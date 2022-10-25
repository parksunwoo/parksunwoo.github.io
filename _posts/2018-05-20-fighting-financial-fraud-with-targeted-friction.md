---
title: Fighting Financial Fraud with Targeted Friction
categories:
  - ML
tags:
  - a/b test 
  - roc
  - auc
last_modified_at: 2018-05-20T09:00:00-00:00
---

Airbnb는 어떻게 일반게스트들에게 미치는 영향을 최소화하면서 사기범을 식별하는가?

Airbnb Data Science Blog에서 [Fighting Financial Fraud with Targeted Friction](https://medium.com/airbnb-engineering/fighting-financial-fraud-with-targeted-friction-82d950d8900e) 포스트 내용을 정리해보았습니다.

Figure1. **Malibu Dream Airstream**

하루밤에만 무려 전세계 191개국 200만명의 사람들이 이용중인 Airbnb.  
이러한 글로벌 커뮤니티의 급속한 성장은 신뢰를 바탕으로 합니다. 이번글에선 신뢰와 관련, 사기범이 도난당한 신용 카드를 사용하지 못하도록 사전에 수행하고 있는 업무에 대해 설명하려고 합니다.

우선 ML모델을 사용해서 감지된 사기범에게 frictions을 유발합니다  
loss function을 최소화하여 모델의 임계값을 선택하는 방법, loss function의 각 부분을 살펴보고 실제예제를 통해 어느정도 효과를 보였는지 살펴보려고합니다

What We’re Fighting: Chargebacks
--------------------------------

Figure2. **Chargebacks (**[https://www.heropay.com/glossary/chargeback-period/](https://www.heropay.com/glossary/chargeback-period/)**)**

Chargebacks 은 카드주인이 카드를 도난 당했을때, 본인이 사용하지 않은 거래를 신고하면 카드회사에서 지불거절을 발행하는것 말합니다. 판매자-여기선 Airbnb-는 카드사로부터 받은 돈을 반환해야합니다. Chargebacks을 막기위해 과거 예약에서 검증된 사기/일반건들로 train된 ML모델을 사용합니다.  
frictions은 사기범을 막기위한 필터링 장치로 이해하면 좋습니다. 실제로 사용할 권한을 가진 user인지 확인하는 방법으로 3-D Secure/청구명세서 검증 등이 있습니다

loss function를 보기전에 **사기범을 식별한다는 것**에 대해 생각해볼 필요가 있습니다.

[

톰과 제리 - Google Search
---------------------

### Edit description

www.google.co.kr

](https://www.google.co.kr/search?q=%ED%86%B0%EA%B3%BC+%EC%A0%9C%EB%A6%AC&source=lnms&tbm=isch&sa=X&ved=0ahUKEwjiiJ-NppLbAhXKnJQKHZSUBx8Q_AUICigB&biw=1058&bih=768)

어릴적 즐겨보던 톰과제리 만화에서 톰과 제리를 예로 들어보겠습니다  
True Positive(참된 긍정) : 톰이 제리를 잡는 경우엔 별 문제가 없어보인다  
(이 경우에도 제리가 빠져나갈 확률은 있지만)  
False Positive(잘못된 긍정) : 톰이 제리인줄 알고 잡았는데 스파이크불독을 잡은 경우 (톰에게 문제가 생길것같다)  
False Negative (잘못된 부정) : 톰이 불독으로 변장한 제리를 못알아보고 놓친경우

제리를 사기범으로, 스파이크불독을 일반유저로 바꾸어 생각해보면  
Airbnb가 부담해야할 손실(비용) 역시 크게는  
“TP의 비용 + FP의 비용 + FN의 비용의 합”일 것입니다

Formula1. loss function _L_

우리의 목표는 위의 식인 전체손실함수 L을 최소화하는 것입니다

Cost of a False Positive
------------------------

**일반사용자에게** frictions**을 잘못 적용하면 Airbnb 를 사용하지 않을 가능성**

FP : 잘못된 긍정의 수  
G : frictions에 노출된 일반 사용자가 거래에서 나갈확률  
V : 양호한 사용자 수명 값 (게스트 평생 사용가치 추정)

양호한 사용자 마찰손실률 G를 구하기 위해 AB테스트를 시행합니다  
AB테스트는 특정 이벤트의 성과를 측정하기 위해 임의 두 집단으로 나누고 두 집단 중 어떤 집단이 더 높은 성과를 보이는지 측정하는 테스팅 기법입니다  
사기발생은 극히 드문 이벤트로 일반유저가 frictions에 적용되는 수를 최소화하기위해 95% control (no friction) / 5% treatment (friction) 정도의 높은 불균형 할당 테스트를 진행합니다  
이때 G _\=_ 1 — ( frictions  그룹의  성공률 ) / ( 대조군의  성공률 ) 입니다

Cost of a False Negative
------------------------

**적발하지 못한 사기사건 비용**

_FN :_ 잘못된 부정의 횟수  
_C :_ 사기 사건의 비용 (사기꾼 지불한 금액, 증가된 카드거절비율 등 간접비)

Cost of a True Positive
-----------------------

**사기꾼이 어떻게든 마찰을 통과하면 손실로 연결**

TP : 참된 긍정의 횟수  
_F_ : frictions에 의해 공격을 받았을 때의 사기범 dropout rate  
_C :_ 사기 사건의 비용

모델이 임계 값 이상의 점수를 가진 사기범을 식별하고 frictions이 사기범을 성공적으로 막는다면 손실은 없습니다. 그러나 frictions 까지 통과한 -제리같은- 사기범들도 분명 있습니다. 사기범에게 마찰을 가하지  않으면  매우 비용이 높기때문에 F에 적용되는 불균형은 G를 구할때와는 반대입니다.

Figure3. Schematic diagram of Good user dropout and Fraudster dropout experiments

실제로 금융기관에서 부정거래를 탐지할때 거래나 이벤트를 철저히 부인하거나 차단하며 보통 이때 _F_ = 1 및 _G_ = 1 인 마찰로 간주 합니다.

Example: Comparing Blocking a Transaction versus Applying a Friction
--------------------------------------------------------------------

Figure 4: Example ROC curve of a ML model predicting P(fraud)

**서로 반비례적인 관계를 가진** 결과 예측의 테스트를 평가하기 위하여 흔히 두 가지 지표, 민감도(sensitivity)와 특이도(specificity)를 사용해서 성과를 측정하고 이를 ROC(Receiver Operating Characteristic ) 커브라 합니다. 화살표 방향처럼 왼쪽 상단으로 향하는 모양일때 더 좋은 성과를 나타냅니다. 곡선의 가운데 부분이 어느방향으로 향할지 — 사기범을 식별하는 것에 비중을 둘지, 일반유저들이 frictions 에 받는 영향을 최소화할지 — 는 비즈니스 측면에서 판단하여 의사결정을 합니다. 글에선 _TPR = ⁵√_\[1-(1- _FPR_)⁵\] 형태의 dummy 커브를 가정하여 성과를 측정합니다.

**Figure 5(a):** Blocking events, a friction with F=1 and G=1. **Figure 5(b):** Applying a friction with F=0.95 and G=0.1

사기범이 사건의 1 %를 시도했다고 가정하고  
사기의 고정비용 C = 10, 양호한 사용자의 가치 V = 1로 봅니다.  
  
거래를 직접 차단하는 일반모델(a)에선 (F=1, G=1 )  
1 %의 트랜잭션 차단으로 최소손실이 6으로 정해지는 반면,  
frictions 를 적용한 모델(b)에선 (F=0.95, G=0.1)  
(사기범의 이탈율을 95%, 일반고객의 이탈율을 10%로 적용하면)  
11 %의 거래에 마찰을 가하여 총 손실이 3으로 최소화됩니다!

추가적으로 고위험 이벤트를 직접 차단하고 중위험 이벤트에 frictions을 가하는 모델에 대해서도 고려해 볼 필요가 있습니다.

Reference  
[A/B 테스팅이란](http://www.boxnwhis.kr/2015/01/29/a_b_testing.html)  
[ROC 커브와 AUC, 그리고 민감도와 특이도](http://pyopyo03.tistory.com/8)
