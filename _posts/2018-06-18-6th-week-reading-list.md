---
title: 6th week reading list
categories:
  - ML
tags:
  - em algorithm 
  - mixture models
  - murphy
last_modified_at: 2018-06-18T09:00:00-00:00
---

Murphy Ch11\. Mixture models and the EM algorithm   
\* 11.1  Latent variable models  숨겨진 변수 leaden variable or lvm   
2가지이유에서 이점이 있다 lvm 은 더 적은 파라미터를 갖는다  
  
  
\* 11.2  Mixture models  다대다 일대다 다대일 일대일 형식의 모델로 구분된다  
  
base distribution 을 mixing 하는것이라 mixture model 이라고 부른다혼합모델은 크게 2개가 있는데 하나가 black-box density model이고 데이터 요약, 이상치 검출, 분류를 생성할때 유용하다  
  
다른 하나는 클러스터링에 적합  
Soft clustering 은 한 개체가 여러 군집에 속할수있음  
Hard clustering 은 한 개체가 여러 군집에 속하는 경우를 허용하지 않는 군집화 방법  
효모유전자 시간경과에 따른 데이터 plot, 16개 클러스터 중심을 만듬  
  
11.5 그림을 보면 mixing weights가 나와있는데 결과가 좋지않다  
모델이 너무 단순해서 숫자의 시각적인 특성들을 잡아내지 못한다  
Likelihood function이 볼록하지않다  
  
mixture of experts   
expert 전문가 또는 학습자  
각각의 하위모델들을 입력공간의 특정지역의 학습자로 여긴다  
Inverse 문제를 푸는데 유용하다  
x축에 값들에 대해 unique한 값을 갖는 y들을 forwards model   
Posterior mean은 좋은 예측치를 만들지 못한다  
그러나 posterior mode는 input 값이 종속적이라면 합리적인 추정치를 제공한다  
  
  
11.3 Parameter estimation for mixture models   
파라미터들을 어떻게 배우는지 이야기해보자  
결측치가 없고 hidden 변수가 없는걸 complete data 라고 한다  
  
K=2 인 2차원 가우시안 혼합 모델의 데이터에 대해서 우도함수는 두 개의 피크를 포함하고 있다.   
사후확률이 다수의 봉우리를 포함한 (multimodal ) 형태로 표현될 수도 있다는 것이다  
Unidentifiability MLE가 unique 하지않다  
유일한 최대우도 (MLE)이 존재하지 않기 때문에, 매개변수는 식별가능하지 않다고 한다.  
그러므로 사전확률이 특정 라벨링에 영향을 주지 않는다면, 유일한 최대 사후확률 추정(MAP)도 존재하지 않는다  
이는 사후확률이 다수의 봉우리를 포함한 형태라는 것과 같은 의미이다  
사후확률이 포함하는 봉우리의 개수가 몇 개인지 찾는 것은 어려운 문제다. NP-hard 문제  
  
MCMC  
  
Globally optimal MLE를 찾는다 따라서 MAP 가 된다  
  
11.4 The EM algorithm   
expectation maximization, or EM   
latent 변수를 알지못하기에 Log likelihood의 기대값을 사전확률분포에 취한다  
K-means 알고리즘에서는 유클리디언 거리 함수를 사용하는 반면에 EM 알고리즘은 log-likelihood 함수를   
사용하여 모델의 적합성을 평가한다. K-means가 거리 기반 군집 방법인 것에 비하여 EMdms 확률 기반 군집이라고 한다.   
  
Estep 추정단계  
k개의 확룰 분포에 대하여 각 레코드들이 속할 확률을 계산하여 weight로 변환하여 배정한다  
  
Mstep 최대화단계  
혼합모델의 매개변수들을 업데이트한다  
  
EM알고리즘과 K-means 알고리즘과의 관계  
K-mean 알고리즘은 임의의 데이터셋을 클러스터링 할 때 쓰이는 알고리즘으로, 가우시안  
혼합 모델의 EM 알고리즘과 비슷하다. K-means 알고리즘의 경우, 각 데이터 포인트에 클러스터를 정확히  
1가지 지정하는데 반해 EM 알고리즘은 여러개의 클러스터에 대해 사후확률에 비례하도록 지정한다  
  
Vector quantization   
연속적으로 샘플링된 진폭값들을 그룹핑하여 이 그룹단위를 몇개의 대표값으로 양자화  
음성압축, 영상 압축 부호화기법, 음성인식, 패턴인식 등 여러분야에 사용  
단점 : 매우 낮은 데이터율에서도 높은 효율성을 갖지만, 반면에 양자화 왜곡을 제어하기 어려움  
장점 : 구조가 단순하며 압축률이 높음  
비록 인코딩 과정에서는 속도가 느리나, 디코딩 과정에서는 새인에 의해 코드북을 1회만 참조하게되므로 속도가 빠름  
  
군집화에서 고려해야하는 중요한 문제점  
1\. 최적의 클래스(확률분포모델) 개수는 몇 개인가? ( K의 개수 결정)  
2\. 주어진 데이터에서 가장 근접한 클래스(확률분포모델)은 무엇인가? (클러스터링 과정)  
3\. 오차가 가장 최소가 되는 클래스(확률분포모델)는 무엇인가? (클래스 특징을 변화)위의 3가지 문제를 고려하여 클래스 구성과 최적화 과정을 EM 알고리즘을 통하여 진행한다  
  
11.4.5 EM for the Student distribution \*   
결측치도 없는데 왜 EM 을 사용하는가? 인공적인 hidden or 보조변수로 알고리즘을 단순화하기 위함이다  
Gaussian scale mixture   
Mixtures of Student distributions   
11.4 도표를 보면 Student 모델은 4개의 에러, 가우시안 모델은 21개의 에러  
Class 조건부 확률밀도에 극단값이 포함되어 가우시안 모델이 잘못된 선택을 하기떄문이다  
  
11.4.7 Theoretical basis for EM \*   
11.4.7.1 Expected complete data log likelihood is a lower bound   
Q(θ ,θ ) = logp(xi|θ ) = l(θ ) (11.93)   
  
EM monotonically increases the observed data log likelihood   
  
11.4.8 Online EM   
incremental EM   
각 데이터셋마다 기대되는 충분한 통계가 저장되어있어야한다  
stepwise EM   
확률적 추정이론  
전역적인 최대치를 찾기위해선 한가지 방법으로 deterministic annealing   
  
11.5 Model selection for latent variable models   
클러스터의 갯수 K를 특정짓는것 모델을 고르는것이다  
\* 11.5.1  Model selection for probabilistic models marginal likelihood lvm의 를 측정하기가 어렵다모델의 수가 큰수라 검색할 필요  
\* 11.5.2  Model selection for non-probabilistic methods  
  
11.21 도표에서 c그림은 무얼 의미하는가?  
K를 어떻게 고를수있나  
K별로 reconstruction errorfmf plot 해보면서 알수있다  
  
11.6 Fitting models with missing data