---
title: uvicorn-gunicorn-fastapi does not support ARM64
created: 2024-03-12 20:11
last-updated: 2024-03-12 20:11
tags:
  - python
  - docker
  - troubleshooting
  - CS
---

## 👯‍♂️ [uvicorn-gunicorn-fastapi](https://github.com/tiangolo/uvicorn-gunicorn-fastapi-docker)가 ARM64 Support를 하지 않는다



```dockerfile
FROM tiangolo/uvicorn-gunicorn:python3.10-slim
LABEL maintainer="Sebastian Ramirez <tiangolo@gmail.com>"
WORKDIR /app
COPY pyproject.toml poetry.lock* /app/
RUN pip install --no-cache-dir poetry
RUN poetry config virtualenvs.create false \
	&& poetry install --no-interaction --no-ansi
COPY ./api /app
```

이 이미지를 빌드하려고하는데  아래와 같은 오류가 발생한다
```
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested
```

- 즉 내 로컬(맥북 m1) 에서는 아래 이미지 빌드가 되지않는다 내 시스템의 아키텍처(`linux/arm64/v8`)와  이미지의 아키텍처(`linux/amd64`)가 일치하지 않는다

- 이는 일반적으로 ARM 기반 시스템(예: Apple M1/M2 칩이 장착된 Mac, 일부 ARM 기반 Linux 서버)에서 AMD64(혹은 x86_64) 아키텍처용으로 빌드된 Docker 이미지를 실행하려고 할 때 발생
	- CPU = 명령어 실행하는 애, 
	- 명령어 집합 = CPU가 이해할 수 있는 명령어 모음
	- 근데 모든 CPU가 똑같이 생긴 명령어를 실행 할 수 있는게 아니다 왜냐면 CPU제조사별로(아키텍쳐별로) CPU가 다르고 이에 따라 이해 할 수 있는 명령어 집합이 달라지기 때문!  
	- 그러니까 지금 내 노트북 m1은 arm64 아키텍처이고 해당 이미지는 amd64만 지원하는 이미지이기 때문에 내 노트북이 알아들을 수 없는 것이다

[fastapi작성사 tiangolo는 python 공식 docker image는 멀티 아키텍처 지원하지만 도커 허브 자동빌드는 ARM 아키텍처로 컴파일 할 수 가없다고 함 따라서 직접 이 리포 클론해서 수정하라고하지만 이상적이지는 않다고한다](https://github.com/tiangolo/uvicorn-gunicorn-fastapi-docker/issues/5)


직접적으로 해당 이미지를 수정하지 않는 방법을 찾아보다가 Docker buildx를 알게되었음


--- 

## 👯‍♂️ Docker Buildx를 이용하자

- docker buildx 명령어는 멀티 아키텍처 이미지를 빌드할 수 있게 해준다.  예를 들어, Linux의 AMD64 및 ARM64 아키텍처를 위한 이미지를 동시에 빌드할 수 있다
- 이 buildx는 `buildkit`을 사용하는데 `buildkit`은 캐싱 메커니즘을 개선하며, 보안을 강화하고 빌드 속도를 향상시킬수있게 도와주는 빌드엔진이다. 제공하는 기능중에는 병렬빌드도 있다..  <mark style="background: #D2B3FFA6;">buildkit은 Docker Engine v23. 0부터 디폴트 빌드엔진이다</mark>

병렬빌드는 아래와 같은 걸 의미함. C++  Go를 베이스를한 이미지빌드 과정이다  
보면 알겠지만 cpp, go 빌드 스테이지를 병렬로 실행 할 수 있을 것같다 그리고 buildkit은 이러한 병렬빌드를 실행시켜준다 

```Dockerfile
FROM ubuntu:18.04 AS build-cpp ########### C++ 빌드
RUN apt-get update && apt-get install -y build-essential
COPY src/cpp .
RUN gcc -o libfoo.so -shared foo.c

FROM golang:1.16 AS build-go ########### Go 빌드 
COPY src/go .
RUN go build -o app

FROM debian:buster
COPY --from=build-cpp libfoo.so .
COPY --from=build-go app .
```
한 번 위의 이미지를 빌드 해봤다(내 작업디렉토리에는 src없다). 신기하게도  
동시에 `src/go`, `src/cpp`에서 오류가 난다!  
![](https://i.imgur.com/9nKMAxH.png)

여튼 다시 본론으로 돌아와서 멀티 아키텍처 빌드를 위해 이미지를 buildx로 빌드한다.
```
docker buildx build --platform linux/arm64/v8 -t icehongssi/uvicorn-gunicorn-fastapi-arm:latest  --load
```
각 옵션에 대한 설명 그리고 docker build

- `--platform linux/amd64, linux/arm64/v8` :빌드 대상으로 AMD64(일반적인 x86_64 아키텍처 컴퓨터에서 사용됨)와 ARM64(ARM v8, 예를 들어 새로운 Apple Silicon Macs나 많은 최신 Android 스마트폰에서 사용됨) 아키텍처를 지정함. 로컬에서만 사용 할 것이므로 arm64로 명시한다 
-  `--push` : 원래 그냥 `dockr build` 에서는 빌드 후에 레지스트리로 곧바로 푸시 할 수 없다 하지만 buildx에서는 --push를 통해 빌드 완료후 생성된 이미지를 도커 허브 또는 설정된 컨테이너 레지스트리에 푸시할 수 있다 
- `--load` : 곧바로 빌드된 이미지를 로컬로 가져온다  

만약 amd64로도 빌드하고 싶다면 아래와 같이 하면 된다.. amd64는 m1에서 호환되지 않으므로 레지스트리에 저장한다
```
docker buildx build --platform linux/amd64,linux/arm64/v8 -t icehongssi/uvicorn-gunicorn-fastapi-arm:latest --push . 
```
`docker buildx build` 를 통해 멀티 아키텍처 이미지를 빌드했다면

`--load`를 통해 이미지를 로컬에서 빌드했으므로  그리고 나서 `docker run`으로 실행시켜주면 해당 이미지를 사용 할 수 있다



--- 

## 👯‍♂️ Ref & LINKS TO THIS PAGE

-  https://github.com/tiangolo/uvicorn-gunicorn-fastapi-docker/issues/5
- [https://medium.com/@artur.klauser/building-multi-architecture-docker-images-with-buildx-27d80f7e2408](https://medium.com/@artur.klauser/building-multi-architecture-docker-images-with-buildx-27d80f7e2408)  
- [https://collabnix.com/building-arm-based-docker-images-on-docker-desktop-made-possible-using-buildx/](https://collabnix.com/building-arm-based-docker-images-on-docker-desktop-made-possible-using-buildx/)
- buildkit https://www.youtube.com/watch?v=5EuDY6ayNs8
- https://blukat.me/2021/07/docker-buildkit-speedup/
- https://minkukjo.github.io/devops/2020/04/06/Docker-01/
- https://docs.docker.com/reference/cli/docker/buildx/