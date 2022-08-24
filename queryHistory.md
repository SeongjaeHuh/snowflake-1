# How to Audit in Snowflake (Managing Govenance in Snowflake)
## Query History
### Querying the QUERY_HISTORY
```
select * from "SNOWFLAKE"."ACCOUNT_USAGE"."QUERY_HISTORY" limit 10;
```
![image](https://user-images.githubusercontent.com/52474199/162957848-db97182d-dd8c-4c1b-a770-c3962eefad0b.png)


## Access Hisotry
### Querying the ACCESS_HISTORY 

#### Who accessed the data? Return the User Access History
```
select user_name
       , query_id
       , query_start_time
       , direct_objects_accessed
       , base_objects_accessed
from access_history
order by 1, 3 desc
;
```
```
# This is one of record in base_objects_accessed (column)
  {
    "columns": [
      {
        "columnId": 10252,
        "columnName": "JSONRECORD"
      }
    ],
    "objectDomain": "Table",
    "objectId": 10248,
    "objectName": "TEST_DB.PUBLIC.JSONRECORD"
  }
]
```
![image](https://user-images.githubusercontent.com/52474199/162992691-e268a949-62c3-4e16-b912-607dada8c144.png)


#### When was the data accessed within 30 days?
```
select query_id
       , query_start_time
from access_history
     , lateral flatten(base_objects_accessed) f1
where f1.value:"objectId"::int=10248 /*objectId*/
and f1.value:"objectDomain"::string='Table'
and query_start_time >= dateadd('day', -30, current_timestamp())
;
```
![image](https://user-images.githubusercontent.com/52474199/162993543-dc45c301-3b3e-4b4b-b0a3-5ac878db035b.png)

#### Who was the data accessed within 30 days?
```
select distinct user_name
from "SNOWFLAKE"."ACCOUNT_USAGE"."ACCESS_HISTORY"
     , lateral flatten(base_objects_accessed) f1
where f1.value:"objectId"::int=10248
and f1.value:"objectDomain"::string='Table'
and query_start_time >= dateadd('day', -30, current_timestamp())
```
![image](https://user-images.githubusercontent.com/52474199/162993818-08a765d4-7e49-46c3-b8e2-b85a08c8543a.png)


#### What columns were accessed?
```
select distinct f4.value as column_name
from "SNOWFLAKE"."ACCOUNT_USAGE"."ACCESS_HISTORY"
     , lateral flatten(base_objects_accessed) f1
     , lateral flatten(f1.value) f2
     , lateral flatten(f2.value) f3
     , lateral flatten(f3.value) f4
where f1.value:"objectId"::int=10248
and f1.value:"objectDomain"::string='Table'
and f4.key='columnName';
```
![image](https://user-images.githubusercontent.com/52474199/162994294-6f6e06a4-dd26-47b0-8e0d-393d0dde4813.png)

#### What Tables were accessed?
```
SELECT   
    ANY_VALUE(DO_ACC_L1.VALUE:objectDomain::STRING) AS OBJECT_TYPE,
    DO_ACC_L1.VALUE:objectName::STRING AS OBJECT_NAME,
    COUNT (DISTINCT AH.QUERY_ID) AS COUNT_USED,
    MIN(query_start_time) as MIN_ACCESS_DATE,
    MAX(query_start_time) as MAX_ACCESS_DATE
FROM
    SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY AH,
    lateral flatten(DIRECT_OBJECTS_ACCESSED) DO_ACC_L1
GROUP BY       
    OBJECT_NAME
HAVING OBJECT_TYPE = 'Table'
ORDER BY
    COUNT_USED DESC;
```
![image](https://user-images.githubusercontent.com/52474199/163530328-dcb22d74-ed51-4fc2-8bb9-1eba61d39770.png)

![image](https://user-images.githubusercontent.com/52474199/166086484-865e03df-088a-46db-8bcd-caa27672acac.png)


