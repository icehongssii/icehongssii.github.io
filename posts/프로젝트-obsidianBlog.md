---
title: 프로젝트-obsidianBlog
created: 2024-03-18 17:48
last-updated: 2024-03-18 17:48
tags:
  - toyproject
  - obsidian
description: 토이 프로젝트 개발기
---

## Obsidian 블로그 개발기

![diagram.excalidraw.svg](https://i.imgur.com/1Gy30lv.png)

## 👯‍♂️ 개요

기술 블로그에 포스팅할려면.. 개인적으로 공부한 내용을 이쁘게/가독성 좋게 꾸미는 과정이 필요합니다....

예를 들면 요즘에 노션에다가 개인 공부 한 내용 많이 저장하던데 이 내용 그대로  복붙해서 네이버 블로그에 못 올립니다.. 마크다운 호환이 안돼서.. 그래서 글씨 두껍게하는거 이런거 다 일일이 바꿔줘야합니다..

<mark style="background: #ABF7F7A6;">이 작업이 귀찮아서 그냥 개인 공부 저장된 메모어플 통째로 블로그로 만들고 싶었습니다..</mark>

사실 옵시디언(메모어플)-> 곧바로 웹페이지 만들어주는 서비스들이 존재합니다만.. 저는 인프라부터 제가 해보고 싶어서 굳이 굳이 처음부터 만들어봤습니다..


- [GIthub](https://github.com/icehongssii/personal-blog-render)
- [Website](https:///www.xxxx.icehogssii.xyz)
- 진행기간  2024.03.05 - 계~에속 진행중 / 마일스톤 추후 업데이트 예정
- 목표 
	- 메모 어플에서 작성한 내용을 곧바로 블로그로 퍼블리싱 가능하게하자
	- 기존에 비슷한 서비스를 하는 곳(obsidian flower show, obisidian publish)이 있긴하지만 직접 서버까지 운영해보고 싶어서 만들었다
- 추가 해야하는 기능
	- ✔️ 검색
	- ✔️ 페이징 
	- ✔️ 카테고리리
	- ✔️ 댓글 
	- ✔️ CI 구현(테스트, 린트)
	- ✔️ 인프라 구조 테라폼으로 작성
	- ✔️ ECS task 무중단 배포 (FastAPI 코드 수정하면 ECR에 올라가지만 이 최신 이미지를 토대로 컨테이너가 생성되지는 않음) -> CD구현



## 👯‍♂️ 역할

- 기획 / 개발 


## 👯‍♂️ 사용기술

- Python
- Docker build, buildX, GitubAction
- AWD ECS, ECR, ALB,

## 👯‍♂️ 프로젝트 진행하면서 경험하고 체득한 부분

- **클라우드 서비스 및 배포**: ECS, Redis, SSL, GitHub Actions
	- [ECS service없에 task에 사설 도메인 붙일 수 있을까?..](https://xxx.icehongssii.xyz/posts/ECS%20service없에%20task에%20사설%20도메인%20붙일%20수%20있을까%3F...md)
	- [API 호출 수를 줄이기 위한 Redis 연결](https://xxx.icehongssii.xyz/posts/API%20%ED%98%B8%EC%B6%9C%20%EC%88%98%EB%A5%BC%20%EC%A4%84%EC%9D%B4%EA%B8%B0%20%EC%9C%84%ED%95%9C%20Redis%20%EC%97%B0%EA%B2%B0.md?ref=main)
	- [github action 알아보기](https://xxx.icehongssii.xyz/posts/github%20action%20%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0.md?ref=main)
- **웹 보안**: SSL 적용, mixed contents 에러
	- [SSL 인증서 적용하니 CSS가 적용이 안된다](https://xxx.icehongssii.xyz/posts/SSL%20인증서%20적용하니%20CSS가%20적용이%20안된다.md)
- **웹 서버 및 프레임워크**
	- [Flask 프로덕션 배포시 usgi+nginx 같이쓰인다](https://xxx.icehongssii.xyz/posts/Flask%20%ED%94%84%EB%A1%9C%EB%8D%95%EC%85%98%20%EB%B0%B0%ED%8F%AC%EC%8B%9C%20usgi%2Bnginx%20%EA%B0%99%EC%9D%B4%EC%93%B0%EC%9D%B8%EB%8B%A4.md?ref=main)
- **호환성 및 배포 문제**
	- [uvicorn-gunicorn-fastapi does not support ARM64; docker buildx](https://xxx.icehongssii.xyz/posts/uvicorn-gunicorn-fastapi%20does%20not%20support%20ARM64%3B%20docker%20buildx.md)



