---
title: 7th week reading list
categories:
  - ML
tags:
  - clustering 
  - link analysis
  - mining massive datasets
  - pagerank
  - social network
last_modified_at: 2018-07-08T09:00:00-00:00
---

[Link Analysis PageRank](http://infolab.stanford.edu/~ullman/mmds/ch5.pdf)
구글의 핵심기술은 PageRank . 무엇이고 어떤방식으로 효율적으로 계산하는지

5.1 PageRank 
5.1.1 Early Search Engines and Term Spam 

inverted index 
search query 가 발급되면 페이지에 있는 terms는 추출된다 inverted index 에서
그리고 순위가 매겨진다 얼마나 사용되었는지
발생빈도가 높을수록 페이지와 관련성이 높을것이라 추측
— 웹 페이지를 부정확하게 묘사하다
Term spam을 막기위해 구글은 2가지 혁신을 소개한다
랜덤페이지에서 시작하는 서퍼를 가정해서 수많은 서퍼들이 방문하는 페이지를
보다 중요한 페이지로 여긴다
페이지에 나타난 단어들로 판단할뿐만 아니라 가까운 링크된 페이지들이 사용하는 단어들도 고려

사용자들은 그들이 생각하기에 괜찮은 페이지들을 링크로건다
랜덤한 서퍼들의 행동은 어떤 페이지가 사용자들의 방문이 잦은지를 알려준다


5.1.2 Definition of PageRank 
pageRank는 페이지가 얼마나 중요한지 측정
재귀방정식의 해이다 어떤페이지가 중요한 페이지들와 링크되어있다면 그 페이지는 중요하다

PageRank는 transition matrix 의 주요한 고유벡터이다
우리는 PageRank를 0이아닌 어떤 값에서 시작해서 현재벡터에 반복적으로 곱함으로써 더 나은 측정값을 가져온다

PageRank 계산은 수많은 랜덤한 서퍼들의 행동을 가정했다 서퍼들은 그들이 생각하기에 유용한 링크에 접속하려는 경향에서 랜덤한 서퍼들 역시 유용한 페이지에 접속하려고 할것

Dead Ends는 나가는 링크가 없는 웹페이지를 말한다
해당 페이지에 들어오면 PageRank계산시 0에 수렴한다

Spider Traps 는 서로만 연결되어 있어서 외부로 나가는 링크가 없는경우

Taxation Schemes 과세계획? 위에서 설명한 spider traps 효과에 대응하기 위해서
일반 계산식을 약간변형 해서 파라미터 베타(0.85) 를 다음단계에서 곱하고
1-베타 / 총 페이지의 수를 더한다.

Very Large-Scale Matrix–Vector Multiplication 
전체 웹을 계산한다는건 거의 불가능하기에 한 기계에서
우리는 k조각으로 전체를 나누고 매트릭스를 k^2 으로 나눈다

Teleport set 에만 tax 를 나눈다?
TrustRank 대학교 홈페이지와 같이 신뢰할 수 있는 페이지들의 모임

5.2 Efficient Computation of PageRank 
PageRank를 계산할때 직면하는 문제

transition matrix of the Web M 은 희소행렬이다 빈값이 많다
0이 아닌 값만 표현하고자한다

MapReduce를 사용할수없다

5.2.1 Representing Transition Matrices 
간결하게 희소행렬을 표현하는 방법은 이런식으로
시작점 degree 도착점
A 3 B,C,D
B 2 A,D
C 1 A
D 2 B,C

5.2.3 Use of Combiners to Consolidate the Result Vector 


6_Social-Network Graphs
Mining Social-Network Graphs 

communities 는 노드들의 집합인데 평소와는달리 강한 결합을 나타낼때

10.1 Social Networks as Graphs 
Social Networks
특정 네트워크에 참여하는 사람들의 집합
페이스북의 경우 이 관계는 친구.. 친구이거나 친구가 아니거나 관계는 2가지뿐이다
때론 관계가 정도를 갖을때도있다 
지역성이나 무작위성이 없다는 가정이있다

소셜그래프는 비방향성이고 때론 방향성을 갖는다 트위터에서 팔로워의 경우

10.1.3 Varieties of Social Networks 
Telephone Networks 
전화네트웍은 친구사이이거나 같은 클럽이거나 같은 회사에 근무하는 사람들

Email Networks 

10.1.4 Graphs With Several Node Types 
복잡한 예에서는 users, tags, and pages 로 구성

10.2 Clustering of Social-Network Graphs 
사회적 네트워크의 중요한 면은 개체들의 공동체를 포함
같은 학교의 친구들이나 같은주제에 대해 연구하는 연구자들

소셜네트워크그래프에서 처음해야할건
거리 측정을 정의하는것..

소셜네트워크 그래프의 Hierarchical clustering 은 
간선으로 연결된 2개의 노드를 결합하는데서 시작

10.2.3 Betweenness 
10.2.4 The Girvan-Newman Algorithm 
각각의 노드를 한번만 방문하고 최단거리의 수를 계산한다
credit이 무얼 의미하는가???


좋은 군집이란 무엇인가?
모여있는 관계는 최대로 밀집되게 하고 떨어진 군집간의 수는 최소화 시키는.

최상의 컷은 그래프의 가운데를 자르는것
Cunductance 
상대적으로 밀집되어있는 그룹과 나머지그룹이 연결되어있는 정도

Cunductance를 적게 가지는 결과로 그래프를 나눈다

D-regular graph

K-way 클러스터링
2가진 기본 접근법
재귀로 2개씩 나누는방법
고유벡터를 찾아서 군집화하는 방법...

Trawling 
웹 그래프에서 작은 communities 를 검색하는 방법

