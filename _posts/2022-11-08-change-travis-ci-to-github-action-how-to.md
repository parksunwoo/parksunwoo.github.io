---
title: Travis CI 에서 Github Action 으로 AWS Elastic Beanstalk 배포하기
categories:
  - Dev
tags:
  - howto
  - ci/cd
  - travis
  - github action
  - elastic beanstalk
last_modified_at: 2022-11-08T23:00:00-00:00
---

# 들어가며

사내 백엔드 시스템에서 사용중이던 Travis CI 를 Github Action 으로 교체하였습니다

Travis CI에서 어떻게 사용하고 있었는지 간단히 설명하고 Github Action 으로 변경을 진행하면서 

어떤 부분에서 이슈가 있었는지 정리해 CI/CD를 이해하는데 조금이라도 도움이 되고자합니다. 

# Travis CI

![TravisCI](https://www.travis-ci.com/wp-content/uploads/2021/12/TravisCI-Full-Color.png)

대표적인 CI툴에는 Jenkins, CircleCI, TeamCity, Gitlab 등이 알려져 있고 Travis CI도 진입장벽이 상대적으로 낮아 많이 사용된다

아래는 Travis CI의 주요특징 ([https://katalon.com/resources-center/blog/ci-cd-tools](https://katalon.com/resources-center/blog/ci-cd-tools) )

**Travis CI key features:**

- Quick setup
- Live build views for GitHub projects monitoring
- Pull request support
- Deployment to multiple cloud services
- Pre-installed database services
- Auto deployments on passing builds
- Clean VMs for every build
- Supports macOS, Linux, and iOS
- Supports multiple languages, such as Android, C, C#, C++, Java, JavaScript (with Node.js), Perl, PHP, Python, R, Ruby, and so on.

Travis CI를 사용하기 위해서는 프로젝트 상단에 .travis.yml 파일이 있어야함

프로젝트 구조

```yaml
1└── test_proj
2    ├── proj
3    │   ├── __init__.py
4    │   ├── asgi.py
5    │   ├── settings.py
6    │   ├── urls.py
7    │   └── wsgi.py
8    ├── manage.py
9    └── requirements.txt
10   └── .travis.yml
```

.travis.yml 은 크게 아래의 단계로 구성됨

1. `env`

이후 단계들에서 공통으로 사용할 환경변수 설정

2. `before_install`

3번 스크립트 수행 전 실행작업 (ex. 도커 로그인, 환경변수 확인)

3. `script`

배포를 수행하는데 필요한 스크립트 실행 (ex. 실행중인 도커 컨테이너 확인)

4. `after_success`

3번 스크립트 수행후 실행작업 (ex. 도커파일 빌드 및 빌드된 도커이미지를 도커허브에 푸쉬, 배포전 변경 필요한 파일 수정)

5. `deploy`

배포환경관련 정보 설정



.travis.yml 예시 파일 전체는 아래와 같습니다.

```yaml
sudo: required

language: generic

services:
  - docker
env:
  global:
    - VERSION="$ENV_TARGET-${TRAVIS_COMMIT::7}"

before_install:
  - echo "Testing Docker Hub credentials"
  - echo "$DOCKER_HUB_PASSWORD" | docker login -u "$DOCKER_HUB_ID" --password-stdin
  - echo "Docker Hub credentials are working"
  - echo "VERSION=$VERSION"

script:
  - docker ps -a

after_success:
  - docker build -t test/backend:$VERSION -f Dockerfile."$ENV_TARGET" .
  - docker build -t test/nginx:$VERSION ./nginx

  - echo "$DOCKER_HUB_PASSWORD" | docker login -u "$DOCKER_HUB_ID" --password-stdin

  - docker push test/backend:$VERSION
  - docker push test/nginx:$VERSION

  - sed -i='' "s/<TAG>/$VERSION/g;s/<ENV>/$ENV_TARGET/g" Dockerrun.aws.json
  - git add Dockerrun.aws.json
  - git commit -m "Replace the <TAG> with the real version"

deploy:
  - provider: elasticbeanstalk
    access_key_id: $AWS_ACCESS_KEY
    secret_access_key: $AWS_SECRET_ACCESS_KEY
    region: "ap-northeast-2"
    app: "docker-app"
    env: "Dockerapp-dev"
    bucket_name: elasticbeanstalk-ap-northeast-2-xxxxxxxxxxxxx
    bucket_path: "docker-app-dev"
    on:
      branch: master
```

env 단계에서 global 변수로 선언한 VERSION, TRAVIS_COMMIT(travis 고유 환경변수) 를 제외한

$DOCKER_HUB_PASSWORD, $DOCKER_HUB_ID, 
$ENV_TARGET, 
$AWS_ACCESS_KEY, $AWS_SECRET_ACCESS_KEY

해당 환경변수들은
Travis CI 사이트에 저장되어있는 환경정보를 가져다 사용합니다

<figure>
  <img src="/assets/images/travis-ci-1.png" alt="Trulli" style="width:650, height:516">
  <img src="/assets/images/travis-ci-2.png" alt="Trulli" style="width:650, height:516">
  <figcaption>Travis CI 사이트 > Settings > Settings 화면</figcaption>
</figure>


브랜치별로도 환경변수 값을 다르게 설정할수 있음 ex. $ENV_TARGET 변수.

# Github Action

Github Action은 다양한 아이디어를 자동화해서 사용해볼수있는게 장점.
아래는 Github Action 사이트 접속시 소개되는 문구

[https://github.com/features/actions](https://github.com/features/actions)

**Automate your workflowfrom idea to production**

GitHub Actions makes it easy to automate all your software workflows, now with world-class CI/CD. Build, test, and deploy your code right from GitHub. Make code reviews, branch management, and issue triaging work the way you want.

위에서 설명한 Travis CI의 각 단계를 Github Action 으로 그대로 옮기는 작업을 진행했습니다.

CI를 변경하는 작업은 자주 있는 작업이 아니다보니 몇가지 부분에서 이슈(~~삽질~~)가 있었습니다.

Github Action 은 Travis CI와 달리 무료로 사용가능하고 (무료 최대 2000분) 
Github 에 있는 다양한 Repo를 블럭처럼 재조합해서 사용할수 있어서 굉장히 강력한 도구같습니다.

CI/CD 뿐 아니라 크롤링작업, 운영 배포시 `semantic-release` (git message를 활용한 버전관리) 에도 활용할수있는 등 사용할수 있는 부분이 많습니다.

Github Action을 사용하기 위해서는 프로젝트 상단 .github/workflows 에 yml 파일이 있어야 합니다.

아래 예시에서는 `deploy-dev.yml` 이 해당 파일입니다

프로젝트 구조

```yaml
1└── test_proj
2    ├── proj
3    │   ├── __init__.py
4    │   ├── asgi.py
5    │   ├── settings.py
6    │   ├── urls.py
7    │   └── wsgi.py
8    ├── manage.py
9    └── requirements.txt
10   └── .github
11       └── workflows
12          └── deploy-dev.yml
```

.deploy-dev.yml 의 주요단계를 뜯어보겠습니다.

name: Dev deployment from Github to AWS EB
#yml 파일의 이름지정, Github Action 탭에서 확인될 Task명

```yaml
on:
  push:
    branches:
      - master_dev
  pull_request:
    branches:
      - master_dev
```

1. `on`

push, `pull_request` 와 같이 Github Action을 수행할 특정 이벤트 종류와, 특정 브랜치를 별도로 지정할 수 있습니다.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
```

1. `jobs`

아래부분에 수행할 작업들을 순차적으로 명시함, jobs > steps 하위에 수행할 작업을 상세하게 기술함

`runs-on: ubuntu-latest` 
action을 수행할 환경을 지정함, 여기서는 ubuntu 최신버전을 사용

```yaml
- name: Checkout Latest Repo
  uses: actions/checkout@master
```

1. `Checkout Latest Repo`

최신버전 repo checkout

```yaml
- name: Setup env variables
  shell: bash
  run: |
    if [[ ${{github.ref_name}} == 'master_dev' ]]; then
        echo "env_target=dev" >> $GITHUB_ENV
        echo "version=dev-${{ github.run_id }}" >> $GITHUB_ENV
    elif [[ ${{github.ref_name}} == 'master' ]]; then
        echo "env_target=prod" >> $GITHUB_ENV
        echo "version=prod-${{ github.run_id }}" >> $GITHUB_ENV
    else
        echo ""
    fi
  id: set_env
```

1. `Setup env variables`

이후 스텝에서 사용할 환경변수 설정, 여기서는 브랜치명을 조회해서 브랜치 별로 환경변수를 별도 설정함

env_target, version 을 환경별로 지정하고 $GITHUB_ENV 에 값을 저장함

```yaml
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v2
```

1. `Set up Docker Buildx`

도커 빌드를 위한 환경구성

```yaml
- name: Login to DockerHub
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}

```

1. `Login to DockerHub`

도커 로그인, `username` 은 이메일주소가 아닌 username 만 가능함! (이메일주소로 로그인을 시도했다가 반복해서 로그인실패 발생했습니다.. 주의하세요)

```yaml
- name: Build and push backend
  uses: docker/build-push-action@v3
  with:
    context: .
    file: Dockerfile.${{ env.env_target }}
    push: true
    tags: test/backend:${{ env.version }}
```

1. `Build and push backend`

해당하는 도커파일 빌드위치를 지정한 후 도커파일 빌드 및 푸쉬 진행, 이미지 태깅작업 수행

```yaml
- name: Mod Dockerrun.aws.json
  shell: bash
  run: |
    sed -i -e "s/<TAG>/${{ env.version }}/" Dockerrun.aws.json
    sed -i -e "s/<ENV>/${{ env.env_target }}/" Dockerrun.aws.json
    git config --global user.email "test@gmail.com"
    git config --global user.name "test"
    git add Dockerrun.aws.json
    git commit -m "Replace the <TAG> with the real version"
  id: mod_dockerun_aws_json
```

1. `Mod Dockerrun.aws.json`

AWS Beanstalk 환경 배포를 위한 Dockerrun.aws.json 파일 변경작업 수행후 git commit 실행
git 사용을 위한 git config 설정포함

```yaml
- name: Get Timestamp
  uses: gerred/actions/current-time@master
  id: current-time

- name: Run String Replace
  uses: frabert/replace-string-action@master
  id: format-time
  with:
    pattern: '[:\.]+'
    string: "${{ steps.current-time.outputs.time }}"
    replace-with: '-'
    flags: 'g'

- name: Generate Deployment Package
        run: zip -r deploy.zip *
```

1. `Generate Deployment Package`

AWS Elasticbeanstalk 배포를 위한 deploy.zip 파일 생성

```yaml
- name: Deploy to EB
  uses: einaregilsson/beanstalk-deploy@v20
  with:
    aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    application_name: docker-app-dev
    environment_name: Dockerapp-dev-1
    version_label: "github-action-${{ github.run_id }}-${{ steps.format-time.outputs.replaced }}"
    region: ap-northeast-2
    deployment_package: deploy.zip
    existing_bucket_name: elasticbeanstalk-ap-northeast-2-xxxxxxxxxxx
    wait_for_environment_recovery: 180
```

1. `Deploy to EB`

AWS Elasticbeanstalk 배포환경관련 정보 설정

name 아래 uses 부분은 깃허브 상 repo와 같고 
([Github Aciton - beanstalk-deploy](https://github.com/einaregilsson/beanstalk-deploy) )



해당 repo에 접근하면 repo관련 정보를 확인할수 있습니다
해당 repo에서만 사용하는 변수정보를 확인할 수있음, 위 예제에서는 with 아래 변수들이 repo 변수에 해당
@v2, @v20 은 repo의 버전정보를 나타냄

`github.run_id` 처럼 github action 에서만 사용할수있는 환경변수가 있음

`steps.current-time.outputs.time` 은 
current-time id를 갖는 step의 출력값으로 time 을 사용하겠다는 의미임



<figure>
  <img src="/assets/images/github-action-example-1.png" alt="Trulli" style="width:650, height:516">
  <figcaption>gerred/actions/current-time@master Github Actino page</figcaption>
</figure>



<figure>
  <img src="/assets/images/github-action-list.png" alt="Trulli" style="width:650, height:516">
  <figcaption>Github Repo 상 Acitons 탭에서 Github Action 리스트 조회</figcaption>
</figure>



<figure>
  <img src="/assets/images/github-action-list-detail.png" alt="Trulli" style="width:650, height:516">
  <figcaption>Github Action 리스트 상세 조회</figcaption>
</figure>



<figure>
  <img src="/assets/images/github-action-detail.png" alt="Trulli" style="width:650, height:516">
  <figcaption>스텝별 진행상황 확인, 수행시간도 확인가능</figcaption>
</figure>


# ps. 더 해보기

Travis CI는 유료툴이라 매달 일정금액이 나가고 있었는데 무료인 Github Action으로 변경하면서 비용절감이 되었습니다. 
물론 Github Action은 무료라 배포에 사용할수있는 무료시간을 2000 분/월 까지 제공합니다.

현재 회사에서는 Team Plan을 사용중이라 3000 분/월 까지 무료로 사용할수 있습니다.

무료시간을 충분히 사용하고 배포속도를 향상시키기 위해서도 배포과정 최적화가 필요합니다.

Github Action 은 앞서 이야기한 것처럼 step마다 repo 들을 블럭처럼 조합해서 사용하는 것이라 

사용하고 있는 업무툴과 연동한다던지 (ex. slack)

배포뿐 아니라 크롤링, sementic release 관리등 업무 자동화부분에도 사용할 수 있을 것입니다.

추가로 Travis CI와 Github Action의 속도비교를 위해 브랜치에서 pr merge시 두개 CI가 모두 동작하도록 설정후 테스트 진행.

2개 CI는 거의 비슷한 속도로 작업이 진행됨 (Github Action은 Elastic Beanstalk에서 배포되고 정상화되는 시간이 포함되어 총 시간이 늘어나지만 해당 작업을 제외시)


# pss. Job을 병렬적으로 실행하기

위 예시에선 Travis CI와 Github Action이 별 차이가 없는 것으로 나와있는데

실제 운영환경에 배포할때에 이슈가 하나 있었습니다. 

.travis.yml 에선 deploy 부분에 여러 task를 생성해 브랜치 기준으로 작업들을 병렬처리할 수 있었으나

Github Action으로 변경후에는 모든 step들이 작성된 순서대로 순차적으로 진행되었습니다.

운영환경에서는 특정브랜치를 기준으로 2개의 웹서버에 배포되고 있어서

A서버와 B서버에 동시 배포되어야하는데 A서버에 배포완료된 후에 B서버 배포가 시작되었습니다.

이 문제를 어떻게 해결할 수 있을지 검색한 결과 아래 내용을 찾을 수 있었습니다.

*Github Action에서 작업(job)은 어떤 이벤트가 발생했을 때 독립된 환경에서 실행되어야 하는 일련의 일을 나타내는 매우 중요한 개념. 워크플로우 (workflow)는 작업(job)의 상위 개념이고 단계(step)은 하위 개념이라고 볼수 있음.*

*즉, 하나 이상의 작업(job)이 하나의 워크플로우(workflow)를 구성하며, 하나의 작업(job)은 순차적으로 수행되는 여러 개의 단계(step)로 이뤄짐*

*Github Action에서 작업 설정을 할 때, 하나의 워크플로우 상에서 각각의 작업이 완전히 격리된 환경에서 실행된다는 것. 2개의 작업을 실행하면 각 작업은 서로 CI 서버, 다른 컴퓨터에서 돌아가는 것*

*작업간의 완전한 격리는 병렬 처리가 가능해져 성능 측면에서 유리.*

아래와 같이 구성되어 있던 yaml 파일을 TOBE 형태로 변경하였다.

## ASIS

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
```

## TOBE - step1

```yaml
jobs:
	build:
    runs-on: ubuntu-latest
    steps:

  deploy1:
    runs-on: ubuntu-latest
    steps:

  deploy2:
    runs-on: ubuntu-latest
    steps:
```

그럼 ASIS에서는 jobs > deploy 하위에 작성된 순서대로 steps 들이 실행되지만

TOBE에서는 jobs 하위 build, deploy1, deploy2가 병렬적으로 실행된다.

여기서 고려해야할 게 deploy job 은 build job이 수행완료된 다음에 실행되어야 최신버전의 코드가 반영될수 있다는 것이다.

## TOBE - step2

```yaml
jobs:
	build:
    runs-on: ubuntu-latest
    steps:

  deploy1:
    runs-on: ubuntu-latest
		needs: build
    steps:

  deploy2:
    runs-on: ubuntu-latest
		needs: build
    steps:
```

실행 순서 제어를 위해선 의존관계가 있는 작업에 `needs: build` 를 추가하여 
위 예시에서는 build job이 수행된 이후 deploy1, deploy2 job이 수행되게 처리할 수 있다.

여기까지 수행하고 나소 Github Action 을 통해 CI/CD를 완전히 옮겼다고 생각했는데

막상 Action이 수행되고 나니 에러가 발생했다.

에러인 즉슨 이전에는 동일한 job내에서 step 이 처리되다 보니 
아래에서 생성된 deploy.zip 파일이 이후 step에서 사용가능하였지만

병렬처리를 위해 job을 구분하다보니 A job의 output을  B job에서 사용할수가 없었다.

```yaml
- name: Generate Deployment Package
        run: zip -r deploy.zip *
```

마찬가지로 job과 job 사이에서 output을 공유하는 방법을 찾았다.

## TOBE - step3

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      replaced: ${{ steps.format-time.outputs.replaced }}
    steps:
			- name: Generate Deployment Package
        run: zip -r deploy.zip *
      - name: Upload Deployment Package
        uses: actions/upload-artifact@v3
        with:
          name: deploy
          path: deploy.zip

	deploy:
	    runs-on: ubuntu-latest
	    needs: build
	    steps:
	      - name: Download Deployment Package
	        uses: actions/download-artifact@v3
	        with:
	          name: deploy
	          path: ./
	
	      - name: Deploy to EB
	        uses: einaregilsson/beanstalk-deploy@v20
					with:
	          aws_access_key: ${{ secrets.AWS_ACCESS_KEY_ID }}
	          aws_secret_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
	          application_name: django-app-dev
	          environment_name: djangoapp
	          version_label: "github-action-${{ github.run_id }}-${{ needs.build.outputs.replaced }}"
```

step 단위의 output을 job의 output으로 선언해 다른 job 에서 사용가능하다

위 TOBE - step3 에서는 outputs:replaced: `{{ steps.format-time.outputs.replaced }}`

를 통해 format-time 이라는 id의 step의 replaced output을 build job의 replaced outputs로 보낼수 있다.

이걸 deploy job 에선 `{{ needs.build.outputs.replaced }}` 이와 같이 호출해서 사용할 수 있다.

그리고 한가지더 build job 에서 최종으로 생성된 deploy.zip 파일을 

actions/upload-artifact@v3 Action을 통해 upload 하고

deploy job 에서는 download하여 파일을 공유할 수 있었다.

이렇게 해서 2개의 웹서버에 무사히 배포되는 것을 확인하였고 

Travis CI에서 Github Action으로 CI/CD를 옮기는 작업을 마무리하였다.

> 참고링크
> 

[https://www.daleseo.com/github-actions-jobs/](https://www.daleseo.com/github-actions-jobs/)

[https://stackoverflow.com/questions/59175332/using-output-from-a-previous-job-in-a-new-one-in-a-github-action](https://stackoverflow.com/questions/59175332/using-output-from-a-previous-job-in-a-new-one-in-a-github-action)

