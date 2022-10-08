---
title: 날씨알려주는 알람을 만들어보자 chap2
categories:
  - Dev
tags:
  - whether
  - alarm
  - jsoup
  - tts
last_modified_at: 2017-05-03T09:00:00-00:00
---

고려해야했던 부분들을 몇가지 짚어보면서 이야기하고자 한다

1\_제일 먼저 생각했던 부분은 어떤 날씨정보를 듣고싶은지였다

기상청 홈페이지에 접속만해봐도 현재날씨, 오늘/내일 예보, 중기예보와 같이 정말 다양한 정보들을 접할수있다

하지만 아침에 일어났을때 많은정보를 듣는게 꼭 도움이 되지는 않을듯했고

내가 듣고싶은정보는 현재지역에 해당하는 미세먼지, 온도, 현재날씨 정도였다

2\_다음으로 고려했던 부분은 날씨정보를 어디서 가져올것인가?

[기상청](http://www.kma.go.kr/weather/main.jsp)

[케이웨더](http://www.kweather.co.kr/main/main.html)

[네이버날씨](https://m.weather.naver.com/m/main.nhn)

우선 기상청 사이트는 기본적인 날씨정보들이 다양하게 나와있고

RSS 서비스가 제공되어 지역별로 원하는 날씨정보를 parsing해서 사용하기 편했다

자신이 원하는 지역정보를 선택하면 아래와 같은 url 정보를 얻을수있다

http://www.kma.go.kr/wid/queryDFSRSS.jsp?zone=1159068000 

뒤에 숫자부분은 지역별 코드값이고

기상청 RSS 정보만으로 부족한부분이 있어서 케이웨더 라이프스타일 예보 날씨개황부분을 추가로 가져오도록 하였다

아래의 소스를 살펴보면

기상청 url로 접속하여 오픈소스인 okHttp를 호출, 결과값을 가져오면 parseXML 함수를 호출하도록 하였다

parseXML 함수에선 XmlPullParser 를 활용하여 태그별로 구분해서 shortForecasts 라는 객체에 data를 저장하도록 했다

아래

doInBackground 함수는 케이웨더 사이트에서 특정부분의 정보를 가져오기 위해 호출하였다

html을 파싱하는데 편리한 jsoup 을 사용하여 원하는 부분의 문구만 가져왔다

```java
protected Long doInBackground(URL... urls) {             String url = "http://www.kma.go.kr/wid/queryDFSRSS.jsp?zone=4119071000";             OkHttpClient client = new OkHttpClient();             Request req = new Request.Builder().url(url).build();             Response res = null;              try {                 res = client.newCall(req).execute();                 parseXML(res.body().string());              } catch (IOException e) {                 e.printStackTrace();             }             return null;         }          protected Long doInBackground(URL... params) {              try {                 org.jsoup.nodes.Document doc = Jsoup.connect(url).get();                 forecastKor = doc.select(".lifestyle_condition_content").html();                 int idx = forecastKor.indexOf("<br />");                 forecastKor = forecastKor.substring(0, idx - 1);              } catch (IOException e) {                 e.printStackTrace();             }             return null;         } 
```

3\_날씨정보를 어떤방식으로 음성으로 출력할 것인가?

우선 화면에 필요한 정보들이 문자로 출력이 되면 tts가 동작하면서 해당문구들을 읽도록 하였다

한국어로 설정을 해주었고 화면의 특정부분을 클릭하면 문구가 나오면 음성이 나온다

```java
myTTS = new TextToSpeech(getActivity().getApplicationContext(), new TextToSpeech.OnInitListener() {             @Override             public void onInit(int status) {                 if (status != TextToSpeech.ERROR) {                     myTTS.setLanguage(Locale.KOREAN);                 }             }         });          myTTS = new TextToSpeech(getActivity().getApplicationContext(), new TextToSpeech.OnInitListener() {             @Override             public void onInit(int status) {                 if (status != TextToSpeech.ERROR) {                     myTTS.setLanguage(Locale.KOREAN);                 }             }         }); 
```

추가로 TTS를 검색하다 재밌는 사이트 한곳을 발견했는데

[문자를 음성으로 출력해주는 재밌는 사이트](http://text-to-speech.imtranslator.net/speech.asp?dir=ko)

안녕하세요, 제 이름은 미아입니다. 저희 언어 포털에 방문하신 것을 환영합니다! 원하시는 것이라면 무엇이라도 읽어드릴 수 있는 새로운 텍스트 음성 변환 서비스를 소개해 드리겠습니다. 저와 제 동료들은 영어, 중국어, 프랑스어, 독일어, 이탈리아어, 일본어, 한국어, 포르투갈어, 러시아어, 스페인어를 할 수 있습니다. 단어 또는 구문을 입력하시거나, 텍스트를 복사하여 붙여 넣기만 하시면 제가 읽어드리겠습니다. 텍스트를 지원 언어 중 어떤 언어로도 번역하실 수도 있으며, 제가 번역문을 읽어드리겠습니다. 발음 연습으로 외국어를 마스터하고 싶으신 경우도 있을 것입니다. 저를 따라 하기만 하세요! 재미있을 거에요!

생각보다 발음이나 속도가 나쁘지않았고 한국어뿐만 아니라 다양한 언어가 가능해서 놀라웠다

한번 들어가서 해보면 재밌을거같다

단순한 아이디어에서 시작한게 완성하고 이렇게 글로 남기는데 시간이 꽤 오랜시간걸렸다

항상 느끼는 거지만 무언가를 만드는건 기쁨이자 고통이다

훌륭한 개발자는 기술이 뛰어나기보다 인내력을 가지고 끊임없이 고쳐나가는게 아닐까 

이른 시간 내에 또 다른 아이디어 구현을 글로써 남겨보고자 한다.

\- 구현한 화면들
<figure>
  <img src="/assets/images/mimicekr1.png" alt="Trulli" style="width:900, height:800">
  <figcaption>Wheather Alarm1</figcaption>
</figure>

<figure>
  <img src="/assets/images/mimicker2.png" alt="Trulli" style="width:900, height:800">
  <figcaption>Wheather Alarm2</figcaption>
</figure>

\- 작업코드
https://github.com/parksunwoo/StopWatchForecast4
