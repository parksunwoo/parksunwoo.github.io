---
title: Learning Structured Output Representation using Deep Conditional Generative Models
categories:
  - ML
tags:
  - slowpaper
  - cgm
  - cvae
last_modified_at: 2019-05-21T22:00:00-00:00
---

# slow paper
=====================================================================

VAE 관련 설명은 해당 [슬라이드](https://www.slideshare.net/ssuser06e0c5/variational-autoencoder-76552518) 와 [블로그](https://ratsgo.github.io/generative%20model/2018/01/27/VAE/) 를 참고했습니다.

지도학습은 많은 인식문제에서 성공적이었습니다. 많은 훈련데이터가 제공되었을때 복잡한 함수를 근사할 수는 있지만 확률론적 추론을 수행하고 다양한 예측을 하는 구조화된 출력 표현 모델링을 하는데는 어려움이 있습니다.

이 논문에서는 가우시안 잠재변수를 사용해서 구조화된 출력 예측을 위한 생성 모델을 만들었습니다. 확률적 feed-foward 추정을 사용하여 빠른 예측이 가능하고 입력값에 노이즈 주입, 다중 스케일 예측과 같은 기법을 사용했습니다.

**Structured prediction 이란?**

[

What is structured prediction?
------------------------------

### (1) there are far too many possible values for y (exponential or infinite) (2) however, these values are not opaque…

www.quora.com

](https://www.quora.com/What-is-structured-prediction)

> 구조화된 예측. 출력 변수가 상호 의존적이거나 제약되는 분류 또는 회귀 문제를 해결하기 위한 프레임웍입니다. 이러한 종속성과 제약 조건은 문제 영역에서 순차적, 공간적 또는 조합적 구조를 반영하며 이러한 상호 작용을 캡쳐하는 것은 입출력 종속성을 캡쳐하는 것만큼이나 중요합니다.

구조화 예측은 분류에 반대되는 용어로 정의됩니다. 분류문제에서는 여러 입력값들이 한개의 값으로 맵핑되나 여기선 하나의 입력값으로 가능한 다양한 출력값을 맵핑하는 것을 말합니다. 일부 입력 데이터 x가 주어진 경우 해당 데이터와 관련된 최상의 구조를 찾는게 목표입니다. 이는 문제를 해결하기 위해 일종의 검색이 필요하다는 것을 의미하기도 합니다.

논문에서 제안하는 conditional variational auto-encoder는 기존 VAE 구조를 지도학습이 가능하도록 인코더와 디코더에 정답 레이블 y를 추가한 형태로 바꾸었습니다.

**3\. Preliminary : Variational Auto — encoder**

생성모델은 A/B를 판별하기보다 범위를 고려하는 것입니다. 생성모델을 통해 우리가 하고 싶은 것은 이미지와 같은 고차원 데이터 X의 저차원 표현 z 를 구할 수 있다면 z 를 조정하여 훈련 데이터셋에서 주어지지 않은 새로운 이미지 생성을 시도해보는 것입니다. 기존 생성모델의 문제점은 데이터 구조에 대한 가정과 모델에 대한 근사가 필요하고 시간이 많이 소요되었습니다.

잠재변수는 다차원의 정규분포로 가정합니다. 입력값으로 잠재변수 분포를 생성하고 이 잠재변수로부터 샘플링을 해서 입력에 가까운 출력을 생성합니다. 잠재변수 z와 VAE 아키텍쳐 관점에서 보면, 인코더는 입력 데이터를 추상화하여 잠재적인 특징을 추출하는 역할, 디코더는 이러한 잠재적인 특징을 바탕으로 원 데이터로 복원하는 역할을 한다고 해석해볼 수 있습니다.

eq(1)

위 식 우변 첫번째 항은 KL Divergence Regularizer에 해당합니다. p(z)와 q(z|x)의 정보적 거리를 의미하는 정규화항입니다. 우변 두번째 항은 reconstruction loss에 해당합니다. 인코더가 데이터 x를 입력값으로 받아 q로부터 z를 뽑습니다. 디코더는 인코더가 만든 z를 입력받아 원 데이터 x를 복원합니다. 이는 현재 샘플된 z에 대한 negative log likelihood, 즉 x에 대한 복원오차를 의미합니다.

**4\. Deep Conditional Generative Models for Structured Output Prediction**

fig1. illustration of the conditional graphical models

4.1 Output inference and estimation of the conditional likelihood  
Conditional Generative Model **(**CGM)은 입력값 X 와 출력값 Y 그리고 잠재변수 Z로 구성되어 있습니다. (a)그림은 일반적인 CNN 모델을 나타내고,  
(b) 그림은 CGM에서도 생성부분을 나타내고 있습니다. 설명을 더하면 입력값 X가 주어질때 잠재변수 Z가 출력값 Y의 조건부 분포 모델링을 제어합니다. CGM 에 적합한 일대다 맵핑을 제안하고 잠재변수가 입력값에 독립적이라는 제약이 있다면 P (z|x) = P (z) 가 성립합니다.  
(d) 그림은 recurrent connection을 나타내는 것으로 직접 입력되는 x값 뿐 아니라 CNN에 의해 초기 추측된 값 ŷ 이 prior 네트워크에 함께 들어갑니다. 컨볼루션 네트워크가 깊어지는 동안 이전 추측값이 계속 수정되어 예측값은 연속적으로 업데이트 됩니다.

4.2 Learning to predict structured output  
CVAE는 훈련하는동안 인식 recognition 네트워크 q(z|x,y)를 사용합니다. 그러나 테스트시에는 사전 prior 네트워크 p(z|x)를 사용해서 표본 z를 추출하고 출력에 대한 예측값을 생성합니다. 인식 네트워크를 위한 입력값으로 y가 주어지고 훈련의 목적은 y를 다시 복원하는 것과 같습니다. 여기선 네트워크 훈련방식으로 train 과 test에서 동일한 예측 파이프라인을 사용, 인식 네트워크와 사전 네트워크를 동일하게 설정합니다. 이러한 모델을 Gaussian stochastic neural network(GSNN) 이라고 합니다. 우리는 GSNN과 CVAE를 결합한 모델을 제안합니다.

4.3 CVAE for image segmentation and labeling  
영상분할은 구조화예측에서 중요한 태스크입니다. 우리가 제안하는 2가지는 멀티 스케일 prediction objective 와 구조화된 입력 노이즈입니다. 이미지 크기가 커지면 커질수록 픽셀수준에서 세밀한 예측이 점점 어려워집니다. 따라서 멀티 스케일 접근방식은 각기 다른 스케일 수준 (1/4, 1/2, 원본크기) 출력값에 따른 네트워크를 훈련합니다.

뉴런에 노이즈를 추가하는 건 딥 뉴럴네트워크를 훈련시킬때 정규화를 위한방법입니다. 입력 데이터 x에 노이즈를 더한값을 x̄라고 하면 네트워크를 최적화시키는 목적함수 L= (x̄, y)입니다. 노이즈를 추가할때는 랜덤하게 squard mask를 하는데 이는 이미지의 너비와 길이 전체의 40% 미만으로 적용시켰습니다.

위 그림은 토이예제로 MNIST를 사용한 실험결과 입니다. 왼쪽 그림은 3사분면만을 입력값으로, 오른쪽 그림은 2,3 사분면을 입력값으로 넣은 것입니다. 베이스라인으로 주어진 NN은 하나의 예측만 할수 있으며 출력이 대부분 흐리게 보입니다. 이와 비교적으로 CVAE는 다양한 샘플들을 볼 수 있습니다. 때로는 실제값과 다르게 변경되는 경우도 보이지만 좀 더 뚜렷한 모습을 보입니다.
