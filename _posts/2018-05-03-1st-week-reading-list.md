---
title: 1st week reading list
categories:
  - ML
tags:
  - bayesian 
  - machine leanrning
last_modified_at: 2018-05-03T09:00:00-00:00
---

**\# Bayesian Learning for hackers Ch1**

[https://github.com/CamDavidsonPilon/Probabilistic-Programming-and-Bayesian-Methods-for-Hackers/blob/master/Chapter1\_Introduction/Ch1\_Introduction\_PyMC3.ipynb](https://github.com/CamDavidsonPilon/Probabilistic-Programming-and-Bayesian-Methods-for-Hackers/blob/master/Chapter1_Introduction/Ch1_Introduction_PyMC3.ipynb)

**\# Machine Learning Murphy** 

##Ch 1.1 - 1.4

다양한 도메인 에서 나타나는 현상은 long tail이다, 작은영역이 일반적이고 대부분의 것들이 희귀한

예를들면 매일 20%의 구글 검색은 지금까지 검색되지 않은 것들이다.

Machine Learning은 보통 2개로 구분되며 예측 or 지도학습 접근법 / 기술적 or 비지도학습 접근법이다

추가로 강화학습이 있다

output yi 의 형태가 범주형이라면 classification or pattern recognition 에 해당하고

real-valued 라면 회귀 문제에 해당한다

지도학습

The need for probabilistic predictions

map estimate (maximum a posteriori) / CTR /  Subset of size 16242 x 100 of the 20-newsgroups data(?) 해당 그림의 정확한 의미는? 

실제세계 application

문서분류 및 이메일 스팸 필터링

input verctor X has a fixed size , bag of a words representation

MNIST : Modified National Institute of Standards 

Face detection and recognition

인식의 문제에선 헤어스타일의 차이가 개체의 identitiy를 결정하는데 중요하지만

감지의 문제에선 별로 중요하지않고 얼굴과 얼굴과 아닌것의 구분에 집중 

matrix completion 은 설문조사에서 응답하지않은 항목이 nan으로 표시되는것을

그럴듯한 값으로 추론하는것을 의미

데이터마이닝은 설명할수있는것을 강조하는반면 머신러닝은 정확도를 높이는 것을 강조한다 

##Ch 2.1 - 2.5

##Ch 6.1 - 6.5


\# Machine Learning 이란 무엇인가?

데이터를 바탕으로 다양한 학습알고리즘을 사용해서 우리에게 필요한 정답, 지식을 추론하는 과정

\- DATA

Fully observed(모든게 명시적인) 

Patially observed(영화추천시스템을 예로 들면 기본데이터를 바탕으로 로맨스,SF  영화장르를 추출)

Actively collected(지속적으로 들어오는 스트리밍 데이터) 로 구성

\- Learning Algorithm

Model based methods

· probablistic models ( 확률모델 )

· parametric models ( 파라메터 모델  파라미터 수가 고정)

· Non parametric models  (파라미터 수가 가변적)

Model free methods

\- Task

Regression (회귀분석)

Classification (분류)

Unsupervised learning (비지도 학습)

\# Bias - Variance decomposition

MSE = Bias^2  + Variance = Total error

![](https://t1.daumcdn.net/cfile/tistory/995DEE4D5AF7E5CF1E)

\# Coin fiips 

HHTTH 

\- 앞면이 나올확률은? 3/5

\- Bernoulli distribution을 사용해서 앞면이 나오는 경우 1, 뒷면이 나오는 경우 0으로 가정

p(X=1) = p (앞면이 나올 확률)

p(X=0) = 1-p (뒷면이 나올 확률)

확률 p는 0과 1사이 

iid 가정을 한다 ( 독립적이고 동일한 분포에서 온 데이터)

독립인 경우 p(X, Y) = p(X) p(Y) 가 성립한다

\- 한개의 동전을 던질때 coin flip 의 확률

![](https://t1.daumcdn.net/cfile/tistory/99AE75405AF7D8EE1A)

p(X=1) = p, p(X=0) = 1-p 성립

\- 여러번 동전을 던질때 coin flip 의 확률

![](https://t1.daumcdn.net/cfile/tistory/9962313C5AF7DAD127)

Nh 는 앞면이 나온 횟수의 합을 나타낸다

log로 변환해서 P에대해 구하면 Nh / N 이 나온다

\# Bayesian Rule

# \- Frequentist (빈도주의자) And Bayesian(베이지안) 의 차이

![](https://t1.daumcdn.net/cfile/tistory/99B34F3B5AF7DE2B32)

 를 보는 관점이 다르다

Bayesian은 frequentist 와 달리 

![](https://t1.daumcdn.net/cfile/tistory/993A45355AF7DBA227)

 를 변화 가능한 값으로 바라본다

반면 D를 고정된 값으로 보아서  D를 확률로 바라보는  frequentist와 차이

![](https://t1.daumcdn.net/cfile/tistory/99EEEA3B5AF7DE0C1B)

![](https://t1.daumcdn.net/cfile/tistory/99FEEA385AF7DE8002)

posterior (사후지식) 은  likelihood 와  prior (사전지식)의 곱으로 나타낼수있다

신인 야구선수의 타율을 계산하는 경우?

처음이라 데이터가 없지만 다른 선수들의 타율이 prior가 되어 평균값에서 출발한다

이후의 추이를 보고 평균값에서 좌우로 조정되는 것

베이지안은 대강의 값이라도 일단 맞춰보고 시작해서 근사한 값을 알수있다

또한 분포를 알수있어서 새로운 모델로 접근이 쉽고 이용할 수 있는 정보도 많다

\# Linear Prediction

The derivation in matrix notation

Starting from 

![](https://t1.daumcdn.net/cfile/tistory/99FB1F3B5AF7E05C11)

 , which is just the same as

![](https://t1.daumcdn.net/cfile/tistory/9982EC495AF7E7120B)

it all comes down to minimizing 

![](https://t1.daumcdn.net/cfile/tistory/99184F415AF7E2A418)

so minimizing ee' gives us:

![](https://t1.daumcdn.net/cfile/tistory/99AE384F5AF7E67707)

![](https://t1.daumcdn.net/cfile/tistory/99C6F63B5AF7E3361F)![](https://t1.daumcdn.net/cfile/tistory/99460B4A5AF7E68801)

\# Linear Prediction

![](https://t1.daumcdn.net/cfile/tistory/9961B23C5AF7E3F815)

 부분이 singular matrix 라면  ex) p >> n 

\-> 역행렬연산이 불가하다

\-> singular matrix 의 경우 고유값이 0

\-> 특정 더미값을 더해서 diagonal  을 0이 아니게 만들어준다면 역행렬 연산이 가능하다!

페널티를 더해서 연산이 가능하게 하는 방법이 Ridge 와 Lasso 와 같은 모델이 된다

![](https://t1.daumcdn.net/cfile/tistory/998A594B5AF7E53F30)

k(x))+E(fk(x))−fk(x))2\]+2E\[(f(x)−fk(x))ϵ\]+σ2\=Var(fk(x))+Bias2(fk(x))+σ2

E\[(Y−fk(x))2\]\=E\[(f(x)+ϵ−fk(x))2\]\=E\[(f(x)−fk(x))2\]+2E\[(f(x)−fk(x))ϵ\]+E\[ϵ2\]\=E\[(f(x)−E(fk(x))+E(fk(x))−fk(x))2\]+2E\[(f(x)−fk(x))ϵ\]+σ2\=Var(fk(x))+Bias2(fk(x))+σ2

E\[(Y−fk(x))2\]\=E\[(f(x)+ϵ−fk(x))2\]\=E\[(f(x)−fk(x))2\]+2E\[(f(x)−fk(x))ϵ\]+E\[ϵ2\]\=E\[(f(x)−E(fk(x))+E(fk(x))−fk(x))2\]+2E\[(f(x)−fk(x))ϵ\]+σ2\=Var(fk(x))+Bias2(fk(x))+σ2

E\[(Y−fk(x))2\]\=E\[(f(x)+ϵ−fk(x))2\]\=E\[(f(x)−fk(x))2\]+2E\[(f(x)−fk(x))ϵ\]+E\[ϵ2\]\=E\[(f(x)−E(fk(x))+E(fk(x))−fk(x))2\]+2E\[(f(x)−fk(x))ϵ\]+σ2\=Var(fk(x))+Bias2(fk(x))+σ2

E\[(Y−fk(x))2\]\=E\[(f(x)+ϵ−fk(x))2\]\=E\[(f(x)−fk(x))2\]+2E\[(f(x)−fk(x))ϵ\]+E\[ϵ2\]\=E\[(f(x)−E(fk(x))+E(fk(x))−fk(x))2\]+2E\[(f(x)−fk(x))ϵ\]+σ2\=Var(fk(x))+Bias2(fk(x))+σ2

E\[(Y−fk(x))2\]\=E\[(f(x)+ϵ−fk(x))2\]\=E\[(f(x)−fk(x))2\]+2E\[(f(x)−fk(x))ϵ\]+E\[ϵ2\]\=E\[(f(x)−E(fk(x))+E(fk(x))−fk(x))2\]+2E\[(f(x)−fk(x))ϵ\]+σ2\=Var(fk(x))+Bias2(fk(x))+σ2