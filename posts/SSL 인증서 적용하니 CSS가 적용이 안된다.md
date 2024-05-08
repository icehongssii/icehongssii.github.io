---
title: SSL 인증서 적용하니 CSS가 적용이 안된다
created: 2024-03-15 16:52
last-updated: 2024-03-15 16:52
tags:
  - troubleshooting
  - python
---

## 👯‍♂️ 문제; Mixed Content Error

![](https://i.imgur.com/1AePkzE.png)

사이트의 내용물은 제대로 나오는데 스타일이 적용되지 않았다!

```ad-error
title: Mixed Content Error
Mixed Content: The page at 'https://xxx.icehongssii.xyz/' was loaded over HTTPS, but requested an insecure stylesheet 'http://xxx.icehongssii.xyz/static/style.css'. This request has been blocked; the content must be served over HTTPS.
```


## 👯‍♂️ Mixed Content Error란?

- 생각보다 정의된지 얼마 안된 에러이다
- 2020년부터 크롬과 파이어폭스에서는 `https://`로 로딩되지 않는 리소스를 차단한다고 발표했다
- [Mixed Content 에러는 웹 페이지가 HTTPS를 통해 안전하게 로드되었지만, 페이지의 일부 자원(예: 스타일시트, 스크립트, 이미지 등)이 안전하지 않은 HTTP를 통해 요청되었을 때 발생. 이 경우 브라우저는 안전하지 않은 자원의 로드를 차단하여 사용자의 보안을 유지함. ](https://cert.crosscert.com/%EF%BB%BFhttps-%ED%81%AC%EB%A1%AC%EA%B3%BC-%ED%8C%8C%EC%9D%B4%EC%96%B4%ED%8F%AD%EC%8A%A4-%ED%98%BC%ED%95%A9-%EC%BB%A8%ED%85%90%EC%B8%A0mixed-content-%EC%B0%A8%EB%8B%A8/)
- [왜냐면 보안상의 문제가 있기 때문. 보안되지 않은 HTTP 프로토콜을 사용하여 하위 리소스를 요청하는 경우 해당 요청은 공격자가 네트워크 연결을 도청하고 양자 간 통신을 보거나 수정하는 수단인 중간자(man-in-the-middle) 공격에 취약하므로 전체 페이지의 보안이 약화된다고 한다](https://cert.crosscert.com/%EF%BB%BFhttps-%ED%81%AC%EB%A1%AC%EA%B3%BC-%ED%8C%8C%EC%9D%B4%EC%96%B4%ED%8F%AD%EC%8A%A4-%ED%98%BC%ED%95%A9-%EC%BB%A8%ED%85%90%EC%B8%A0mixed-content-%EC%B0%A8%EB%8B%A8/)



## 👯‍♂️ 해결방법; 웹 페이지내 모든 리소스를 Https로 변경한다

- 처음에는 리버스 프록시를 사용해야하나 싶었지만 이미내 웹사이트 ECS service에 ALB를 붙였고 이미 로드 밸런서 보안그룹에서 80번 포트로 오는 요청은 443으로 리다이렉트가 되도록 설정되어있다. 따라서 모든 요청은 https를 통해서 이루어지고있다  
![](https://i.imgur.com/RLMxmaB.png)

- 그래서 조금 더 찾아보니  확인해보니 코드레벨에서 수정해야하는 부분이 있는듯하다. 정확한 원인은 모르겠지만.. 나와 비슷한 케이스가 있었다. url_for이 https대신 http요청을 보내는 경우였다. 따라서 아래의 두가지 예제를 참조해 다음과 같은 코드를 작성했다 
	- [웹, 스택오버플로우, url_for generates http instead of https when running by docker](https://stackoverflow.com/questions/51287411/flask-url-for-generates-http-instead-of-https-when-running-by-docker)
	- [웹, 스택오버 플로우, FastAPI links created by url_for in Jinja2 template use HTTP instead of HTTPS](https://stackoverflow.com/questions/70521784/fastapi-links-created-by-url-for-in-jinja2-template-use-http-instead-of-https)

```python
from starlette.middleware.base import BaseHTTPMiddleware
from starlette.requests import Request
from starlette.responses import Response

class HTTPSRedirectMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        # Change the scheme to https
        request.scope['scheme'] = 'https'
        response = await call_next(request)
        return response
app.add_middleware(HTTPSRedirectMiddleware)
```

그런데 로컬에서는 이제 아예 경로를 못읽는 문제가 발생한다;; 아예 이미지가 로딩이 되지 않는다 (Invalid request) 왜냐면 로컬에는 SSL 인증서가 없기때문에.. 따라서  아래 내용을 추가했다
```python
from fastapi import FastAPI
from starlette.middleware.httpsredirect import HTTPSRedirectMiddleware
import os

app = FastAPI()

# 환경 변수를 체크하여 로컬 환경이 아닐 때만 HTTPSRedirectMiddleware를 적용합니다.
if os.getenv("ENVIRONMENT") != "local":
    app.add_middleware(HTTPSRedirectMiddleware)
```

![](https://i.imgur.com/ohTywYl.png)



--- 

## 👯‍♂️ Ref & LINKS TO THIS PAGE

- [웹, ecs fastapi와 리버스 프록시 적용예시  ](https://www.miketheman.net/2021/12/28/container-to-container-communication/)
- [웹, 스택오버플로우, url_for generates http instead of https when running by docker](https://stackoverflow.com/questions/51287411/flask-url-for-generates-http-instead-of-https-when-running-by-docker)
- [웹, 스택오버 플로우, FastAPI links created by url_for in Jinja2 template use HTTP instead of HTTPS](https://stackoverflow.com/questions/70521784/fastapi-links-created-by-url-for-in-jinja2-template-use-http-instead-of-https)
- [웹, alb 리다이렉트](https://stackoverflow.com/questions/70521784/fastapi-links-created-by-url-for-in-jinja2-template-use-http-instead-of-https)
- https://cert.crosscert.com/%EF%BB%BFhttps-%ED%81%AC%EB%A1%AC%EA%B3%BC-%ED%8C%8C%EC%9D%B4%EC%96%B4%ED%8F%AD%EC%8A%A4-%ED%98%BC%ED%95%A9-%EC%BB%A8%ED%85%90%EC%B8%A0mixed-content-%EC%B0%A8%EB%8B%A8/
