---
title: Google Machine Learning Bootcamp KR 2023
categories:
  - ML
tags:
  - bootcamp
  - google
  - kaggle
  - coursera
last_modified_at: 2024-01-14T22:00:00-00:00
---

## 구글에서 진행하는 머신러닝 부트캠프 2023을 수료하고 그 과정에서 학습하고 느낀 점을 정리했습니다

---

처음 부트캠프를 알게되어 지원하고 또 그 과정에서 무엇을 얻었으며 앞으로 어떤 것을 조금 더 공부해볼지 정리해보려 한다

나의 정리된 글이 막연히 머신러닝 부트캠프를 알기만 하고 지원을 망설이는 사람이나

또 관심은 있지만 제한된 정보로 아쉬운 사람들에게 도움이 되길 바란다.

구글 머신러닝 부트캠프 모집을 처음 알게된 것은 TensorFlow KR 페이스그룹에서 

권순선 님의 구글 머신러닝 부트캠프 4기 모집글을 보게되면서였다

https://korea.googleblog.com/2023/07/ml-bootcamp-2023.html?fbclid=IwAR2chNCj6jXOiMSgCJlafRO_5mxWBEpTx5-S7HT4REFDKigNvsf52VXziak

사실 이전 기수 모집을 시작할때에도 관심을 갖고 지원을 했었지만

메일함을 뒤져보니 20년도에 1기로 지원을 했었고 그때는 응답이 없었다

모집글을 보고 3년만에 다시 지원을 하게 된 것인데

지원 절차는 구글폼 링크에 들어가 아래의 폼 양식에 맞춰 기본정보/ 개발능력 설문/ 레벨테스트(10문제)/ 프로그램 참가관련 설문 문항에 응답을 하고 제출하면 된다.

![Google for Developers Machine Learning Bootcamp 2023 참가지원서](/assets/images/post-google-bootcamp-invite.png)

얼마나 답장을 기다렸을까? 7월 중순에 지원하고 8월 말에 합격안내 메일을 받았으니 한달 넘게 기다렸었다

지원서가 통과되었다는 내용과 함께

부트캠프에서 진행되는 2개의 교육과정 안내가 나온다

