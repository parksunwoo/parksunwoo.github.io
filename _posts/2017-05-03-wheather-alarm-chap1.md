---
title: 날씨알려주는 알람을 만들어보자 chap1
categories:
  - Dev
tags:
  - whether
  - alarm
  - android
  - mimicker
last_modified_at: 2017-05-03T09:00:00-00:00
# layout: posts
# actions:
#   - label: "Learn More"
#     icon: github  # references name of svg icon, see full list below
#     url: "https://github.com/parksunwoo/StopWatchForecast4"
---

날씨알려주는 알람 아이디어를 실현시키기 위해 
​
오래전 공부하던 안드로이드 교재들을 꺼내서 필요한 정보들을 수집하기 시작했다
​
 먼저, 안드로이드 개발에 대한 감을 익히기 위해 검색을 통해
​
간단히  stopWatch를 따라서 만들어보았다
​
튜토리얼을 따라 자신에 속도에 맞춰서 함께 하다보면 감을 익히기에는 괜찮은듯했다
​
\- [안드로이드 앱 만들기 스톱워치1](https://www.youtube.com/watch?v=U34sMlcMZV4)
​
나의 계획은 완성된 stopWatch 어플에서 시작버튼을 눌러 동작을 시킨다음
​
정지버튼을 클릭하면 내가 원하는 날씨정보를 가져와서 아래 남는화면에 날씨에 관한 정보를 보여주고
​
그것을 TTS (Text-To-Speech) 기능을 활용하여 음성으로 들을수 있게 구현하는 것이었다
​
크게 2가지 부분으로 나누어서 진행을 했다
​
1\. 적당한 알람 찾기
​
2\. 날씨정보를 가져와서 음성으로 읽어주는 기능 
​
갤럭시 휴대폰에 탑재되어있는 기본 알람을 사용하고 싶었으나
​
오픈소스를 이용해서 만들었다는 정보만 있을뿐 
​
기본 알람의 소스를 구하지 못했다
​
현재 갤럭시7을 사용하는 중인데 1-2달전 업데이트된 소프트웨어 기능중에
​
알람 종료시 현재 시간 읽어주기 기능이 추가되어 단순한 기능이었지만 흥미로웠다
​
(날씨 읽어주는 기능을 개발하기 시작한 동기가 되었다고 할까)
​
알람을 구현한 오픈소스가 있을거라는 생각에 깃허브를 검색해보기 시작했고
​
그 결과, MS에서 예전에 진행한 [Mimicker Alarm](https://github.com/parksunwoo/ProjectOxford-Apps-MimickerAlarm) 을 찾았다!
​
알람의 기본 기본기능들이 잘 구현되어있을뿐 아니라 알람을 종료하면 Mimic 이라 불리는 단순한 게임들을
​
완료해야 알람이 정상적으로 종료되게 된다
​
셀카사진을 통해 제시하는 표정을 지어야하는 Express yourself with [Emotion API](https://www.projectoxford.ai/emotion)
​
해당하는 색의 사물을 찾아야하는 Color capture with [Vision API](https://www.projectoxford.ai/vision)
​
제시되는 문장을 정확히 읽어야하는 Tongue twister with [Speech API](https://www.projectoxford.ai/speech)
​
이 외에도 자신이 원하는 Mimic을 직접 구현하여 알람에 추가할수가 있다는 등 
​
내가 구현하고자하는 날씨알림 Service와의 연계에 있어서도 적합해 보였다
​
안드로이드 프로젝트를 설정하는데 초반에 어려움을 겪었지만
​
\- GitHub에서 안드로이드 프로젝트 다운로드시 설정
​
Mimicker Alarm 소스를 다운받아  기존에 작성된 소스를 살펴보며 app의 기능과 구조를 파악하기 시작했다
​
날씨정보를 가져와 음성으로 들을수있게 하는 기능을 기존 Fragment를 변형해서 만들어가기로 마음을 먹었고
​
작업을 시작했다


\- 작업코드
https://github.com/parksunwoo/StopWatchForecast4
