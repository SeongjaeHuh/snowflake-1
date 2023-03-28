# Trouble Shooting of Network Error

## 1. 현상
1. smilegate 는 VPC Endpoint로 구성되어 있음
> SQL 수행시 알 수 없는(?) Error 발생

![image](https://user-images.githubusercontent.com/52474199/172679835-cc7e2a10-f758-4ec7-a1cc-f0108d7cde3c.png)

2. Snowflake에서 Query 실행시, 초기 Result Set은 정상 Return
3. Result Set을 Scroll Down 하여 다음 Record를 조회하는 경우 Internal Error 발생
![image](https://user-images.githubusercontent.com/52474199/208307415-8d5e25e2-ac9e-47fe-b308-009eeb86377c.png)

![20221006_145232](https://user-images.githubusercontent.com/52474199/204763866-f8fcdb83-e3a1-4849-be0f-c3e43d5b57c1.jpg)


## 2. 원인 파악

### Step 1. Packet 식별 (using Wireshark, Devtool)

Network 트래픽을 분석하여 문제가 되는 패킷을 식별함
```
https://sfc-kr-ds1-customer-stage.s3.ap-northeast.amazonaws.com
```

![image](https://user-images.githubusercontent.com/52474199/208307344-ddde08e8-bdee-4a60-881b-ded1e2dad020.png)

1. Snowflake에서는 쿼리를 실행할 때 Result Set 이 충분히 크면 먼저 Internal Stage(S3 버킷)에 저장한 다음 브라우저에서 가져옴
2. 그러나 브라우저와 Internal Stage 간의 네트워크 통신 오류로 인해 네트워크가 단절되어 쿼리는 실패
3. 동일한 Internal Stage는 Worksheet를 저장하는데 사용되나, 네트워크 단절로 아래와 같이 **"Changes Not Saved"** 이라는 메시지를 볼 수 있음

![image](https://user-images.githubusercontent.com/52474199/208307378-5e71e1ac-ceac-4ea0-8903-e25df78b393b.png)


### Step 2. Support 문의 후 답변

```
There are 2 possibilities to fix this issue. 
The first one is easier to implement: 
you allow the S3 URL in your firewall/proxy so that your browser can communicate to the internal stage.

The second option is to configure a PrivateLink connection to the internal stage. 
This is something that requires a change in your VPC, similar to the change when PrivateLink was initially configured. 
Complete instructions on how to configure this are in the link below:
```
### 무슨 말이냐면,
```
1. S3 URL을 방화벽/Proxy 등록 (방화벽은 사이트 특성상 불가)
2. Internal stage 와 PrivateLink 구성
```

## 3. 해결책

### Step 1. Proxy Server 사용 (성공)
> Port Forwarding(443, 80)으로 사용자 PC <-> Proxy <-> VPC Endpoint <-> S3로 Network 연결

> Local 환경 구성의 복잡도 증가 및 IP 변경 등에 따른 NW Error 발생시 원인 파악 어려움(운영 Risk 증가)

### Step 2. Internal Stage와 PrivateLink 구성 (성공)
> Internal Stage와 VPC Endpoint 구성
> 상세 구성 방법

https://community.snowflake.com/s/article/HOW-TO-configure-private-connectivity-to-aws-internal-stages
