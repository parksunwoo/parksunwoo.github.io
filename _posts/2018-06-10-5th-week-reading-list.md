---
title: 5th week reading list
categories:
  - ML
tags:
  - clustering 
  - k-means
  - pca
  - unsupervised learning
last_modified_at: 2018-06-10T09:00:00-00:00
---


An Introduction to Statistical Learning

Ch10. Unsupervised Learning _ 


X에 대한 Y값이 없는 문제, 
principal components analysis, supervised tech을 사용하기전 시각화 또는 전처리 작업에 사용
clustering, 세부그룹을 알지못하는 데이터를 그룹핑하는 방법

10.1 The Challenge of Unsupervised Learning
분석의 정확한 목표가 없다? Y 값이 없다! 
지도학습과는 달리 비지도학습은 정확한 답이 없어서, 작업(모델)이 정확히 수행되는지 알수 없고, 암환자 식별, 온라인쇼핑몰 고객타게팅, 검색엔진 등 다양한 분야에서 비지도학습이 요구

10.2PrincipalComponentsAnalysis
주성분분석, 주성분방향은 특성공간에서의 방향 원본데이터와 높은 관련성있는 주요성분은 무엇인가?
가능한한 많은 variation 을 포함하는 더 낮은 차원의 데이터셋을 찾는다
p개의 특성들로 이루어진 선형결합
첫번째 주요성분은 특성들의 set이다 normalized 선형결합
두번째 주요성분은 첫번째 주요성분을 제외하고 최대의 variation을 나타내는 선형결합
두번째 주요성분은 첫번째 주요성분과 직교하는가?

# USArrests data 
거주자 10만당 범죄혐의자의 수
폭행, 살인,강간을 나타내는 범죄와 관련된 변수들은 서로 가까이 위치해서 지역에따라 더 높은 범죄발생율을 보이고 도시에서 인구 비율과는 떨어져있다 

10.2.2 Another Interpretation of Principal Components 
첫번째 주요성분은 p차원 공간의 직선이다 n개의관측치들과 가장 가까운
직선이 관측치들의 좋은 요약이 될수있다?

###Euclidean distance 

unscaled 된 변수들을 pca하게되면 변수들중 높은 variance를 가진 애들은 훨씬 큰 값을 갖게된다
scaled함으로써 결과에 상당한 영향을 미친다 보통 PCA를 하기전에 standard deviation을 각 변수에 취한다

주요성분으로 나타내게 되면 얼마나 많은 정보들이 사라지게 되는가? 하는 자연스런 질문 -> PVE 라는 수치로 
얼마나 많은 주요성분을 사용할지 정하자
주요성분의 갯수를 선택하는 비교적 단순함을 지도학습의 한 부분이다 좀 더 명확하고 객관적으로 표현할수있다는

10.3 ClusteringMethods
도메인 특화된 지식이 필요, 군집화, 2개이상의 관측치가 비슷하거나 다르다는건 무얼 의미하고 어떤기준으로 나뉘어야 하는가
PCA 와 cluster 작은수를 통한 요약으로 간단화하는게 공통점이나 둘은 좀 다르다
PCA는 더 낮은 차원으로 표현해서 좋은 주요성분을 찾는것이고
Clustering은 동질의 세부그룹을 찾는것이다!
마케팅에서 고객을 그룹핑 하는건 특정광고에 타케팅, 특별한 물건을 사게끔 할수있다

10.3.1 K-Means Clustering 
겹치지 않는 K개의 그분된 데이터셋으로 K를 정해야한다
1. 각각의 관측치는 K개의 군집중 적어도 한개에는 속해야한다
2. 1개보다 더 많은 군집에 속할수는 없다

### pairwise squared Euclidean distances ??

1 to K 랜덤하게 수를 할당해서 각각의 K 군집의 cluster centroid를 계산
더나은 결과치가 나오지않을때까지 최소화하는 값을 찾고, 다시 재할당하는 작업을 반복
초기설정값에 따라 결과가 달라진다
따라서 다른 랜덤한수들을 할당해서 알고리즘을 run하는게 중요하다
10.7 figure 색깔만 다르지 235.8은 같은 모습이다

10.3.2 Hierarchical Clustering 
K를 정할필요가없다 vertical axis 를 토대로 한 두 관측치의 유사성을 봐야한다
군집을 자르는 높이가 곧 K mean에서 K와 같은 역할이다

다름의 개념 -> linkage 개념으로 확장된다 
complete, average, single, and centroid 
Largest , smallest, undesirable inversions 

Dissimilarity를 측정방법을 정하는 건 최종결과에 많은 영향을 미친다
Euclidean distance 를 사용한다면 구매이력이 없는 고객은 구매이력이 없는 다른고객과 clustering 되겠지만 correlation-based distance 는 높은 연관성을 가진다면 같은 그룹으로, Euclidean distance가 높다고해도
관측치 또는 특성을 standardized 할 것인가?