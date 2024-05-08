---
title: ecs service 만들지 않고 task에 service 붙일 수 있나?
created: 2024-03-16 22:16
last-updated: 2024-03-16 22:17
tags:
  - troubleshooting
  - aws
description: ecs task 직접 사설 도메인 붙이는건 안티패턴이다
---
2018.05-2019.03

## 👯‍♂️ 붙일 수 있지만 Best Practice는 아닌 듯 하다

- AWS ECS에서는 서비스 없이 단독으로 실행되는 태스크에 직접 ALB(Application Load Balancer)를 연결하는 것은 지원하지 않는다
- ALB는 ECS 서비스에 등록된 태스크 세트와 통합되어 동작하기 위해 설계되어있기 때문이다
- 따라서 ECS 서비스를 생성하고 ALB와 연동해서 트래픽을 태스크로 라우팅
- 서비스 없이 단독 태스크를 실행하는 경우에는 ALB를 사용하지 못하고, 태스크가 제공하는 특정 포트에 직접 접근해야한다.. 근데. 이는 보통 개발 환경이나 일시적인 배치 작업에 사용되는 패턴이라고한다 






## 👯‍♂️ Ref & LINKS TO THIS PAGE

-  https://shaungc.medium.com/protect-your-aws-ecs-service-behind-application-load-balancer-alb-and-create-them-by-terraform-6db816d329f7
- 
