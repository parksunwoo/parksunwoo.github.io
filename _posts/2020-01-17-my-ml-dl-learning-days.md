---
title: 나의 머신러닝 답사기
categories:
  - ML
tags:
  - ml
  - dl
  - learning list
  - retrospect
last_modified_at: 2020-01-17T22:00:00-00:00
---

1부
==

2018.03.04 — 초고작성

> **사랑하면 알게 되고 알면 보이나니 그때 보이는 것은 전과 같지 않으리라.**  
> ㅡ 나의 문화유산답사기 서문

이제 3년차 엔지니어가 되었다 예전 Spring관련 conference 3년차 개발자의 발표에선 무언가 기술에 대한 자부심같은게 느껴졌었다 나도 나만의 무기를 가져야겠다고 마음먹은건 그 때쯤 이었던 것 같다

비전공자 출신의 내가 어떻게 머신러닝에 관심을 가지게되었는지,  
무엇보다 프로그래밍, AI에 관심을 갖기시작한 학생, 직장인들에게 어느부분부터 시작하면 좋을지 조금이나마 도움이 되었으면 하는 마음에 계속 공부를 해야하는 나의 입장에서도 도움이 될것이라.  
이제까지 공부해왔던 과정을 글로 정리해서 남기고자 한다

2017년 4월. 한창 코세라 강의들에 관심을 가지고 스칼라며 몇몇 강의들을 살펴보다가 머신러닝에 대해 알게되었다. 머신러닝에 대한 관심은 자연스럽게 처음시작할만한 참고자료를 찾기 시작했고 모두를 위한 딥러닝 강좌를 알게되었다

