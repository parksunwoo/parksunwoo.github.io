---
title: 한글문서 기반 QA 생성모델 만들기-part2
categories:
  - ML
tags:
  - nlp
  - korean
  - document
  - daanet
last_modified_at: 2019-03-01T22:00:00-00:00
---


# Dual Ask-Answer Network for Machine Reading Comprehension

다른 측면에서 QA&QG 모델에 접근한 논문을 찾아서 현재 진행중인 QA모델에 적용해볼만한 부분이 있다고 생각했습니다. 전체적인 내용을 정리해보는 시간을 가져보려 합니다.

기계독해에는 질문, 답변 그리고 문맥 이렇게 3가지 요소가 있습니다.  
question answering / question generation 의 목표는 문맥이 주어졌을때 (질문을 통해 답변을/답변을 통해 질문을) 추론하는 것입니다. 훈련되는 동안 모델은 질문-문맥-답변을 입력값으로 받아 계층적 어텐션 과정을 통해 cross modal interaction 을 감지합니다.

— cross modal interaction : 서로 다른 감각을 통해 인지하는 과정. 여기에선  
(1) 질문&문맥-> 답변  
(2) 답변&문맥->질문  
(1), (2) 모두 활용하여 정보를 추론하는 것을 말함

**Introduction**

figure1. question answering and question generation of three learning paradigms

위 그림은 QA모델과 QG모델의 패러다임 변화에 대한 그림입니다. 먼저 a그림은 QA와 QG 모델이 서로 구분되어있습니다. 입출력되는 데이터부분에서도 모델자체 부분에서도. b그림은 데이터부분에선 구분되어있으나 모델부분은 통합되어 있습니다. 입력되는 값이 변화함에 따라 출력값은 달라지지만 모델 자체는 하나 이기때문에 QA와 QG를 큰 틀에서는 같은 task로 바라보았다는 점이 특징입니다. c그림은 이번 논문의 모델구조를 나타내고 있습니다. 구분되어있던 부분이 완전히 사라지고 입력값에는 질문-문맥-답변이 출력값에는 답변과 질문이 함께 있습니다. 이 모델구조에서는 QA와 QG가 동시에 수행되고 있습니다.

figure2. model architecture of Dual-Ask-Answer Network

계층구조는 크게 4개부분으로 구성되어있습니다. 임베딩, 인코딩, 어텐션, 출력값. 검은색 화살표 영역이 QG모델부분, 빨간색 화살표 영역이 QA 모델부분을 나타냅니다. 색칠 된 부분 (인코딩 부분, 포인터 생성)은 QA모델과 QG모델에 공통적으로 사용되는 부분입니다.

**Dual Ask-Answer Network**

*   Problem Formulation  
    문맥은 C:={c\_1,…,c\_n}, 질문문장은 Q:={q\_1,…,q\_m}, 답변문장은 A:={a\_1,…,a\_k} 라고 하면  
    C가 주어졌을때, QA task 라 함은 Q를 기반으로 A를 찾는 것으로 정의  
    C가 주어졌을때, QG task 라 함은 A를 기반으로 Q를 찾는 것으로 정의합니다. 이전 기계독해 모델들이 A를 C의 일부 영역(시작위치와 끝위치)를 찾는 것으로 간주하였지만 이 논문에서는 QA와 QG를 생성의 문제로 간주하여 시퀀스모델로 해결하고자 합니다
*   Model Description

Embedding Layer : 임베딩에서는 문맥, 질문, 답변 모두에서 파라미터가 공유됩니다. 단어 임베딩에선 pretrain된 256차원의 Glove 벡터가 사용, 훈련되는 동안 고정됩니다. 범위를 벗어난 단어는 <UNK>토큰으로 맵핑됩니다. 캐릭터 임베딩에선 200 차원의 훈련가능한 벡터로 나타내어집니다. 시퀀스 길이는 16으로 맞추고 최종적으로는 캐릭터 임베딩과 단어 임베딩을 연결후 300차원의 공간으로 만듭니다

figure3. the architecture of a context/question/answer encoder

Encoding Layer : 인코딩은 문맥,질문, 단어 인코더를 포함합니다. 파라미터를 공유하는 것은 훈련할때에 정규화 효과가 있으며 또한 일반적이고 견고한 모델을 만드는데 도움이 됩니다. 그림3은 각 인코더를 구성하는 가장 기본적인 블록을 나타냅니다. 질문&답변 인코더에선 LSTM 이 단방향이고 self-attention 이 사용될때 forward-mask가 사용됩니다.(현재 task에서 질문과 정답을 찾는 것이 target이 되고 남겨진 정보들에서 target 값이 누출되는 것을 막기위함) 문맥 인코더에선 feed-forward 네트워크와 3개로 쌓인 양방향 LSTM 그리고 4 head self attention이 사용됩니다.

