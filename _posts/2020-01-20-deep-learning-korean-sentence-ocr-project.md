---
title: 딥러닝을 활용한 한글문장 OCR 프로젝트
categories:
  - ML
tags:
  - ml
  - project
  - ocr
  - retrospect
last_modified_at: 2020-01-20T22:00:00-00:00
---


**한글 및 한국어 정보처리 학술대회** 2019
===========================

**2019년 4월부터 10월까지 6개월동안 딥러닝 개인프로젝트를 진행했고 그 과정에서 주제선정부터 논문작성까지 배우고 느낀것들을 정리해보려 한다**

프로젝트의 시작
========

처음 개인프로젝트의 주제로 정했던 건 딥러닝을 활용한 시험 문제 예측 서비스 구현이었다. 미래문 이라는 일본 최초 AI를 활용한 자격시험 문제 예측 서비스 관련 기사를 읽고 미래문의 기록을 깨보자는 생각에서 출발했다.

[인공지능에 의한 국가시험문제 출제 예측 ‘미래문’의 개요](https://lh6.googleusercontent.com/f_BRvMYBp6OMP3V7JvCzKUakMqQb5nNxcWaiyEquXQSbvJWqkl8MroUJCjUFgBr_tG8_NgFZnqR4aCPQnc-4GD2-5TDB-Xfc0dQCvQaCgjWLCqTncBLhWM5M34ySJ8XJPRfGHiHN)

과거 기출문제 그리고 문제집, 위키백과 데이터를 이용해서 128개 정도의 카테고리로 분류를 하고 RNN을 사용해서 올해 출제될 문제의 카테고리를 추출하는 방식이었다. 5월 한달동안은 데이터 수집 및 분석을 하기로 마음먹고 공인중개사, 공무원시험, 국가자격시험 데이터를 모으기 시작했다.

우선은 주요타겟이 될 시험을 선정해야했고 나름의 선정기준을 정해야했다

*   가장 중요한 기출문제 데이터를 쉽게 구할수 있고 시험횟수가 최소 10회 이상
*   시험응시 인원이 많아서 예측결과가 실제로 도움이 될수있게
*   계속 시험이 유지되어야 (사법시험…)

선정기준을 고려했을 때 공인중개사 시험이 2018년까지 29회 진행되었고 기출문제도 2005년 이후(15회 분량)은 박문각 사이트에서 쉽게 구할수있었다. 연1회 시험이나 1,2차 5개과목 \* 과목당 40문항으로 계산했을 때 5지선택형이라 어림 계산만으로 3000문항 정도가 되었다. EBS 온라인 모의고사 데이터도 4년 \* 7회 \* 200문항 으로 5600 문항 분량이었다. 여기까지 모든게 순조로웠다.

하지만 발목을 잡는 문제가 생겼는게 데이터 전처리 부분이었다. 기출문제와 EBS 모의고사는 모두 pdf 파일 형식이나 모의고사 데이터는 글자인식이 가능한 타입이고, 기출문제는 단순 이미지 파일이라 데이터로 변환하기 위해선 수작업을 하거나 OCR 프로그램을 돌려야 했다

[stackoverflow: How to extract text from a PDF file?](https://stackoverflow.com/questions/34837707/how-to-extract-text-from-a-pdf-file)

이같은 문제는 많은 사람들이 해결방법을 찾고있는 문제이고 문자인식 성능이 그리 좋지않으나 한글은 더욱 성능이 좋지 않았다…

[최강 자격증 기출문제 전자문제집 CBT](https://www.comcbt.com/)  
검색을 통해 기출문제를 따로 모아두는 사이트를 발견했으나 여기에선 기출문제 복원이라는 (이미지 파일을 수작업으로 한글문서화) 작업을 진행하고 있었다.

OCR 이게 주제가 될 수 있을까?  
이미지로 되어있는 기출문제를 문자데이터로 변환가능한가?

이 때부터 OCR 관련 블로그와 서비스, 기술논문들을 찾아보기 시작했다.

*   OCR 관련 블로그  
    [아날로그 기상 데이터를 OCR로 디지털화할 수 있을까?](https://brunch.co.kr/@kakao-it/319)  
    [카카오 OCR 시스템 구성과 모델](https://brunch.co.kr/@kakao-it/318)  
    [텐서플로로 OCR 개발해보기: 문제점과 문제점과 문제점](https://brunch.co.kr/@kakao-it/304)  
    [딥러닝과 OpenCV를 활용해 사진 속 글자 검출하기](https://d2.naver.com/helloworld/8344782)
*   OCR 관련 논문 리스트  
    [Scene-Text-Understanding](https://github.com/tangzhenyu/Scene-Text-Understanding)
*   OCR 관련 서비스  
    [online OCR](https://www.onlineocr.net/)

리서치를 통해 기업에서도 풀고싶어하는 문제이고 영문에 비해 한글의 경우 이렇다할 서비스가 제공되지 않고 있어 충분히 진행해볼만한 프로젝트라는 생각이 들었다.

한글 데이터셋 준비
==========

기존 한글 OCR 데이터가 있는지, 있다면 사용할 수 있는 수준인지 확인이 필요했다. [ICDAR 2019 Task 1: Multi-script text detection](https://rrc.cvc.uab.es/?ch=15) 에 한국어 이미지 데이터가 있었으나 1000개 정도이고 짧은 단어로 되어 있었다.  
[KAIST Scene Text Database](http://www.iapr-tc11.org/mediawiki/index.php/KAIST_Scene_Text_Database) 는 카이스트에서 제공하는 이미지 데이터셋으로 디지털카메라 또는 휴대폰으로 촬영한 간판, 책표지, 기타 이미지였다. 하지만 생성된 일자가 2011년이고 진행하려는 한글문서 이미지와는 조금 차이가 있어서 활용하기에 힘들었다.

한글 데이터셋을 어떻게 구할지, 구하지 못하면 포기해야하나 하는 생각이 들었다. 궁하면 통한다고 깃허브에서 OCR 프로그램 트레이닝을 위한 데이터셋을 생성하는 소스 [TextRecognitionDataGenerator](https://github.com/Belval/TextRecognitionDataGenerator)를 발견했다. 해당 깃허브에서는 라틴어계열만 제공되었는데 다행인건 폰트와 사전데이터만 있으면 한글 이미지 데이터도 생성할 수 있었다. 폰트는 네이버 [나눔글꼴](https://hangeul.naver.com/2017/nanum)과 [폰트코리아](http://www.font.co.kr/yoonfont/free/main.asp) 에서 모은 99종의 폰트데이터와 국립국어원에서 한국어 학습용 어휘목록 데이터를 구해 사용했다. (5965개 단어)

한글 문장 데이터를 생성해야 해서 한글 입숨이나 반크이야기, 한국소설에서 문장을 가져올까 하다가 어휘목록의 단어들을 10개씩 랜덤으로 뽑으면 문장의 길이와 일치하는 임의의 문장이 생성될것이라 생각해 그대로 진행했다. 생성되는 문장타입은 지정한 방식에 따라 아무런 변형이 없는 기본, 기울기, 왜곡, 흐리게 처리, 배경이미지 포함으로 5가지 타입이 있었다. 문서이미지에 많이 분포할 것을 보이는 기본과 기울기 타입은 각 9000 개씩, 나머지 타입은 각 3000 개씩 생성해서 27,000 개의 한글문장 이미지 데이터셋을 생성했다.

각 이미지 데이터의 다양한 예시는 아래 깃허브에서 확인가능하다. 참고했던 TextRecognitionDataGenerator 에서도 손글씨체 타입이 있었고 프로젝트를 진행하는 중에도 네이버 클로바에서 [나눔손글씨 글꼴](https://clova.ai/handwriting) 을 제공해 추가로 연구를 진행한다면 손글씨 폰트를 포함해서 다양한 데이터셋을 확보해서 진행해보려 한다.

OCR 개요
======

OCR 모델은 크게 문자탐지와 문자인식으로 구성되고 이 둘을 따로 구분하지 않고 end to end 방식으로 연결해서 진행하는 방식이 있다. 문자탐지, Text Detection은 아래 논문을 참고했다

[TextBoxes: A Fast Text Detector with a Single Deep Neural Network](https://arxiv.org/abs/1611.06779)  
[PixelLink: Detecting Scene Text via Instance Segmentation](https://arxiv.org/abs/1801.01315)  
[FOTS: Fast Oriented Text Spotting with a Unified Network](https://arxiv.org/abs/1801.01671)  
[Character Region Awareness for Text Detection](https://arxiv.org/abs/1904.01941)

기존 문자탐지 모델은 (이미지 속 글자에 적합한 후보를 생성하고 필터링하고 그룹화하는) 우리가 처음 생각하기 쉬운 방식대로 진행하다보니 시간이 많이 소요되고 여러 단계로 나뉘어져 있어 튜닝이 어려웠다. 실시간으로 진행하는건 기대하기도 어려웠다. 하지만 물체감지에서 성능향상을 보인 SSD 아이디어를 가져와 TextBoxes를 사용해서 성능과 정확도 향상을 가져왔다. 그리고 TextBoxes에서 발견된 문제점을 해결하기 위해 의미분할을 사용한 PixelLink가, 문자탐지와 문자인식을 연결해 인식률을 높이기 위한 방법으로 FOTS가 제안되었다.

위 논문들을 읽고 일부논문은 요약정리를 시도했다. 그리고 깃허브에서 해당코드를 찾아서 수행해보기 시작했는데 스타를 다수 받은 깃의 경우에도 실제로 수행해보면 문제가 좀 많았다. loss 가 nan 으로 출력되기도 하고. 여러 모델 코드들을 수행하면서 훈련했던 모델이 날라가기도 하고 linux 명령어에 익숙치 않아 찾아보면서 하는데 시간이 많이 소요되었다.

문자인식, Text Recognition은 아래 논문을 참고했다.

[An End-to-End Trainable Neural Network for Image-based Sequence Recognition and Its Application to Scene Text Recognition](https://arxiv.org/abs/1507.05717)  
[Gated Recurrent Convolution Neural Network for OCR](https://papers.nips.cc/paper/6637-gated-recurrent-convolution-neural-network-for-ocr.pdf)  
[Focusing Attention: Towards Accurate Text Recognition in Natural Images](https://parksunwoo.github.io/ocr_kor2/Focusing%20Attention:%20Towards%20Accurate%20Text%20Recognition%20in%20Natural%20Images)  
[MORAN: A Multi-Object Rectified Attention Network for Scene Text Recognition](https://arxiv.org/pdf/1901.03003.pdf)  
[Text recognition (optical character recognition) with deep learning methods](https://github.com/clovaai/deep-text-recognition-benchmark)

문자인식은 앞서 문자탐지에서 넘어온 입력값에서 정확한 문자를 분별해내는 과정이다. 단일모델만으로는 한계가 있어서 CNN과 RNN을 합친 CRNN과 Gate + RNN + CNN 의 GRCNN이 제안되었다. Gate는 CNN과 RNN의 정보를 균형있게 제어하는 역할을 수행한다. 또한 어텐션 드리프트 문제를 해결하기 위해 어텐션 네트워크와 포커싱 네트워크를 합친 FAN(Focusing Attention Network)도 제안되었다.

그리고 이러한 문자탐지와 문자인식을 별개로 구분하지 않고 연결되는 하나의 모델로 구현한 것이 Aster, TextSpotter 이다

[ASTER: Attentional Scene Text Recognizer with Flexible Rectification](https://ieeexplore.ieee.org/abstract/document/8395027/)  
[An end-to-end TextSpotter with Explicit Alignment and Attention](https://arxiv.org/abs/1803.03474)  
[An Overview of the Tesseract OCR Engine](https://static.googleusercontent.com/media/research.google.com/ko//pubs/archive/33418.pdf)

한글 OCR에 대한 딥러닝 연구가 이전에도 진행되었으나 단일모델을 사용하고 글자단위의 이미지로 진행되어 한계점을 지니고 있었다. 기존 문자탐지와 문자인식 모델들에 앞에서 생성한 한글문장 데이터를 사용해서 성능을 확인해본결과 문자탐지는 잘 수행되었고 문자인식 부분에서 개선할 부분이 있다고 판단하였다.

모델구조와 실험결과
==========

문자탐지와 문자인식으로만 구분되던 것을 좀 더 세분화해서 변환, 특성추출, 시퀀스, 예측 4개모듈로 구분하였다.(모델구성은 아래 네이버 clova 팀의 논문을 참고) 변환 모듈은 TPS(Thin Plate Spline) 사용여부에 따라, 특성추출 모듈은 VGGNet, RCNN, ResNet 으로, 시퀀스 모듈은 BiLSTM 사용여부에 따라, 예측 모듈은 CTC(Connectionist Temporal Classification) 과 어텐션 으로 구성되었다. 따라서 4개 모듈조합은 전체 경우의 수가 2 x 3 x 3 x 2 = 24가지가 나와 한글 문장에 적합한 모듈조합을 찾고자 모든 조합에 대한 실험을 진행했다. [실험기록](https://github.com/parksunwoo/ocr_kor/issues)

24개의 모델을 하이퍼파라미터를 튜닝해가면서 가장 높은 정확도의 모델을 찾는 작업은 생각이상으로 많은 시간이 소요되었다. 한개 모델을 수행하는데 batch\_size 192에 num\_iter 를 300,000 회 기준이다보니 2–3일정도 시간이 걸렸다. Pytorch에서 Multi-GPU로 학습하는 게 실험시간을 줄이는데 무엇보다 중요하다고 생각했고 당근마켓 블로그의 [PyTorch Multi-GPU 제대로 학습하기](https://parksunwoo.github.io/ocr_kor2/[https://medium.com/daangn/pytorch-multi-gpu-%ED%95%99%EC%8A%B5-%EC%A0%9C%EB%8C%80%EB%A1%9C-%ED%95%98%EA%B8%B0-27270617936b](https://medium.com/daangn/pytorch-multi-gpu-%ED%95%99%EC%8A%B5-%EC%A0%9C%EB%8C%80%EB%A1%9C-%ED%95%98%EA%B8%B0-27270617936b)) 포스트를 참고했다. 결론적으로는 상황에 따라 다소 차이가 있긴하지만 Nvidia Apex를 사용하면 multi gpu를 활용할 수 있다는 내용이었다.

네이버 clova 팀의 [What Is Wrong With Scene Text Recognition Model Comparisons? Dataset and Model Analysis](https://github.com/clovaai/deep-text-recognition-benchmark) 논문에서 최고 정확도의 모델조합은 TPS-ResNet-BiLSTM-Attn (영문 단어기준 84%) 이었으나 한글문장 데이터로 실험한 결과 TPS-VGG-BiLSTM-Attn 으로 88.24% 의 정확도가 나왔다.

향후 연구과제 그리고 느낀점
===============

높은 정확도가 나왔지만 실시간 서비스에 활용되기에는 정확도와 시간을 더 줄여야할 것으로 보인다. 모델부분에서는 최신 언어모델을 활용해 모델조합을 더 늘려야 하고 데이터 부분에서는 얼마전 공개된 네이버의 손글씨체 폰트를 활용해서 더 다양한 한글문장 이미지 데이터셋을 확보할 수 있을 것으로 보인다. 실험이 어느정도 진행된다면 본래 진행하려던 한글문서로 범위를 확장해서 진행해보고 싶다.

이미지화 되어있는 기출문제를 디지털 문서로 변환하는데서 출발했지만 해당 문제는 실제로 기업 비즈니스에서도 현재 중요한 문제로 이야기되고 있다고 한다. -[AMA with 우정훈 (KPMG NY Data Scientist)](https://www.youtube.com/watch?v=h1Szcj0_kYU&list=PLIXnubKzTJX6-lOj5BkbrUy3GEqnqSz2W)  
계약서를 비롯한 기업의 여러문서들이 인쇄물로 보관되어있는 게 많고 이를 디지털화 하는 작업이 선행되어야 데이터분석 및 ML/DL 이 가능해진다. OCR 기술은 중요한 기술이었던 것이다.

논문을 학회에 제출해보면서 몇가지 알게된 것들이 있다. 학회는 우선 포스터발표와 세션발표가 있고, 세션발표는 2–30분정도 시간을 갖고 강의실에서 발표할 기회가 주어진다. 그리고 포스터발표는 자신의 연구내용을 A1 사이즈 전지에 정리해 넓은 공간에서 관심있는 사람들에게 설명할 수 있는 1–2시간 정도의 발표시간을 갖는다.

조금 충격적이었던 것은 학회논문을 개인이 혼자서 발표하는 케이스는 나말고는 없었다. 보통은 기업의 연구팀에서 오던지 아니면 팀원들과 함께 논문을 작성해서 제출하는 것이 보통이었다. 이럴 수 밖에 없는게 논문을 작성하는 건 시간투자가 많은 일이라 정확한 이해관계가 없이 단순 호기심으로만 진행하기에는 다소 무리가 있어 보였다. 그리고 딥러닝 관련 논문들은 논문내용을 잘 정리해서 작성하는 것과 진행하려는 실험을 코드를 작성해서 시간안에 원하는 결과를 가져오는 것이 구분되어있다. 따라서 팀원들끼리 각자의 역할을 정해서 분업화 하지 않으면 안된다는 생각이 들었다.

학회에 논문을 투고하면, 투고하는 논문양식에는 저자에 관한 정보없이 오로지 논문의 내용만 담긴다, 얼마간의 심사기간을 거쳐 (나의 경우 2주일) 심사위원의 심사점수 및 심사평과 함께 게재여부가 회신된다.

심사결과 : “게재가” 메일을 확인했을 때가 올해 가장 행복한 괴로운 순간이었다.

심사위원의 심사평에는 보완해야할 부분에 대한 내용도 기재가 된다.

위원회 멤버 #1 실험과 관련된 분석이 더 추가되었으면 좋을 것 같습니다.

위원회 멤버 #2 기존의 OCR모델과 비교하여 제안방법의 차이를 보다 분명히 언급하면 좋을 것 같습니다.

최종본 (Camera-ready Paper)를 작성하기까지는 이틀정도의 시간이 주어졌는데 부족한 부분을 보완하기 위해 거의 잠을 못잤다. 그래도 학회에 가볼수있다는 사실이 설레게 해서 학회발표일이 많이 기다려졌다.

대전 카이스트에서 열린 학회는 한글과 자연어처리에 관심많은 사람들이 발표하고 질문하는 장이었다. 다음에 기회가 된다면 더 견고한 논문을 작성해서 우수논문으로 채택되고 싶다는 생각을 해본다.

마지막으로 임해창 교수님의 강연에서 인상깊었던 내용을 적으며 긴글을 마치고자 한다.

> 과거의 연구들은 과거에 머물러있는게 아니라 현재 그리고 미래의 연구를 이해하고 발전시키는 데 밑거름이 될것입니다.

[딥러닝을 활용한 한글문서 OCR 연구: https://github.com/parksunwoo/ocr\_kor](https://github.com/parksunwoo/ocr_kor)