[모두를 위한 머신러닝/딥러닝 강의](http://hunkim.github.io/ml/)
=================================================

홍콩과기대 김성훈 교수님의 친절한 설명으로 머신러닝의 바다로 나아간다참고로 [김성훈 교수님의 이력](http://m.mt.co.kr/renew/view.html?no=2013052615345024115&googleamp)을 알게되면서 더 관심을 가지고 따르게(?) 되었다 lec과 lab으로 나뉘어 이론에 대한 설명을 듣고 그 내용이 어떻게 소스로 적용이 되었는지 알수있다

TensorFlow 설치부터 시작해서 Linear Regression, Linear/Softmax Classification, Regression, 딥러닝의 기본개념, CNN. RNN 까지 거의 모든 내용을 다루지만 그 깊이가 얕지않다

처음 시작하는 입문자들에게는 한국어로 된 아주 좋은 강의자료이다교수님도 다른 여러자료들을 참고하여 만드신 강의라 공부할 거리들을 많이 알려주셔서 더 좋았다.

해당 강의를 제공하는 [김성훈 교수님의 Youtube 재생목록](https://www.youtube.com/user/hunkims/playlists) 에는 모두를 위한 RL강좌, PyTorchZeroToAll(in English), PR12 딥러닝 논문읽기 모임 등 다양한 공부거리가 있다

[TensorFlow KR](https://www.facebook.com/groups/TensorFlowKR/)
==============================================================

모두를 위한 머신러닝/딥러닝 강의 를 듣게 되면 자연스럽게 알게되는 커뮤니티. 많은 사람들이 계속해서 관심있는 주제에 대해 글과 자료를 공유해서 꾸준히 읽어보는것만으로도 도움. [김성훈 교수님이 네이버 선행AI연구팀 CLAIR 에 합류한다는 기사](http://m.zdnet.co.kr/news_view.asp?article_id=20170907095125#imadnews)를 접하고 교수님을 꼭 만나보고 싶다는 생각이 들었다 그래서 아마 이때부터 자료들과 책들을 좀 더 깊이있게 찾아보기 시작한것같다

[밑바닥부터 시작하는 딥러닝](http://www.hanbit.co.kr/store/books/look.php?p_code=B8475831198)
=================================================================================

혼자서 공부를 하기에 좋은 책 검색을 하다가 여러 책들중 고르고 골라서점에 직접가서 읽을만한 책을 찾은 결과다. 딥러닝으로 이미지를 인식하는데 필요한 기술을 배운다. 퍼셉트론, 신경망, 신경망학습, 오차역전파법, CNN, 딥러닝 까지 아래의 책소개처럼 라이브러리나 프레임워크를 사용하지않는데 이는 핵심개념을 이해하는데 더 도움이 된다고 생각한다

**\*직접 구현하고 움직여보며 익히는 가장 쉬운 딥러닝 입문서\***

_이 책은 라이브러리나 프레임워크에 의존하지 않고, 딥러닝의 핵심을 ‘밑바닥부터’ 직접 만들어보며 즐겁게 배울 수 있는 본격 딥러닝 입문서입니다. 술술 읽힐 만큼 쉽게 설명하였고, 역전파처럼 어려운 내용은 ‘계산 그래프’ 기법으로 시각적으로 풀이했습니다. 무엇보다 작동하는 코드가 있어 직접 돌려보고 요리조리 수정해보면 어려운 이론도 명확하게 이해할 수 있습니다. 딥러닝에 새롭게 입문하려는 분과 기초를 다시금 정리하고 싶은 현업 연구자와 개발자에게 최고의 책이 될 것입니다._

딥러닝 책과 함께 출퇴근 시간 시간나는대로 참고할만한 블로그, 사이트들도 찾기시작했다

[라온피플 머신러닝 아카데미](http://blog.naver.com/PostList.nhn?blogId=laonple)
===================================================================

블로그에는 주제별로 자세히 나누어 찬찬히 설명을 해준다. 이런블로그를 운영하고 꾸준히 사람들에게 공유하려고 한다는데 라온피플 이라는 회사에 입사할수 있다면 괜찮겠다는 생각까지 했었다. 강소기업이라고 생각하며 시간이 될때마다 읽어보려 노력중이다

[다크 프로그래머](http://darkpgmr.tistory.com/)
========================================

영상처리, 프로그래밍에 대해 다소 난이도 있는 주제들에 대해 자료를 정리해두었다. 단순히 지식을 아는것이 아니라 그것의 원리를 알고 이야기를 한다는데서 내공을 느낄수있었다. 기계학습, 수학이야기 그 외 다른 카테고리들도 참고할만하다.

머신러닝에 대한 호기심에서 시작했던 공부가 시간이 갈수록 학문에 대한 갈증과 더 많은 호기심을 자극하기 시작했다 김성훈 교수님이 계신 CLAIR 팀에 합류하고 싶어서 준비했던 것들은 그동안 내가 보내왔던 시간들을 정리해볼수 있는 계기가 되었고 대학생때 수강했던 금융공학 수업 교수님, SCSA 부장님, 함께 근무했던 차장님께 지원동기를 말씀드리고 추천서를 받는 과정은 다른 사람들에게서 신뢰를 받는다는 기분이 들어서 나에게는 아주 큰 의미가 있었다.

영문 이력서 작성, 팀에 합류해서 같이 구현해보고 싶은 주제설명, 논문을 소스코드로 직접구현하는 과정은 많은시간이 소요되었고 결과적으로는 합류하지 못했지만 적지않은 소득이 있었다

이론적으로 지식만 아는것과 직접 소스코드로 구현해보는건 하늘과 땅의 차이다. 기본적인 학습이 완료되었다고 생각한다면 아래의 논문들을 찾아서 읽고 소스로 구현해보는것을 추천한다

논문을 읽기 시작하면서 선형대수를 포함한 수학/통계적 지식, 영어공부의 필요성을 느꼈고 이것들을 어떻게 보완 준비해나가는지는 아래에서 소개하겠다

읽어볼만한 논문들

*   StarGAN (Using TF or something. non PyTorch: [https://arxiv.org/abs/1711.09020](https://arxiv.org/abs/1711.09020) )
*   Seq-to-Seq translation (https://arxiv.org/abs/1409.0473 )
*   VQ-VAE (http://papers.nips.cc/paper/7210-neural-discrete-representation-learning )
*   CapsuleNet (https://arxiv.org/abs/1710.09829 )
*   Deep Speaker (https://arxiv.org/abs/1705.02304 )
*   Show, attend, and tell (https://arxiv.org/abs/1502.03044 )
*   SqeezeNet (https://arxiv.org/abs/1602.07360 )
*   Recurrent Highway Network (https://arxiv.org/abs/1607.03474 )
*   Tacotron2 — excluding WaveNet-vocoder (https://arxiv.org/abs/1712.05884 )
*   Generative Adversarial Imitation Learning (https://arxiv.org/abs/1606.03476 )
*   DARLA (https://arxiv.org/abs/1707.0847)

2부
==

2018.09.29 — 초고작성

나의 머신러닝 답사기1 이 혼자서 고군분투한 내용이라면 이제부터 쓸 내용은 배움을 찾아서 다녔던 이야기이다

논문을 코드로 직접 구현해보는 경험을 한 이후로 부족한 부분들을 알게되었고 크게 3가지 부분이었다 파이썬. 수학. 머신러닝 이론

[파이썬 라이브러리를 활용한 머신러닝](http://www.hanbit.co.kr/store/books/look.php?p_code=B6119391002)
======================================================================================

책의 목차를 따라 예제들을 직접 실습해보았고 책의 역자인 이해선 님의 오프라인 강의가 있다는 걸 알게되어서 시간나는대로 참석해서 설명을 들었다 [홍대 머신러닝 Meetup](https://www.meetup.com/ko-KR/Hongdae-Machine-Learning-Study/). 우선 책을 직접 번역하셨다보니 설명을 깔끔하게 해주셔서 이해하기가 한결 쉬웠다

밑바닥부터 시작하는 딥러닝이 발을 내딛게 한 시작이었다면 scikit-learn 라이브러리를 활용해서 좀 더 다양한 데이터들, 다양한 이론들을 간결한 설명으로 접할 수 있었다

[핸즈온 머신러닝](http://www.hanbit.co.kr/store/books/look.php?p_code=B9267655530)
===========================================================================

위 책에 이어서 다음 강의로 진행되는 핸즈온 머신러닝. 좀 더 심화된 내용으로 scikit-learn 과 Tensorflow 로 실무에서 참고가능한 수준이다 현재 읽고 있는 책이기도 하고 역시 이해선 님의 오프라인 강의가 현재 진행되고 있다

[선형대수 칸 아카데미](https://ko.khanacademy.org/math/linear-algebra)
=============================================================

수학의 부족함을 절실히 느꼈고, 모두의 연구소라는 곳을 알게되었다. 모두의 연구소에서는 3개월 단위로 풀잎스쿨 과정이 진행되는데 선형대수 과목을 수강하게되었다. 대학교수님이 오셔서 강의를 해주셨는데 평소 받던 일방적 방식이 아니라 서로 이야기하고 설명하고 강의를 해주신다기보다 가이드를 해주신다는게 더 맞는 표현인것같다.

선형대수 풀잎과정에서 3개월동안 칸 아카데미 선형대수 과목을 들으면서 진행되었다 1주일동안 정해진 범위를 미리 수강하고나서 금요일 저녁에 모여서 같이 이야기 나누는 방식으로 혼자 공부하기가 막막하다면 10 주정도로 진도범위를 정해서 진행한다면 좋을 것같다. (3개 과목으로 구성) 3개월이라는 시간이 길어보이지만 짧은 강의들이 꽤 많아서 일주일동안 정해진 분량을 듣는게 쉽지는 않았던 것 같다

[네이버 ai 해커톤 2018](https://github.com/naver/ai-hackathon-2018)
=============================================================

페이스북에서 네이버 ai 해커톤이 개최된다는 소식을 접했다. 아직 공부중이기는 했지만 참가해보는 것만으로도 큰 도움이 될 것같아서 휴가를 내고 참가를 했다 미션은 아래 2가지 였는데 나는 네이버 영화 평점예측을 시도했다

*   네[이버 지식iN 질문 유사도 예측](https://github.com/naver/ai-hackathon-2018/blob/master/missions/kin.md)
*   [네이버 영화 평점 예측](https://github.com/naver/ai-hackathon-2018/blob/master/missions/movie-review.md)

예선을 운좋게도 통과해서 1박 2일동안 춘천 커넥트원에서 진행되는 결선에 갈수 있었다. CNN 을 이용해서 문장분류를 시도했고 결선참가팀중에 24등이라는 성적을 거두었다 우승팀들은 다들 팀으로 되어있었고 개별 참가자들은 그리 많지않았다.

몇몇 사람들을 만났고 이튿날에는 김성훈 교수님도 뵐수있었다. 아쉬움이 남았지만 그래도 예선전포함해서 한달정도 논문을 찾아보고 또 코드로 구현해보면서 공부가 많이 되었다 네이버 ai 해커톤 깃허브에는 다른 참가자들의 모델들도 올라와있으니 한번 찾아보는 것만으로도 다양한 모델들을 구경할 수있을것같다

[DataBootcamp, Spring study](https://github.com/parksunwoo/DBSZeroToAll)
========================================================================

운이 좋게도 카카오톡 파이썬 머신러닝 오픈채팅방에서 다니엘 님을 만나게 되었다. 다니엘 님은 현직 데이터 사이언티스트로 회사에 입사하는 새내기 데이터 사이언티스트들을 위한 프로그램을 데이터 사이언티스트에 관심있는 사람들에게 진행해보겠다는 계획을 세우고 1기를 모집했고 내가 참가하게 되었다 데이터과학과 관련된 내용을 이론부터 실습까지 전체적으로 경험해볼 수 있었고 올해 나의 최고의 선택이었다

아래 적혀있는 도서들을 읽고 요약했고(일부는 발췌독) , 미디엄 블로그 작성, 팀을 구성해서 실전 데이터 분석대회 참여까지 진행이 되었다.

5월~8월 매주 토요일마다 만나서 토론식으로 수업이 진행되었다. 수업전 미리 공지된 분량의 도서를 읽고 요약(학습일지 카테고리 참고), 개별과제에 대해 이야기를 나누었다

무엇보다 같은 뜻을 갖고 공부하는 사람들을 만날 수 있어서 좋았고, 직접 일하는 분의 강의를 듣다보니 실무 관련된 이야기들도 들을 수 있었다.

아래의 내용은 깃허브에 작성해둔 주차별 학습내용이다.

Reading List
============

*   Pattern Recognition and Machine Learning, Christopher Bishop
*   Machine Learning: A probabilistic perspective, Kevin Murphy.
*   The Elements of Statistical Learning: Data Mining, Inference and Prediction, Trevor Hastie, Robert Tibshirani, Jerome Friedman.
*   An Introduction to Statistical Learning: with Applications in R, Gareth James, Daniela Witten, Trevor Hastie, Robert Tibshirani

Table of Contents
=================

1st week
--------

*   Teory — Probability Models: Estimators, Guarantees, MLE (Murphy 2, 6)
*   [Bayesian Learning for hackers Ch1](https://github.com/CamDavidsonPilon/Probabilistic-Programming-and-Bayesian-Methods-for-Hackers/blob/master/Chapter1_Introduction/Ch1_Introduction_PyMC3.ipynb)
*   [Data Analysis — House Prices(Hands on Machine Learning example)](https://github.com/parksunwoo/DBSZeroToAll/blob/master/1st%20week/1st%20hw_EDA_housing.ipynb)

2nd week
--------

*   Theory — Linear Prediction, Maximum Likelihood, Linear Regression (Bishop)
*   [Data Analysis — House Prices](https://github.com/parksunwoo/DBSZeroToAll/blob/master/2nd%20week/2nd_kaggle_housing2.ipynb)

3rd week
--------

*   Theory — Classification, Tree-Based Methods (ISLR)
*   [Data Analysis — Bike Demand](https://github.com/parksunwoo/DBSZeroToAll/blob/master/3rd_week/bike_modeling.ipynb)
*   [Code — Making Gaussian distribution simulation data, Modeling Linear Discriminant Analaysis, Logistic regression, QDA, Naive Bayes](https://github.com/parksunwoo/DBSZeroToAll/blob/master/3rd_week/models_accuracy_effect.ipynb)
*   [Blog — Bayesian vs. Frequentist](https://medium.com/@sunwoopark/오늘-나의-운세는-bayesian-vs-frequentist-로-바라본-운세-83077992c17)
*   [Blog — Fighting Financial Fraud with Targeted Friction](https://medium.com/mighty-data-science-bootcamp/airbnb-fighting-financial-fraud-with-targeted-friction-4d1b52077444)

4th week
--------

*   [Blog — Data Science and the Art of Producing Entertainment at Netflix](https://medium.com/@sunwoopark/netflix-data-science-and-the-art-of-producing-entertainment-at-netflix-d710174a8c16)
*   [Kaggle Competition](https://github.com/parksunwoo/DBSZeroToAll/blob/master/4th_week/NAVER-home-credit-default-risk.ipynb)

5–6th week
----------

*   Theory — Unsupervised Learning: Clustering, Kmeans (Hastie), Unsupervised Learning: Clustering: Mixture of Gaussians, Expectation Maximization
*   [Code — Kmeans, PCA](https://github.com/parksunwoo/DBSZeroToAll/blob/master/5th_week/01_PCA_Kmeans.ipynb)
*   [Blog — Kaggle\_Home Credit Default Risk\_Part 1](https://medium.com/mighty-data-science-bootcamp/kaggle-도전기-home-credit-default-risk-part-1-735030d40ee0)

7th week
--------

*   Theory — Link Analysis, Social Network Graphs (Mining Massive Datasets)
*   [Code — Spectral Clustering](https://github.com/parksunwoo/DBSZeroToAll/blob/master/7th_week/Spectral_Clustering_from_scratch.ipynb)

8th week
--------

*   [Data Analysis — Walmart Recruiting: Trip Type Classification](https://github.com/parksunwoo/DBSZeroToAll/blob/master/8th_week/Walmart_TripTypeClassification.ipynb)

9–10th week
-----------

*   [Data Analysis — Traffic Accident\_road attribute](https://github.com/parksunwoo/DBSZeroToAll/blob/master/9th_week/road_attribute_analysis.ipynb)
*   [Data Analysis — Traffic Accident\_urban rural](https://github.com/parksunwoo/DBSZeroToAll/blob/master/9th_week/urban_rural_analysis.ipynb)
*   [Data Analysis — Traffic Accident\_pedestrian](https://github.com/parksunwoo/DBSZeroToAll/blob/master/9th_week/pedestrian_analysis.ipynb)
*   [Data Analysis — Traffic Accident\_maps modeling](https://github.com/parksunwoo/DBSZeroToAll/blob/master/9th_week/maps_modeling.ipynb)
*   [White paper — Traffic Accident](https://github.com/parksunwoo/DBSZeroToAll/blob/master/10th_week/EDA_Traffic_accident_data.pdf)

\+ 추천사이트
========

다니엘님은 데이터 관련 기업들의 블로그가 다양한 주제를 재미있게 설명하고 있어서 꾸준히 읽어볼 것을 강조했다

*   [Data Scientist with Python Track](https://www.datacamp.com/tracks/data-scientist-with-python)
*   [stitchfix blog](https://multithreaded.stitchfix.com/blog/)
*   [reddit machineLearning](https://www.reddit.com/r/MachineLearning/)
*   [neflix blog](https://medium.com/netflix-techblog/tagged/data-science)
*   [big cloud blog](http://www.bigcloud.io/what-we-think/)
*   [stanford 동영상강의 Mining Massive Datasets](https://www.stanford.edu/)  
    Link Analysis, Social-Network Graphs 강의를 수강했습니다
*   [edwith 홈페이지](https://www.edwith.org/)  
    아래 강의들을 수강 및 수강중이고 양질의 강의가 한글로 되어있어서 큰 도움이 되고있다.
*   인공지능을 위한 선형대수
*   인공지능 및 기계학습 개론 1, 2, 심화
*   딥러닝을 이용한 자연어처리

[모두의 연구소 Deep Learning College](http://dlc.modulabs.co.kr/)
===========================================================

주중에 스터디가 3개가 진행되는 강행군 이었지만 끝나고 나서 어떤 부분을 더 공부해볼지 고민이되었다. 그러다 지난번 참여했던 모두의 연구소에서 Deep Learning College 2기를 모집한다는 공지를 보게되었다

1년 과정이고 또 팀프로젝트, 개인프로젝트가 진행. 무엇보다 논문을 작성해야한다는게 맘에 들어서 지원하게되었다. 학부과정에선 논문을 써볼일이 없었고 대학생때 참여했던 파생상품대회에서 팀을 이뤄서 보고서 형식으로 작성해보긴 했지만 내가 직접 논문을 써보고 싶었다. 배운 내용을 활용해서 연구해보고 싶은 주제를 정해 그것에 대해 논리적으로 작성해보는 것 말이다. 현재 나는 DLC 자연어처리 과정을 진행하고 있고 2개월 정도 지나 기초과정 수강은 끝이 보인다.

늦은 시간 집으로 돌아오는 길은 힘들고 때론 지치지만 뜻이 있는 길에 도움을 주는 사람들이 있으니 좀더 힘을 내보려고 한다.

3부
==

[나의 머신러닝 답사기2](https://parksunwoo.github.io/ml_voyage2/)가 18년 9월에 작성되었으니 1년이라는 시간이 훌쩍 지났다. 나는 그동안 무얼 공부했고 또 어떤 모습으로 변화되었을까 2019년의 마지막에서 간단히 내용을 정리해보려 한다

[모두의 연구소 Deep Learning College](http://aic.yangjaehub.com/)
===========================================================

Deep Learning College (DLC) 2기 자연어처리 Track에 참여했다. 현재는 AI College 라는 이름으로 바뀌었지만 1년여 과정동안 기초강의 3개월, 팀프로젝트 4개월, 개인프로젝트 4개월로 이어졌다.

자연어처리 기초강의에서는 SKT 연구원 분들이 직접 방문해서 gluon과 네이버 영화리뷰 데이터셋을 이용한 sentiment classification 같은 내용까지 설명을 들을 수 있었다

팀프로젝트 주제는 Question Answer Generation 을 정해서 연구를 진행했다. 질문생성만 하는 모델과 답변만 생성하는 모델들은 찾을 수 있었지만 이 둘을 동시에 진행하는 모델을 찾기 어려웠고 DAANet: Dual Ask-Answer Network for Machine Reading Comprehension 이라는 논문과 코드를 찾아 수행해보았으나 논문과는 달리 잘 수행이 되지 않아 큰 벽에 부딪혔다. 쉽지 않은 연구주제를 설정했음을 알게되었고 연구를 진행할때 데이터셋의 중요성을 실감하게 되었다. 연구를 진행하는 동안에 다행히도 KorQuAD 데이터셋이 나왔지만 질문-답변 동시생성 모델에 대한 가시적인 성과는 얻지 못했다.

DLC과정중 Flipped Learning 방식의 과목을 1개 수강할 수 있게 되는데 마지막 과제인 개인논문을 잘 작성하기 위한 준비로 좋은 논문들을 읽고 싶어 Slow Paper 라는 딥러닝 논문읽기 과목을 수강하게 되었다. 혼자서는 읽기 어려운 논문을 여러 사람이 모여서 3시간동안 읽게되니 그래도 끝까지 읽을 수 있었고 읽은 논문들을 하나씩 블로그 글로 정리했다. 8편의 논문이었지만 기초적인 개념들을 정리하고 다양한 분야를 접할 수 있는 좋은 기회였다.

1주차 (1/12)  
[Mobilenetv2: 거꾸로 된 잔차와 선형 병목](https://medium.com/@sunwoopark/slow-paper-mobilenetv2-inverted-residuals-and-linear-bottlenecks-6eacaa696b54?source=friends_link&sk=21cf2623994618735bfc94bdf13034c0&fbclid=IwAR1uV5iMRzHd-Lh1CrCcnG7-uYd47QSaHAv3rCEAYS1DZ6iRrzRuwu-9UeM)

2주차 (1/19)  
[라디오 신호를 이용한 벽의 포즈 추정](https://medium.com/@sunwoopark/slow-paper-through-wall-human-pose-estimation-using-radio-signals-47b76a76ec59?source=friends_link&sk=dcd82f35919a9315a99b3799e6bc93b2&fbclid=IwAR0xUqU6676ImMHydRrf9zEvSTqyBR-0SRfShgXjMqUlq5tE7eQSVLigJq0)

3주차 (1/26)  
[밤: 병목 주목 모듈](https://medium.com/@sunwoopark/slow-paper-bam-bottleneck-attention-module-abbf680a869?source=friends_link&sk=66bd689843e28ed0c3b6b2b3a9783441&fbclid=IwAR3lRgNDKV4tS9bvT9gowwKAUJZNxdl1C4Olvic2O5QuKavnTIyEuMZ-2EY)

4주차 (2/9)  
[강화 학습으로 신경 건축 검색](https://medium.com/@sunwoopark/slow-paper-neural-architecture-search-with-reinforcement-learning-6de601560522?source=friends_link&sk=148ff3d2b667cf753ce01118ac74e95a&fbclid=IwAR2KQvBx-OUjJbHhqI7HmYRN1EoBVAXSWlYCIKWyhSGMOgWkHJaRU5rQyvo)

5주차 (2/16)  
[글로우: invertible 1 x1 convolutions와 함께 생성 흐름](https://medium.com/@sunwoopark/slow-paper-glow-generative-flow-with-invertible-1x1-convolutions-837710116939?source=friends_link&sk=285d1ae8885cc78bfb431630974c562f&fbclid=IwAR36cppOT36VDyPeLkp0g1V04gHdvuxmVRPWqyiZwW_CePY3XrhgBQq3tAs)

6주차 (2/23)  
[Timbretron: a wavenet (cyclegan (cqt (오디오))) 뮤지컬 음색 전송을 위한 파이프라인](https://medium.com/@sunwoopark/slow-paper-timbretron-a-wavenet-cyclegan-cqt-audio-pipeline-for-musical-timbre-transfer-7b41f90ab62c?source=friends_link&sk=a62a878834a1a7b50471e49a2b655d72&fbclid=IwAR0puM65gkNOs8dqLA0w0VMTACmuJ06j1qBSvjI8zBV1sDoByFkHqPFroGc)

7주차 (3/9)  
[깊은 조건부 생성 모델을 이용한 구조적 출력 표현 학습](https://medium.com/@sunwoopark/slow-paper-learning-structured-output-representation-using-deep-conditional-generative-models-692b8dd69cf6?source=friends_link&sk=eac89c7de7beb958b045cdfce9a79afa&fbclid=IwAR2RslBeUpj2SA8EbCah7NeaKpoLgyyoybBozZgzlVUrjK-Btkx3IFEC6_0)

8주차 (3/16)  
[스타일-attentional 네트워크로 임의 스타일 전송](https://medium.com/@sunwoopark/slow-paper-arbitrary-style-transfer-with-style-attentional-networks-8e0c3206168e?source=friends_link&sk=fa81992704dc7414b3a0142d8271afd0&fbclid=IwAR38horX6UnN-UFyi-IA4E2Z3nywszsdGrY7JlNjsxVByufR2r-v1ePPD8M)

마지막 개인프로젝트는 논문작성이었고 주제를 정하기가 쉽지않았다. 결국에는 한국어문장에 대한 딥러닝 OCR 모델에 대한 내용으로 정하고 진행하였다. OCR 프로젝트는 네이버에서 진행된 프로젝트를 많이 참고했고 한국어 문장 데이터셋은 따로 없어서 폰트와 많이 사용되는 어휘사전을 이용해서 직접 생성해서 진행했다. 해당 프로젝트 상세내용은 [다른 포스트](https://parksunwoo.github.io/ocr_kor1/)로 내용을 정리하고자 한다. 처음 작성한 논문이 HCLT 2019 에 채택되어서 포스터발표를 하게되는 영광을 누리게되었다.

MIT\_데이터 사이언스 기초
================

[강의링크](https://www.edwith.org/datascience/lecture/33888/)  
[강의정리 Git](https://github.com/parksunwoo/TIL/blob/master/data-science/README.md)  
15 chapter의 데이터 사이언스 기초강의를 듣고 Git에 정리했다. 예전 Data Science Bootcamp 내용을 복습할겸 전체내용을 다시 정리하기에 좋았다.

하버드\_확률론 기초: Statistics 110
===========================

[강의링크](https://www.edwith.org/harvardprobability/joinLectures/17924)  
[강의정리 Git](https://github.com/parksunwoo/TIL/blob/master/statistics/README.md)

아직도(?) 진행중이긴 하지만 확률은 강의를 듣고 정리해두면 두고두고 사용할수 있을 것이라 남은 내용들도 다시 듣고 정리해보려 한다. [edwith](https://www.edwith.org/)에는 좋은 강의들을 엄선해서 다시 번역까지 해놓은 곳이라 필요한 과목들만 골라서 보기에 추천하는 사이트이다.

올해에도 여러 컨퍼런스에 참여했고 보고 들은것을 기록으로 남겼다.  
[파이콘 2019](https://parksunwoo.github.io/pyconkr2019/)  
[한글및한국어정보처리학술대회 2019 1일차](https://parksunwoo.github.io/HCLT2019_day1/)  
[한글및한국어정보처리학술대회 2019 2일차](https://parksunwoo.github.io/HCLT2019_day2/)  
[모두콘 2019](https://parksunwoo.github.io/moducon2019/)

DLC 과정을 시작할때에는 막연히 1년과정이 끝나면 좋은 기회가 생기지않을까 하는 기대감이 있었다. 데이터 사이언티스트를 꿈꾸며 길을 걷고 있었지만 파이썬이라는 언어의 매력에 빠지게 되었다.

그리고 모두의연구소 라는 곳에서 공부하면서 그곳에 있는 사람들과 그곳이 나아가는 방향에 깊은 공감을 하게되었다. 내년부터는 모두의연구소의 Backend Engineer 로 더 많은 사람들이 AI와 ML/DL를 배울 수 있는 연구생태계를 만들어보려 한다.

지금까지 하고 있던 웹개발을 Django라는 새로운 언어로 하게 되어 현재는 강의를 들으며 django를 공부중이다. 그럼 나의 머신러닝 답사기는 다시 웹개발로 돌아오게되어 끝이 난건가? ;)

나는 이제 Ai Engineer라는 새로운 목표를 세우고 어떤 분야 어떤 주제를 파볼지 정해보려 한다.  
놀이터에서 일하게 되었으니 다양한 분야를 좀더 가까이에서 접해보고 연구원분들과도 되도록 많은 이야기를 나눠보고 싶다.  
그곳에서의 새로운 경험이 나를 또 다른 길로 이어지게 할것이고 2020년은 지금까지보다 더 기대되는 한해가 될것같다.

Github : [https://github.com/parksunwoo](https://github.com/parksunwoo)  
Linkedin : [https://www.linkedin.com/in/sunwoo-park-a613aa131/](https://www.linkedin.com/in/sunwoo-park-a613aa131/)
