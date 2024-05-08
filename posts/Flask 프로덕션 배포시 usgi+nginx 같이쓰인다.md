---
title: Flask 프로덕션 배포시 usgi+nginx 같이쓰인다
created: 2024-03-11 16:23
last-updated: 2024-03-11 16:23
tags:
  - python
  - web
description: 단순한 서버/개발용이라면 fastapi 사용. 복잡+프로덕션은 다른 프레임워크 고려
---


## 👯‍♂️ Flask로만 프로덕션 환경에 배포 하지 않는다 .

앗.. 단순리 flask app만든다고하면 그냥 이건 도커라이징해서 ECR에 올리고 ECS로 배포하면 되지 않아?라고 생각했었는데 좋지 않은 생각이라고 한다..

```ad-warning
 **Do not use the development server when deploying to production. It is intended for use only during local development. It is not designed to be particularly secure, stable, or efficient.**
 - [flask 홈페이지](https://flask.palletsprojects.com/en/3.0.x/deploying/) - 
```


### 개발 vs. 운영(Production)

- **개발(Development)**: 애플리케이션을 만들고 테스트하는 과정에서 사용. 개발용 서버는 일반적으로 디버깅이 용이하고 코드 변경 사항이 자동으로 적용되지만, 보안이나 안정성, 효율성 측면에서는 적합하지 않다
- **운영(Production)**: 개발이 완료된 애플리케이션을 사용자들이 실제로 사용하는 환경을 말한다. 실제 사용자가 접속하고, 실제 데이터를 다루게 됩니다. 따라서 보안, 안정성, 효율성이 매우 중요함

### WSGI 서버

<mark style="background: #FFF3A3A6;">Flask와 같은 Python 웹 애플리케이션은 WSGI(Web Server Gateway Interface)라는 표준을 사용하여 웹 서버와 통. 운영 환경에서는 Flask의 개발용 서버 대신 Gunicorn, uWSGI와 같은 WSGI 서버를 사용한다. 이 서버들은 Flask 애플리케이션을 안정적으로 운영할 수 있도록 설계되었으며, 다음과 같은 이유로 권장함:</mark>

- **보안**: WSGI 서버는 공개적으로 사용될 수 있도록 보안 측면에서 강화됨
- **안정성**: 동시 요청 처리와 에러 관리가 더 잘 되어 있어, 애플리케이션이 더 안정적으로 실행
- **효율성**: 리소스(메모리, CPU 등) 사용이 최적화되어 있어, 더 많은 사용자를 처리



### 따라서 보통은 프로덕션에서는 Flask + Nginx + WSGI로 배포한다

![](https://i.imgur.com/xN8zNr9.png)

저글[^1] 에서도 아래 처럼 말한다..

```ad-note
flask에 기본적으로 개발용 서버 포함되지만 테스트와 개발 목적으로 주 목적이다.  
따라서 flask 그냥 사용하지 않고 nginx와 wsgi와 같이 사용하게 되는 많다.  

- nginx로는 느린 네트워크 연결 가진 사용자 요청 효율적으로 처리 가능, 보안강화 역할
- wsgi는 nginx로 부터 받은 웹요청 flask로 어플리케이션 전달하고 flask 어플리케이션 응답을 다시 nginx로 보내는 역할
```

이렇게 자주 쓰이는 기술 스택이다 보니 [uwsgi-nginx-flask-docker](https://github.com/tiangolo/uwsgi-nginx-flask-docker) 이렇게 하나의 이미지로 묶어서 사용되는 듯하다..  그러나 해당 리포 리드미를 읽어보니 

![](https://i.imgur.com/AhXLx3A.png)

- WSGI말고 ASGI를 사용한  fastapi나 starlette를 추천하고 있다
- 왜냐면 이 둘을 사용사용하면 uwsgi-nginx-flask를 사용한 것보다 8배의 성능을 낼 수 있어서
- 여전히 그래도 플라스크가 필요하다면 대안으로 [meinheld-gunicorn-flask-docker](https://github.com/tiangolo/meinheld-gunicorn-flask-docker) 이 이미지를 추천하는데 확인해보니 이 이미지에서 지원되는 파이썬 최신버전은 3.9까지여서 사용 불가 


![](https://i.imgur.com/CfBwn1K.png)  
[마찬가지로 ](https://woolbro.tistory.com/94)[^3] nginx-wsgi-flask로 쓰이는걸 보여주신다..





## 👯‍♂️ Ref & LINKS TO THIS PAGE

- [웹,  flask tutorial about Deploying Produciton](https://flask.palletsprojects.com/en/3.0.x/deploying/)
- [웹, uwsgi-ngix-flask-docker 이미지](https://github.com/tiangolo/uwsgi-nginx-flask-docker)
- https://woolbro.tistory.com/94
- [^1] [reddit, # Dockerifying a Flask App for ECS -> Nginx+Gunicorn+Flask in 1 container?](https://www.reddit.com/r/devops/comments/e3riwk/dockerifying_a_flask_app_for_ecs/)