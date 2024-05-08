---
title: k8s-서비스-외부로 서비스 노출하는 세가지방법
created: 2023-05-02T14:55:12
last-updated: 2023-05-22T14:55:12
tags:
  - 2023-k8s-bootcamp
  - k8s
description: 노드포트, 서비스 로드밸런서, ingress 이용해 외부로 서비스 노출
---
## 외부로 서비스 노출하는 세가지방법


서비스를 노출하기 위한 방법
- nodeport : 노드 자체 포트 사용해 포드로 리다이렉션. 노드 자체 포트 열고 있음. 로드 밸랜서 사용하면 앞에 실제 로드 밸런서가 생긴다 그리고 뒷쪽에는 클라우드 환경에서 지원하는 쿠버네티스 클러스터 노드 3개가 떠있다 즉 쿠버네티스 클러스터. 이 쿠버네티스 클러스트는 노드의 포트를 열고 있는데 이걸 노드포트라고함. 
- loadbalancer : 이는 클라우드 환경에서 자주 사용함. 사내 프라이빗 클라우드 구성하고 이를 제공할 떄 로드밸런싱할 수 있도록 loadbalancer라는..클러스터를 클라우드 환경에서 제공할떄 로드밸런서라는걸 이용하면 쉽게 이용가능하다
- ingress : 인그레스는 하나의 아이피로 다수 서비스 이용가능.. lloadblaancer가 L4라면 ingress는 L7. 어디로 서비스 보낼지..? 도메인 이름 해석하는, 서비스를 어디다가 해

로밸(서비스), 잉그레스 -> 클라우드에서 자주사용  
노드포트(서비스) 자체 -> 물리적인 기능 즉 vm에서 자주 사용하고 vm에서 로밸 잉그레스 실습은 좀 어려움

