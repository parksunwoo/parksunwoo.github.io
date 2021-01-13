---
title: "CI/CD 끝장내기"
categories:
  - Dev
tags:
  - CI/CD
  - DevOps
  - Azure 
last_modified_at: 2021-01-12T08:06:00-00:00
---
어느정도 개발이 진행되고 나면 이제는 개발서버에서 벗어나 실제 운영서버를 갖추어야한다. 

보통은 운영서버와 운영서버와 환경이 같은 스테이지 서버를 준비하게 된다.

코드변경에 따라 build / deploy 를 해야하는데 이 작업을 수작업으로 하게 되면 변경할때마다 반복해서 여러 작업을 해야한다.

---
<br>

처음 개발을 진행할때 클라우드 환경은 가성비가 좋은 DigitalOcean을, 코드저장소는 GitLab을 사용했고 코드변경이 있을때마다 서버에 반영하는 작업은 아래와 같았다.

1. DigitalOcean Droplets(VM)에 ssh 로 연결 <br>
```shell
$ ssh username@203.0.113.0
```

2. 서버에 붙어서 실행중인 프로그램을 종료

3. 코드가 저장되어있는 dev 폴더로 이동해 개발 branch pull 하기
```shell
$ git pull origin master_dev
```

4. 코드를 최신화했으니 이제 다시 서버를 재시작
```shell
$ gunicorn --env DJANGO_SETTINGS_MODULE=myproject.settings myproject.wsgi
```


1-4번까지 정상적으로 수행된다면 다행이지만 코드를 pull 하면서 에러가 발생하면 잘못된 부분을 직접 찾아 수정해야하는 경우도 잦았다.

개발초기에 소수의 사용자이지만 (10명 남짓) 실제 사용자가 사용하는 서버를 운영하게 되었다. 서버를 중단할때 혹시라도 발생할 사고를 대비해 30분~ 1시간정도 사용이 어렵다고 사전 고지를 하고 서버작업을 시작했다.

개발서버 (1-4번 작업) + 운영서버 (1-4번 작업) + 코드 충돌날때의 긴장감 까지 더하면 

그리고 향후 추가될 스테이지서버까지 고려한다면 수작업으로 진행하는 것은 더이상 어렵다는 판단을 내렸다.

<br><br><br>

