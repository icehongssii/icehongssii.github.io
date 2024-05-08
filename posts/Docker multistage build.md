---
title: Docker multistage build
created: 2024-03-19 00:16
last-updated: 2024-03-19 00:16
tags:
  - docker
  - ci/cd
description: docker image 사이즈와 빌드속도 최적화
---

## 👯‍♂️ Intro & tl;dr

<mark style="background: #CACFD9A6;">도커이미지 사이즈 줄이는 방법중 하나 멀티스테이지.  
멀티 스테이지는 하나의 도커파일에서 이미지를 빌드하는 단계를 여러개로 나누어 필요한 것만 마지막에 도커 이미지로 만들어 질 수록 함!</mark>

아래는 python3.9이미지를 베이스로 이미 주피터와 판다스 빌드시킨 builder 이미지를 사용한다

```dockerfile
# temp stage
FROM python:3.9 as builder
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /wheels jupyter pandas

# final stage
FROM python:3.9-slim
WORKDIR /notebooks
COPY --from=builder /wheels /wheels
RUN pip install --no-cache /wheels/*
```

그럼 이렇게 사이즈가 가벼워집니다


--- 

## 👯‍♂️ 멀티 스테이지 빌드 목적

- 도커 이미지 크기를 줄이는게 주 목적
- 여러 단계의 빌드 단계를 만들고, 다른 단계의 결과물에서 특정 폴더만 복사해올 수 있다.
- 컴파일이나 패키지 설치 과정에서 부수적으로 발생하는 불필요한 파일들은 배제하고, 애플리케이션 구동에 필요한 것만 선택하여 가져오므로 이미지를 경량화시킬 수 있다.

## 👯‍♂️ 다른 베이스 이미지에서 필요 한 것만 가져 올 수 있다

```dockerfile
FROM golang:1.16 AS builder
WORKDIR /go/src/github.com/alexellis/href-counter/
RUN go get -d -v golang.org/x/net/html  
COPY app.go ./
RUN CGO_ENABLED=0 go build -a -installsuffix cgo -o app .

FROM alpine:latest  
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /go/src/github.com/alexellis/href-counter/app ./
CMD ["./app"]
```

두 번째 `FROM` 이 새 빌드 스테이지 시작 `alpine:latest` 를 베이스로 `COPY --from=0` 이 그전 스테이지에 있는 artifact를 가져와서 사용함. 

Go SDK랑 중간에 사용하지 않는 것들은 파이널 이미지로 사용하지 않음 . 

예제에서는 `--from=builder` 스테이지 이름 사용

```
docker build -t alexellis2/href-counter:latest .
```


## 👯‍♂️ 특정 스테이지에서 멈출 수 있다!

위의 예제에서 `builder`를 대상으로 한 이미지를 빌드하는 거 가능

```
docker build --target builder -t alexellis2/href-counter:latest .
```

- --target builder : 이 옵션은 Dockerfile내에서 여러 빌드 스테이지 있을 때 특정 스테이지까지만 빌드하라는 의미임!  
builder 스테이지까지만 빌드 수행하고 생성된 이미지를 href-counter에 저장하라는 의미이다

## 👯‍♂️ Ref & LINKS TO THIS PAGE

- [https://testdriven.io/blog/docker-best-practices/](https://testdriven.io/blog/docker-best-practices/)

- [https://docs.docker.com/build/building/multi-stage/](https://docs.docker.com/build/building/multi-stage/)