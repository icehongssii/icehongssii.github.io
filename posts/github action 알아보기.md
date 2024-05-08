---
title: github action 알아보기
created: 2024-03-11 11:47
last-updated: 2024-03-11 11:47
tags:
  - github
  - ci/cd
  - aws
description: repo 푸쉬, 커밋시 생기는 이벤트를 github action으로 관리하자
---

## 👯‍♂️ Intro & tl;dr

- 아래의 내용들을 구현하고 싶어서 githubaction 공부 
	-  develop 브랜치에 푸쉬가 일어나고 -> 내가 작성한 테스트 케이스가 모두 성공한다면 ->  ecr에 이미지 올리기
	- 필요한것
		- github workflow 파일
		- github worflow가 내 ecr에 접근 할 수 있도록 IAM 계정 생성
		- 이  IAM 계정 github workflow 변수에 넣기
		- ECR 생성

```ad-note
- 새로 알게 된 거
	- public registry 아무나 push안돰
	- githubaction내에서 aws 로그인하는 방법;  github OIDC
	- workflow에서 스테이지별 환경변수; github enviroments
```

--- 


## 👯‍♂️ 깃허브 워크 플로우 용어 알아보자

### Event
- 워크 플로우 트리거 시키는 repo 내에서 발생하는 동작(예; 메인 브랜치에 푸쉬하는거, 혹은 issue 실행, PR만들기등)

### Jobs
- 워크 플로우 내에 정의 된 step 묶음 ~~안습 설명을 잘 못하겠음~~
- 각 job은 같은 `runner`에 의해 실행됨
- 각 스텝은 사용자가 정의한 순서대로 진행됨 그리고 같은 `runner`에 의해 실행되므로 데이터를 서로 공유함 
- 
### Actions
- github action 플랫폼에서 작동하는 커스텀 앱으로 반복되는 작업을 실행시키게 해줌
- 

### Runner
- 서버
- 이벤트가 발생하면 워크 플로우를 실행시키는 서버
- 각 러너는 single job 한 번에 하나의 job만 실행 가능함
- 깃허브는 워크 플로우 실행시켜주기 위해 ubuntu, ms, mac os 등 제공
- 각 워크 플로우는 새로 프로비전된 VM에서 실행가능


### 워크 플로우 예제파일 구경하기

각 워크 플로우 파일은 yaml파일로 각자 코드 리포에 `.github/workflows` 이렇게 들어가 있으면 된다 

```yaml
name: learn-github-actions
run-name: ${{ github.actor }} is learning GitHub Actions
on: [push]
jobs:
  check-bats-version:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '14'
      - run: npm install -g bats
      - run: bats -v
```

- event  `push`  : 푸쉬 할 때마다 발생함. 예제에서는 브랜치 명시 되어있지 않으므로 그냥  리포지토리에 푸쉬 될 때마다 발생함 
- job `check-bats-version` : 이 job은 여러개의 step들로 구분 되어있음 
	- `use` 이 키워드는 `actions/checkout/` v4 이용한다고 명시하는거임.  내내 리포에 체크 아웃해 기타 작업(빌드, 테스트등) 실행 할 수 있도록 하는 것! <mark style="background: #FFF3A3A6;">워크 플로우에서 내 리포에 있는 코드 사용 할 때마다 이 체크아웃 작업은 필수다</mark>


## 👯‍♂️ 궁금했던 것들

### 워크 플로우AWS 이미지 푸쉬 할 때 AWS 계정 정보는 어떻게 저장하나? -> `Github Env`

```ad-note
prod, dev, stag등 각 단계에서 배포 할 수 있는 환경변수도 설정가능하다
```


- 스테이징 별로(dev 버전의 환경변수, prod 버전의 환경변수등) 참조 할 수 있는 env(환경변수&배포 룰 등을) 미리 정의 할 수 있다
- `deployment protection` rules 은  이러한 env를 참조하기전에 특정 조건 통과해야하는 룰들인데 PR룰들이랑 비슷하다. (예를들면 main머지하기전에 몇명의 리뷰어에게 통과받는다든가..) 
	- `required reviewers` 누구에게 이 배포 리뷰를 받을건지? 이들에게 승인된 후에 job이 실행되게 할 수 있음
	- `wait timer` - 타이머를 이용해 job이 트리거가 되도 일정시간동안 job이 지연되도록 설정할 수 있음
	- `deployment branched and tags` - 특정 브랜치나 태그만 특정 환경에 배포 할 수 있도록 제한 하거나 아니면 이러한 제한 아에 없애거나 
		- 만약에 `release/*` 로 배포 브랜치를 설정했다면  `release`로 시작한느 브랜치만 해당 환경에 배포 할 수 있음
