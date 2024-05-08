---
title: AWS 비용폭탄, 왜 15만원이나
created: 2024-04-12 13:33
last-updated: 2024-04-12 13:33
tags:
---

## 👯‍♂️ Intro & tl;dr

![](https://i.imgur.com/xMRyunq.png)



3월한달간 가장 많이 사용한 서비스

| 서비스명 | 금액     |
| ---- | ------ |
| VPC  | $19.41 |
| ELB  | $16.26 |
| ECS  | $58.41 |

## 왜 비용이 이만큼 나왔는지 궁금하다면? `Bills`확인!

왜 VPC 가격이 저만큼 나왔는지 계속 찾아다녔는데 시간낭비하지 말고  
`Billig and Cost Management > Bills`   에 가면 기본 화면에 나오는 것보다 더 자세하게 서비스 사용 내역을 볼 수가 있다.

| Billing ad Cost Mangement > Cost Explore | Billing ad Cost Mangement > Bills        |
| ---------------------------------------- | ---------------------------------------- |
| ![](https://i.imgur.com/HxulQoP.png)<br> | ![](https://i.imgur.com/d8cQblM.png)<br> |





## 👯‍♂️ VPC 비용

Amazon VPC(가상 사설 클라우드) 자체의 생성 및 사용에 대한 추가 요금은 없다.  
하지만 VPC를 제어, 연결, 모니터링 보안과 같은 서비스를 추가적으로 이용 할 경우 이에 대해 과금됨

VPC 과금이 되었다면 아래 4가지를 의심해 볼 수 있다.  
하지만 보통 `퍼블릭 IPv4주소`에서 가장 많이 청구가 되는 것 같다.[^1]

- NAT 게이트웨이
- IPAM
- 네트워크 분석
- **퍼블릭 IPv4주소**

### NAT 게이트웨이 요금

- VPC에 NAT 게이트웨이를 생성하는 경우 게이트웨이를 프로비저닝하고 사용하는 각 ‘NAT 게이트웨이 시간’에 대한 요금이 부과된다


NAT 게이트웨이(Network Address Translation Gateway)는 AWS에서 제공하는 서비스로, VPC(Virtual Private Cloud) 내의 프라이빗 서브넷에 있는 인스턴스들이 인터넷에 접속할 수 있도록 해주면서도, 인터넷에서는 이 인스턴스들로 직접 접속할 수 없게 하는 역할을 합니다. 즉, NAT 게이트웨이는 보안과 격리를 유지하면서 인터넷 접근이 필요한 작업(예: 소프트웨어 업데이트, API 호출)을 수행할 수 있게 해주는 중요한 네트워크 구성 요소입니다.

- NAT 게이트웨이의 주요 기능:

1. **인터넷 접속 제공**: 프라이빗 서브넷의 인스턴스가 인터넷에 접속해 필요한 데이터를 다운로드하거나 외부 API와 통신할 수 있도록 합니다.
2. **인바운드 연결 차단**: 외부에서 프라이빗 서브넷의 인스턴스로 직접 접근하는 것을 차단하여 보안을 강화합니다.
3. **IP 주소 마스킹**: NAT 게이트웨이는 프라이빗 서브넷의 인스턴스가 인터넷에 접속할 때 고정된 IP 주소(게이트웨이 자체의 IP)를 사용하여 출발지 IP 주소를 변환하고 응답을 해당 인스턴스로 라우팅합니다.
### IPAM 요금

### 네트워크 분석 요금

### 퍼블릭 IPv4주소 

퍼블릭 IPv4 주소는 인터넷에 연결되는 모든 기기가 가지는 특별한 번호임. 인터넷 게임을 하거나 웹사이트 접속 할 때 이 주소를 사용함. 우리집에 우편물 보내려면 정확한 주소가 필요한 것처럼 인터넷에서 각 기기의 정확한 위치 알려주는 주소가 필요한거고 이게 퍼블릭 IPv4주소다.

AWS VPC에서 시작하는 대부분의 리소스들은 서로 연결되어야 하기때문에 IP주소와 함께 제공된다. 이때 대부분은 프라이빗 IP주소(인터넷에 공개되지 않음)로 제공되지만 인터넷에 직접 연결되어야하는 경우 퍼블릭  IP주소를 사용한다.
(내가 만든 웹서버를 인터넷 통해 보여주고 싶다든가 할 때)

이때!  VPC에서 시작된 리소스와 연결된 모든 퍼블릭 IPv4주소에 요금이 부과되는데. 주소별로 시간당 0.005 USD이다. (리로스 연결과 상관없이 과금됨)


예를들면 아래의 경우 이번달에 `IP주소 10개 * 24시간 * 30일 * $0.005 = $36` 만큼 과금된다. 

• 사용 중인 퍼블릭 IPv4 주소가 1개씩 있는 Amazon EC2 인스턴스 3개  
• 사용 중인 퍼블릭 IPv4 주소가 2개 있는 Elastic Load Balancer 1개(포함)  
• 사용 중인 퍼블릭 IPv4 주소가 1개 있는 Amazon RDS 데이터베이스 1개  
• AWS 계정의 유휴 탄력적 IP 주소 4개

이의 계산대로 보면 
나는 IPv4가 한 5~6개를 쓰고 있는걸로 확인된다.. `19.41/(24*30*0.005)` (ALB )

그런데 내가 사용하고 있는  IPv4는 1개 일텐데?  내 블로그 도메인과 연결된 ALB에서 
![](https://i.imgur.com/vd4vo1H.png)
아그런데 이게 VPC1, 서브넷4개에서 사용중이기 떄문에 총 5개.. 그러면 계산이 맞다. 



![](https://i.imgur.com/ZwU7Oac.png)
![](https://i.imgur.com/mMCePwq.png)
![mMCePwq.png](https://i.imgur.com/mMCePwq.png)




##  ECS 비용



![](https://i.imgur.com/33l5Yqr.png)

ECS total 58.34

- ECS APN2 fargate ARM-GB-hours -> aws fargate ARM memory 0.15hrs=$0
- ECS APN2 fargate ARM-vCPU-hours:perCPU -> aws fargate ARM vCPU 0.05hrs=$0
- ECS APN2 fargate GB-hours -> aws fargate memory 2,828.023hrs = $14.45
- ECS APN2 fargate vCPU-hours:perCPU -> aws fargate vCPU 942.674hrs = $43.89

## 👯‍♂️ Ref & LINKS TO THIS PAGE



https://aws.amazon.com/ko/blogs/korea/new-aws-public-ipv4-address-charge-public-ip-insights/
- [^1]: [reddit, VPC 왜 요금청구 되는거죠?](https://www.reddit.com/r/aws/comments/1ajvdn9/comment/kp3qw8x/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)