![](https://i.imgur.com/0XowSm3.png)


## 노드포트란?

포트(svc),타겟포트(pod) 
- 노드 자체에 포트를 제공 
- 노트포트가 열려있고 클러스터 내에 서비스가 있음 근데 서비스가 동작하는게 아니라 네트워크 담당하는 kube-proxy가 패킷을 받아서 서비스에 등록된 룰을 적용시켜준다. 
- 30001번 포트가 큐브 프록시에 등록되어 라운드로빈으로 적절한  pod에 연결해준다 만약에 Pod가 노드에 없으면 kube proxy가 알아서 다른 노드에 있는 적절한 kube proxy와 연결해 Pod으로 연결해준다.
- 3가지 단계 -> 노드포트-서비스-포드
- 이때 노드포트의 범위는 30000-32767사이에 있는것만 사용가능하다 
- 노드에 직접 연결 할 수 있게 만드는 포트라고 생각하면 될 듯함

|     |     |
| --- | --- |
| ![](https://i.imgur.com/41F8o98.png)| ![](https://i.imgur.com/TIDIAPd.png) | 
   


> [!important]  
> 구글클라우드에서 CP에서 방화벽 해체줘야 노드로 직접 접속가능하다


![](https://i.imgur.com/g6mqbo8.png)


- 포트가 하나 연결되어 있으면 클러스터 ip 테이블 설정에 따라서 넷필터(들어오는 트래픽 정해진 규칙 따라 포워딩) 적절한 pod으로 분배가 된다 

## 노드포트 실습
한편 서비스만 있으면 소용이 없으므로 pod이 띄어져있어야하는데 아래의 clusterIP타입의 서비스와 deploy를 생성하는 yaml로 리소스를 생성한다 
```yaml
# http-go-deploy.yaml
apiVersion: v1
kind: Service
metadata:
  name: http-go-svc
spec:
  selector:
    run: http-go
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
---

apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    run: http-go
  name: http-go
spec:
  replicas: 1
  selector:
    matchLabels:
      run: http-go
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: http-go
    spec:
      containers:
      - image: gasbugs/http-go
        name: http-go
        ports:
        - containerPort: 8080
        resources: {}
status: {}                
```

그리고 노드포트 타입 생성

```yaml
# http-go-np.yml
apiVersion: v1
kind: Service
metadata:
  name: http-go-np
spec:
  type: NodePort
  selector:
    run: http-go
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30001
```

![](https://i.imgur.com/J7A1z0O.png)  
이때 노드포트는 external IP가 없는데 저 옆에 있는 포트로 갈 수 있다. 


```
kubectl get nodes -o wide
```
![](https://i.imgur.com/2F3w5GN.png)

이때 각 노드의 IP:30001번으로 외부에서 접근가능하다 

> [!important]  방화벽을 열어둬야 외부에서 접근가능
> ```
> gcloud compute firewall-rules create http-go-svc-rule --allow=tcp:30001


## 서비스 로드밸런서란?

- 클라우드 환경에서만 사용가능
- 노드포트의 확장서비스
- 로드 밸런서 IP주소를 통해 서비스에 접근 
- 이때 이 로드밸런스 타입의 서비스는 노트포트의 기능을 포함한다 

|     |     |
| --- | --- |
| ![](https://i.imgur.com/dE4IdDF.png)| ![](https://i.imgur.com/UAln1JD.png)    |




vm에서 안되고 클라우드 서비스에서만 가능  
L4장비


![](https://i.imgur.com/U1sKGjQ.png)

30533은 임의로 정해진건  
로드밸런서는 노드포트에 직접 연결되고 클러스터 IP로 통신함

## 로드밸런서 + GCP

그전에 생성한 리소스 지우고 실행 
```yaml

# http-go-lb.yaml
apiVersion: v1
kind: Service
metadata:
  name: http-go-lb
spec:
  type: LoadBalancer
  selector:
    run: http-go
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080

---
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    run: http-go
  name: http-go
spec:
  replicas: 1
  selector:
    matchLabels:
      run: http-go
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: http-go
    spec:
      containers:
      - image: gasbugs/http-go
        name: http-go
        ports:
        - containerPort: 8080
        resources: {}
status: {}
```

ip 할당 받을 때 까지 기다려야함📝  
이제는 노드외부IP:30001아니라 80포트로 접속됨  
구글 클라우드에서 GCP 로드밸런서를 생성해줌  
|     |     |
| --- | --- |
|![](https://i.imgur.com/2SOrUJi.png)     | ![](https://i.imgur.com/DMlPemq.png)

```
curl 35.202.128.171
```



## 📝 연습문제

- tomcat 노드포트로 서비스(30002 포트)
- tomcat을 로드 밸런스로 서비스하기(80번 포트)

## 📝 연습문제 답안


- tomcat 노드포트로 서비스(30002 포트)

필요한게  deploy, service(노드포트)


```
# 톰캣 디플로이
:k  create depoy tomcat --image=consol/tomcat-7.0 --dry-run=client --port=8080 -o yaml >> tomcat-deploy-lb-np.yaml 
```

```
# 노드포트
k create service nodeport tomcat-np --tcp=80:8080 --dry-run=client -o yaml 
```
를 `tomcat-deploy-lb-np.yaml`에 붙여넣고  
`NodePort: 30002` 붙이고 서비스 selector에 app=tomcat으로 태그 변경 

```
# tomcat-deploy-np-lb.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: tomcat
  name: tomcat
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcat
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: tomcat
    spec:
      containers:
      - image: consol/tomcat-7.0
        name: tomcat-7-0-xqbmr
        ports:
        - containerPort: 8080
        resources: {}
status: {}
---
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: tomcat-np
  name: tomcat-np
spec:
  ports:
  - name: 80-8080
    port: 80
    protocol: TCP
    targetPort: 8080
    nodePort: 30002
  selector:
    app: tomcat
  type: NodePort
status:
  loadBalancer: {}
```


```
k scale rs [생성된rs이름] --replicas=5
curl [노드외부IP]:30002
```
두개의 노드에서 확인해보면 tomcat 페이지 나오는거 확인가능

- 노드포트 사용하는경우 30002 풀어야

![](https://i.imgur.com/wLIGbJg.png)


- tomcat을 로드 밸런스로 서비스하기(80번 포트)

```
k create service loadbalancer tomcat-lb --dry-run=client --tcp=80:8080 -o yaml
```

이때 나온 결과 `# tomcat-deploy-np-lb.yaml `에 붙여넣 고 `k apply -f tomcat-deploy-np-lb.yaml `  
이때 `k get svc`에 나온 로드밸런서의 외부 IP:80으로 curl해보면 접근가능한 것 확인 가능



## Ingress란?
![](https://i.imgur.com/vAUHAkg.png)

ingress앞에 배치하면 도메인이나, URI, URL에 따라서 gasbug.com 그때는 1번 포드, nodejs.gasbug면 2번 포드, 3번째는 URI, 요청들어오면 서비스 연결한다.  
사용자는 하나의 ip를 쓰면서 URI, 포트등으로 나눌 수 있다.


> 하나의 IP와 하나의 포트로 다수의 어플리케이션을 사용할 수 있다.


## Ingress-nginx이용


|     |     |
| --- | --- |
| ![](https://i.imgur.com/qmMrwtP.png)    | ![](https://i.imgur.com/vww5GVl.png)     |


- 클러스터안에 수많은 노드가있고 거기에 nginx띄어놓. nginx는 내부적으로 포워딩하는 기능 포함함. 근데 nginx ingress사용하면 포트 분석해서 그 룰을 동적으로 세팅하게끔 해주는 기능을 사용할 수 있다.  외부 사용자가 클러스터에 접근하면 nginx가 적절한 서비스를 연결하게 할 수 있다. 
- gcp에서 제공해주는 클러스터앞에 ALB가 nginx ingress같은 룰을 적용함

이때 둘의 아키텍처는 다르지만 클러스터 내부에서 분산되는거고,  URI나 도메인으로 분산시스템 구축하는건 같다.



## Ingress-nginx 설치(230522)
```yaml
git clone https://github.com/kubernetes/ingress-nginx/
kubectl apply -k `pwd`/ingress-nginx/deploy/static/provider/baremetal/
# 마지막은 웹훅구성삭제. 이기능으로  ingress 잘 안되는 현상있어서
kubectl delete validatingwebhookconfigurations.admissionregistration.k8s.io ingress-nginx-admission
```

nginx ingress설치하면 ingress라는 룰을 그 자원을 만들면 정보를 읽어서 그룰대로 세팅

![](https://i.imgur.com/OuVPY9b.png)



## Ingress실습

```yaml
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: http-go-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /welcome/test
spec:
  rules:
    - http:
        paths:
          - pathType: Exact
            path: /welcome/test
            backend:
              service:
                name: http-go
                port: 
                  number: 80
EOF
```


ingress-nginx 네임스페이스에 생성되어있다 
```
k get all -n  ingress-nginx
```

서비스 구성

```sh 
kubectl create deployment http-go --image=gasbugs/http-go:ingress # 인그레스 테스트용 http-go
kubectl expose deployment http-go --port=80 --target-port=8080
```

```
kubectl exec -it http-go-759bdc4cf8-rv9gf --bash
# 404 페이지가 뜬가
$ curl 127.0.0.1:8080 
# ingress 룰 설정이 잘 된 모습
$ curl 127.0.0.1:8080/welcome/test 

```




어디로 요청보내야 뒤에있는 노드로 전달할수있을까?  
kubectl get all -n ingress-nginx 네임스페이스에서 `ingress-nginx-controller`로 접근하면된다

![](https://i.imgur.com/rQk6NgW.png)

ingree-nginx-controller 31471 포트로보내면 파드가 아니라 앞에 있는 nginx가 요청을 받게 되므로

`curl 127.0.0.1:31471/welcome/test` 로 요청을 보내야  
파드의 결과가 나온다  
![](https://i.imgur.com/aterMgk.png)



## Ref
- 인프런 - devops를 위한 쿠버네티스