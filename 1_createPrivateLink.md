# AWS PrivateLink & Snowflake
## AWS Private Link란 무엇인가?
### 정의
PrivateLink는 VPC 내부에서 외부에 있는 다른 서비스를 연결하기 위한 기술이다. PrivateLink를 이용하면, 퍼블릭 네트워크(인터넷)을 거치지 않고도 AWS의 서비스와 다른 서비스(다른 계정의 네트워크, 다른 VPC에 존재하는)를 호출 할 수 있다.
### PrivateLink를 쓰면 좋은 점
 - 서비스 간 연결이 외부로 노출 안됨(내부망에서만 트래픽 움직임)
  - 서비스 제공자가 연결 제어 가능 (단방향 설정 같은)
  - 그래서 보안/성능적으로 좋음
[(Private Link 참조)](https://aws.amazon.com/ko/privatelink)

### Private Link 구성도
![image](https://user-images.githubusercontent.com/52474199/173366947-815ef1a8-5abb-4aaa-bd42-ea7da4f50723.png)

## AWS VPCe - Snowflake 연동 구성도

![image](https://user-images.githubusercontent.com/52474199/172679835-cc7e2a10-f758-4ec7-a1cc-f0108d7cde3c.png)

#### Snowflake Private Link 설정은 아래 3가지 단계로 이뤄진다.
1. Private Link 활성화
2. AWS VPC 환경 구성하기(VPCE 구성, 보안그룹 생성 및 Subnet구성)
3. CNAME Record 생성

#### [참고] Endpoint 생성 관련 주의사항
PrivateLink의 Endpoint를 생성하려면 서비스(Endpoint Service)가 제공되는 region과 동일한 region에서 작업해야 함
PrivateLink가 존재하는 region A에 VPC A를 생성하고, 이 VPC에 Endpoint 생성
이때 VPC만 생성하면 되는 것이 아니라, Subnet(AZ기반) 등을 적절히 생성해야 한다. 

## 1. [AWS PrivateLink 활성화](https://docs.snowflake.com/en/user-guide/admin-security-privatelink.html)
snowflake 계정의 AWS Private Link 활성화 하기 위한 단계
### (1). AWS CLI 환경에서 다음 명령을 실행하고 출력을 저장 (출력은 다음 단계에서 인자값으로 사용됨)

```
aws sts get-federation-token --name sam
```
IAM 사용자가 GetFederationToken 작업을 호출하고 SAM 이라는 이름의 사용자에 대한 임시 자격 증명을 반환한다.

"FederatedUserId" 값에서 12자리 숫자를 추출합니다(빨간색 부분 = <aws_id>). 

![image](https://user-images.githubusercontent.com/52474199/172307567-1a30a721-4312-409b-8878-6f2d1ffa691d.png)

### (2).Snowflake Role은 ACCOUNTADMIN 으로, 다음 Select 문으로 Snowflake 계정에 대해 AWS PrivateLink를 인증(즉, 활성화)합니다.

```
select SYSTEM$AUTHORIZE_PRIVATELINK ( '<aws_id>' , '<federated_token>' );
```
<'aws_id'> = AWS의 계정을 문자열로 고유하게 식별하는 12자리 식별자

<federated_token> = 페더레이션 사용자에 대한 액세스 자격 증명을 문자열로 포함하는 페더레이션 토큰 값

![image](https://user-images.githubusercontent.com/52474199/172308637-d7dc4f7a-9763-4010-a599-7a9d569755ae.png) 

![image](https://user-images.githubusercontent.com/52474199/172308669-6a3d8b5f-a95f-477c-8fe1-0c8ac937df39.png)

'''

"privatelink-account-name":"ab25370.ap-northeast-2.privatelink",
"privatelink-vpce-id":"com.amazonaws.vpce.ap-northeast-2.vpce-svc-0870e889ef8b7b491",
"privatelink-account-url":"ab25370.ap-northeast-2.privatelink.snowflakecomputing.com",
"regionless-privatelink-account-url":"atixoaj-smilegate.privatelink.snowflakecomputing.com",
"privatelink_ocsp-url":"ocsp.ab25370.ap-northeast-2.privatelink.snowflakecomputing.com",
"privatelink-connection-urls":"[]"

'''


## 2. AWS VPC 환경 구성하기

### (1). VPC 엔드포인트(VPCE) 생성 및 구성

VPC Endpoint는 VPC를 AWS 서비스 혹은 다른 VPC의 서비스를 연결하기 위한 종단점(endpoint)다.

Snowflake 계정 관리자(즉, ACCOUNTADMIN 시스템 역할을 가진 사용자)로서, SYSTEM$GET_PRIVATELINK_CONFIG 함수를 호출하고 privatelink-vpce-id 값을 기록합니다.
```
select system$get_privatelink_config();
```
![image](https://user-images.githubusercontent.com/52474199/170653808-964bc5b4-a69a-4317-aff0-e01f24b1caff.png)


#### 1). AWS VPC Account로 로그인 > 엔드포인트 메뉴 선택

![image](https://user-images.githubusercontent.com/52474199/171126643-ac6b393b-20ac-48a5-9e41-147d430f422b.png)


#### 2). 엔드포인트 설정

![image](https://user-images.githubusercontent.com/52474199/172310639-cfb5fb93-b171-4632-af52-9d89c0b8823b.png)



#### 3). 서비스 이름은 "privatelink-vpce-id": 에서 가져옴 (Snowflake 로의 직접 접속을 위한 정보)

![image](https://user-images.githubusercontent.com/52474199/171126936-dc045abc-2fbf-438f-a795-cb6815c06882.png)


#### 4). vpc 선택

![image](https://user-images.githubusercontent.com/52474199/172311294-ce35d4c9-d6ab-4f08-b786-e9f2c424dc93.png)


#### 5). 서브넷 설정 - 프라이빗으로 선택

![image](https://user-images.githubusercontent.com/52474199/172312178-597267ec-efa7-4fc2-8cda-e3f39342085d.png)


#### 6). 보안그룹 - 그룹 ID 선택

![image](https://user-images.githubusercontent.com/52474199/171127609-2d9eb5f6-f1ef-49fb-9e69-f7f0c645a371.png)


#### 7). 인바운드 규칙 443, 80 Port 추가 (OCSP cache server listens on port 80, all other Snowflake traffics on port 443)

![image](https://user-images.githubusercontent.com/52474199/172312452-705d1a25-3489-4cf4-92c0-5cf2c302e97e.png)


#### 8). 엔드포인트 생성

![image](https://user-images.githubusercontent.com/52474199/172313036-4a29f120-88d1-4f65-8f3b-1aff5c9d8738.png)


#### 9). 생성완료

![image](https://user-images.githubusercontent.com/52474199/172313158-b16d0772-7727-4b07-a9c6-1ce93f41f871.png)

### (2). VPC 네트워크 구성
AWS PrivateLink 엔드포인트를 통해 Snowflake에 액세스하려면 복잡한 URL이 아닌 쉬운 URL로 접속해야 한다.
이를 위해, DNS에 CNAME 레코드를 생성하여 SYSTEM$GET_PRIVATELINK_CONFIG 함수의 privatelink-account-url 및 privatelink-ocsp-url 값을 VPC 엔드포인트의 DNS 이름으로 설정해주어야 한다.

#### 1). ROUTER53 메뉴 이동 > 호스팅 영역 선택
호스팅 영역은 privatelink.snowflake.com 같은 도메인과 관련 하위 도메인에 대한 트래픽을 라우팅하도록 설정해준다.

![image](https://user-images.githubusercontent.com/52474199/172313474-863947f4-5855-4bf2-aeab-937e05abfb8b.png)


#### 2). 호스팅 영역 생성

![image](https://user-images.githubusercontent.com/52474199/171129576-3d261dc7-71d2-422b-a0e7-540498fa8f8a.png)


#### 3). 호스팅 영역 구성

![image](https://user-images.githubusercontent.com/52474199/172313703-450a3698-7e76-4564-97b1-49179c7e678f.png)


#### 4). 호스팅 영역과 연결할 VPC 정보 입력

![image](https://user-images.githubusercontent.com/52474199/172314130-259233c9-2011-48cd-bdc9-74c1093b08ec.png)



#### [참고] DNS 서비스에 CNAME을 왜 등록해줄까? (Endpoint에 URL이 적용되어야 한다)
자체 DNS를 통해 CNAME(Canonical Name)으로 endpoint를 읽기 좋게 감싸서 제공해주는데, 이때는 VPC에서 제공되는 URL을 타고 목적 서비스의 IP에 도달할 수 있다.

![image](https://user-images.githubusercontent.com/52474199/173383276-9dc064ff-084e-44dc-bd1e-043f617f8267.png)

##### Endpoint-specific DNS hostname
Endpoint를 생성하고 Endpoint 세부정보의 DNS name을 보면 여러개의 AWS에서 제공했을 법한 읽기 힘든 URL을 볼 수 있는데 우리는 이를 통해 서비스에 접근할 수 있다.

```
예) oi09824.ap-northeast-2.aws.privatelink.snowflakecomputing.com
vpce-00000xxxxx00000cc-aaaabbbb.vpce-svc-00000xxxxx00000aa.us-east-1.vpce.amazonaws.com (Z7XXXXXUULQXV)
```

##### Private DNS name
Endpoint Service에서 서비스의 DNS 설정을 통해 URL과 동일하게 Endpoint를 제공해주는 경우에는 위에 언급한 바와 같이 해당 URL을 통해 바로 접근이 가능하다.
```
예) internal-api.service.com (Z069XXXXXM0LMA0004WUC)
````

## 3. 호스팅 영역 CNAME 등록

AWS PrivateLink 엔드포인트를 통해 Snowflake에 쉬운 URL로 액세스하려면, 추가 CNAME 레코드를 생성한다. 

SYSTEM$GET_PRIVATELINK_CONFIG 출력의 privatelink-account-url 및 privatelink-ocsp-url 값을 조합

```
예) oi09824.ap-northeast-2.aws.privatelink.snowflakecomputing.com
vpce-00000xxxxx00000cc-aaaabbbb.vpce-svc-00000xxxxx00000aa.us-east-1.vpce.amazonaws.com (Z7XXXXXUULQXV)
```


![image](https://user-images.githubusercontent.com/52474199/172314406-2b30a905-074e-4a92-afc5-3292007b9c8f.png)

![image](https://user-images.githubusercontent.com/52474199/172314608-105c0fde-09d4-423d-9dbb-3d0930c1b658.png)


### (1). 엔드포인트의 DNS 정보를 읽어와야 함
![image](https://user-images.githubusercontent.com/52474199/172315485-544606f6-49a6-4db2-9886-96e2d4bd1559.png)

![image](https://user-images.githubusercontent.com/52474199/172315699-9459bd70-ddb2-439f-97d2-d6ae60db926b.png)

![image](https://user-images.githubusercontent.com/52474199/172317474-dccc191f-451b-488a-8a66-54aebf5979bf.png)

![image](https://user-images.githubusercontent.com/52474199/172316389-c7a876c9-e052-46f1-8e49-c5db464e69cc.png)

![image](https://user-images.githubusercontent.com/52474199/172316663-885b738e-4ea8-4c05-8582-142c3ade551b.png)


### (2). 레코드생성

![image](https://user-images.githubusercontent.com/52474199/172318009-fdca1509-d8b8-4b5d-be8f-89fae7694acc.png)
![image](https://user-images.githubusercontent.com/52474199/172318156-1c2aacd0-8308-497a-a1d0-f3d38130be2e.png)


## 4. NW 정책 등록
### (1). Snowflake Web Host에 접속할 수 있는 경로는 AWS VPC로만 열어주어야 하기 때문에, Public IP 접속을 막아준다.
![image](https://user-images.githubusercontent.com/52474199/171535746-0122d82d-e79a-49fe-8166-391790c06f27.png)

![image](https://user-images.githubusercontent.com/52474199/171137591-cf00c314-ecf0-4699-a4a3-98ac62fc85ad.png)

### (2). White List 외 IP에 대해서 정상 차단 
![image](https://user-images.githubusercontent.com/52474199/172319034-1086fa94-b3c5-4fb3-9a21-766d1f51e8b3.png)


## 5. 접속 확인 
### (1). EC2-windows 생성
![image](https://user-images.githubusercontent.com/52474199/171610542-0d939671-1763-45e8-8ed9-8145972844ca.png)

### (2). VPC EC2 Instance 에서 Private Link 접속 성공
![image](https://user-images.githubusercontent.com/52474199/172319226-8949fbe3-f12c-4678-b652-e6ab4e82734a.png)




## 6. 실패사례
### (1). HTTPS 주소로 입력 해야함
![image](https://user-images.githubusercontent.com/52474199/171137679-ae2406a4-7053-415f-9446-d81e0315919d.png)

![image](https://user-images.githubusercontent.com/52474199/173404535-87700c16-adf8-4f7f-8589-635ab75f6731.png)


### (2). EC2 Windows 내에서 Chrome 브라우저 사용해야함 (Edge, IExplorer사용시 Log-in Hang)
