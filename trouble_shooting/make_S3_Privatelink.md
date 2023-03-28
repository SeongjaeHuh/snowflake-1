# HOW TO: CONFIGURE PRIVATE CONNECTIVITY TO INTERNAL STAGES WITH AWS S3 PRIVATE ENDPOINTS
## 1. Enable the PrivateLink Internal Stage feature
In your Snowflake account, run the below command to enable the private connectivity to the internal stage feature: 

```
use role accountadmin;
alter account set enable_internal_stages_privatelink = true;
```


## 2. Retrieve your Snowflake Account's Internal Stage URL
Run the below command and retrieve the privatelink-internal-stage value. Record this for later. 
```
use role accountadmin;
select key, value from table(flatten(input=>parse_json(system$get_privatelink_config())));
```
![image](https://user-images.githubusercontent.com/52474199/206359060-63225254-e2e5-41ee-b5a7-95da8a4c3fd3.png)


## 3. Create an S3 Private Endpoint
In your AWS console, navigate to Endpoints > Create Endpoint. Choose AWS services and select the S3 Interface type. 

![image](https://user-images.githubusercontent.com/52474199/206359237-14ebf943-63cd-4d28-9eb7-a16d2587f114.png)

## 4. Record the VPCE DNS Name
Once the S3 Private Endpoint is created, go to the resource and record the VPCE DNS name to be used in the next step: 

![image](https://user-images.githubusercontent.com/52474199/206359509-8598ec95-c4d2-4464-b81c-a5bc06531633.png)

In this example, the DNS name is: *.vpce-0f4407f7e8b7c7212-ed5t2tw9.s3.eu-central-1.vpce.amazonaws.com
Note: do not record the VPCE DNS zonal name.


## 5. Summary:
You now know two values: 

* *<b> privatelink_internal_stage </b> retrieved in step 2.*   
* *<b> S3 VPCE DNS </b> name retrieved in step 4.*

For the purpose of this example, the values we will use are: 

* *privatelink_internal_stage: <b> sfc-eu-ds1-9-customer-stage.s3 </b>.eu-central-1.amazonaws.com*
  - The highlighted part above is the S3 bucket name assigned to your Snowflake account. Anything before .s3 is the name of your S3 bucket. 
* *S3 VPCE DNS: *.vpce-0f4407f7e8b7c7212-ed5t2tw9.s3.eu-central-1.vpce.amazonaws.com*   

The goal of this DNS configuration and the outcome you must achieve is that whenever a client (browser, driver, etc) tries to access the internal stage URL (sfc-eu-ds1-9-customer-stage.s3.eu-central-1.amazonaws.com), it is routed through the S3 Private Endpoint you created. 


## 6. DNS configuration
To achieve the purpose explained in step 5, you can use a Private Hosted Zone in AWS Route 53 service. Use one of the two methods below: 

### Method 1: Create a private hosted zone named s3.<region>.amazonaws.com. 
![image](https://user-images.githubusercontent.com/52474199/206359672-b7976c61-23d1-43c6-97c1-9ed915639108.png)

  In this zone, create a new <b> CNAME </b> record where: 

* *Record name is the S3 bucket name highlighted in step 5.*
* *Value is (S3 bucket name).(S3 VPCE DNS name)*

Such as below for this example: 
* <b>Record:</b> *sfc-eu-ds1-9-customer-stage*
* <b>Value:</b> *sfc-eu-ds1-9-customer-stage.vpce-0f4407f7e8b7c7212-ed5t2tw9.s3.eu-central-1.vpce.amazonaws.com*

![image](https://user-images.githubusercontent.com/52474199/206366175-fbcc06cc-d7aa-45c8-bd95-cbe99835f67b.png)
  
<span style="color:red">Important: Read the following if you intend to connect to other S3 buckets than your Snowflake internal stage from the attached VPC.</span>
  
Creating a private hosted zone with domain s3.<region>.amazonaws.com without explicitly adding all your S3 bucket names to the zone, will prevent access to these other S3 buckets.
 
Should you need to access other non-Snowflake S3 buckets in this region from the VPC, it is preferable to use Method 2 explained below: 

  
### Method 2: Create a private hosted zone named <bucket>.s3.<region>.amazonaws.com. 

  You first need to retrieve the <b> Private IP address </b> of your S3 Private Endpoint. 
Because you cannot create a <b>blank CNAME record</b>, we will use an <b>A record</b> in this DNS configuration. 
Navigate back to Endpoint resources, select your S3 Private Endpoint and choose the Subnets tab. Record the <b> IPv4 Address: </b>

  ![image](https://user-images.githubusercontent.com/52474199/206363654-2ffc4c1a-5146-4130-bc62-5630b93c82c7.png)
  
Back in Route 53 service, create a new Private Hosted Zone with domain name: <bucket>.s3.<region>.amazonaws.com

In this example, we use: <b> sfc-eu-ds1-9-customer-stage.s3.eu-central-1.amazonaws.com</b>

![image](https://user-images.githubusercontent.com/52474199/206363788-9f7a7a31-740a-4c26-9cd0-e63c3fa37d11.png)
  
  
Once the zone is created, select <b> Create Record </b> where: 

* Record name should be left empty
* Record type should be A record
* The value should be the private IP address of your S3 Private Endpoint retrieved above. 

![image](https://user-images.githubusercontent.com/52474199/206364425-d2f9107e-5f24-4667-8564-cec851aa7987.png)
  
By creating a private hosted zone with the domain name being explicitly the URL of your Snowflake internal stage S3 bucket, you ensure this zone won't apply to other S3 buckets in this region used by the VPC. 
  
However, you must <b><u>ensure</u> that a static private IP address</b> is assigned to the network interface of your S3 private endpoint.

To summarize, what you've achieved with this DNS configuration: 

### With method 1: 
* Your client will try to access the internal stage URL sfc-eu-ds1-9-customer-stage.s3.eu-central-1.amazonaws.com
* With the private hosted zone configuration, the DNS resolves sfc-eu-ds1-9-customer-stage.s3.eu-central-1.amazonaws.com to sfc-eu-ds1-9-customer-stage.vpce-0f4407f7e8b7c7212-ed5t2tw9.s3.eu-central-1.vpce.amazonaws.com
* AWS routes the traffic originally destined for sfc-eu-ds1-9-customer-stage.s3.eu-central-1.amazonaws.com through the S3 Private endpoint. 
  
### With method 2: 
* Your client will try to access the internal stage URL sfc-eu-ds1-9-customer-stage.s3.eu-central-1.amazonaws.com
* With the private hosted zone configuration, the DNS resolves sfc-eu-ds1-9-customer-stage.s3.eu-central-1.amazonaws.com to the private IP address of your S3 Private Endpoint (10.0.10.37)
* AWS routes the traffic originally destined for sfc-eu-ds1-9-customer-stage.s3.eu-central-1.amazonaws.com through the S3 Private endpoint. 
  
## 7. Testing the configuration
You may now launch an EC2 instance in the relevant VPC. 
Open a terminal or command prompt, by using nslookup or dig, ensure that your Snowflake internal stage URL resolves to the S3 private endpoint. 

![image](https://user-images.githubusercontent.com/52474199/206364992-0eef3fd6-b37a-492b-aa75-b6901f73784e.png)
  
User's worksheets being saved and retrieved from the Internal Stage, the UI showing the worksheet being saved is a good sign that you are able to access your Snowflake internal stage over the S3 private endpoint. 
  
![image](https://user-images.githubusercontent.com/52474199/206365081-8a1e2c36-b2d2-4776-9793-ef7681e41f44.png)
