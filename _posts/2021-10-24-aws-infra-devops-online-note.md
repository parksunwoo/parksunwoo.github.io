---
title: "한 번에 끝내는 AWS 인프라 구축과 DevOps 운영 초격차 패키지 Online - 강의노트"
categories:
  - Dev
tags:
  - aws
  - infra
  - devops
  - architecture
  - ci/cd
last_modified_at: 2021-10-24T09:45:00-00:00
---

패스트 캠퍼스 온라인 강의
[한 번에 끝내는 AWS 인프라 구축과 DevOps 운영 초격차 패키지 Online](https://fastcampus.co.kr/dev_online_awsdevops) 
를 수강하며 강의 주요 내용을 정리해보았습니다

강의를 들으면서 좋았던 점은 막연히 알고있던 DevOps에 대해 전체 큰 흐름을 알수있었던 것 같습니다
아쉬웠던 부분은 여러개의 섹션으로 나뉘어 섹션별로 강사분이 다르다보니 강의 수준이 고르지 않고, devops 강의다보니 명령어를 따라서 해보는 일이 많은데
따로 스크립트나 코드가 지원되지 않는 부분에서는 따라가기가 어려웠습니다.

강의를 들으면서 질의응답을 할수있는 창구가 있어 진행되며 겪는 이슈들에 어려움이 필요한 떄에 해결된다면 강의에 좀더 몰입할수 있지 않을까 생각해봅니다
강의 분량이 워낙 많다보니 끝까지 강의를 듣는데 생각보다 오랜시간이 들고 강의를 수강한 부분들에 대해서만 내용이 정리되었고 아직 강의를 수강하고 있는 중입니다.

아래 강의정리 자료가 강의를 선택하시는데 조금이나마 도움이 되시기 바랍니다.

---

# Part 1. DevOps 기본 개념 및 강의 준비

**devOps 정의**

제품의 변경사항을 품질을 보장함과 동시에 프로덕션에 반영하는데 걸리는 시간을 단축하기 위한 실천 방법의 모음

넷플릭스 기술블로그.

개발과 운영의 벽을 허물어 더 빨리 자주 배포하자!

**devOps vs DevOps 엔지니어**

데브옵스 엔지니어는 조직에 데브옵스 문화를 정착시키는데 도움을 주는 역할.

**데브옵스 팀의 업무 도메인**

네트워크

개발 및 배포 플랫폼

오케스트레이션 플랫폼

관측 플랫폼

클라우드 플랫폼

보안 플랫폼

데이터 플랫폼

서비스 운영

행위기반 분류 : 구축, 설정, 운영, 사용, 교육, 문서화

**데브옵스 팀의 핵심 지표**

장애복구 시간, MTTR (Mean Time To Recovery)

변경으로 인한 결함률

배포 빈도

변경 적용 소요 시간

**데브옵스 엔지니어 로드맵**

circle CI ← 스타트업 위한 무료 플랜이 있음

Elastic stack ← 로그관리에서 시작해서 기술 일원화 가능함 

Datadog 여유가 있다면 오픈소스 활용한다면 Prometheus, Grafana

기능성 

안정성, 자동화

보안, 최적화 - 성능 및 비용

장애가 발생하면 근본 원인을 찾고 장애 기록을 남기는 습관을 기르자.

지금 조직에서 깊게 공부해 볼 문제 혹은 기술을 선택하여 시간 투자를 해보자.

모르는 것에 솔직해지고 동료에게 도움을 구하자

정말 잘 하고 싶다면 영어는 필수다.

강의에서 사용되는 운영체제 환경

macOS, Homebrew



# Part 2. AWS 기반 소규모 & 중규모 아키텍트 설계

## ch 01. AWS 기초와 VPC

**AWS 과금방식**

탄력성 기반의 종량 과금제 방식

**API 중심의 서비스 설계**

AWS의 거의 모든 

**AWS 가격모델**

절감계획

1년 혹은 3년 기간 약정시 할인

규모의경제

더많이 사용할수록 단가를 낮출수있음

**AWS  대표 무료 서비스 - EC2**

유의사항

함께 사용하는 EBS 저장소의 사용량이 프리티어 수준인가?

공인 IP를 위해 사용한 Elastic IP는 모두 EC2 인스턴스에 연결되어 있는가?

네트워크 사용량이 무료 제공량(수신무료/ 송신 월 1GB 무료 제공)을 초과하지 않았는가?

T2 계열 인스턴스의 무제한 크레딧(Unlimited Credits ) 방식을 활성화하지 않았는가?

VPC의 많은 리소스는 생성하더라도 과금되지 않음

대표적인 VPC 과금 서비스: NAT Gateway PrivateLink Client VPC

**IAM**

![AWS IAM](/assets/images/aws-iam.png)

→ pem 개인키에 대한 권한을 최소권한으로 부여해야 접속이 가능함

권한변경후 정상접속확인

![AWS PEM](/assets/images/aws-pem.png)

**AWS CLI 자격증명: AWS 액세스 키 발급**

AWS 계정 혹은 IAM 사용자의 액세스 키 발급필요

Access Key ID

자격증명 주체를 가리킴

인증 요청한 사람이 누구인가?

Secret Access Key

자격증명 주체 본인임을 인증

인증 요청한 사람이 정말 A가 맞는가?

**AWS CLI 자격증명 설정 : EC2 인스턴스 프로파일**

인스턴스 프로파일

IAM 역할(Role)을 EC2 머신에 부여하기 위한 목적

EC2 내에서 AWS 서비스에 대한 권한을 수행할 수 있게 됨

여러 AWS 계정 운영

동일 계쩡 내 여러 리전 운영

동일 계정 내 여러 IAM 역할 수행

AWS SSO 조직 내 SSO 역할 수행

 

AWS CLI는 v2부터 파이썬 패키지 매니저가 아닌 공식 설치 가이드를 제공

windows WSL의 경우에도 동일하게 설치 진행 가능

AWS 계정 MFA 활성화

**다단계 인증, MFA (Multi Factor Auth)**

정규 자격증명 방법 외에 서비스 제공자가 지원하는 추가 보안 인증 방법을 수행하는 것

지원하는 MFA 유형

가상 MFA 디바이스 : 스마트폰 또는 기타 디바이스에서 실행되며 물리적 디바이스를 에뮬레이션하는 소프트웨어 app

authy, 1password 를 사용하라

U2F 보안키

다른 하드웨어 MFA 디바이스

SMS 문자 메시지 기반 MFA

IAM 사용자만 사용 가능

TODO. 루트계정과 sub 계정을 분리해서 관리, 각 계정은 2FA로 관리

**AWS 비용 이슈 대처법**

테스트 후 리소스 정리

오픈소스 도구 : aws - nuke

AWS 계정 내 모든 리소스 혹은 특정 리소스 제거 계정 전체를 정리할 경우에 용이

프리티어 계정이 만료된 경우

Gmail 에서 Plus (+) 사용하기

사용자의 아이디 뒤에 '+0000' 접미사를 붙여 새로운 이메일 주소 생성 가능

a+dev@gmail.com a+test@gmail.com 으로 이메일 보내더라도 a@gmail.com 계정에서 이메일 수신

**AWS의 주요 서비스 소개 - 컴퓨팅 서비스**

EC2 : 사양과 크기를 조저할 수 있는 컴퓨팅 서비스

Lightsail : 가상화 프라이빗 서비스

Auto Scaling : 서버의 특정 조건에 따라 서버를 추가/삭제할 수 있게 하는 서비스

workspaces : 사내 PC를 가상화로 구성하여 문서를 개인 pc에 보관하는 것이 아니라 서버에서 보관

**AWS의 주요 서비스 소개 - 네트워킹** 

route 53 : DNS 웹서비스

VPC : 가상 네트워크를 클라우드 내에 생성/구성

Direct Connect : on premise 인프라와 aws 를 연결하는 네트워킹 서비스

ELB : 부하 분산(로드 밸런싱) 서비스

**AWS의 주요 서비스 소개 - 스토리지 데이터베이스**

S3: 여러가지 파일을 형식에 구애받지 않고 저장

RDS: 가상 SQL 데이터베이스 서비스

DynamoDB: NoSQL 데이터베이스 서비스

ElasticCache:  In-memory 기반의 cache 서비스 (빠른 속도 필요로 하는 서비스와 연계_

 

**AWS의 주요 서비스 소개 - 데이터분석 & AI**

redshift : 데이터분석에 특화된 스토리지 시스템

EMR : 대량의 데이터를 효율적으로 가공 & 처리

sagemaker : 머신러닝 & 데이터분석을 위한 클라우드 환경 제공

**네트워킹의 기본**

컴퓨터 사이에 통신을 하려면 컴퓨터의 위치값을 알아야한다. 각 컴퓨터의 위치값(주소)을 IP주소라고 지칭한다.

A Class

앞 8자리 network bit

뒤 8*3자리 host bit

B Class

앞 8*2자리 network bit

뒤 8*2자리 host bit

C Class

앞 8*3자리 network bit

뒤 8*1자리 host bit

**Network 나누기 - CIDR**

ip를 2개 subnet으로 나눔

subnet A 

211.11.124.0/25

subnet B

211.11.124.128/25

ip를 4개 subnet으로 나눔

211.11.124.0/26

211.11.124.64/26

211.11.124.128/26

211.11.124.192/26

VPC  (Virtual Private cloud)

가상 네트워크, 고객의 자체 데이터 센터에서 운영하는 기존 네트워크와 매우 유사

- 계정생성 시 default로 VPC를 만들어줌
- EC2, RDS, S3등의 서비스 활용 가능
- 서브넷 구성
- 보안설정
- VPC Peering (VPC 간의 연결)
- IP 대역 지정 가능
- VPC는 하나의 Region에만 속할 수 있음

VPC의 구성요소

Availablity Zone : 

물리적으로 분리되어있는 인프라가 모여있는 데이터 센터

각 AZ는 일정 거리 이상 떨어져 있음

하나의 리전은 2개 이상의 AZ로 구성됨

각 계정의 AZ는 다른 계정의 AZ와 다른 아이디를 부여받음

subnet:

VPC의 하위단위

하나의 AZ에서만 생성가능

CIDR 블록을 통해 subnet 을 구분

Internet gateway (IGW):

인터넷으로 나가는 통로

Private subnet 은 IGW로 연결되어 있지 않다

Route Table:

트래픽이 어디로 가야 할지 알려주는 테이블

NACL / Security Group:

보안검문소

Access Block 은 NACL 에서만 가능

**NAT instance / gateway**

 데이터베이스 같은 보안이 중요한 부분은 VPC Subnet(private) 에 위치

private subnet 이라도 업데이트를 위해 때로는 외부접속을 허용해야할 때가 있다

private subnet 에서 public subnet 으로 쏘고 우회적으로 접근

private subnet 안에 있는 private instance가 외부의 인터넷과 통신하기 위한 방법

**Bastion host**

private instance 에 접근하기 위한 수단

public subnet 내에 위치하는 EC2 

VPC endpoint

AWS의 여러 서비스들과 VPC를 연결시켜주는 중간 매개체

- AWS에서 VPC 바깥으로 트래픽이 나가지 않고 aws의 여러 서비스를 사용하게끔 만들어주는 서비스
- private subnet 같은 경우는 격리된 공간인데, 그 상황에서도 aws의 다양한 서비스들 연결할 수 있도록 지원하는 서비스

- Interface Endpoint
- Gateway Endpoint

stateful : 이전 상태를 기억함, security group 

stateless :  이전 상태를 고려하지 않음, nacl

넷게이트 웨이를 사용하면 안되는이유는 

트래픽이 외부에 노출되어서 보안에 취약하다

VPC Endpoint 기능을 이용해 S3에 접근할 수 있다 (넷게이트 웨이 사용이 아닌)








## ch 02. 소규모 아키텍트

1 (개요) 모놀리식 아키텍쳐 vs 마이크로서비스 아키텍쳐

#모놀리식 아키텍쳐

- 장점

e2e 테스트가 용이

빠르게 간단한 서비스를 만들 수 있음

- 단점

조그마한 수정사항이 있어도 전체를 다시 빌드하고 배포

유지보수도 힘듬

덩치가 너무 커져 구동시간이 늘어남

일부분의 오류가 전체에 영향을 미침

각 기능에 따라 다르언어를 선택할 수 없음

#마이크로서비스 

- 장점

유지보수 용이

거대한 서비스도 빠르게 수정 가능

각 기능에 따라 다른 언어를 선택할 수 있음

- 단점

모니터링이 힘듬

e2e 서비스 구동 불편 (테스트가 불편)

3 도메인 주도 설계 개요

소프트웨어 개발과 도메인, 모델과의 불일치 발생 (도메인이 복잡하기 때문에)

기획과 개발의 불일치 발생

소통 어려움

- 보편언어

도메인에 대한 어휘를 이해관계자 들이 공통적으로 이해할 수 있도록 정의

- 모델 주도 설계

도메인 분석과 설계의 간극을 최소화

분석/설계/구현의 모든 단계를 관통하는 하나의 모델을 유지

모델 = 코드

- 도메인 주도 설계

소프트웨어가 복잡한 이유는 도메인이 복잡하기 때문

도메인 전문가와 개발자가 어떻게 협업할 것인지가 중요

보편언어, 모델 주도 디자인

- 도메인인이란?

영역,집합

비즈니스 domain을 의미

예를 들어 주문, 고객, 주소관리 등등과 같이 분리

- 전략적 설계

비즈니스의 상황에 맞게 설계

모든 context를 이벤트 스토밍을 통해 공유

각 context를 그룹핑

컨텍스트 매핑을 통해 bounded context 간의 관계를 정의

전략적 설계의 결과물 : 도메인 모델(서비스를 추상화한 설계도)

4 어플리케이션 이벤트스토밍

step1 domain event 정의

- 이벤트는 actor가 action을 해서 발생한 결과
- 각자 생각나는 event를 적고 더 이상 생각이 안 날때까지 붙입니다
- 서로 상이하면ㅇ서 중복된 것을 없애거나 합침
- 이벤트가 발생하는 시간 순서대로 붙임. 동시 수행되는 이벤트는 수직으로 붙임
- 비즈니스 용어로 무슨 일이 발생했는지를 적는것, 시스템 내에서

step2 프로세스 그룹핑

step3 command 정의

step4 trigger 정의

step5 aggregate 정의

step6 bounded context 정의

음식, 주문, 배송

step7 context map 작성

CH02_18. (인프라) 로드밸런서(L4 , L7) 의 동작원리와 AWS ELB

대규모 트래픽을 대응할때 1) 서버 1개의 용량을 높이거나 2) 로드밸런서를 통해 서버의 갯수를 늘리거나

