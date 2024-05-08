---
title: Docker 이미지 크기 최적화
created: 2024-03-18 18:26
last-updated: 2024-03-18 18:26
tags:
  - docker
  - ci/cd
description: 성능에 도움되는 dockerfile 작성법
---


## 👯‍♂️ 골든 이미지를 찾아내자


모든 상황에 정답인 실버 불릿은 없다  
따라서  서비스마다 개발 환경마다 적절한 이미지를 사용하는게 중요하다.

예를 들면 파이썬 앱 개발시 무조건 베이스 이미지로  크기가 작은  alpine을 추천하는 경우가있다.  
근데 [웹, 2021, Using Aplpine can make Pytho builds 50x slower](https://pythonspeed.com/articles/alpine-docker-python/) 라는 글을 보면 알겠지만 (~~물론 해당 글은 무조건 alpine 쓰지마세요! 라는 내용이지만~~) 머신러닝과 같이 시스템 라이브러리가 필요한 복잡한 경우에는 alpine 이미지가 적절치 않다 (gcc를 필요로하는등 오히려 이미지 크기가 더 커진다)

반대로 Flask 같이 가벼운 웹 프레임어크를 사용한다면 alpine 정도면 훌륭하다.



## 👯‍♂️ Dockerfile 작성시 수정 자주 일어나는 곳은 파일 하단에 작성

도커파일 작성시 명령어 순서가 중요하다!  
Docker build 과정은 `정렬된` 빌드 명령어로 구성되어 있기 때문이다.  
아래 이미지처럼 Docker builder가 각 명령어를 이미지 레이어 스택으로 변환한다

![](https://i.imgur.com/20KgSW6.png)

Docker는 각 단계(각 레이어)를 캐시에 저장해서 다음에 image 빌드 속도를 높인다.  
이전 이미지랑 비교해서 레이어가 변경되지 않은 경우 builder는 빌드 캐시에서 레이어를 가져온다 근데 <mark style="background: #FFF3A3A6;">특정 레이어가 변경되었으면 그 뒤의 있는 모든 레이어를 다시 빌드한다</mark>  
따라서 변경사항이 자주 일어나는 명령어는 Dockerfile의 하단에 작성하는게 좋다. 그래야 최대한 빌드캐시에 있는 레이어들을 재사용할 수 있으니까

따라서 아래의 도커 파일은 비효율적이다

```dockerfile
FROM python:3.9-slim 
WORKDIR /app 
COPY sample.py . 
COPY requirements.txt . 
RUN pip install -r /requirements.txt
```

- requirments를 인스톨하기전에 이미 애플리케이션 코드를 복사했다(`COPY sample.py`)
- 따라서 `sample.py`에서 수정이 일어날 때 마다 다커 빌드가 계속해서 파이썬 패키지를 인스톨할 것임 😨 

만약에 개발 환경을 컨테이너 내에서 사용 중이라면 위의 도커파일은 확실히 구리다고 말하 수 있다(`sample.py`  가 계속 바뀔테니)  
따라서 아래와 같이 바꿔주는ㄴ게 낫다 

```dockerfile
FROM python:3.9-slim 
WORKDIR /app 
COPY requirements.txt . 
RUN pip install -r /requirements.txt
COPY sample.py .
```


## 👯‍♂️ 레이어의 개수를 줄인다; RUN 자주 쓰지말자

```dockerfile
FROM ubuntu:22.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

위에서 말했듯이 도커의 image는 여러개의 레이어로 구성 되어있는데 이 레이어는 결국에 Dockerfile에 작성된 각 명령얼르 나타낸다. 이때 `정렬된 명령어 순서대로` 레이어는 쌓여있다.  

항상 각 줄의 명령어가 새로운 레이어를 만드는 것은 아니지만  
위 Dockerfiled에서 각 줄의 명령어는 각 각 하나의 레이어를 만들어낸다 (계속해서 변경사항 발생함)
- FROM : 우부툰 이미지를 통해 새 레이어 생성
- COPY :  dockerfile 있는 현재 디렉토리에서 파일 추가  
  RUN  : make 사용해 앱 빌드  
  CMD   : 컨테이너 내에서 실행 할 명령어 지정 

그런데 저러한 명령어들은 새 레이어를 만들게 되므로결국에 이미지 사이즈가 커지게 된다.  
![](https://i.imgur.com/O79foVm.png)

 COPY, RUN이 이미지 사이즈 늘리고 있다 . 따라서 RUN 명령어를 사용 할 때는 아래처럼 뭉탱이로 쓰는게 좋다. 그리고 가독성을 위해 정렬도 해주고 아래처럼 줄바꿈 문자도 사용해준다 
```dockerfile
RUN apt-get update && apt-get install -y \ 
	git \ 
	gcc \ 
	matplotlib \ 
	pillow \ 
	&& rm -rf /var/lib/apt/lists/*
```



## 👯‍♂️ Ref & LINKS TO THIS PAGE

- [웹, 공식 도커 홈페이지, Layers](https://docs.docker.com/build/guide/layers/)
- [웹, 공식 도커 홈페이지, Dockerfile](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [웹, test-driven, 도커 best-practice]([https://testdriven.io/blog/docker-best-practices/](https://testdriven.io/blog/docker-best-practices/))
- [웹, 2021, Using Aplpine can make Pytho builds 50x slower](https://pythonspeed.com/articles/alpine-docker-python/)
- [웹,ycombinator, pytho Alpine 이미지 관련 댓글](https://news.ycombinator.com/item?id=22182226)