- 관리자 계정은 이러한  protection rules를 건너뛰고 곧바로 배포 할 수 있도록 설정 할 수 있다
- 설정 방법  환경변수 설정하고 싶은 repo > settings > enviroments 생성! 그리고 yaml에서는 `jobs.enviroment`에 내가 방금 만든 환경 이름을 명시해준다

| ~~실수로 환경이름을 dd로 만들었다~~               |     |
| ------------------------------------ | --- |
| ![](https://i.imgur.com/IiA39aj.png) |     |
```yml
name: Build and Push to ECR

on:
  push:
    branches:
      - develop  # Set your default branch here

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    environment: dd # evn

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.10'  # Set your Python version here

    - name: Install Poetry
      run: |
        pip install --upgrade pip
        pip install poetry

    - name: Install dependencies using Poetry
      run: |
        poetry config virtualenvs.create false
        poetry install --no-interaction --no-ansi

    - name: Run unit tests
      run: |
        cd api
        poetry run python -m unittest test_app.py

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
```




### 워크 플로우에서 AWS계정 어떻게 로그인하나?

AWS credential 설정 -> `aws-actions/configutre-aws-crendential` 

```ad-important
- 앗! AWS ECR 퍼블릭 레지스트리로 생성했다고 해서 아무나 이미지를 Push 할 수 있는게 아니였다. 
- 누구나 Pull 할 수 있지만  Push는 권한이 필요하다! 
- 그렇다는건 내가 정의한 깃허브 워크 플로우에서 권한있는 계정으로 로그인하는게 필요함
```


그래서 ECR 푸쉬하기전에 aws credential 설정하는게 필요하다[^1] 이 필요하다

```yaml
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4 # More information on this action can be found below in the 'AWS Credentials' section
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}

		  aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
		  aws-region:  ${{ secrets.AWS_REGION }}
          
```
이때 나는 `aws-access-key-id`를 사용했지만 aws에서는 [Github's OIDC provider](https://github.com/aws-actions/configure-aws-credentials?tab=readme-ov-file#using-this-action)[^2]를 사용하는 것을 추천했다



### 워크 플로우에서 ECR Push할 때 필요한 최소 권한은 뭘까?

- ECR public에 로그인 할 때 필요한 권한[^3]
```yml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "GetAuthorizationToken",
      "Effect": "Allow",
      "Action": [
        "ecr-public:GetAuthorizationToken",
        "sts:GetServiceBearerToken"
      ],
      "Resource": "*"
    }
  ]
}
```
- ECR public에 push할 때 필요한 권한
```yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowPush",
      "Effect": "Allow",
      "Action": [
        "ecr-public:BatchCheckLayerAvailability",
        "ecr-public:CompleteLayerUpload",
        "ecr-public:InitiateLayerUpload",
        "ecr-public:PutImage",
        "ecr-public:UploadLayerPart"
      ],
      "Resource": "arn:aws:ecr-public::123456789012:repository/my-ecr-public-repo"
    }
  ]
}

```

### ECR Registry와 Repository 차이?

 - 개념상으로 ECR Repository **⊂** ECR Registry
 - Public registry URI 생김새 `public.ecr.aws/레지스트리명/리포지토리명:이미지태그`
 - Registry
	 - repositories 집합
	 - AWS 내에서 계정당 생기는 이미지 저장소 
 - Repository
	 - Registry의 일부이다. 각 Repository는 여러개의 도커 이미지 버전 가질 수 있다(=하나의 리포지토리에 백엔드 이미지, 프론트이미지 넣을 수 있음)

## 👯‍♂️ 의사결정

- ECR private VS public -> `public` 리포 사용
	- private으로 하면 접근 제한 할 수 있어 인증된 유저만 사용가능하다. 단  private ECR 리포 밖으로 데이터 전송시에 비용이 부가된다. 다만 AWS 같은 리전이나 private ECR/ECS/EKS간 데이터 전송은 무료
	- 내 어플리케이션에는 중요한 정보가 없고 가능한 저렴한 가격에 서버 운영하고 싶기에 public으로 결정




## 👯‍♂️ Ref & LINKS TO THIS PAGE

- [웹, github docs, githubaction](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)
- [웹, githubaction, using env for deployments](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [^1]: [웹,github action,  aws ecr login](https://github.com/aws-actions/amazon-ecr-login?tab=readme-ov-file#building-and-pushing-an-image)
- [^2]: [웹, github action, aws credential recommendation](https://github.com/aws-actions/configure-aws-credentials?tab=readme-ov-file#using-this-action)
- [^3]:[웹, github action, ecr-login and ecr push permission ](https://github.com/aws-actions/amazon-ecr-login?tab=readme-ov-file#ecr-public)



```

```


