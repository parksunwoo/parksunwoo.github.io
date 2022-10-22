---
title: 3rd week reading list
categories:
  - ML
tags:
  - classification 
  - tree based mothods
last_modified_at: 2018-05-23T09:00:00-00:00
---

4\. Classification

-   4.1 . An Overview of Classification 

연간소득과 월간 신용카드잔액을 바탕으로 개인이 어느정도 카드를 사용할지 예측해보자

Default(Y)는 balance(X1) 과 income(X2)로 정해지고 수치가 아니다

-   4.2 . Why Not LinearRegression  
    범주형은 선형회귀방법이 왜 적절하지 않을까?  범주형의 결과값은 자연적인 순서가 없다 예를 들면 mild, moderate, severe 처럼  
    2개의 상황밖에 없다면 이진분류를 해도되지만

-   4.3 . Logistic Regression  
    default는 Yes, no 2개 분류로 끝나지만 LogisticRegression은 Y가 특정 카테고리에 속할 확률로 나타난다  
    balance가 0에 가까워지면 음의확률이 나오고 balance가 아주 큰 값을 갖을때는 확률이 1을 넘어선다. 이게 말이되나? 이걸 해결하려고 logistic regression이 나왔다  
    그리고 odd 개념이 나오는데 확률대신에 사용되었다? 경주마 베팅할때, 동일한 신용카드잔액이라면 학생이 학생이 아닌경우보다 덜 위험하다

-   4.4 . Linear Discriminant Analysis  
    LDA는 관측치가 적거나 X의 분포가 평균분포일때에 더 견고하다 선형판별분석 두 범주의 중심은 최대화하고, 분산은 최소화하는게 목표  
    현실에선 베이즈 분류를 계산할수가없다.. 판별함수는 x에 관한 Ax+b의 꼴이 되고 선형함수가 되어 선형이 앞에 붙는다  
    LDA / Bayes decision boundary가 서로다르다, Fig 4.4. 경계에 걸치거나 두 경계 사이에 있는 데이터는 그럼 어떻게 판별하는가? 데이터의 양이 작아서 경계가 다르게 나오는것이라면 데이터가 많아진다면 서로 일치하게 되는가? 트레이닝 에러가 테스트 에러보다 낮다 오버피팅떄문  
    threshold를 어떻게 잡느냐에 따라 에러율은 감소하지만 default로 판별하는 비율도 꽤 높아진다 최적의 threshold를 정할떄는 도메인 지식을 활용한다LDA QDA trade off 관계 분산을 어떻게 가져갈것인지에 따라 선택

-   4.5 . A Comparison of Classification Methods  
    LDA는  관측치가 가우시안 분포를 가진다고 가정  
    로직스틱 회귀는 가우시안 분포가 아닐때 더 좋은 성과 LDA에 비해서 KNN는 decision boundary 가 비선형일때 다른방법들에 비해 더 나은모델  
    상황마다 적합한 모델은 조금씩 다르다 decision boundaries 가 선형이라면 LDA, 로지스틱이 비선형이라면 QDA가 더 나은 성과를 보인다

8. Tree-Based Methods

-   8.1 . The Basics of Decision Trees  
    4년이상 플레이 했는지, 118 타점이상인지 여부로 선수들의 연봉을 구분해보자  
    predictor를 겹치지 않는 몇개의 공간으로 나누자 RSS를 최소화하는 게 목표 RSS?? 8.3의 식  
    좋은 예측값을 만들기 위해 가지를 분리해내지만 너무 복잡해져도 overfitting되어서 모델이 복잡해진다 큰 나무를 만들어서 적당히 가지치기를 하자 그럼 어떻게 가지치기를 해야하나?  
    튜닝 파라미터 a 가 하위트리의 복잡도와 trainingdata에 적합하게 맞추는것의 trade off 를 조정한다node purity 의 정확한 의미? 트리모델은 시각화해서 보여주기 좋고 비전문가도 이해하기 쉽다 범주형 변수들도 dummy 변수로 만들필요없이 쉽게 다룰수있다  
    그러나 일반적으로 정확도가 조금 낮고 견고하지못해서 작은변화에도 결과값이 크게 변할수있다

-   8.2 . Bagging,RandomForests,Boosting 의사결정트리는 높은 분산문제가 있다, bootstrap or Bagging 은 분산을 낮춘다. 트레이닝 데이터셋을 여러개로 나누어서 단일의 낮은 분산값을 얻는다?  
    Bagging 수백, 수천개의 트리를 결합해서 정확도를 높인다 정확도가 올라가지만 많은 수의 트리를 bag하다보면 더 이상 쉽게 설명할수없다  
    모델에서 정확도만큼이나 설명력도 중요한가? 랜덤포레스트 : 알고리즘은 사용 가능한 예측 인자의 대다수를 고려하는 것을 허용하지 않습니다.???  
    Bagged tree는 높은 연관성을 갖고 분산 감소로 이어지지 못한다. Boosting : 트리수 B, 감소파라미터 람다, 트리를 나누는 수 d