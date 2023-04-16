---
title: 엔지니어에게 직접 배우는 GCP, kubernetes, Firebase 환경세팅(1)
categories:
  - Dev
tags:
  - devops
  - gcp
  - firebase
  - iam
  - cicd
  - tasks
  - dns
  - filestore
last_modified_at: 2023-04-16T09:10:00-00:00
---

쿠버네티스 관련 책을 사두고 1번정도 읽어보기는 했지만

어느날 갑자기 kubernetes(k8s) 기반의 시스템을 인수인계 받는 상황에 부닥치게 되었다

k8s 는 거스를 수 없는 대세기술이고 배워서 알고있으면 좋다는걸 알고있지만 난이도와 학습의 어려움으로 인해

쉽게 다가서기가 어려운 것도 사실이다

k8s 시스템 엔지니어로부터 직접 차근차근 설명받으면서 배웠던 내용을

회사정보를 제외하고 기본적인 내용 중심, GCP GKE 기준으로 설명하고자 한다

---

[실습환경에서 전체 시스템 구조](https://www.dropbox.com/s/7p1s9tam1ne1jj2/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-16%20%EC%98%A4%EC%A0%84%2012.46.27.png?dl=0)

현재 구동중인 백엔드 시스템이 있고 해당 시스템에서 유저의 특정 이벤트 발생시 유저에게 필요한 컨테이너 생성요청이 발생한다.

생성 요청이 발생하면 정해진 API가 call 되면서 Cloud Run에 전달되고

Cloud Run에서는 Firebase Realtime Database에 데이터를 새롭게 등록하거나 기존 데이터를 업데이트한다

또한 비동기로 처리할 큐들을 Cloud Tasks에 등록한다

이와 동시에 cloud run 에서 cpu 컨테이너 생성요청을 보내게 되면 Kubernetes Engine에서는 아래와 같은 자세한 과정을 거쳐서 필요로 하는 컨테이너 자원을 생성한다.

	temp-cpu pod를 생성하여 노드 생성 요청을 보내게 되고 
	해당 요청으로 nodepoolname(라벨)에 해당하는 스펙의 노드를 생성하게 되고
	해당 노드가 생성되면 prepull-cpu pod가 생성되어서 이미지 풀을 받게 되고
	이미지 풀을 받게 되면 노드에 image.pull.ready=True 라는 라벨을 붙이는 과정을 거치고
	위 과정에서 workspace pod도 같이 생성이 되어 node들 중에 cpu라벨과, image.pull.ready=True을 가지고 있는 노드에 할당이 되는 구조
	
	그럼 위 흐름대로라면 cpu라벨을 가진 노드풀이 없는 경우에는 노드부터 생성이 안되고
	temp-xxx는 중간에 제거되고,
	workspace pod는 계속 원하는 노드 -위에서 말한 두가지 라벨(cpu, image.pull.ready=True)을 가지고있는-가 생성이 될 때까지 계속 기다리게 된다.

아직 설명이 전부 이해 가지않을수있다.

단계별로 하나씩 세팅을 해보면서 파악해보자

## **Firebase(Realtime Database)**

firebase 홈페이지에 접속해 새 프로젝트를 생성한다.

실습에서는 test-workspace로 프로젝트를 생성했다.

Firebase Console에서 데이터베이스 또는 스토리지 인스턴스를 만들 때 Firebase 보안 규칙으로 데이터 액세스를 제한(**잠금 모드**)할 것인지 아니면 모든 사용자의 액세스를 허용(**테스트 모드**)할 것인지를 선택한다. Cloud Firestore 및 실시간 데이터베이스에서 **잠금 모드**의 기본 규칙은 모든 사용자의 액세스를 거부한다. Cloud Storage에서는 인증된 사용자만 스토리지 버킷에 액세스할 수 있다.

우선은 위 설명을 참고해서 테스트모드로 시작후 사용중인 rule을 등록하면 규칙기반으로 데이터 액세스를 제한하는 잠금모드로 변경된다.

프로젝트 개요 > 프로젝트 설정 에서 서비스 계정을 선택하고 새 비공개 키 생성 버튼 클릭후 저장한다

Firebase Admin SDK용 SA파일이 생성되는데 이 파일은 이후 Cloud Run에서 사용될 예정이다.

[Firebase Admin SDK용 SA파일생성](https://www.dropbox.com/s/yrjnu1e7uaycbby/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%EC%A0%84%2011.26.48.png?dl=0)

이후에는 실시간 데이터베이스에서 데이터 탭을 클릭해 실제 서비스에서 사용할 테이블을 세팅한다

실습에서는 clusters (클러스터 정보) 와 localschools (로컬스쿨 정보) 테이블을 세팅하고 필요한 필드도 함께 세팅했다.

## Google Cloud Platform (GCP)

Firebase 와 Goolge Cloud는 연결되어있다. 두 플랫폼을 함께 사용할 수 있고 제품, 프로젝트, 결제 등의 공통 인프라를 공유하고 있다([관련링크](https://firebase.google.com/firebase-and-gcp?hl=ko)). 앞선 단계에서 Firebase를 test-workspace 프로젝트로 먼저 세팅해었기 떄문에 GCP 에서도 test-workspace 프로젝트를 사용할 수 있다.

현재 실습에서는 Kubernetes Engine과 Cloud Tasks, Cloud Run, Colud DNS, Filesotre 서비스를 사용할 예정이고 서비스 사용이 처음이라면 활성화가 필요하다. 활성화 과정에서 결제 정보를 추가해야할 수 있다.
처음 GCP를 사용하면 크레딧을 제공하지만 제공한 크레딧과는 별개로 결제 정보 추가는 필요하다.

처음 사용하는 제품들의 경우 아래와 같은 화면이 나오고 사용버튼을 클릭해야 활성화 후 제품을 사용할 수 있다.

[제품 활성화](https://www.dropbox.com/s/5qefdh2c0khxrvw/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%EC%A0%84%2011.41.39.png?dl=0)

### IAM

IAM 메뉴에서는 사용자를 권한에 따라 추가할수 있고 서비스 계정을 생성할 수 있다.

외부 관계자를 프로젝트에 초대할 경우가 발생할 수 있는데 이 때는 조건을 추가해 만료되는 시간을 지정해서 관리할 수 있다. 시간 또는 리소스 유형을 지정해 관리할 수 있다.

[IAM 액세스 권한부여](https://www.dropbox.com/s/i8zmth8eexesnya/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%EC%A0%84%2011.45.34.png?dl=0)

위 화면에서 + IAM 조건추가를 클릭하면 아래와 같은 탭이 나온다.

[IAM 조건 추가](https://www.dropbox.com/s/ehxmtcdpwames0i/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%EC%A0%84%2011.46.08.png?dl=0)

IAM 메뉴에서 필요로 하는 실제 사용자 이메일주소로 액세스 권한을 부여해보았고

사이드 메뉴에 서비스 계정 메뉴가 있는데 서비스 계정 메뉴를 선택하고 서비스 계정 만들기를 선택한다.

계정이름을 지정하고 (test-server) 역할은 소유자로 지정 후 완료버튼을 클릭하면 서비스 계정이 새로 생성된다.

새로 생성된 서비스 계정을 클릭후 가장 우측에 있는 작업 컬럼에서 키 관리 > 새 키 만들기 > json > 만들기 > 다운 을 클릭하면 서비스 계정의 키를 다운로드 받을 수 있다.

해당 서비스 계정 키는 보관하고 있다가 Firebase Admin SDK용 서비스 계정 파일과 함께 이후 Cloud Run에서 사용될 예정이다.

### Cloud Run

cloud run은 완전 관리형 플랫폼에서 원하는 언어(Go, Python, Java, Node.js, .NET, Ruby 등)로 작성된 확장 가능하고 컨테이너화된 앱을 빌드하고 배포하기 쉽다.

CI/CD 구축을 쉽게할 수 있는데

소스 저장소에서 지속적으로 새 버전 배포 선택후 “Cloud Build 로 설정”을 클릭하면

소스저장소를 선택 및 authenicate 설정을 할 수 있다.

[Cloud Run 서비스 만들기](https://www.dropbox.com/s/ln2yfmqrvbgurxa/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%ED%9B%84%2012.05.11.png?dl=0)

[Cloud Build 소스저장소](https://www.dropbox.com/s/53tfckpeuife3gj/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%ED%9B%84%2012.05.39.png?dl=0)

authenicate 설정을 완료했다면 연결된 저장소 관리에 연동한 저장소가 조회되고 저장소를 선택하고 다음을 누르면 브랜치와 Dockerfile을 지정할 수 있다.

[Cloud Build 빌드구성](https://www.dropbox.com/s/vfgmevo1anbd2n1/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%ED%9B%84%2012.05.57.png?dl=0)

이렇게만 세팅하면 지정한 소스저장소의 타겟 브랜치에 새로운 변경사항이 반영되면
cloud run 이 트리거되어 변경사항이 동시에 반영된다.


자신에게 맞는 서비스 이름(test-api-server)을 지정하고 리전(us-central1)을 선택한다.

여기서는 CPU가 항상 할당되는 것으로 선택했고 자동확장 옵션에서

인스턴스 최소개수와 최대 인스턴스 수는 각각 1개로 지정했다.

컨테이너, 네트워킹, 보안 추가설정을 클릭해서 컨테이너 포트는 80으로

용량은 CPU 2, Memory 8GiB 로 지정했다. 해당 부분은 본인 서비스에 맞게 적절한 값을 세팅하면 된다.

조금더 아래로 내려가면 환경변수를 추가할 수 있는데

여기에 등록하는 환경변수들은 Cloud Run과 연동되는 저장소에서 사용하는 변수들이 된다.

본인 서비스에서 사용하는 환경변수들은 추가하면 된다.

여기서는 이전 단계에서 따로 저장해두었던 Firebase 서비스 계정정보, IAM 서비스 계정에서 생성했던 test-server 의 서비스 계정정보 그리고 내외부 슬랙알림 webhook url, sentry 레포팅을 위한 key값을 환경변수로 지정해 등록하였다.

등록한 환경변수는 바로 위에서 연결한 소스저장소 로직과 연관되어 있는 변수들이고
본인이 사용해야할 다른 변수가 있다면 적절히 추가하면 되겠다.

[Cloud Run 환경변수 입력](https://www.dropbox.com/s/y3ps8f7k9lola77/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%ED%9B%84%2012.13.47.png?dl=0)[https://www.dropbox.com/s/y3ps8f7k9lola77/스크린샷 2023-04-13 오후 12.13.47.png?dl=0](https://www.dropbox.com/s/y3ps8f7k9lola77/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%ED%9B%84%2012.13.47.png?dl=0)

환경변수 등록까지 하고 만들기를 하면 Cloud Run에 새로운 서비스가 등록되고 build 가 바로 시작된다.

Cloud Build에서 트리거를 만들수 없습니다. Cloud Build 서비스 계정에 필요한 역할을 설정하는 중에 오류가 발생했습니다 문제가 발생한다면

cloud build service account 설정에서 ([링크](https://console.cloud.google.com/cloud-build/settings/service-account))

Cloud Run 관리자와 서비스 계정 사용자를 “사용 설정” 으로 변경한다.

[서비스 계정권한](https://www.dropbox.com/s/r06helrnednumzb/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%ED%9B%84%2012.24.25.png?dl=0)

### Cloud tasks

Cloud Tasks는 대량 분산형 태스크의 실행, 디스패치, 전송을 관리할 수 있는 완전 관리형 서비스

Cloud Tasks를 사용하면 사용자 또는 서비스 간 요청 이외의 작업을 비동기적으로 수행할 수 있다.

Cloud Tasks는 App Engine 또는 임의의 HTTP 엔드포인트에서 실행할 수 있다.

그럼 test-server 라는 이름의 tasks 큐를 만들어보겠다.

지역은 이전 단계 Cloud Run과 일치되는 us-central1 로 지정하겠다.

[Cloud Tasks push큐 만들기](https://www.dropbox.com/s/9sl479r9czaoeea/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%ED%9B%84%2012.41.41.png?dl=0)

여러 설정값들이 나오는데

재시도 구성 영역을 보면 최대 시도 횟수, 최대 재시도 시간, 최소 백오프, 최대 백오프가 나온다.

최대 시도 횟수는 작업당 시도 횟수입니다. 무제한은 -1로 설정하세요.

최대 재시도 시간은 작업이 처음 시도된 시점부터 측정됩니다. 0으로 설정

최소 백오프는 큐에서 작업을 재시도해야 한다고 지정한 경우 작업이 실패한 후에 최소 백오프와 최대 백오프 시간 사이에 작업 재시도가 예약됩니다. 10분이라면 600 으로 설정

### Cloud DNS

GCP Cloud DNS에 접속해 새 DNS 영역을 만들어보겠다.

새로 생성한 DNS는 실제로 DNS를 관리하는 곳이 있다면 CGP 에서 생성되는 ns 레코드들을 실제로 DNS를 관리하는 곳 에 등록해주어야 한다.

여기서는 AWS Route53에 등록하는 것까지 해보겠다.

[DNS 영역 만들기](https://www.dropbox.com/s/wr5uswqlf8qn3ua/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%ED%9B%84%2012.48.11.png?dl=0)

DNS 영역 생성이후 NS 유형의 리소스 레코드 세트 세부정보를 확인하면

"ns-cloud-xx.googledomains.com." 형태의 서로 다른 데이터(xx 부분)가 4개 생성되어있고

이 라우팅 데이터를 AWS Route53 과 같이 도메인을 관리하는 곳에

"test-workspace.dev.test.io" 도메인 하위값으로 등록하면 된다.

### Filestore

Filestore는 스토리지이고 이번 실습에서는 개별 컨테이너가 생성되었을때 각 컨테이너가 공유 스토리지 형태로 연결되는 구성이다.

아래에서는 인스턴스 ID를 test로 인스턴스 유형은 기본, 스토리지 유형은 HDD, 용량은 1TiB 로 설정하였다.

[Filestore 인스턴스 만들기](https://www.dropbox.com/s/zwbie1pvyvajc0f/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%ED%9B%84%2012.57.05.png?dl=0)

만들기를 누르면 인스턴스가 생성되고

설정한 값들을 확인할 수 있고 고유 IP주소가 할당된다.

이 IP주소를 가지고 이후 클러스터 세팅시 사용할 것이다.

[Filestore 인스턴스 조회](https://www.dropbox.com/s/d3wtycg7c0o9gls/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-04-13%20%EC%98%A4%ED%9B%84%202.26.57.png?dl=0)