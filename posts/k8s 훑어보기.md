---
title: k8s 훑어보기
created: 2023-05-10 15:32
last-updated: 2023-05-10 15:32
tags:
  - 2023-k8s-bootcamp
  - k8s
description: k8s 기초
---

## k8s 워크로드

k8s는 선언적. yaml로 리소스를 구성 할 수 있는데 최초의 10개의 컨테이너를 유지하겠다고 작성하고 실행시켰지만 이를 8개로 수정시킨다면 두개의 리소스를 지울 필요 없이 8로 수정하면 k8s이 알아서 상태를 업데이트 해줌

> [!tip]  
> 미리 알아두면 좋은것
> - kubectl 별칭 k로 지정
> - `--dry-run=client` 
> - 업데이트시에 히스토리를 남기기위해 `--record=true`옵션

### Pod
- k8s 기본 빌딩블럭
- 보통 하나의 pod에 하나의 컨테이너 사용
- 하나의 pod는 단일 노드에만 배치 (하나의 pod이 두개 이상의 노드에 있을 수 없음)
- pod 네트워크? 

```
# run 명령어로 디폴트 네임스페이스에 젠킨스 POD 배치 
kubectl run [POD이름] --image=jenkins/jenkins --port:8080

# 조회
kubectl get po 
kubectl describe po [POD이름]
kubectl get po --show-labels

# 출력물
kubectl get po [POD이름] -o yaml 
kubectl get po [POD이름] -o wide

# POD 이름만 보기 
kubectl get po -o --no-headers custom-columns="metadata.name"
```

- 상태조회
	- readiness probe
	- liveness probe
	- startup probe

### 레이블달기

```
kubectl label [리소스타입] [리소스명] 키=밸류
kubectl label pod http-go foo=test
kubectl label pod http-go foo=test2 --overwrite
```

### Replicaset
- pod들을 관리하는 상위 객체. 
- pod에 문제가 생겼을 때 몇개를 생성할것인지
- 다수의 객체를 생성하는 것이기 때문에 각각의 pod의 붙여진 레이블로 관리한다 
- 1.8버전 전까지  replication controller라는 api 를 사용했으나 큰 차이는 없음 다만 replicaset이 더 다양한 레이블 선택 가능케함 

- 세가지방법
	- edit
	- apply
	- scale --replicas=5  

### Deploy
- replicaset을 관리하는 상위 객체
- pod 배포시 어떻게 업데이트를 할 것이니 관리하는 객체
- undo, revision등으로 기존 버전으로 갈 수 있음
```

# deploy 생성
k create deploy tomcat --image=consol/tomcat --dry-run=client -o yaml
# 롤아웃 변경사항 저장
kubectl edit deploy http-go --record=true
# 히스토리
kubectl rollout history deploy/tomcat
```
- 세가지방법
	- edit
	- apply
	- rollout

### Namesapce

- 클러스터 내에 여러개의 네임스페이스 가질 수 있음
- 게임서버가 여러개 있는 것 마냥 하나의 게임의 여러 네임스페이스

```
# 모든 네임스페이스 내에 있는 리소스 조회
kubectl get all --all-namespaces
# office 네임스페이스 내에 deployment 조회 
kubectl get deploy -n office

```

## k8s 서비스

- k8s 클러스터 내의 리소스 끼리 통신하거나 클러스터와 외부와 통신하기 위한 k8s 의 자원 이름
- 직업 pod에 통신 할 수도 있지만 스케일링시 바뀌는 IP 때문에 고정된 IP를 가진 서비스라는 리소스가 필요하게됨
- 서비스의 종류 
	- ClusterIP 
		- Endpoints 리소스 
	- NodePort
	- LoadBalancer
- k8s의 리소스를 외부와 통신 가능하게 만들려면?
	- Nodeport
	- Loadbalancer
	- Ingress

### clusterIP

- YML 파일 또는 `kubectl expose ` 로 생성한 디폴트 서비스.
- deploy 통해서  rs, pod등을 생성하게 되면 여기와 통신하게 될 클러스터 내 고정 IP
- 외부와 통신 X
- 로드 밸런싱되므로 이 IP로 접근하게 되면 클러스터 내 여러 pod과 통신가능 다만 로그인 기능 같은 경우 세션이 유지되어야하는데 이떄는 `sesionAffinity`를 통해 설정가능

### 노드포트

- 외부와 통신하게 도와주는 노드포트 
- 노드 자체에 포트를 열어줌
- vm이나 클라우드 사용시 방화벽 해제해줘야함
- port, targetPort, 노드자체포트를 yaml에 선언해줘야함

### 로드밸런서

- 외부와 통신하게 도와주는 k8s 포트 일종
- 클라우드 이용시에 자주쓰이게 됨

### Ingress

- 외부와 통신하게 도와주는 k8s 
- nginx-ingress 설치후 실습했었음
- 하나의 endpoint. 하나의 포트로 다양한 서비스에 접근할 수 있게 도와주는 일종의 룰 테이블? 
- 자체적으로 로드 밸런서 기능있음 /welcome/test 로 요청이 들어올 때만 이벤트가 일어나게 할 수 있음 

![](https://i.imgur.com/RK6YWyL.png)


## 스토리지

- emptyDir : pod내에서 컨테이너끼리 
- hostPath : pod와 컨테이너간
- 수동적 pv, pvc
- 동적 pvc, storageclass 그리고 headless service 
- configMap
- 스테이트풀셋 : 포드 상태 유지와 관련된 컨트롤러