![포기하고 싶었습니다](https://www.dropbox.com/s/z2dlitn56t8ipvj/Screen Shot 2021-01-12 at 8.08.08 AM.png?raw=1) <br>
포기하고 싶었습니다..

<br><br><br>

수작업에서 벗어나기위해 CI/CD에 대해 찾아보기 시작했다.



# CI/CD는 무엇인가?

> CI/CD는 새로운 코드 통합으로 인해 개발 및 운영팀에 발생하는 문제(일명 "인테그레이션 헬(integration hell)")을 해결하기 위한 솔루션입니다.


인테그레이션 헬 용어를 한글로 번역하면 통합지옥.

여러 개발자가 동시에 개발을 진행하면 개발이 완료된 후에 코드들을 한데 모아야 한다. 아주 다행히도 A개발자와 B개발자가 다른 소스를 수정했다면 문제가 없겠지만 A개발자와 B개발자가 같은 소스를  수정하는 순간 매끄럽게 수정하는데 많은 시간이 걸린다.

CI 는 개발자를 위한 자동화 프로세스로 지속적인 통합 (Continuous Integration) 을 의미한다 
새로운 코드 변경사항이 정기적으로 빌드, 테스트되서 공유 리포지토리에 통합되는 과정이다. 이렇게 되면 공유 리포지토리는 정상적인 최신버전임을 보장하게 된다. 여러명의 개발자가 개발할 경우에도 코드가 충돌하는 문제를 해결할 수 있다.

CD 는 지속적인 서비스 제공(Continuous Delivery) 및 지속적인 배포(Continuous Deployment)를 의미한다.
지속적인 제공은 개발자들이 애플리케이션에 적용한 변경 사항이 버그 테스트를 거쳐 리포지토리에 자동으로 업로드 되는 것을 뜻한다. 
지속적인 배포란 개발자의 변경 사항을 리포지토리에서 고객이 사용가능한 프로덕션 환경까지 자동으로 릴리스하는 것을 의미한다.

<br>

![자동화! 자동화를 하면 일련의 반복되는 작업들을 하지 않아도 된다.](https://s3-us-west-2.amazonaws.com/public.notion-static.com/98d6f347-ca2d-47b7-bf48-47511630182f/maarten-van-den-heuvel-400626-unsplash.jpg)
자동화! 자동화를 하면 일련의 반복되는 작업들을 하지 않아도 된다.

위에서 코드변경시 서버에 반영한 수작업에 빗대어 보면 서버에 있는 코드를 변경된 코드로 수정하는 과정이 CI에, 최신버전 코드를 개발/스테이지/운영 서버에 배포하는 과정이 CD에 해당한다.

<br>

자동화와 함께 운영 / 스테이지 서버 운영환경 고민했을때 가성비 보다는 장애없이 운영되는 안정성에 더 높은 비중을 두게되어 고민하게 되었다. 

Django와 React로 서비스를 운영하기 위해 정말 많은 자료들을 검색해보다 [리액트와 함께 장고 시작하기](https://www.youtube.com/watch?v=617PFR-A30s) 강의를 많이 참고하게되었다.

강의에서 Docker기반으로 Azure에서 진행하기도 했고 Azure의 DevOps 서비스, 회사에서 Microsoft 계정을 사용하고 있는 부분도 클라우드 플랫폼 결정에 한몫을 했다. 

<br>

![Azure DevOps](https://www.dropbox.com/s/tq4jdvekz5ztdsn/Screen Shot 2021-01-10 at 5.45.18 PM.png?raw=1)
[출처 : https://azure.microsoft.com/ko-kr/services/devops/]
<br><br><br>

클라우드 환경을 Digital Ocean 에서 Azure 로, 코드저장소는 Azure DevOps와 호환되는 GitHub 로 이동을 결정하고 진행하게되었다. GitHub 에서도 private 기능이 무료화 되면서 저장소를 private 하게 지정했다. GitHub 로 소스코드를 이동하면서 원래 프론트와 백엔드가 같은 곳에 모여있던걸 각기 다른 저장소로 분리하는 작업을 동시에 진행했다.

아래 그림은 Azure 로 이사한 후 완성된 CI/CD 전체흐름도이다.
프론트엔드와 백엔드는 1~3번이 서로 구분되나 같은 흐름이라 편의상 같에 표시했고 4번부터 진행되는 과정을 구분되게 표시했다.

![Azure DevOps CI/CD](https://www.dropbox.com/s/ly6ivz76h3pbwcv/Screen%20Shot%202021-01-12%20at%208.22.57%20AM.png?raw=1)
<br><br>
1. 개발자가 코드 변경사항이 있어 IDE에서 코드 수정후 commit / push 를 한다
2. push한 브랜치 기준으로 깃허브에 코드가 반영된다
3. 깃허브에 변경사항이 있는걸 감지한 Azure DevOps 가 동작한다
4. 백엔드 코드가 변경되었다면 Dockerfile을 기준으로 image를 Build 하고 Push 한다 
새롭게 생성된 image는 별도 Container registry 에 저장된다 <br>
 프론트엔드 코드가 변경되었다면 npm install / npm run build 를 한 후 빌드된 내용이 zip 파일 형태로 Archive 되어 Storage에 파일이 올라간다.
5. Azure Web App 에 Container를 Deploy 하면 백엔드 변경작업이 완료되고 DB에서 데이터를 조회한다 <br>
 Storage 는 정적사이트로 앞단은 Endpoint로 연결되어있고 이 Endpoint 가 사용자가 접속하는 custom domain에 연결되어있다.
 
 
 사용자가 공개된 도메인으로 사이트에 접속하면 이것은 Endpoint에 접속하는 것이고
사이트에서 클릭을 하거나 특정 이벤트가 있을때마다 Rest Api 를 통해 백엔드와 통신한 결과값을 화면에 출력하게 된다.

진행한 프로젝트가 Django 와 React를 사용하다보니 저장소도 분리해서 가져가고 파이프라인 구분되어 만들어졌다. 일반 프로젝트라면 구조가 더 단순할수도 있을 것 같다. 

<br>
<br>


CI/CD를 도입하고 나서 코드를 push하고 난후 10분 이내로 개별서버에 반영된다.

코드변경작업 좀더 안정적으로 진행하게 되었고 무엇보다 반복되던 서버작업에서 해방되었다.

물론 아직도 개선할 부분은 있다.

CI/CD 하는데 걸리는 전체 시간을 줄여야하고
(개발서버는 9분, 스테이지 서버는 18분, 운영서버는 18분 걸린다)

스테이지와 운영서버에서 시간이 배로 걸리는 것은 스테이지와 운영서버에 이중화를 구성을 해두었는데
서로 다른 App Service가 각각의 파이프라인을 가지고 있다. 이 부분은 불필요한 중복이라 2개의 App Service가 서로 같은 파이프라인을 바라보게 변경해야할 것 같다.

<br><br>
CI/CD 를 처음 구성하는데 어려움을 겪을 분들에게 조금이나마 도움이 되었으면 하고
도커로 환경을 구성한 부분은 다른 포스트를 통해 이야기 해보려고 한다.
CI/CD 개념 관련해선 아래 페이지를 참고했다.

<br>
**RedHat CI/CD(지속적 통합/지속적 제공): 개념, 방법, 장점, 구현 과정** : [https://www.redhat.com/ko/topics/devops/what-is-ci-cd](https://www.redhat.com/ko/topics/devops/what-is-ci-cd)


