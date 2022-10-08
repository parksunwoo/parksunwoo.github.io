---
title: CNN으로 영화 평점 예측 모델개발하기
categories:
  - ML
tags:
  - cnn
  - pytorch
  - sentence classification
last_modified_at: 2018-04-28T09:00:00-00:00
---

"한계를 넘어 상상에 도전하자!" [네이버 AI 해커톤 2018](https://github.com/naver/ai-hackathon-2018) 에 참가했습니다 

예선 1라운드부터 시작해서 결선은 4월 26일~ 27일 1박 2일간 진행되었습니다

4월 한달간 즐거운 시간이었고 영화 평점 예측 모델개발 후기를 남겨보려 합니다

주어진 미션은 기존 영화 평과 평점을 학습해서 주어진 영화 평에 가장 알맞는 평점이 몇 점인지 예측하는 모델을 개발하는 것이었습니다

데이터의 구조는 영화 평과 평점이 따로 주어졌습니다

최초로 기본적인 전처리와 영화리뷰 예측을 위한 [Regression](https://github.com/naver/ai-hackathon-2018/tree/master/missions/examples/movie-review/example) 모델이 제공되었습니다 

평소 cnn이 이미지 처리에만 사용되는줄 알고있었으나 김성훈 교수님의 해커톤 소개영상에서 

cnn을 활용하는 것이 좋을것이라는 약간의 힌트를 참고 삼아서 시작했습니다

cnn 을 활용한 문장분류에 대한 논문과 현업 연구자들을 위한 cnn 민감도분석에 대한 논문을 찾을수있어 2개의 논문을 참고했습니다

[Convolutional Neural Networks for Sentence Classification](https://arxiv.org/abs/1408.5882)

[A Sensitivity Analysis of (and Practitioners' Guide to) Convolutional Neural Networks for Sentence Classification](https://arxiv.org/abs/1510.03820)

cnn sentence classification을 pytorch로 구현하였고

2번째 논문을 참고하여 hyperparameter search에 집중했습니다

최종 결과는 24등에 그쳤지만 교수님 말씀대로 이번 실패에서 다시 배우라는 말씀이 기억에 남습니다

추가로 수상한 팀들의 모델에 대한 이야기를 들으면서 다음 사항들에 대해 더 연구해보면 좋겠다는 생각이 들었습니다

 - 한국어 자연어처리의 경우 음소, 음절, 단어단위에 대한 고민필요

\-  Bidirectional RNN 과 RNN과 CNN 의 결합모델

\- 하이퍼파라미터의 경우 learning decay도 중요 고려대상

구현한 소스가 저장된 링크 남깁니다

문의사항이 있다면 댓글로 남겨주세요

[https://github.com/parksunwoo/naver-ai-hackathon-2018](https://github.com/parksunwoo/naver-ai-hackathon-2018)