figure4. attention layer

Attention Layer : QG를 기준으로(왼쪽) t 시점에 먼저 인코딩된 답변을 인코딩된 문맥에 fold 시킵니다. 그 결과값이 C\_a로 표기되고 t-1시점까지 생성된 Q의 인코딩값에 두번째로 fold 됩니다. 자세히 설명하면 문맥-답변의 어텐션은 문맥과 답변 단어의 표준화된 유사도를 계산하는 것입니다.

eq. (1)

eq(1)의 C, A는 d x d matrix로 가정, U, V는 column vector이고 CU, AV 2개의 d 차원을 dot product 하면 스칼라 벡터가 구해집니다. 이러한 어텐션 스코어 함수를 multiplicative attention 또는 dot-product attention이라고도 합니다.

eq. (2)

eq(2) 문맥-답변 어텐션은 eq(1)에서 계산된 값에 답변들의 가중합으로 계산됩니다. 첫번째 fold-in 과정에서 나온 출력값 C\_a는 n x d 행렬이고 문맥 시퀀스와 동일한 크기입니다. 질문-문맥 그리고 답변-문맥 어텐션을 계산하는 두번째 fold-in 과정은 의미있는 질문과 답변을 생성하는데 필수적인 과정입니다. 이 부분에서 첫번째 fold-in 에서 나온 문맥 시퀀스의 모든 부분과 이전까지 생성된 시퀀스의 모든 부분이 서로 어텐션할 수 있습니다.

eq. (3)

t시점에서 질문-문맥 어텐션의 입력값은 이전 인코딩 레이어에서 나온 Q<t와 문맥-답변 어텐션에서 나온 C\_a이고 최종출력값은 eq(3)과 같습니다.

figure5. the architecture of the output layer

Output Layer : 출력부분에선 매 t마다 한개의 단어를 생성합니다. 모델에서 생성된 이전값이 다시 모델에 사용되어 다음 값을 생성하는 auto-regressive한 성격을 가집니다. 포인터 생성부분이 출력부분에서 핵심인데 문맥에서 단어를 복사하거나 미리 구축된 단어사전에서 단어를 샘플링하는 역할을 합니다.  
이것은 특히 QA 모델부분에서 문맥의 단어범위를 벗어나는 다른 단어가 출현할수 있게 합니다. 최종적으로 출력되는 값은 두개의 분포가 람다t 와 (1-람다t)와 각각 곱해진 값을 혼합한 분포가 됩니다.

eq. (4)

eq(4)에서 W\_shared와 b\_shared는 QA, QG모두에서 공유되는 파라미터이고  
반면 W\_1, W\_2, b\_1, b\_2 는 QA, QG 각 모델에 특화된 파라미터입니다.

P\_vocab 은 사전에 정의된 단어의 분포를 P\_context 는 어텐션 스코어에 기반한 현재 문맥에 있는 중복된 단어가 제거된 분포를 나타냅니다.  
이렇게 공유 부분이 있는 것의 장점을 꼽자면 먼저 파라미터 수가 급격히 감소하고 어텐션 층에서 cross-modal 효과가 더 높아지는 효과가 있습니다. 두번째로 모델은 자신의 공간에서 디코딩 된 정보를 공통의 잠재 공간으로 투영한 다음 어휘 공간에 투영합니다.

eq. (5)

*   Loss Function: 손실함수 부분은 2개부분으로 구성되어있는데 — eq(5)에선 QG부분 손실함수만 표시 — 시퀀스 변환 모델에서 흔히 사용되는 negative log likelihood loss와 동일한 텍스트가 반복해서 생성되는 걸 막기위한 coverage loss 로 이루어져 있습니다. t시점 이전에 생성된 Q와 A,C가 주어졌을때 예측된 q\_t의 값과 실제값과의 비교부분이 negative log likelihood loss 가 됩니다. Q,A,C 가 주어졌을때 모든 모델 파라미터에서 eq(5)의 손실함수 가 최소화되는 값을 찾는것이 목표입니다.

**Experimental Results**

논문에선 데이터셋으로 SQuAD, MSMARCO, WikiQA, TriviaQA를 사용,  
평가지표는 BLEU-1,2,3,4, Meteor, ROUGE-L 을 사용했습니다.

**Conclusion**

논문에서 제안하는 Dual Ask -Answer Network는 양방향 생성 모델로 질문생성과 질문답변 모델을 해결하는 모델입니다. 공유부분을 통해 기존 기계독해부분에서 더 나은 성능을 보였습니다. 나아가서 생성된 질문과 답변을 상위 모델로 보내는 방식과 같이 DAANet을 여러개 쌓는 실험을 해본다면  
데이터 부족이 심한 영역에서 효과적으로 적용될 수 있을 것입니다.
