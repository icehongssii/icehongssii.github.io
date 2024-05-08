---
title: k8s-파드디자인패턴과잡실행
created: 2023-06-02T14:55:12
last-updated: 2023-06-02T14:55:12
tags:
  - 2023-k8s-bootcamp
  - k8s
description: 사이드카,어댑터, 앰버서더 컨테이너와 크론잡
---


## 사이드카 컨테이너

- 이륜차 기존기능 향상/확장(=파드의 기능 향상)
- 파드 파일 시스템 공유하는 형태
- 보조컨테이너
- k8s에서는 파일시스템공유하는 형태
- app컨테이너가 파일시스템에 로그저장하면 스티리밍 컨테이너 통해서 stdin stdoutㅇ로 출력하는 기능필요 그 기능위해 사이드카 컨테이너 사용  
![](https://i.imgur.com/ILaiSVF.png)  
![](https://i.imgur.com/rt7VW3q.png)
- nginx보조한느 두개의 컨테이너
- emptyDir볼륨 구성하고 (이름으로 배칭), 사이드컨테이너는 비지박스

```
# nginx-sidecar.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-sidecar
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: varlognginx
      mountPath: /var/log/nginx
  - name: sidecar-access
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/nginx/access.log']
    volumeMounts:
    - name: varlognginx
      mountPath: /var/log/nginx
  - name: sidecar-error
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/nginx/error.log']
    volumeMounts:
    - name: varlognginx
      mountPath: /var/log/nginx
  volumes:
  - name: varlognginx
    emptyDir: {}
```



```
// 배포 
vim nginx-sidecar.yaml
kubectl apply -f nginx-sidecar.yaml
// 액세스 로그 남기도록 유도 
kubectl exec nginx-sidecar -c nginx -- curl 127.0.0.1 -s
// 액세스 로그는 sidecar-access 에서 보도록 한다.
# kubectl logs nginx-sidecar sidecar-access
```


## 어댑터 컨테이너

- 본질적으로 이질적인 응요프로그램을 저용간으하도록 개선하는 컨테이너
- 그 컨테이너가 원래 a사에서 실행시키던거, 근데 그 컨테이너가져와서 b사에 적용하려고함 근데 로그남기는규칙이거나 출력개선만하면 사용가능. ㄴ가 만든게 아닌 컨테이너라 수정하기가 힘듬 따라서 기존프로그램 유지하고 출력 만 변경하려고함 호환성
- 원본컨테이너에 대한 변경사항없이 현재 커네이너 기능을 시스템에 적용시키는 기능

https://gasbugs.notion.site/ade367d87dc94bf2aee3d7c46fe3b440?v=7c3e779220b241b9bb9e3e7190b63261&p=341c3e8e5a2a43538b33e9f9ebce3889&pm=s

- 예제코듭 분석  
![](https://i.imgur.com/zhE3At1.png)

kubectl apply -f https://raw.githubusercontent.com/bbachi/k8s-adaptor-container-pattern/master/pod.yml


- 도커파일 ; 어댑터 컨테이너
- manifests.yaml : k8s에 있는 busybox()

kubectl port-forward adapter-container-demo 8080:3080


## 엠버서더 컨테이너

- 에버서더, 국가공무원 외교대표하는 대사의미
- 엠버서더 커네이너는 파드 이부의 서비스를 액서스를 간소화하는 특수 유형
- ㅠㅏ드에 앰버서더 컨테이널르 배치해 통신을 대신해주는 역할 소화
- 서비스 인증, 데이터 변조, 감시등 다양한 작업 가능
- 프로기  
![](https://i.imgur.com/59LiSph.png)


![](https://i.imgur.com/y3QUGkw.png)

외부사용자로부터 3000번 요청이 들어오면 엠버서더 컨테이너로 요청 보냄 다른경로로 요청 넘긴다  
(프록시태워서 remote service로)

kubectl exec -it ambassador-container-demo -c ambassador-container -- curl localhost:9000 (응답이 안옴 그게 정상임)
- 현재 정상작동되지않음  
kubectl logs ambassador-container-demo main-container  
/node_modules/request-promise-core/lib/errors.js:32:15


## Job과 크론잡

- 예약된시간에 포드 실행될수잇도록 도와주는것
- rs, deploy는 서비스 유지하고 지속적 서비스 유지하기위한 파드 직접실행..
- 하드를 일회성으로 정회진 기능만큼 수련하기위해
- k8s 클러스터 운영시 일정주기마다 돌아가야하는작업 존재함
- 넷플릭스 추천서비스 좋은거 많이필요, 빅데이터 ㅜ집 및 분석 필요 이거 넷플리스에서도 함..
- 온라인에서는 실제 유저들 접속하고 관심있게 보는걸 실시간 맞춤형, nearline은 사용자 보고있을 때 뒤에서 분석함, offline은 완전히 많은 리소스 소비, 하루에 한번 정도 분석 이런거는 서비슷 ㅏ용자가 적을 때 하ㅡㄴ게 맞다.
- ![](https://i.imgur.com/vfF40X9.png)

job 
- 하나이상 하드만들고 지정된 수 파드가 성공적으로 종료될때까지 pod 실행 계속 재시도
- 프로세스 0 반환하면 complete 
- 파드가 complete될때까지 실행
- 생성한 파드 작어이 다시 재게될때까지 활성 파드가 삭제
- 작업을 사용하여 여러파드를 병렬로 실행가능(병렬1 완료1) 실패하면 재시작회수 backofflimit 최대수행계수
- 잡에 대한 병렬실행방법
	- 비병령잡 : 기본값, 한번에 하나씩만 실행됨 파드종료시잡 완료
	- 정해진횟수 반복하늕는 잡: spec.completations에 0아닌 양수 지정
	- 병렬실행가능 수 지정 : spec paralleism에 0이 아닌 양수 지정하면 정해진 개수만큼 파드가 동시에 실행가능

## 크론잡

- job을 시간 예약해서 
- pod여러개 띄울때 어느것부터 .. 정책을 설정  
![](https://i.imgur.com/p5K16yb.png)