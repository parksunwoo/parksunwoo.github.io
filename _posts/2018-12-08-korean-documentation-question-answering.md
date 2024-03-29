---
title: 한글문서 기반 QA 생성모델 만들기\_part1
categories:
  - ML
tags:
  - squad
  - nlp
  - question-answering
  - korean
last_modified_at: 2018-12-08T09:00:00-00:00
---



한글문서 기반 QA 생성모델 만들기\_part1
==========================

Introduction (feat. Dok2.QA)

한국어 기반의 문서가 주어졌을 때 문서에 대한 질문과 그에 맞는 답을 추출하는 Question Answering (QA) 모델을 만들고자 합니다. 질문이 주어지고 그에 맞는 답을 하는 기존 방식에서 한단계 나아가 최초 질문 생성까지 자동으로 해보겠다는 겁니다.

예를들어, 코모도왕도마뱀에 대한 위키문서가 주어졌다고 해봅시다. 위키문서가 입력값으로 모델에 들어가면 그에 맞춰 질문-답 을 출력하는 겁니다.

figure1. 멸종위기 코모도왕도마뱀

Q- 코모도왕도마뱀의 몸 길이는 몇 미터인가요?  
A- 코모도왕도마뱀은 현생 도마뱀 가운데 가장 커서 다 자라면 몸 길이 3m에 달합니다.

이러한 모델을 생성하면 어떤 부분에 사용할 수 있을까요? 현재 대부분의 챗봇은 사람들이 주어진 글 ( 고객 상담내용, 유저 가이드 문서) 을 읽고 고객의 질문은 무엇이고 또 상담원의 답변은 어떤 부분이었는지를 맵핑하는 작업을 하고 있습니다. 이런 가공된 데이터 쌍을 챗봇에 입력해주어야 챗봇이 제 기능을 할 수 있습니다. 우리가 QA모델을 만든다면 이런 길고 지루한 작업에서 벗어날 수 있을겁니다. 또 다른 부분에도 적용할수 있겠죠? 예를 들면, 뉴스 기사가 주어지면 기사 내용을 바탕으로 독자의 예상 질문과 준비된 답을 만들 수 있을 겁니다. 활용할 수 있는 분야는 상상할 수 있는 모든것이라고 할 수 있겠네요.

몇 가지 고민해봐야할 부분이 있습니다. 먼저 데이터셋입니다. 현재까지 영문기반으로는 이러한 시도가 계속 되었고 모델 생성에 있어 가장 중요한 데이터의 양도 많이 확보되어있습니다.

figure2. [https://rajpurkar.github.io/SQuAD-explorer/](https://rajpurkar.github.io/SQuAD-explorer/)

**S**tanford **Qu**estion **A**nswering **D**ataset (SQuAD) 는 위키피디아 문서를 바탕으로 크라우드소싱 인력들이 해당하는 질문과 답변을 생성했습니다. 현재 2.0 버전이 나와있고 100,000 개의 질문이 있습니다. squad 페이지의 리더보드에는 다양한 모델들로 높은 성능을 내려는 시도를 볼 수 있습니다.  
반면 한글은 이런 데이터셋이 매우 부족한 상황입니다. ETRI 공개 말뭉치 목록에 번역된 SQuAD 한국어 QA 데이터셋이 있고 추가로 확보한 K-QuAD 데이터셋이 있어 우선 이를 활용해 보려합니다.

그리고 질문과 답의 순서입니다.  
1)질문을 먼저 만들고 그에 맞는 답을 찾을지  
2) 답을 먼저 찾고 그에 맞는 질문을 만들지 정해야합니다.  
마치 닭이 먼저냐? 달걀이 먼저냐? 같은 질문과 비슷해보입니다.