- 로드밸런서(L4
    
    빠르고 싸다, 단순
    
    microservice에 불리
    
- 로드밸런서(L7
    
    스마트하다.  microservice에 유리
    
    데이터를 뜯어봐야 하므로 속도 이슈, 비용 이슈
    

CH02_23. (인프라) AWS Route53 DNS 기반 인증서 발급과 ALB에 https 설정하기

HTTP

웹 페이지의 정보를 웹 서버에 전송요청하는 프로토콜

클라이언트와 서버가 통신할 때, 정보를 있는 그대로 전송

보안에 취약

HTTPS

HTTP에 암호화 기능을 추가

암호화 기능이 추가되었으므로 보안성이 좋음

그러나 느림

서버 보안인증서를 발급받아야 함

CH02_24. (인프라) AWS CloudFront 와 CDN의 동작원리

- CloudFront = Cache + CDN
- 기본적으로는 Cache 서버
- Cache 서버는 전 세계에 흩어져 있는 인프라를 활용하기 때문에 추가저긍로 CDN의 기능도 보유
- 웹 서버의  비용을 감소시키며 전 세계의 유저를 대상으로 고속으로 웹 섭시르르 제공하도록 하는 서비스

Cache - 기존 방식

- 클라이언트가 요청할 때마다 서버가 응답해주는 방식
- HTML 문서를 준비해놨다가 뿌려주는 것이 아니라 동적으로 생성하는 방식
- 유저 입장에선 느림
- 서버 입장에선 서버비용이 많이 나감

Cache - 새로운 방식

- 클라이언트가 요청하여 응답된 결과를 cache 로 저장
- 다음 번에 클라이언트가 요청할 때는 기존 server에 요청할 필요 없이 cache server에 요청하여 저장된 정보를 추출
- 요청할 때마다 동적으로 생성하는 방식에서 벗어나, 준비된 데이터를 cache 에 저장하고 그걸 그대로 뿌려줌

CDN 

- 전 세계 어느 위치에서 접속하더라도 빠른 속도로 서비스할 수 있도록 하는 서비스
- Content Delivery Network
- 전 세계에 흩어져 있는 Edge Location (캐시 서버)을 활용

CH03_1 (백엔드) Django Session 활용 1

session 쿠키 내용 추가필요

CH03_3 (인프라) Docker 개요

문제점

- r개발 환경을 그대로 복제해서 같은 환경에서 배포하고 싶다
- 같은 서버에서 개발 환경을 여러개로 분리하고 싶다

가상환경

- 하나의 서버 내에서 프로그램별로 환경을 확실하게 구분해준다.
- 그러나 비효율성이 생긴다.
- 각 가상환경별로 컴퓨팅 파워를 나누어서 쓴다.

Docker

- 컨테이너 형태로 환경을 구분해준다
- 이미지를 통해 같은 환경의 가상 컴퓨터(컨테이너)를 무한히 생성할 수 있다(auto scaling)
- 컴퓨팅 자원을 공유해서 쓰므로 유동적이다

CH03_14. (인프라) AWS KMS 개요

데이터 암호화(Data Encryption)란?
- 허가받지 않은 외부의 침입자로부터 데이터가 유출되거나 조작되지 않도록 보호하기 위한 수단
- 데이터의 실제 내용을 허가된 사용자만 확인할 수 있도록 은폐하는 기법이 Data Encryption

- 전송 중인 데이터 암호화
- https
- 저장되어 있는 데이터 암호화
    - Client-side 암호화(내가 직접)
    - Server-side 암호화(aws가 알아서)
    

**Client-side** 암호화

내가직접암호화키를관리

필요하다면 AWS의 KMS를 활용할 수 있음

**Server-side** 암호화
AWS가 알아서 서버에 저장된 데이터를 암호화시켜놓음
암호화 키를 자동으로 관리
S3, RDS, DynamoDB, Redshift 등은 모두 암호화 기능을 기본으로 갖추고 있음
이들은 암호화 키를 관리하기 위해 AWS의 KMS를 우리가 모르는 사이에 사용함

AWS KMS(Key Management Service)란?
- AWS의 encryption key(암호화 키)를 관리해주는 서비스
- 관리하는 암호화 키를 CMK(Customer Master Key)라 부름
- CMK를 HSMs(Hardware Security Modules)라는 저장소에 저장함
- HSMs에 있는 CMK를 활용하기 위해 KMS API를 사용
- AWS Cloudtrail로 누가 어떤 Key를 어떻게 사용했는지 로그를 남김

- CMK는 생성된 **region**을 떠나지 않는다(region 설정 필수)
- CMK를 통해선 최대 4KB의 데이터만 암호화할 수 있다.
- 더 큰 데이터를 암호화할 땐 CMK가 아닌 data key를 활용한다.
- KMS는 CMK만 관리하고 data key는 관리하지 않는다.
- Data keys는 cmk를 통해서 생성된다.
- 하나의 cmk를 통해 여러 개의 data keys를 생성할 수 있다.
- 일단 큰 데이터를 암호화하기 위해 plaintext data key를 활용해서 암호화
- 암호화가 끝난 뒤엔 plaintext data key를 삭제

CH03_16. (인프라) AWS CodeCommit

AWS CodeCommit이란?
- GitHub와 유사한 코드 저장소 및 형상관리 서비스
- AWS CodeCommit은 git 기반의 리포지토리를 호스팅
- S3에 모든 코드를 암호화하여 저장(AWS KMS)
- IAM 인증을 통해 push/pull 에 대한 권한 관리
- 5명 미만까지는 무료

CH03_17. (인프라) AWS CodeDeploy1

AWS CodeDeploy란?
AWS Codedeploy is a deployment service that automates application deployments to **Amazon EC2 instances, on-premises instances, serverless Lambda functions, or Amazon ECS services**

**Part 3. AWS 기반 대규모 아키텍트 설계**

CH01_01. (개요) 마이크로서비스의 장단점

모놀리식 아키텍쳐

장점

e2e 테스트가 용이

빠르게 간단한 서비스를 만들 수 있음
단점

조그마한 수정사항이 있어도 전체를 다시 빌드하고 배포

유지보수도 힘듬

덩치가 너무 커져 구동시간이 늘어남

일부분의 오류가 전체에 영향을 미침

각 기능에 따라 다른 언어를 선택할 수 없음

마이크로서비스 아키텍쳐

장점

유지보수 용이

거대한 서비스도 빠르게 수정 가능

각 기능에 따라 다른 언어를 선택할 수 있음

단점

모니터링이 힘듬

e2e 서비스 구동 불편(테스트가 불편)

마이크로서비스 설계 - Domain event 정의

- 이벤트는 Actor가 Action을 해서 발생한 결과입니다.
각자 생각나는 Event를 적고 더 이상 생각이 안 날때까지 붙입니다.
서로 상이하면서 중복된것을 없애거나 합칩니다.
이벤트가 발생하는 시간 순서대로 붙입니다. 동시 수행되는 이벤트는 수직으로 붙입니다.
비즈니스 용어로 무슨 일이 발생했는지를 적는것이지, 시스템 내에서 발생되는것을 찾는게 아닙니다.

2 (설계) 마이크로서비스 간의 통신
방식(RabbitMQ)
Synchronous Solution
동기적
Rest API
서비스 A에서 서비스 B로 직접 요청을 보내고 동기적으로 응답을 기다림

Asynchronous Messaging
비동기적
메시지 브로커를 사용하여 서비스 A에서 서비스 B로 메시지를 보냄
서비스 A는 응답을 기다리지 않음
서비스 B는 일반적으로 동일한 메시징 시스템을 통해 결과를 사용할 수
있을 때 (결과가 예상되는 경우) 결과를 보냄
RabbitMQ와 Apache Kafka

Synchronous Solution의 문제점
A가연결을시도할때B가오프라인상태일수있다.
그런 경우 어딘가에 요청을 저장하고 나중에 다시 시도해야 하는가?
언제 다시 시도해야 하는가?
얼마나 자주 시도해야 하는가?
A가응답을기다리는동안시간이오래걸리거나실패할수있다.
B가 데이터 처리를 완료했지만 A가 오프라인 상태인 경우 어떻게

Asynchronous Messaging
A(Producer)에서 B(Consumer)로 메시지를 전달하고 B에서 A로 메시지를 전달할 때 중개자 역할을 하는
"메시지 브로커"

메시지 브로커는 Producer에서 메시지를 수신하여 Consumer에게 메시지를 전달하여 작업을 수행
메시지 브로커는 Consumer의 연결이 끊어졌을 때 임시 메시지 저장소를 제공
마이크로서비스 별로의 독립성을 확보

CH01_19. (인프라) 마이크로서비스 아키텍처 개발 시 고려해야 하는 사항들

**초기 설계의 중요성**

마이크로서비스 구분(독립성이 최우선)

API설계(거의 변경사항이 없어야 한다고 생각하고 신중히 설계)

데이터 스키마 관리(데이터 중복 최소화)

독립적으로 스케일링(메모리, GPU 등)

**end to end 테스트 환경 구축**

end2end 테스트 환경 구축

업데이트 전 테스트 서버에 업로드 후 3일간 QA기간

QA에서 모아진 내용들을 토대로 테스트 시나리오 구성(기획자가)

업데이트 전 테스트 서버에 업로드 후 1일간 테스트 시나리오 테스트(수동)

업데이트 전 테스트 서버에 업로드 후 1일간 테스트 시나리오 테스트(자동)

**마이크로서비스 배포 방식**

- 호스트 하나에 여러 개를 배포
    - 관리하기가 편함
    - 서비스들 간의 독립성이 떨어짐
    - 독립적으로 자원을 최적화할 수 없음

- 호스트마다 서비스 하나씩 배포
    - 호스트를 분리시켜서 각 서비스를 분리
    - 가상머신 기반, 컨테이너 기반(운영 체제 수준 가상화)이 있음
    

CH01_20. (인프라) 마이크로 아키텍처 개발 시 문제점들

처음 마이크로서비스 도입할 때 겪은 문제점들

모니터링과 관리 프로세스 /툴이 마이크로서비스 성공의 핵심이다

- 조그마한 코드 수정도 배포하기 위해 복잡한 프로세스 + 오랜 시간 필요
- 에러가 났을 때 debugging 하기가 상당히 까다로움 (로그들도 분리되어 있음)
- 너무 거대해서 한 눈에 안 잡힘

CI/CD 도입

- 코드 변경사항을 주기적으로 빈번하게 머지
- 빌드, 테스트 (테스트 시나리오 + 데이터 consistency), 머지의 자동화
- AWS Codebuild, CodeDeploy, Codepipline 활용

로그수집

- 모든 환경에서 logging을 기록 (s3, glacier 에 저장)
- 얼마나 오래 기록할것인지 결정
- 각 인스턴스의 성능 체크
- critical event 설정 (slack 과 연동)

업무분장 명확히

CH02_01. (개요) 서버리스 아키텍트 소개

- 서버가 없다 == 관리해야 할 필요가 없다

- 개발자가 서버를 관리할 필요 없이 애플리케이션을 빌드하고 실행할 수 있도록 하는 클라우드 네이티브 개발 모델 (서바가 없는 거이 아니라 추상화된 것임)
- 항상 대기하는 전용 서버가 없어서 실행을 완료하면 자원을 반납
- AWS Lambda, Google Firebase

장점

- 서버관리 (자동확장, 장애방지)가 불필요
- 관리보다 개발에 집중 가능
- 사용한 만큼만 과금(FasS, BasS)
- 급격한 트래픽 변화에 유연

단점

- 다른 클라우드 컴퓨팅 자원보다 비쌈
- 느림 (호출과 동시에 서버가 세팅되기 때문)
- 장기적인 작업에는 적합하지 않음
- 함수의 처리 결과에 따라 상태를 따로 저장

BaaS

- 클라우드 공급자가 제공하는 서비스를 이용해 백엔드 기능들을 쉽게 구현
- Customizing 어려움
- google firebase

FaaS

- FaaS는 기능을 하나의 함수로 구현
- 함수가 실행할 때마다 서버 자원을 할당받아 사용
- 로직을 개발자가 작성하므로 Customizing 가능
- AWS Lambda

AWS Lambda

- 어떤 일이 일어났을 떄 트리거되는 함수(코드)
- 이렇게 개발된 함수를 Lambda function (람다 함수)라고 지칭
- 트리거 설정 가능 (API Gateway, S3, DynamoDB, RDS)
- 제한시간이 있음 (그 이상으로 넘어가면 에러)
- 각기 다른 코드로 독립적으로 구성 가능 (마이크로서비스와 유사)
- 관리의 이슈가 생길 수 있음

AWS Lambda 요금

- 요청수, 코드를 실행하는데 걸리는 시간
- 월 100만건, 40만 초 까지는 무료
- 사용한 만큼만 과금
- 다른 클라우드 컴퓨팅 자원보다 비쌈

CH02_03. (API 게이트웨이) API 게이트웨이란

**AWS API 게이트웨이  == redirect** 

- 개발자가 API를  손쉽게 생성, 게시, 유지관리, 모니터링 및 보안 유지할 수 있도록 하는 완전관리형 서비
- AWS에서 제공하는 API Gateway 서비스
- API Gateway가 API가 지나갈수 있는 유일한 통로로 설정할 시 관리(로깅, 액세스 제어, 모니터링) 관리
- AWS API Gateway 를 통해 Lambda 와의 연동 가능 (마이크로서비스, serverless 에 적합)
- 3가지 제품으로 이루어져 있음
- HTTP API, Rest API, Websocket API
- API에 일괄적으로 인증을 붙힐 수 있음(권한 부여)

CH02_04. (API 게이트웨이) API 게이트웨이의 구성 요소들

CORS (Cross-Origin Resource Sharing)

- origin 이 다른 API를 호출하는 경우(fastcampus.com 과 fastcampus.org)
- 보안상에 문제가 생길 수 있는 여지가 있는 API호출 (마이크로서비스)

1. 다른 도메인
2. 다른 포트
3. 다른 프로토콜

- 원래는 같은 도메인이 아닌, 다른 도메인의 API를 호출할 수 없음
- 어떤 도메인에서나 호출하게 하고 싶거나, 특정 도메인에서만 호출하게 하고 싶다면 해당 부분을 설정해줘야 함 (마이크로서비스)
- 적합한 사용자가 호출하는지 인증하는 역할

CORS API 호출

1. 권한 체크(CORS check)
2. 허용
3. 실제 API 콜

Canary 배포

CH02_10. (AWS Lambda) 스케일링과 동시성

AWS Lambda Coldstart

- 처음 Trigger 후 Lambda Container (EC2)가 세팅됨(평균 0.8초 소요)
- 10분 지나도 아무런 트리거가 없으면 해당 Container를 다른 사용자에게 넘겨줌
- 10분 이내에 새로운 트리거 발생시 해당 Container 에서 실행됨
- Coldstart에 시간(0.8초)이 소요되므로 Coldstart를 막기 위한 방법들이 있음

1. 나만의 컨테이너를 지정받아 할당받는 방법
2. 자동으로 5분마다 무조건 한 번씩 Tigger시킴

- Coldstart 의 시간을 결정짓는 가장 중요한 요소는 Lambda 함수 코드의 사이즈

AWS Lambda Scaling

- 동접 1명일 때
- 동접 5명일 때
- 동접 100명일 때
- 한 번에 최대 1000개의 람다 컨테이너 운영 가능
- Reserved concurrency를 설정할 수 있음

버전관리와 Alias

CH02_18. (DynamoDB) 데이터 아키텍처의 변화

기존의 데이터베이스 형태

- 관계형 데이터베이스 시스템
- 데이터를 쌓기 전에 데이터를 쌓을 형식을 지정
- 일관성, 변동성 무, 예측 가능성
- 기업이 활용할 수 있는 데이터가 다양해지며 단점이 부각됨
- 데이터 형식(스키마)의 변동이 잦음, 예측 불가능성

DataLake, Nosql

- 데이터 레이크란 필요한 모든 종류의 데이터를 있는 그대로(데이터 변환작업 없이) 저장하는 단일 저장소
- ETL → ELT (Extract, Load, Transform)
- NOSQL (Not Only SQL)
- 형식을 지정해두지 않음

CH02_19. (DynamoDB) DynamoDB 개요와 Index 구성
DynamoDB

- AWS의 Nosql 서비스
- 빠른 쿼리 속도
- 수평확장 (Scale-out)이 쉽고 유연
- Auto-Scaling 지원/ 일정 기간 백업 지원
- 스키마 지정 필요 없음
- 트랙잭션/ 조인과 같은 복잡한 쿼리 불가능
- Ideal of application with known access patterns

Table, Item, Attribute, Index

- Table - mysql의 테이블과 개념이 같음
- Item - 하나의 객체
- Attribute - key/ value
- Index - 인덱스
    - Partition Key
    - Sort Key
    - Primary Key = Partition Key + Sort Key

CH02_23. (부록) AWS SAM 개요 및 실습

- AWS SAM (AWS Serverless Application Model)
- 서버리스 애플리케이션 개발을 위해 필요 기능들을 코드로 관리
- AWS Lambda 콘솔에서 하던 일들을 코드화, 체계화시킬 수 있음
- Cloudformation 보다 훨씬 간단하게 가능
- 실습 전에

**Part 4. 코드를 통한 인프라 관리 (IaC)**

**IaC (infrastructure as code)**

네트워크, 로드밸런서, 저장소, 서버 등의 인프라 자원을

수동 설정이 아닌 코드를 이용하여 프로비저닝하고 관리하는 것

대표적인 IaC 도구로 Terraform, CloudFormation, Pulumi, Azure ARM Template 등이 있음

 

형상관리 (Configuration Management)

서버 운영체제 상에 필요한 소프트웨어를 설치하고 원하는 설정으로 관리하는 것

Configuration as Code 라고도 불림

대표적인 형상 관리 도구로 Ansible Puppet, Chef, Salt Stack 등이 있음

이미지 빌드

AWS EC2, VMware, VirtualBox, Docker 등 여러 플랫폼에서 재사용 가능한 머신 이미지를 빌드하는 것

대표적인 이미지 빌더로 패커, AWS EC2 Image Builder 등이 있음

코드로 관리한다는 것 

사람이 수동으로 처리하는 것을 코드로 작성하여 관리

⇒ 휴먼 에러 방지/ 재사용성 / 일관성

소프트웨어 개발처럼 Git과 같은 버전 관리 시스템 (VCS) 활용 가능

→ 코드 리뷰 / 변경내용 추적 / 버전 관리 / 협업

선언형 설정 과 절차형 설정의 차이

CH02_18. 테라폼 Provisioner와 EC2 Userdata

![AWS Terraform](/assets/images/aws-terraform.png)

CH03_01. 패커 소개

![AWS Packer](/assets/images/aws-packer.png)

**Ch 04. 앤서블을 이용한 서버 형상 관리**

CH04_01. 앤서블 소개

서버 형상관리는 무엇인가

형상관리 (configuration Management)

서버 운영체제 상에 필요한 소프트웨어를 설치하고 원하는 설정으로 관리하는 것

configuration as code 라고도 불림

대표적인 형상 관리 도구로 ansible, puppet, chef, salt stack 등이 있음

왜 앤서블을 사용해야 할까?

간단한 YAML 문법

멱등성을 보장하여 여러번 실행해도 안전함

ssh / win_rm 기반으로 통신 → 대상 서버에 에이전트 설치가 필요하지 않음

여러 서버를 대상으로 동시 실행

특정 서버들을 타겟팅할 수 있음

버전관리하기에 용이함 → GitOps 가능

CH04_02. 인벤토리 (Inventory)

inventory

대상 서버 호스트를 관리하는 파일

그룹 기능 지원

static (정적)

dynamic (동적) - aws cloud

CH04_04. 플레이북 (Playbook)

# 플레이북 (Playbook): YAML로 정의. 순서대로 정렬된 플레이(작업 목록) 절차.

# 플레이 (Play): 작업 목록(Tasks). 특정 호스트 목록에 대하여 수행

# 작업 (Task): 앤서블의 수행 단위. 애드혹 명령어는 한 번에 단일 작업 수행.

# 모듈 (Module): 앤서블이 실행하는 코드 단위. 작업에서 모듈을 호출함.

# 콜렉션 (Collection): 모듈의 집합.

**Part 5.**

**Part 6.**

**Part 7. 모니터링 서비스 구축 및 운영**

CH01_01. 모니터링 개요

개발 → CI/CD → 운영 → 고객

모니터링이 스타팅 포인트가 될수있음, 데이터 제공, action plan

CH02_01. AWS CloudWatch 개요
**AWS CloudWatch**

-AWS 리소스 모니터링

-AWS내에 구동중인 application 모니터링

-다양한 AWS 서비스랑랑 integration

-지표를 통한 임계값 초과 시 alarm 발생

-자동화 시스템 구축

-지표 backup

**AWS CloudWatch 개념**

namespaces

metrics

- time-ordered set of data series

dimensions

statistics

resolutions

alarms

CH02_02. AWS EC2 monitoring #1

AWS EC2 monitoring (defalut metrics)

CH02_03. AWS EC2 monitoring #2

cloud watch agent 설치안내

기본적으로 제공하는 지표이외 memory 관련 지표도 추적가능

[https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/metrics-collected-by-CloudWatch-agent.html](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/metrics-collected-by-CloudWatch-agent.html)

[https://github.com/dev-chulbuji/devops_infra](https://github.com/dev-chulbuji/devops_infra)

CH02_05. AWS ALB monitoring

alb logging 

cloud watch 로는 메트릭을 보내고

log는 s3에 먼저쌓는다 s3에 쌓은 로그데이터를 athena를 이용해서 조회하거나

lambda 로 cloudwatch 에 연결할 수 있다

**Ch 03. metric 모니터링 시스템 구축**

CH03_01. prometheus 설치 #2

metric collecting (push vs pull)

push

- coupling metric backend system
- require (agent | logic) per application

pull

- require service discovery
- easy to update setting

metric types 

[https://prometheus.io/docs/concepts/metric_types/](https://prometheus.io/docs/concepts/metric_types/)

- counter
- gauge
- histogram
- summary