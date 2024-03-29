---
title: RDBMS NoSQL 차이
categories:
  - Dev
tags:
  - rdbms
  - nosql
  - database
  - backend interviews
last_modified_at: 2022-12-11T08:00:00-00:00
---



## RDBMS vs NoSQL 의 비교되는 특징과 데이터베스 선택시 어떤 점을 고려해야할지 정리해보았습니다.

---


# RDBMS란 무엇인가?

- 관계형 데이터베이스(RDB)는 테이블, 행, 열의 정보를 구조화하는 방식. RDB에는 테이블을 조인하여 정보 간 관계 또는 링크를 설정할 수 있는 기능이 있어, 여러 데이터 포인트 간의 관계를 쉽게 이해하고 정보를 얻을 수 있다.
- 관계형 데이터베이스는 비즈니스에서 데이터를 구성, 관리, 연결하는 데 도움이 되는 스프레드시트 파일 모음
관계형 데이터베이스의 모든 테이블에는 행에서 고유하게 식별 가능한 기본 키 라는 속성이 있으며, 외래 키(다른 기존 테이블의 기본 키를 참조)를 사용하여 각 행에서 서로 다른 테이블 간의 관계를 만드는 데 사용
- 데이터가 사전 정의된 관계로 구성되므로 데이터를 선언전으로 쿼리할수 있음. **선언적 쿼리**는 시스템이 결과를 어떻게 연산해야 하는지 표현하지 않고 시스템에서 추출할 내용을 정의하는 방법. **관계형 시스템의 핵심**
- 관계형 데이터베이스 관리 시스템(RDBMS)은 관계형 데이터베이스를 만들고 업데이트하고 관리하는 데 사용

## 관계형 데이터베이스의 이점

직관적인 데이터 표현 방법을 제공하고 관련 데이터 포인트에 쉽게 액세스할 수 있다는 점
그래서 관계형 데이터베이스는 인벤토리 추적부터 트랜잭션 데이터 처리 및 애플리케이션 로깅에 이르기까지 대량의 구조화된 데이터를 관리해야 하는 조직에서 가장 많이 사용

- 유연성: 필요할 때마다 간편하게 테이블, 관계를 추가 또는 삭제하고 데이터를 변경
- ACID 규정 준수: 관계형 데이터베이스는 ACID(원자성, 일관성, 격리, 내구성) 성능을 지원하므로 오류, 실패, 기타 잠재적 오작동에 관계없이 데이터 유효성을 검사할수 있음
- 사용 편의성, 공동작업, 내장된 보안 기능, 데이터베이스 정규화

# NoSQL은 무엇인가?

- nosql은 빅데이터의 등장으로 인해 데이터와 트래픽이 기하급수적으로 증가함에 따라 rdbms에 단점인 성능을 향상시키기 위해서는 장비가 좋아야 하는 scale-up의 특징이 비용을 기하급수적으로 증가하기 때문에 데이터 일관성은 포기하되 비용을 고려하여 여러 대의 데이터에 분산하여 저장하는 scale -out을 목표로 등장
- 다양한 형태의 저장기술은 rdbms 스키마에 맞추어 데이터를 관리해야 된다는 한계를 극복, 수평적 확장성을 쉽게 할 수 있다는 장점, 데이터를 연결되지 않은 개별 파일로 저장하며, 문서 또는 리치 미디어 파일과 같은 복잡하고 구조화되지 않은 데이터 유형에 사용
- NoSQL 데이터베이스는 유연한 데이터 모델을 따르므로 자주 변경되는 데이터를 저장하거나 다양한 유형의 데이터를 처리하는 애플리케이션에 적합

## NoSQL의 특징

- RDBMS와 달리 데이터 간의 관계를 정의하지 않는다, 일반적으로 테이블 간의 Join도 불가능
- 더 대용량의 데이터를 저장할 수 있음, 페타바이트급의 대용량 데이터
- 분산형 구조: 하나의 고성능 머신에 저장하는 것이 아니라, 일반적인 서버 수십 대를 연결해 데이터를 저장 및 처리하는 구조를 갖음. 여러 대의 서버에 분산해 저장하고, 분산 시에 데이터를 상호 복제해 특정 서버에 장애가 발생했을 때에도 데이터 유실이나 서비스 중지가 없는 형태의 구조
- 고정되지 않은 테이블 스키마

## Key -value database

- key는 value에 접근하기 위한 용도로 사용되며, 값은 어떠한 형태의 데이터라도 담을 수 있다. 심지어 이미지나 비디오도 가능. 간단한 API를 제공하는 만큼 질의의 속도가 굉장히 빠른편
- Redis, Rick, amazon dynamo db, Oracle Coherence
- 단일 연산에 의하여 처리를 완료하거나 취소할수 있는 케이스
    - 사용자의 프로필 정보, 웹서버의 클러스터를 위한 세션정보, 장바구니 정보, URL 단축 정보 저장 등