1. 8주 내 코세라  [Deep Learning Specialization](https://www.coursera.org/specializations/deep-learning) 교육 과정 수강

2. 3개월 내 캐글 미션 중 선택 달성
    
    A.  [Featured Competition](https://www.kaggle.com/competitions?hostSegmentIdFilter=1)에 참여하여 상위 40% 내 랭킹

    B. 팀으로 [Play Ground Competition](https://www.kaggle.com/competitions?hostSegmentIdFilter=8) 혹은 [Community Competition](https://www.kaggle.com/competitions?hostSegmentIdFilter=10)에 참여하여 상위 20% 내 랭킹


# 코세라 강의

처음에는 호기롭게도 코세라 교육과정을 8주간 수강하는 것을 어렵지 않게 생각했다. 

8주라는 시간이 충분해 보이기도 했고 예전에 잠깐이지만 딥러닝을 공부했던 경험도 있고해서

하지만 막상 강의를 듣기 시작했을때 처음 듣는 내용들도  있어서 시간이 충분하지 않았다.

![Coursera Site](/assets/images/post-coursera.png)


구글 메일로 코세라에서 과정 초대메일이 온다. 초대메일의 링크를 클릭해서 코세라 사이트에 들어가면 새롭게 Machine Learning Bootcamp  소속이 생기고

Deep Learning 특화과정을 수강할수 있게 된다.

Deep Learning 특화과정은 80만명이 수강한 deep learning의 유명한 강의로 일반적인 강좌 5개가 패키지로 구성되어있다.

각 강좌는 아래와 같다.

강좌1. **[Neural Networks and Deep Learning](https://www.coursera.org/programs/learning-program-qth60/learn/neural-networks-deep-learning?specialization=deep-learning) (24h)**

강좌2. **[Improving Deep Neural Networks: Hyperparameter Tuning, Regularization and Optimization](https://www.coursera.org/programs/learning-program-qth60/learn/deep-neural-network?specialization=deep-learning) (23h)**

강좌3. **[Structuring Machine Learning Projects](https://www.coursera.org/programs/learning-program-qth60/learn/machine-learning-projects?specialization=deep-learning) (6h)**

강좌4. **[Convolutional Neural Networks](https://www.coursera.org/programs/learning-program-qth60/learn/convolutional-neural-networks?specialization=deep-learning) (35h)**

강좌5. **[Sequence Models](https://www.coursera.org/programs/learning-program-qth60/learn/nlp-sequence-models?specialization=deep-learning) (37h)**

따로 시간을 표기한것은 어느정도 분량인지를 알려주기 위해서인데 5강좌를 합치면 100 h 시간이 조금 넘는다.

100시간을 몰아서 들을수도 있지만 시간의 여유가 없는 직장인에게는 퇴근후 2-3시간을 공부한다고 했을때에 꾸준히 3-40일을 수강해야 (매일 공부했을때의 가정이고 그렇지 않다면 시간 여유가 없다) 완료할 수 있다는 계산이 나온다.

![Coursera Week1](/assets/images/post-coursera-w1.png)

강좌 1을 예시로 코세라 강의가 어떻게 구성되어있는지를 조금더 상세히 살펴보면

강좌 1은 다시 4개 주차로 구성되어있고 1개 주차는 코세라에서 예상하기에 1주일정도 학습분량을 의미한다.

동영상을 시청하는 부분이 있고 읽기자료가 있고 

중간중간에 학습내용을 제대로 이해했는지를 묻는 퀴즈가 있다. 퀴즈는 난이도가 조금 달라서 어려운 퀴즈는 합격기준을 넘지 못하는 경우도 빈번했는데

기준을 넘지 못하면 재응시할 수 있게 기회를 다시 준다. 제한된 일일 기회안에 합격기준을 넘지못하면 더이상 퀴즈를 풀수 없고 하루를 기다린다음 응시할 수 있다.

![Coursera Week2](/assets/images/post-cousera-w2.png)

프로그래밍 과제의 경우는 앞서 공부한 내용을 바탕으로 쥬피터 노트북의 코드를 수행하게 된다.

프로그래밍 과제 페이지에서는 간단히 어떠한 내용을 학습하는지 안내가 나오고 또 아래의 “브라우저에서 작업하기” 버튼을 클릭하면 쥬피터 화면으로 이동한다.


![Coursera- Assignment](/assets/images/post-coursera-assignment.png)

쥬피터 노트북화면에서는 상세한 과제 내용에 대한 설명이 처음에 나온다. 어떤 것들을 배우는지 또 어떤 예제들이 있는지에 대한 안내가 나온다.


![Coursera- Jupyter Notebook](/assets/images/post-coursera-jupyter.png)

작성해야하는 예제 부분에서는 도움말이 제공되고 코드셀 내에 “YOUR CODE STARTS HERE” “YOUR CODE ENDS HERE” 부분에

예제에서 요구하는 코드를 작성하면 된다. 되도록 예제에서 요구하는 사항 (라인수 등) 을 충족해야 자동채점이 제대로 수행된다.

![Coursera- Jupyter Notebook2](/assets/images/post-coursera-jupyter2.png)

Deep Learning 특화과정 은 영어로 진행되지만 설정을 통해 한글자막을 지원한다. 번역된 한글이 이해하기에 더 어려운 경우도 있지만 한글자막을 통해 강의 이해 및 수강속도를 조금더 높일 수 있었다. 여기까지 읽었다면 음..  부트캠프가 코세라 강의를 혼자 수강하는 것과 어떤 부분에서 다르지? 하는 생각이 하실수 있을 것 같다.

부트캠프에서는 8주 수강기간동안 5번의 미션이 나온다. 코세라 사이트에서도 스케쥴이나 지연알림 메시지가 나오지만 이것과는 부트캠프에서 별도 작성된 엑셀시트에

일자와 해당일까지 수강완료해야하는 강의가 나온다. 해당기간까지 목표강의를 완료하지 못한다면 부트캠프에서 자동 탈락이다.

![Bootcamp sheet](/assets/images/post-bootcamp-excel.png)

강의를 수강완료할떄마다 강의별 수료증이 나온다. 수료증이 발급되면  뿌듯함과 함께 다음 강의를 열심히 수강해야겠다는 생각이 들게된다

또한 수료증을 본인만 보는 것이 아니라 공유기능을 통해 sns에 자신의 학습성과를 자랑할 수 있게 되어있다.

링크드인을 통해 자신이 학습중인 과정의 수료증을 공유하면 본인의 관심사를 다른사람에게 어필할수 있어 좋고 또 다른 사람에게 공유했기에 쉽게 포기하지않고 지속하게 만드는 것 같았다.

![Coursera Certificate](/assets/images/post-coursera-certificate.png)

위 수료증은 5개의 강좌를 모두 완료하고 받은 수료증이다. 강좌를 완료하고 수료증을 받았을때 기뻤다. 학습을 하는데 많은 시간이 들었고 생각이상으로 쉽지 않았기 때문이다.



# 디스코드/게더

처음 합격 이메일을 받게되면 코세라 강의를 비롯해 앞으로의 교육 과정 스케쥴이 안내되면서 디스코드 채널 가입도 설명이 나와있다.

Google Developer Community 디스코드에 초대되는데 부트캠프 참여자들은 공개 오픈되어있는 여러채널에 참여할 수 있고 별도로 mlb- 로 시작하는 부트캠프 전용 채널에 따로 들어가게된다. 해당 채널에서는 부트캠프 관련된 공지사항 전달, 커뮤니티 이벤트 안내, 과정진행상황 체크 및 질의응답을 할 수 있게되어있다. 

과정에 대한 질문을 하면 먼저 학습하거나 알고있는 교육생이 답변을 해주면서 해결되는 모습을 볼수 있다.

디스코드에서는 또 캐글 팀원 모집이나 스터디 모집글을 채널에서 보고 참여할 수 있다.

부트캠프에서는 스터디 모임 개설 및 참여를 장려하는데 부트캠프 게더사이트가 따로 있어서 엑셀시트에 희망하는 날짜와 주제를 간단히 작성하고 스터디원을 모집해

스터디를 진행할 수 있다. 논문읽기모임, 강의를 듣는 모임, 기업 분석 모임 등 다양한 스터디 모임이 개설되고 사람들이 서로 정보를 공유했다. 게더타운 이벤트에 응모하면 구글 굿즈를 받을수 있다.

![Discord Study](/assets/images/post-discord-study.png)

내가 생각하는 부트캠프의 매력 중 하나는 기타 프로그램의 구성이다.

수요일 점심, 저녁을 이용해 테크톡/오피스 아워/레쥬메 클리닉이 구글 미트를 통해 진행된다.

이름만 들으면 알만한 기업에서 연사 분이 초청되어 어떤 일들을 진행하는지 회사 홈페이지에는 나와있지 않은 내용들까지 자세히 알려주신다.

이벤트를 진행하기 전에는 미리 사전질문을 받아서 궁금한 사항들을 질문할수 있다.

야놀자, 넷마블, 넥슨코리아, 롯데e커머스, KT, 몰로코, 쏘카, 노타, CJ올리브영, 보이저엑스 와 같은 유명한 기업에서 온라인 구글미트를 통해 테크톡이 진행되거나

선착순으로 인원을 모집해 오피스 아워가 진행되었다. 나는 따로 오피스 아워에 참여하지는 않았다.

코세라 강의를 들으며 조금 지쳐있다가도 연사분들의 이야기를 들으면서 현장에서는 어떠한 문제를 풀고 고민하고 있는지 들을 수 있어서 이야기들이 더 와닿았다.

또 Google Developer Program의 운영자 분이 레쥬메 클리닉을 진행하시거나 이전 기수 수료생 분들의 부트캠프 경험담과 커리어 설계에 대해서도 이야기 들을 수 있었다.

감사하게도 이러한 프로그램이 끝나면 부트캠프 교육생분들이 잘 정리해서 디스코드 채널에 공유를 해준다. 시간이 안되어 참여하지 못해도 이러한 공유내용을 참고할수 있다.

![Discord Share](/assets/images/post-discord-share.png)

그리고 부트캠프 중간정도에 번개로 구글코리아 사무실에서 운영자 분들이 자리를 마련해주신 행사가 있었다.

격려의 이야기도 해주셨고 부트캠프를 같이 진행중인 동료들의 이야기도 들을 수 있었다. 무엇보다 네트워킹 시간에 함께 간단히 저녁을 먹으면서 부트캠프를 진행하면서 어려운 점이나 또 어떻게 진행중인지 서로 이야기 나눌수 있었다. 온라인으로 주로 진행하다보니 소속감이 낮아질 수 있는데 이러한 여러 행사들을 통해

서로 교류하고 또 중도에 포기하지 않고 계속해서 나아갈수 있게 운영진분들이 많은 부분에서 고려하고 힘써주셨다.

# 캐글

코세라 강좌를 수료하기 전부터 캐글 미션을 준비하기 시작했다. 코세라 강좌를 모두 다 듣고 시작할수도 있었지만 강의를 계속해서 듣다보니 긴장감이 떨어졌고 캐글을 병행하면서 조금 일찍 시작해도 무리가 없을 것이라 생각했다. 디스코드 채널에 올라오는 글들 중 조금 더 해보고 싶은 캐글팀 모집 글을 보고 참여하게 되었다.

**Child Mind Institute - Detect Sleep States ([링크](https://www.kaggle.com/competitions/child-mind-institute-detect-sleep-states))**

해당 캐글 대회는 어린이의 손목에 착용한 가속도계 데이터 를 사용해 정확한 수면의 시작과 종료를 예측하는 것이었다.

수면은 모든 연령대의 개인, 특히 어린이의 기분, 감정, 행동을 조절하는 데 매우 중요한 부분이고 성공적인 대회 결과를 통해 연구자들은 수면 패턴을 더 깊이 이해하고 어린이들의 수면 장애를 더 잘 이해할 수 있다고 한다.

참가한 캐글 대회에 대한 과정은 따로 글을 남기기로 하고 팀으로 캐글 대회를 어떻게 진행했는지에 대해 계속해서 이야기해보려 한다.

디스코드 채널을 통해 팀원이 모집되었고 참여하는 사람들만 들어가있는 디스코드 채널을 개설했다. 해당 채널에서 진행되는 상황들을 서로 공유하고 매주 일요일 저녁마다 디스코드 채널에 모여 각자 진행한 분석내용과 실험진행상황을 공유했다. 진행된 상황을 공유하고 다음주까지 진행할 사항을 간단히 논의했다. 

캐글이 처음인 팀원들을 위해 도움될만한 자료들도 함께 공유되었다. **Child Mind Institute - Detect Sleep States 대회** 에서 어려웠던 점은 데이터의 크기가 크다보니 모델을 수행할때에 메모리 이슈가 발생해 모델 수행이 더이상 진행되지 않았다. 제한된 자원사용량 안에서 모델을 수행하기 위해 데이터를 어떻게 전처리 할지에 대해 신경을 많이 썼다.

캐글 대회 미션에 대해 조금더 자세히  설명하면

캐글 대회 미션은  (팀 혹은 개인으로) 현재 진행중인 Featured competition 에 참여해 상위 40% 내 랭킹을 달성하거나

https://www.kaggle.com/competitions?hostSegmentIdFilter=1

팀으로 Playground, Community 대회에 참여해 상위 20% 내 랭킹을 달성해야 한다.

https://www.kaggle.com/competitions?hostSegmentIdFilter=8

Featured 는 상업적 대회로 최신의 자료와 조금더 높은 난이도를 보이고 Playground 는 학습용 대회로 캐글을 처음 시작하거나 학습을 위한 목적으로 개최되는 대회이다.

조금더 난이도가 있는 Featured 에 참여하게 되었고 처음 주어지는 overview, data 내용을 꼼꼼히 읽으면서 대회와 데이터를 이해하려 애썼다.

이후에는 Code, Discussion 페이지에서 공유되는 노트북과 의견들을 확인했다.

![Kaggle Overview](/assets/images/post-kaggle-overview.png)

캐글에는 전세계에서 데이터와 AI/ML에 관심있는 고수들이 참여하는 곳이기에 다양한 의견을 서로 교환되고 또 자신이 이해하고 공부한 것들을 공유한다.

베이스라인으로 삼기에 적절하다고 판단된 노트북을 선정해 해당 노트북을 하나씩 수행해보고 EDA 내용을 확인하면서 이해도를 높여나갔다.

![Kaggle Team](/assets/images/post-kaggle-member.png)

캐글 팀은 5명까지 구성할수 있고 5명까지 구성되어서 결과를 제출할수 있는 횟수도 5회 제공되었다. 처음에는 하루 제출횟수 5회가 적다고 생각되었는데 물론 잘못제출된 경우를 포함이다보니 조금은 신중하게 제출해야한다. 매일 5회씩 제공되다보니 여러 실험을 수행해 계속 제출하면 부족할수도 있지만 매일 5회를 채우지는 못했다.

새롭게 공유되는 인기있는 노트북을 참고해 계속해서 모델과 코드를 업데이트 하다보니 사용하는 모델도 다양해지고 데이터 전처리나 다른 코드 작성에 있어서도 효율성이 올라갔다.

최종적으로 제출된 결과는 private score/ public score 로 0.734 / 0.709 를 기록 , 총 1877팀 중 283 등 / 289 등 (15%) 으로 마감하였다. 

캐글 대회에 참여하면서 몇가지 배운 점은

- 캐글에서 제공하는 자원을 최대한 활용하되 활용할수있는 가용 GPU 자원이 있다면 좀 더 효율적인 실험이 가능
- weights & Biases([링크](https://wandb.ai/))와 같은 experiment traking 툴을 사용하면 실험 관리 및 설계가 더 편함
- 타임라인을 제대로 확인할 것, 이미 대회에 참여 중이라면 entry/ team merger deadline 은 크게 신경쓰지말고 Final Submission Deadline 만 신경쓸것
괜히 타임라인이라고 해서 집중을 늦추지말고 가장 마지막 dealine까지 계속해서 제출해볼 것


![Kaggle Timeline](/assets/images/post-kaggle-timeline.png)
![Kaggle Timeline2](/assets/images/post-kaggle-timeline2.png)

# 수료식

코세라 미션과 캐글의 여정이 마무리 되고 11월 30일 드디어 졸업식 당일이 되었다.

압구정에 있는 카페 건물을 대관해 1, 2층을 졸업식에 맞게 꾸민것을 보고 졸업식을 제대로 준비했다는 생각이 들었다.

건물 입구에 들어서자마자 간단히 성명을 확인하고 준비되어있는 졸업식을 기념하는 메달과 학사모, 졸업식가운, 리본이 준비되어있었다. 

구글 코리아 대표 분의 축사와 함께 머신러닝 부트캠프를 운영하시면서 우리에게 배움의 기회를 주기위해 노력하셨던 권순선님, 금빛누리님의 머신러닝 부트캠프 돌아보기 시간이 진행되었다.

무엇보다 좋았던건 이번 오프라인 행사에서도 앞서 부트캠프를 수료한 선배 분들이 직접 나와 어떻게 부트캠프 과정을 지나왔는지 그리고 그 이후에도 

어떠한 노력들로 새로운 커리어에서 활약하고 있는지 설명하는 시간이 있었다.

![Bootcamp Graduate Ceremony](/assets/images/post-bootcamp-graduate.jpeg)

끝으로 함께 부트캠프 과정을 보낸 동료들의 졸업생 소감 발표 시간이 있었는데 이 시간은 놀라움의 연속이었다.

부트캠프 기간동안 여러 사람들과 네트워킹을 진행하고 오프라인 행사에도 빠짐없이 참석 그 와중에도 캐글에서도 유의미한 성과를 보인 동료가 있었고

그리고 처음 도전하는 캐글대회에서 2인 팀으로 오랜 분석시간동안 데이터를 모으고 실험을 위한 자료들을 준비해

캐글 은메달을 수상한 동료도 있었다.

마지막 동료는 나와 조금은 비슷한 상황이라 더 몰입해서 발표를 듣게되었다. 육아와 학습을 병행하다 보니 매일 21시 -24시를 고정된 학습시간으로 확보해

게더타운에서 스터디모임을 계속해서 열고 꾸준히 학습해 나가는 동료까지 

다양한 곳 다양한 자리에서 AI/ML 에 대한 관심을 갖고 새로운 도전을 하고 있는 동료들의 모습이 너무나 자랑스럽게 느껴졌다.

졸업생 소감 발표를 끝으로 준비된 식사를 먹으며 네트워킹 시간을 가졌다.

부트캠프에서는 잘 준비된 이벤트를 통해 서로 명함을 교환하거나 또 건물입구에서 선택했던 스티커의 다른색을 모으는 이벤트 등 

졸업생들의 네트워킹을 장려하는 이벤트를 준비해주셨고

나는 캐글을 함께 했던 팀원들을 다시만나 그동안 있었던 이야기도 나누고 일과 학습을 병행하면서 쉽지않았지만 우리가 이런 졸업의 순간을 갖게되었음을

서로에게 축하해주는 시간을 가졌다.

23년 9월부터 시작했던 머신러닝 부트캠프가 종료되었지만 나는 캐글 대회를 참여하면서 느꼈던 아쉬움에

MLOps에 대해 더 파보고 싶다는 생각을 하게 되었다.

현재도 코세라의 MLOps강의를 비롯 관련된 책을 찾아서 따로 학습을 진행하고 있다.

지금까지 스타트업과 기업에서 SW 엔지니어로써, 또 DevOps 엔지니어로써, AI/ML 연구자로써 학습을 해왔던 경험을 살려

이 3개 분야를 잘활용할 수 있으면서도 흥미가 있는 분야가 MLOps 분야라 생각되었고

앞으로 어느정도 학습이 끝난이후에는 MLOps 관련해 도움이 되었던 자료도 정리해보고 또 여러 툴들도 다양하게 사용해봄으로써

MLOps 에 대해 이해하고 또 경험을 쌓을수있는 기회를 마련해보고 싶다.


![Google Bootcamp Goods](/assets/images/post-google-goods.jpeg)

![Google Bootcamp Final](/assets/images/post-bootcamp-final.jpg)