순서에서 볼 수 있는 것처럼 QA모델은 크게 2부분으로 나뉩니다. 문서에서 질문을 찾거나 생성하는 Question Generation (QG) 부분과 질문에 맞는 답을 찾는 Question Answer (QA) 부분입니다. 방향을 잡은 것은 주어진 문서를 요약하는 방식으로 중요한 문장들을 뽑아내어 이를 답이 되는 문장으로 먼저 사용(QA)하고, 중요 문장을 질문생성 모델(QG)에 넣어 질문을 출력하려고 합니다.

모델을 잘 만들었다는 걸 어떻게 알수있을까요? 여기서 평가지표를 어떻게 선정하는지가 중요합니다. 아무리 모델을 잘 만들었어도 그게 어느정도 좋은지를 모른다면 아무런 소용이 없을겁니다. QA모델에 빗대어 이야기한다면 출력값으로 나온 질문과 답변이 주어진 문서에서 나올수 있고 또 적절한지를 판단해야할 것입니다. 그리고 질문과 답변이 서로 연관이 있는지도요.  
이전 이야기했던 SQuAD 리더보드에선 EM 과 F1 스코어로 모델간 순위를 매기고 있습니다.

F1과 Exact Match(EM) 은 모델의 정확성을 측정하는 지표들입니다.  
F1 measures the portion of overlap tokens between the predicted answer and groundtruth, while exact match score is 1 if the prediction is exactly the same as groundtruth or 0 otherwise.

간단하지만 향후 모델을 개발하는데 있어서 중요한 baseline을 만들어 보았습니다. 먼저 영문기반으로 작성해보았고 textrank 알고리즘을 사용해서 문서(위키피디아 국/영문 조선)를 요약해서 중요문장을 추출했습니다. 추출된 중요문장의 part of speech (pos, 형태소품사) 태그를 분석해서 what 형태의 의문문을 생성하는 방식입니다.

최초 baseline 을 사용해서 나온 결과는 아래와 같습니다

Question: What has Joseon? / 조선은 무엇입니까?  
Question: What creation is was? / 어떤 창조물이 있었습니까?  
Question: What The Joseon expressing? / 조선은 무엇을 표현 하는가?  
Question: What marks Joseon? / 조선은 무엇인가?  
Question: What is Samsa? / Samsa 란 무엇입니까?  
Question: What is Office? / Office 란 무엇입니까?

조선은 무엇입니까? 같은 의미심장한 질문이 나오기도 했는데요. 질문을 비교적 잘 생성하는 걸 알 수 있습니다.

그럼 동일한 내용의 위키피디아 한글 문서를 사용하면 어떤 결과가 나올까요?

문서요약을 3문장정도로 하는 소스를 사용했을때 입력값이 작아서인지 질문을 잘 생성하지 못했습니다. 그래서 동일한 textrank 알고리즘이나 좀더 길게 요약하는 모델을 사용하였고 아래와 같은 문장들이 나왔습니다.

```
조선(朝鮮, 중세 한국어:  리조조선, 리씨조선, 약칭: \[5\] 1393년 2월 15일에는 국명을 “조선(朝鮮)”으로 정하였고\[6\], 1394년에는 한양을 도읍으로 하여 여러 개혁을 단행했다.  성종은 개국 이후의 문물 제도를 정비했다.  15세기 말부터 지방의 사림 세력이 정계에서 세력을 키우기 시작했다... (중략)
```

소스를 자세히 보시면 아시겠지만 pos 태그를 분석해서 질문을 생성하는 부분에서 what -> 무엇정도로만 간단히 변경했고 최종적으로 아래 질문들이 나왔습니다.

Question: 무엇 것은 국왕을?  
Question: 무엇 지금의 태종?  
Question: 무엇 조선 명종이?

질문의 상태가 아직은 만족할 수준이 아닙니다. 개선할 부분이 많다는건 즐거운 일이죠. 다음 부분에선 여러 참고할만한 논문들을 찾아 사용한 모델구조와 특이한 점들을 정리해보고 우리 모델에 적용해볼만한 부분을 찾아보려 합니다. 다음에 또 만나요!