## Document database

- key와 document 의 형태로 저장, value가 계층적인 형태인 도큐먼트로 저장된다는 것
- 객체지향에서의 객체와 유사하며, 이들은 하나의 단위로 취급되어 저장
- 하나의 객체를 여러 개의 테이블에 나눠 저장할 필요가 없어진다
- 객체-관계 맵핑이 필요하지 않음
- 검색에 최적화
- 단점이라면 사용이 번거롭고 쿼리가 SQL과는 다르다는 점
- 도큐먼트 모델에서는 질의의 결과가 json 이나 xml 형태로 출력되기 때문에 그 사용 방법이 rdbms에서의 질의 결과를 사용하는 방법과 다르다
- mongoDB, CouchDB 등이 있다
- B트리의 특성으로 인하여 한번 작성되면 자주 변하지 않는 정보를 저장하고 조회하는데 적합
    - 중앙집중식 로그저장, 타임라인 저장, 통계 정보 저장

## Wide column database

- Column family model 기반의 database, 이전의 모델들이 key-value 값을 이용해 필드를 결정했다면, 특이하게도 이 모델은 키에서 필드를 결정
- 키는 row(키 값) 와 column-family, column-name을 가진다. 연관된 데이터들은 같은 column -family 안에 속해 있으며, 각자의 column-name을 가진다.
- 관계형 모델로 설명하자면 어트리뷰트가 계층적인 구조를 가지고 있는 셈
- 이렇게 저장된 데이터는 하나의 커다란 테이블로 표현가능하며 질의는 row, column-family, coulmn-name을 통해 수행
- Hbase, Hypertable 등이 있다
- 쓰기에 더 특화. 데이터를 먼저 커밋로그와 메모리에 저장한 후 응답하기 떄문에 매우 빠른 응답 속도 제공
    - 채팅 내용 저장, 메일 저장소, 알림 내용 저장, 실시간 분석을 위한 데이터 저장소등의 서비스 구현 적합
    - 카산드라는 페이스북에서 메일과 쪽지의 알림을 처리하기 위하여 개발, Hbase는 대용량 데이터 분석

## Graph database

- 데이터를 node와 edge, property 와 함께 그래프 구조를 사용하여 데이터를 표현하고 저장하는 database
- 개체와 관계를 그래프 형태로 표현한 것이므로 관계형 모델이라고 할수 있으며, 데이터 간의 관계가 탐색의 키일 경우에 적합하다
- 소셜 네트워크에 적합하고 연관된 데이터를 추천해주는 추천 엔진이나 패턴 인식 등의 데이터베이스로 적합
- Neo4J

# 데이터베이스 유형을 선택할때 어떤 것을 고려해야할까?

## 1) 원자성 → 관계형 데이터베이스

데이터베이스의 일관성 향상, 원자성 트랜잭션의 원칙에 의존,  예를 들면 계좌 A에서 계좌 B로 돈을 이체하는 작업

## 2) 수직 또는 수평 확장

데이터 전략이 수직 확장을 기초로 하는 경우 관계형 데이터베이스, 사용자 수가 제한적이고 관련된 쿼리가 많지 않은 경우 선호, 수직 확장의 기본적인 장점은 속도와 단순성.
사용자나 쿼리 측면에서 더 큰 부하가 예상되는 경우 수평 확장이 훨씬 저렴한 솔루션, NoSQL

## 3) 속도

ACID 준수보다 속도가 더 중요한 경우 문서 데이터베이스와 같은 비관계형 데이터베이스가 적합.
예를 들어 센서데이터와 같은 실시간 데이터의 경우

## 4) NoSQL과 기존 RDBMS와의 차이

NoSQL은 데이터를 저장하고 Key에 대한 Put/Get만 지원한다 아래 기능에 대한 고민이 필요

- Sorting, Join, Grouping, Range Query, Index
- 애플리케이션과 RDBMS는 네트워크 연결을 구축하고 유지, 애플리케이션은 종료될때 연결을 끊는다
반면 NoSQL 데이터베이스의 일종인 DynamoDB는 별도의 웹서비스이며 해당 데이터베이스와 상호작용은 상태 비저장된다. 애플리케이션과 지속적인 네트워크 연결을 유지할 필요가 없음. 
DynamoDB와의 상호작용은 HTTPS 요청 및 응답을 사용하여 이루어진다.

![RDB_NoSQL_AWS_DynamoDB](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/images/SQLtoNoSQL.png)



## 참고자료

[관계형데이터베이스](https://cloud.google.com/learn/what-is-a-relational-database?hl=ko)

[사용 사례에 적합한 데이터베이스는 무엇일까요?](https://www.integrate.io/ko/blog/which-modern-database-is-right-for-you-ko/)