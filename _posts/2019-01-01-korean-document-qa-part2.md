---
title: 한글문서 기반 QA 생성모델 만들기-part2
categories:
  - ML
tags:
  - nlp
  - korean
  - document
  - qa
last_modified_at: 2019-01-01T22:00:00-00:00
---


# Related Work

part1 에서 언급했던 것처럼 QA 문제에 접근한 몇 개의 논문을 살펴보려고 합니다. 논문의 전체내용을 설명하는건 논문을 직접 읽어보는게 더 도움이 될것이라 주요 특징들만 정리해보았습니다

먼저, 우리가 생성하려는 Doc2QA 모델은 크게 2개의 task로 나뉘는데

*   **Question Answering** (QA): infer an answer given a question and the context;
*   **Question Generation** (QG): infer a question given an answer and the context.

첫번째 논문인 QANET 은 QA 부분을, 두번째 논문인 Learning to Ask 는 QG 부분을 나타낸다고 할수 있습니다.

**첫번째 논문은** [**QANET: COMBINING LOCAL CONVOLUTION WITH GLOBAL SELF-ATTENTION FOR READING COMPRE- HENSION**](https://arxiv.org/abs/1804.09541) **입니다**

#3가지 주요아이디어  
RNN 없이 NLP 모델에서 가장 깊은 130 층 구조  
Transfer Learning  
back-translation 을 통한 Data Augmentation

기계독해 모델에 주로 사용되던 RNN 대신 CNN 과 Global self-attention을 사용했습니다. RNN 은 연속된 시퀀스 데이터를 사용하기에 병렬로 처리가 힘들고 따라서 모델을 훈련시키는데 시간이 많이 소요됩니다.

이 논문의 목표는 RNN을 제거하고 CNN으로 그 자리를 채워 같은 성능의 모델을 더 빠른시간안에 수행하려고 합니다. CNN이 문맥의 지역적 특징들을 포착한다면 self-attention은 단어들 사이의 전역적 상호관계를 학습하기에 두가지 특성 조합은 좋은 성능을 가져올 것입니다. RNN 과 비교했을 때, 9-13배 정도의 속도를 보이면서 말입니다. 또한 데이터 증식기법을 사용해서 사용한 데이터 셋의 크기를 늘렸습니다

Fig1. QANet 모델 구조

왼쪽에는 몇몇 인코더 블록들이 보이고, 한 개의 인코더 블록을 확대한 것이 오른쪽 그림입니다. Context-Query Attention에서 가중치를 공유하고 이를 세 개의 출력 인코더 블록으로 보냅니다. 오른쪽 그림의 Position 인코딩은 입력값의 처음에 추가되어 각 Encoder 층의 sin, cos 함수를 구성합니다. 이후에는 한개의 Convolution, self-attention 또는 feed-forward 층으로 이루어져있습니다

또 다른 특성은 Backtranslation 을 사용해서 데이터 증식을 시도했다는 점입니다. 두 개의 번역 모델, 하나는 영어에서 프랑스어 그리고 또 하나는 프랑스어에서 영어로 번역하는 모델입니다. 이러한 방법은 훈련 데이터의 양을 증가시킬 수 있습니다. 더 많은 데이터를 가지고 더 좋은 모델을 만들 수 있습니다.

*   SQuAD 데이터셋 / NLTK 토크나이저 / 300차원의 Glove 임베딩을 사용
*   다른 모델들과의 속도 비교  
    BiDAF와 동일한 F1 스코어를 내기위해서 1/5 정도 시간소요
*   데이터 증식은 + data augmentation ×3 (3:1:1) 의 경우 가장 높은 성능  
    기존 크기의 3배, 괄호안은 샘플링 비율 ( 원본:영어-프랑스어-영어:영어-독일어-영어)
*   더 도전적인 데이터셋인 TriviaQA (길이가 길고, 라벨링이 되어있지않은 데이터가 더 많고, 문맥과 관련없는 정답포함) 로 실험한 결과에서도 기존 모델보다 더 좋은 성능을 보임

**두번째 논문은** [**Learning to Ask: Neural Question Generation for Reading Comprehension**](https://arxiv.org/abs/1705.00106) **입니다**

어텐션 기반의 시퀀스 학습모델을 소개하고. 문장 vs 단락 수준 정보의 효과를 비교합니다. 이전 연구들이 규칙기반이거나 복잡한 NLP 파이프라인을 따랐다면 end-to-end 로 훈련을 시키는 seq2seq 학습방법을 사용했습니다.

Question generation(QG)는 문장 또는 단락단위 정보가 주어졌을때 자연스런질문을 생성하는 것이 목표입니다. QG 의 task 를 다시 정의해본다면 아래와 같습니다

**입력 문장 x 가 주어졌을때, 자연스런 질문 y (문장과 연관된 정보) 를 생성**

y 는 임의 길이의 시퀀스 \[y1, …, yy\]  
입력 문장의 길이를 M으로 가정, x는 시퀀스의 토큰으로 표기 \[x1, …, xm\]  
QG는 Y’를 찾는 게 목표가 됩니다

Fig1. The QG task

RNN 인코더-디코더 구조를 사용하여 조건부 확률로 모델링 했습니다. global 어텐션을 적용해서 각 단어를 디코딩하는 동안 모델이 입력값의 특정요소에 포커싱할수 있게 했습니다

4.1 Decoder

Fig2. decoder probability

yt는 입력문장 x와 이전에 생성된 (y < t) 단어들을 기반으로 예측된 값을 나타내는데 좀더 상세히 보면 ht 는 t 시점의 RNN의 상태값, ct 는 디코딩 t 시점 인코딩된 x에 생성되는 어텐션을 나타냅니다  
기본 모델은 문장기반이나 단락 수준의 모델에선 Y 모양의 네트워크 구조를 보입니다. 문장과 단락 단위 정보가 2개의 RNN branch로 인코딩되는

4.2 Encoder

문장을 인코딩하는데 bidirectional LSTM을 사용합니다. 아래 수식의 bt는 t시점의 화살표 방향(왼쪽에서 오른쪽)의 hidden state 를 나타내고 반대방향 bt도 있습니다.

Fig3. sentence encoder

디코딩 t시점에 어텐션 기반의 인코딩 값 x는 ct로 나타내고 가중치 평균으로 (bt에 a를 곱함) 구해집니다.

Fig4. context dependent token

4.3 Training and Inference

문장-질문 쌍의 훈련 말뭉치가 주어지면 세타로 나타내어지는 모든 파라미터에 대해 훈련 데이터의 negative log-likelihood 값을 최소화 시키는 것이 모델의 훈련 목표입니다.

Fig5. negative log-likelihood of the training data

입력 문장에서 단어사전에는 없는 희소한 단어들이 발생할 수 있습니다. 따라서 UNK 를 적절히 교체하는 후처리 작업이 필요합니다. t 시점에 UNK를 디코딩 할 때, 입력문장에서 가장 높은 어텐션 스코어를 가지는 단어로 교체하는 방법이 있습니다.

*   SQuAD 데이터셋 / 토크나이징 및 문장 구분 Stanford CoreNLP 사용
*   training (80%)/ development (10%)/ test set (10%)  
    문장의 평균 토큰수는 32.9 개 — 질문의 평균 토큰수는 11.3 개
*   evaluation 스크립트를 사용 BLEU1,2,3,4 , METEOR and ROUGE 측정
*   단락정보를 인코딩했을 때 “w/ paragraph” 카테고리의 질문에서 가장 높은 성능을 보이는 것은 질문이 해당 문장 주변의 정보들까지 고려했을 때 효율적인 것을 증명한다 (모든 부분에서는 아님)

다음 시리즈에서는 영어모델에 한국어를 어떻게 적용할 것인지 그리고 데이터셋으로 사용되는 SQuAD에 대한 이야기를 해보려고 합니다.  
그럼 다음시간까지 안녕!
