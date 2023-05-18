# DCR 이란?

## 1. OVERVIEW
1. ‘데이터 클린룸(Data Clean Room; DCR)’이란 데이터 세트를 물리적으로 교환하거나 개인식별정보(PII), IP 식별자 또는 기타 유형의 사용자 개인정보 보호를 손상시키지 않고 여러 데이터 소스를 공유할 수 있는 안전하고 중립적인 환경을 말함 ([참고](https://www.ciokorea.com/t/2996/%EB%B9%85%20%EB%8D%B0%EC%9D%B4%ED%84%B0/277891#csidxee702383fe360faa07c3adf4b4cdc9c))
2. 두 개 이상의 조직에서 Snowflake DCR을 활용하여 기본 데이터를 복사, 이동 또는 공유하지 않고 데이터를 결합하고 고성능 및 확장성으로 대량의 데이터에 대한 분석을 수행 가능
3. First Party에 Data가 있고, 이 Data를 Private하게 Multi-Party에서 결합하고 Access하고 싶은 경우에 적용 가능

### 1-1. Data Clean Room이 필요한 이유
1. 기업들은 사용자 프라이버시를 침해하지 않으면서 데이터를 수집, 공유, 분석할 수 있는 대안을 찾아야 하고 데이터 클린룸과 같은 개인정보 보호 솔루션이 중요한 역할을 할 수 있음
2. 모든 민감한 데이터가 안전하게 비공개로 유지되는, 이른바 데이터의 스위스와 같다. ([참고](https://www.ciokorea.com/t/2996/%EB%B9%85%20%EB%8D%B0%EC%9D%B4%ED%84%B0/277891#csidxee702383fe360faa07c3adf4b4cdc9c))
 
![image](https://user-images.githubusercontent.com/52474199/235078224-6b31b9e4-091f-4202-938b-68e3f5863b2e.png)


### 1-2. DCR 기반기술
![image](https://user-images.githubusercontent.com/52474199/235840698-96f8a368-1e42-4b97-8108-843f2ddf1ecb.png)


## 2. 배울 것
1. 둘 이상의 Snowflake 계정 간에 DCR 환경을 만들고 배포하는 방법
2. DCR 쿼리 요청이 시작, 검토 및 승인(또는 거부)되는 방법
3. 승인된 DCR 쿼리 요청이 실행되는 방식

## 3. 활용 사례
1. 3rd Party 쿠키 또는 집계 Data를 활용하지 않고 직접 Access하여 first party Data를 더 쉽게 연결, 활용할 수 있는 사례
2. 소매업체와 CPG(consumer packaged goods:포장소비재)
   A. 해당 소매업체는 고객이 무엇을, 얼마나, 자주 구매하는지 인사이트를 활용할 수 있으며, 
   B. 이러한 인사이트를 CPG 브랜드와 공유할 수 있다. 
3. 기대효과
   A. 브랜드는 Targeting 효율을 높이고, 적절한 청중에 도달하며, 광고 낭비를 줄일 수 있다. 
   B. 소매업체는 수익을 개선하고, 향상된 고객 경험을 제공할 수 있다

## 4. Prequisition
1. 2개의 Snowflake 계정 (Provider, Consumer)
2. Edition : Enterprise & CSP : AWS


## Overview
### 1. 두 참여자가 각각의 customers 테이블을 갖고 있고,
### 2. party2(consumer)가 party1(provider)로부터 join 하여 column add 하여 조회하고 싶다
### 3. 단 party1은 접근 통제 정책(query, column)을 사전에 설정하여 소유한 Data의 security 확보
![image](https://user-images.githubusercontent.com/52474199/220887464-eb1de5ea-1d57-4b20-a3ba-e6324ba73c70.png)

![image](https://user-images.githubusercontent.com/52474199/221126990-e69a3cdc-3927-4db2-a8ea-ebf9e880da12.png)



## 5. DCR 동작 방식
### Step 1. 두 참여자(Account)로 DCR을 구성한다.
![image](https://user-images.githubusercontent.com/52474199/235817759-7e49a654-368b-4b6b-8818-ad91dc0443a4.png)

### Step 2. 각각 customer 데이터를 갖고 있으며
![image](https://user-images.githubusercontent.com/52474199/235110596-b6b7b2ac-f3ef-45ca-933d-57c0307b9873.png)

### Step 3. Consumer는 내가 갖고 있는 Column을 join key로 하여 Provider의 Data Set를 조회하고 싶다.
![image](https://user-images.githubusercontent.com/52474199/235817788-718cf21b-46a2-4528-a722-f8ef8b63654e.png)

### Step 4. Provider쪽 Data Set을 보고자 하는 경우 Provider쪽 DCR_DB.query_templates에 사전에 정의된 query 를 참조한다. Join key는 email_address 이다.
![image](https://user-images.githubusercontent.com/52474199/235818546-edd09945-fc31-4a95-b251-2022c7f52071.png)

### Step 5. 다 볼순 없고, Provider가 열어준 Available value만 볼 수 있다.
![image](https://user-images.githubusercontent.com/52474199/235818506-e6664a4a-e251-48d5-ab67-fa7cb275d858.png)

### Step 6. (Consumer) Analyst가 (Provider) Party1_SOURCE_DB를 보고 싶다.
다 보여주지 않는다. 허락된 쿼리와 허락된 Value만 볼 수 있다.
![image](https://user-images.githubusercontent.com/52474199/235111984-0788bccd-4efa-44e6-9402-bf25991b076a.png)

### Step 7. (Consumer) DCR_DB에 있는 query_template과 available_value를 Stored Procedure로 호출하여 query_request를 생성하고, 
![image](https://user-images.githubusercontent.com/52474199/235112626-77852e0d-bb9f-472f-9876-dd31bb740657.png)

### Step 8. StoredProcedure로 Provider에 승인을 요청한다.
![image](https://user-images.githubusercontent.com/52474199/235820116-07e64dc5-172e-4385-ba25-9271fc2931e2.png)

### Step 9. 쿼리가 승인되면 approved_query_requests에 저장한다.
![image](https://user-images.githubusercontent.com/52474199/235121021-e6a26ddb-6247-4b4c-85c1-d8120f0342bb.png)

### Step 10. (consumer)는 provider에 Query_Request 공유한다
![image](https://user-images.githubusercontent.com/52474199/235121492-af025ae0-88dd-45b8-b1f6-140bca6919a1.png)

### Step 11.provider는 consumer에게 query_templates, available_values, request_status를 공유한다.
![image](https://user-images.githubusercontent.com/52474199/235121233-7bc87f3f-41c1-4dac-961b-ad0e0850b3ca.png)

### Step 12. party1_customers 테이블을 consumer에서 조회하는데, 강력한 접근 통제, 즉 approved_query_request에 저장되어 있는 query를 통해서만 조회된다.
이외 다른 Query 건은 empty record로 조회된다.

![image](https://user-images.githubusercontent.com/52474199/235123347-bdfe7af8-7728-4089-bb86-3c723e24ecea.png)

### Step 13. query_result_set은 consumer(party2_dcr_db) 의 local table로 저장된다. 이런 작업을 통해 양측의 교집합(intersection) 부분의 많은 데이터 set을 확보할 수 있다.
![image](https://user-images.githubusercontent.com/52474199/235124261-a9f2da37-3664-40c0-b6b0-2227d85679fe.png)

### 참고 1. 아키텍처를 보면, 가운데 부분이 쉐어 되어 있고,
![image](https://user-images.githubusercontent.com/52474199/235125137-fe959957-5828-4a8d-98ce-0d34cf7e89a9.png)


### 참고 2. 이를 통해 DCR 환경을 확인할 수 있다.
![image](https://user-images.githubusercontent.com/52474199/235125221-39d66e1a-d550-4698-88c4-7c13a7070485.png)



## 6. Code 분석
### 1. party 1 

1. object
![image](https://user-images.githubusercontent.com/52474199/235841054-34cdb878-4044-41fa-a75b-28d75efaf6a4.png)

2. available_value
![image](https://user-images.githubusercontent.com/52474199/235841113-1bb1bc24-8e86-48b3-ab63-bdb48323f2c7.png)



### 2. party 2
1. object
![image](https://user-images.githubusercontent.com/52474199/235841200-8accdb66-3c4f-44cb-8121-b6f5940f25b2.png)

2. (city, postal_code) column은 Party 1으로부터 조회 필요
![image](https://user-images.githubusercontent.com/52474199/235841454-4a3331b6-ca75-440f-b9ea-17d0024d97c3.png)

3. join key는 mail_address
![image](https://user-images.githubusercontent.com/52474199/235841601-a65098e5-4304-470d-b27c-34dbd0509254.png)


### 3. party 1
1. 아키텍처를 다시 보자.
![image](https://user-images.githubusercontent.com/52474199/235841933-0de6dc86-c038-4d7e-925b-82ad3f0f56a8.png)

2. 아직 query_request는 예상대로 없다.
![image](https://user-images.githubusercontent.com/52474199/235842049-f22e001a-82f2-4d22-8174-b4e6eae23c72.png)



### 4. party 2
1. 아키텍처 보면 party1.DCR_DB를 공유 받았다
![image](https://user-images.githubusercontent.com/52474199/235842150-4c72a9b7-78ce-4881-8d60-aac5042cb8fe.png)

2. object보면, 샘플 쿼리와 가능한 컬럼 정의되어 있다.
![image](https://user-images.githubusercontent.com/52474199/235842399-252b5d69-b248-4734-9aa7-b4c770950161.png)

3. PARTY1_SOURCE_DB.SOURCE_SCHEMA.PARTY1_CUSTOMERS 보면 no record
![image](https://user-images.githubusercontent.com/52474199/235842566-0146b4fa-27ba-4503-871e-664c36a4efbb.png)

4. 왜냐하면 access policy로 막기 때문
![image](https://user-images.githubusercontent.com/52474199/235842955-6acf0c1c-a0cc-4264-b50a-f477be1c8b25.png)

5. 그러나 우리가 궁금해 하는 컬럼이 있는 것을 확인할 수 있다.
![image](https://user-images.githubusercontent.com/52474199/235842830-ac44817c-03fb-414c-a9f9-aceadd856e25.png)

6. 조회할 수 있도록 해보자. (SP를 수행하여 template에 있는 쿼리와 column을 조합하여 request query 생성)
![image](https://user-images.githubusercontent.com/52474199/235843141-9316a26c-33b5-44ca-8cc4-95f7c049a957.png)

7. 즉, 이 부분
![image](https://user-images.githubusercontent.com/52474199/235843391-a5ec2342-d158-4dd4-9b98-cc0276d24806.png)

### party 1
1. Query Request 확인 (1건 요청 들어옴)
![image](https://user-images.githubusercontent.com/52474199/235843641-0fae669e-13d3-4084-9310-3dedd83b08c8.png)

2. SP를 수행하여 approve함
![image](https://user-images.githubusercontent.com/52474199/235843763-472bfdbc-df7a-47c9-94f5-3306db481bee.png)

3. 즉 이 부분
![image](https://user-images.githubusercontent.com/52474199/235843848-21e12fb0-77ff-40d4-86b6-8c7d50397bc6.png)

승인하면 
![image](https://user-images.githubusercontent.com/52474199/235843932-07906638-7de9-47fa-9471-02b6d23ba4b2.png)


### party 2
1. 승인 됐네
![image](https://user-images.githubusercontent.com/52474199/235844012-f8be85c3-993c-481c-9983-43a912b6c5d4.png)

2. 조회해보자

![image](https://user-images.githubusercontent.com/52474199/235844176-da2cdf70-55a4-4870-97cd-c92a7c29af13.png)

![image](https://user-images.githubusercontent.com/52474199/235844145-27fccb65-4166-4fa9-9944-35ef7e9dcf3c.png)

### party 1
1. request_status 가보면 approved 된 record가 1건 보인다.
![image](https://user-images.githubusercontent.com/52474199/235846128-581647cd-cd13-4ece-8df1-daf6141f4b5b.png)

2. 즉 이 부분
![image](https://user-images.githubusercontent.com/52474199/235845722-9f29f9ba-7658-490f-9622-bbd82ebfe0e0.png)


### party 2
1. party 1의 query_template를 보자
![image](https://user-images.githubusercontent.com/52474199/235846314-f655eb02-b87d-4a07-88f7-2a0ae1b43060.png)

2. 조회할 수 있는 항목들 (param : 호출하려는 table, query_template_name, available_value)
![image](https://user-images.githubusercontent.com/52474199/235846705-e63b46b5-388c-4840-9120-071676402ce1.png)

3. SP 실행 (firewall off)
![image](https://user-images.githubusercontent.com/52474199/235847614-ca24063f-673e-44e5-b5d5-0612cae28982.png)


### party 1
1. SP 실행해서 validation 확인
![image](https://user-images.githubusercontent.com/52474199/235847918-6cc891c2-7fb9-4baa-b543-0e132a51bd35.png)

### party 2
1. 승인 된 것 확인
![image](https://user-images.githubusercontent.com/52474199/235848214-c838386d-73c5-4a96-bc11-c6a829bb2ede.png)

2. Request한 Data 확인
![image](https://user-images.githubusercontent.com/52474199/235848366-c151854c-be5c-4965-8c11-097d3f94e934.png)



## Scenario 2
> available_value 에 없는 항목 조회하는 것

### party 2
1. available_value 에 없는 column을 조회요청 했더니 declined됨
![image](https://user-images.githubusercontent.com/52474199/235848646-fd426b43-ba17-4568-bf93-645a8ce5023b.png)

2. party 1 조회 좀 하게 해줘


### party 1
1. 그래 알았어 (insert into available_value)
![image](https://user-images.githubusercontent.com/52474199/235849004-ee5c24b9-f9c6-4726-9955-febdf5adbf35.png)

### party 2
1. 넣어 줬구나?
![image](https://user-images.githubusercontent.com/52474199/235849397-352f9431-1870-4740-8034-5c3ac3e9030b.png)

2. sp 재 수행
![image](https://user-images.githubusercontent.com/52474199/235849475-03cbcc66-2594-4e94-bfe4-a7260ae6651f.png)

### party 1
1. validate query
![image](https://user-images.githubusercontent.com/52474199/235849791-e4052289-14eb-4d30-8bfd-1dcaaf593d26.png)

### party 2
1. approved 확인함
![image](https://user-images.githubusercontent.com/52474199/235849545-a4ecd3c4-ba97-4475-b81a-0ee35e033d0e.png)

2. 잘 나오네.
![image](https://user-images.githubusercontent.com/52474199/235840456-8d709ba5-187f-4566-a2aa-8c4e96eb00dd.png)


## 참고 영상
[![image](https://user-images.githubusercontent.com/52474199/235077987-8653fe73-f11b-41c5-9e97-cae8f02a829d.png)](https://youtu.be/UI5na73_9cA)

[참고](https://quickstarts.snowflake.com/guide/build_a_data_clean_room_in_snowflake_advanced/#1)
