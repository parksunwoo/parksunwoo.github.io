---
title: Travis CI로 AWS ElasticBeanstalk 배포하기
categories:
  - Dev
tags:
  - cicd
  - elasticbeanstalk
  - aws
  - https
last_modified_at: 2021-04-30T22:52:00-00:00
---

소개:
이번 글에서는 지속적인 통합과 배포를 위해 Travis CI를 사용하여 AWS Elastic Beanstalk에 애플리케이션을 배포하는 방법을 다룹니다. 
이 튜토리얼은 개발자를 대상으로 하며 이해하기 쉬운 설명, 중급 코드 예제, 단계별 프로세스 분석이 포함되어 있습니다.

---

목차

1) Travis CI에 연결하기
2) .travis.yml 파일 작성하기
3) Dockerrun.aws.json 파일 생성하기
4) Docker 컴포즈 파일 변환 및 생성하기
5) PostgreSQL 설정
6) Elastic Beanstalk 구성하기
7) HTTPS 설정


1. Travis CI는 애플리케이션을 빌드, 테스트 및 배포하는 프로세스를 자동화하는 지속적 통합 서비스입니다. GitHub 리포지토리를 Travis CI에 연결하려면 다음 단계를 따르세요:
- 아직 가입하지 않은 경우 [Travis CI](https://www.travis-ci.com/) 계정에 가입하고 로그인합니다.
- 계정 설정으로 이동하여 GitHub 계정을 연결합니다.
- 리포지토리 이름 옆의 스위치를 토글하여 애플리케이션이 포함된 GitHub 리포지토리를 활성화합니다.


2. .travis.yml 파일 작성
프로젝트의 루트 디렉토리에 **`.travis.yml`** 파일을 생성합니다. 이 파일은 빌드 프로세스 중에 Travis CI에 수행할 작업을 알려줍니다. 다음은 예제입니다:

```yaml
language: node_js
node_js:
  - "14"
services:
  - docker
before_install:
  - docker build -t my-app .
script:
  - docker run my-app npm test
deploy:
  provider: elasticbeanstalk
  region: "us-west-2"
  app: "my-app"
  env: "MyApp-env"
  bucket_name: "elasticbeanstalk-us-west-2-1234567890"
  bucket_path: "my-app"
  on:
    branch: main
  access_key_id: $AWS_ACCESS_KEY
  secret_access_key: $AWS_SECRET_KEY
```

예제의 값을 자신의 프로젝트 세부 정보로 바꿉니다.

3. Dockerrun.aws.json 파일 생성하기

프로젝트의 루트 디렉터리에서 **`Dockerrun.aws.json`** 파일을 생성합니다. 이 파일은 Elastic Beanstalk에 Docker 컨테이너를 배포하고 실행하는 방법을 알려줍니다. 다음은 예제입니다:

```json
{
  "AWSEBDockerrunVersion": "1",
  "Image": {
    "Name": "my-image:latest",
    "Update": "true"
  },
  "Ports": [
    {
      "ContainerPort": "80"
    }
  ]
}
```

4. Docker 컴포즈 파일 변환 및 생성하기

여러 서비스(예: 웹 서버, 데이터베이스)와 함께 애플리케이션을 실행하려면 **`docker-compose.yml`** 파일을 사용할 수 있습니다. 
기존 **`Dockerfile``을 **`docker-compose.yml`** 파일로 변환하거나 처음부터 새로 생성하세요. 다음은 예제입니다:

```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "80:80"
  db:
    image: "postgres"
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
```

5. PostgreSQL 설정

**`docker-compose.yml`** 파일에서 `db` 서비스가 Docker Hub의 PostgreSQL 이미지를 사용하도록 구성합니다. 데이터베이스 사용자 및 비밀번호 등 필요한 환경 변수를 입력합니다.

6. Elastic Beanstalk 구성

애플리케이션을 위한 Elastic Beanstalk 환경을 생성합니다:

- AWS 관리 콘솔에 로그인합니다.
- Elastic Beanstalk 서비스로 이동합니다.
- "새 애플리케이션 생성"을 클릭하고 지시를 따릅니다.
- "환경 계층" 섹션에서 "웹 서버 환경"을 선택합니다.
- 플랫폼으로 "Docker"를 선택하고 적절한 버전을 선택합니다.
- 소스 코드 번들을 업로드하며, 여기에는 **`Dockerrun.aws.json`** 파일이 포함되어야 합니다.
- "환경 생성"을 클릭하여 프로비저닝 프로세스를 시작합니다.

환경이 생성되면 Elastic Beanstalk 콘솔에서 애플리케이션 URL을 확인할 수 있습니다.

[Elastic Beanstalk Docker 환경 구성](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/create_deploy_docker.container.console.html)

7. HTTPS 설정

HTTPS로 애플리케이션을 보호하려면 SSL 인증서를 획득하고 이를 사용하도록 Elastic Beanstalk 환경을 구성해야 합니다.

- 도메인에 대한 SSL 인증서를 받습니다. AWS 인증서 관리자(ACM) 또는 다른 인증 기관(CA)을 사용하여 인증서를 받을 수 있습니다.
- AWS 관리 콘솔에서 Elastic Beanstalk 서비스로 이동하여 애플리케이션의 환경을 선택합니다.
- 왼쪽 사이드바에서 "구성"을 클릭합니다.
- "로드 밸런서" 섹션에서 "편집"을 클릭합니다.
- "수신기" 설정에서 프로토콜로 "HTTPS"를 선택하고 이전에 획득한 SSL 인증서를 선택하여 새 HTTPS 수신기를 추가합니다.
- 변경 사항을 저장합니다.

이제 애플리케이션에 HTTPS로 액세스할 수 있습니다. 애플리케이션의 코드를 업데이트하여 HTTP에서 HTTPS로의 리디렉션을 처리하고 모든 내부 링크와 자산이 HTTPS를 사용하도록 해야 할 수도 있습니다.


결론:

이 블로그 포스팅에서는 Travis CI를 사용하여 AWS Elastic Beanstalk 애플리케이션을 배포하는 방법을 보여드렸습니다. 
Travis CI에 연결하기, **`.travis.yml`** 파일 작성, **`Dockerrun.aws.json`** 파일 생성, **`docker-compose.yml`** 파일 변환 및 생성, PostgreSQL 설정, Elastic Beanstalk 구성 및 HTTPS 설정에 대해 다루었습니다. 
이러한 단계를 따르면 애플리케이션 배포 프로세스를 간소화하고 애플리케이션이 항상 최신 변경 사항으로 최신 상태로 유지되도록 할 수 있습니다.
