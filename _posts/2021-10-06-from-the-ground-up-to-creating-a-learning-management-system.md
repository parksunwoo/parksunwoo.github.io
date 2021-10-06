---
title: "맨땅에서 AI학습플랫폼을 만들기까지"
categories:
  - Dev
tags:
  - django
  - lms
  - wagtail
  - howto
image:
  path: /assets/images/ground-up-lms.png
  thumbnail: /assets/images/ground-up-lms.png-small.png
last_modified_at: 2021-10-06T09:45:00-00:00
---

10월 2일- 3일 열린 파이콘 2021에서 AIFFEL 학습플랫폼 개발기를 발표하고 해당 내용을 글로 정리해보았습니다

---

저는 딥러닝과 파이썬에 관심이 많은 개발자이며
현재 모두의연구소 AIFFEL 개발팀에서 Django 기반으로 AI학습플랫폼을 개발하고 있습니다

AI 학습플랫폼, Learning Management System 을 밑바닥부터 만들면서
중요하다고 생각되었던 3가지를 발표의 주제로 선정하였습니다.
이 3가지 주제는 AI학습플랫폼의 주요 요구사항과 맞닿아 있습니다.

![AI학습플랫폼의 주요 요구사항](https://www.dropbox.com/s/rnuh1ehkppqj83f/Screen%20Shot%202021-10-06%20at%2012.21.26%20AM.png?dl=1)

AI학습플랫폼을 설계하면서 주요 사용자인 교육생, 에디터, 개발자의 요구사항을 먼저 확인했습니다.

AI학습플랫폼을 직접 사용하는 교육생 입장에서는 딥러닝을 빠르게 공부하고 싶어도
항상 환경설정이 진입장벽으로 작용했습니다.
그래서 환경설정 지옥에서 벗어나 딥러닝 코드를 웹에서 수행하고 결과를 바로 보고싶어 했습니다.

학습 콘텐츠를 제작하는 에디터 입장에서는
콘텐츠를 쉽게 교열 및 편집하고 콘텐츠 형상관리도 편하게 하고싶어 했습니다.

학습플랫폼을 개발하는 개발자 입장에서는 적은 resource에서도 개발진행이 가능하게
오픈소스 및 패키지가 풍부한 기술스택을 원했습니다.

개발팀은 이러한 3가지 요구사항을 만족하는 솔루션을 찾아야했고
Django 와 React 기반의 기술스택을 선택해 3가지 요구사항을 해결할수 있다고 판단하였고 개발을 시작했습니다 .
이제부터는 해당 문제들을 어떤 방식으로 풀어나갔는지에 대한 우리의 길고 긴 여정에 대한 이야기입니다

# **웹에서 딥러닝코드를 돌리게한다?!**

- 무거운 딥러닝코드를 환경설정없이 웹에서 동작하려면

어떻게 하면 웹에서 딥러닝코드를 동작시킬 수 있을까요?
차근차근 단계를 나누어서 이야기해보겠습니다

![How a Python interpreter runs code](https://www.dropbox.com/s/pna8swiafr9fyio/Screen%20Shot%202021-10-06%20at%2012.35.02%20AM.png?dl=1)

[https://www.quora.com/How-does-a-Python-program-get-executed-Is-there-any-concept-of-compile-time-and-run-time-in-Python](https://www.quora.com/How-does-a-Python-program-get-executed-Is-there-any-concept-of-compile-time-and-run-time-in-Python)

먼저 파이썬코드의 동작순서를 살펴봅시다

(1) 파이썬으로 작성한 프로그램을 실행하기 위해서는 </br>
(2) 먼저 해당 프로그램을 파이썬 가상 머신이라는 인터프리터를 통해 기계어로 변환해야 합니다. </br>
인터프리터는 내부적으로 컴파일을 수행하여 </br>
(3) 소스 코드를 기계어인 바이트 코드로 변환하고 확장자가 .pyc인 파일을 생성합니다. </br>
(4) 이렇게 확장자가 .pyc 인 파일이 생성되면 인터프리터에 의해 해석되어 코드가 실행됩니다 </br>

![Running Python code in web browser](https://www.dropbox.com/s/89vlhmhq7fwl7e8/Screen%20Shot%202021-10-06%20at%2012.35.42%20AM.png?dl=1)

[https://yasoob.me/2019/05/22/running-python-in-the-browser/](https://yasoob.me/2019/05/22/running-python-in-the-browser/)

그럼 파이썬 코드를 본인 노트북이나 데스크탑이 아닌
웹 브라우저에서 수행하려면 어떤 방법들이 있을까요?

파이썬 코드를 웹에서 동작시키는 방법은 크게 2가지 기준으로 구분할수있습니다

한가지는 어떻게 동작하느냐에 따라

- 파이썬 코드를 JavaScript 로 컴파일하는지
- 브라우저에서 파이썬을 직접 실행시키는지로 구분할 수 있고

다른 한가지는 언제 컴파일 되느냐에 따라

- (1) 컴파일 하는 시점이 페이지를 불러오기 전인지
- (2) 페이지를 불러 올 때인지
- (3) 페이지를 불러오고나서 컴파일 하는 것인지

로 구분할 수 있습니다.

![TRANSCRYPT](https://www.dropbox.com/s/87xgsj86nlc4yw4/Screen%20Shot%202021-10-06%20at%2012.39.13%20AM.png?dl=1)

[https://www.transcrypt.org/live/transcrypt/demos/plotly_demo/plotly_demo.html](https://www.transcrypt.org/live/transcrypt/demos/plotly_demo/plotly_demo.html)

어떻게 분류하는지 기준을 살펴봤으니 이제 각 방법들을 좀 더 자세히 들여다봅시다

처음 예제는 TRANSCRYPT를 사용한 예제입니다

[https://github.com/qquick/Transcrypt](https://github.com/qquick/Transcrypt)

Transcrypt는 Python으로 작성된 파일을 transcrypt 명령어로 자바스크립트 버전의 js로 변환합니다.
이 변환작업은 브라우저가 동작하기 전에 진행됩니다.

버튼 이벤트를 통해 간단히 바꿔보는것도 가능하지만
이미 컴파일된 코드라 미리 정의된 이벤트만 사용가능 합니다

보여지는 샘플예제는 transcrypt로 plotly.js 를 브라우저에서 직접 사용해 도표를 표시한 예제입니다
컴파일된 Transcrypt 코드는 매우 작고 빠르지만 예제의 plotly.js 와 같이 원래 무거운 라이브러리를 사용할때는
로딩하는데 다소 시간이 걸릴 수 있습니다

두번째 예제는 BRYTHON 입니다

[https://github.com/brython-dev/brython](https://github.com/brython-dev/brython)

script tag "text/python" 을 사용하며 javascript를 작성하는 방식과 동일합니다
Brython은 파이썬 코드를 javascript 로 컴파일해서 생성된 코드를 실행합니다
이런 변환 과정은 전체 성능에 영향을 미치기 때문에, Brython은 사용자가 원하는 만큼의 성능이 안나올수도 있습니다

다음 예제는 SKULPT 를 사용한 예제입니다

[https://github.com/skulpt/skulpt](https://github.com/skulpt/skulpt)

아주 단순한 코드이지만 간단히 입력값을 수정해서 바뀌는 결과값을 보는 것이 가능합니다
Transcrypt, Brython 과 달리 웹상에서
파이썬 프로그래밍 환경을 구축하는데 더 포커싱 두었습니다.

![Pyodide](https://www.dropbox.com/s/zfh3727uswza8s3/Screen%20Shot%202021-10-06%20at%2012.41.52%20AM.png?dl=1)

[https://alpha.iodide.io/notebooks/300/](https://alpha.iodide.io/notebooks/300/)

쥬피터 노트북은 데이터를 바로 동작해보고 시각화해보고 다양하게 사용할수 있지만
어딘가에 호스팅된 서버가 있어야합니다

Pyodide는 

[https://github.com/pyodide/pyodide](https://github.com/pyodide/pyodide)

Mozilla WebAssembly Cpython 프로젝트로
웹어셈블리 안에서 NumPy, SciPy, Matplotlib, Pandas 등 패키지들이 재컴파일 되어있어 바로 사용할 수있습니다
이러한 결과로 왼쪽에서 보는 것처럼 Jupyter Notebook을 브라우저에서 실행시킬 수 있습니다.

![Jupyter-like model Architecture](https://www.dropbox.com/s/rp73ujpbmmac2t9/Screen%20Shot%202021-10-06%20at%2012.43.07%20AM.png?dl=1)

[https://jupyter.readthedocs.io/en/latest/projects/architecture/content-architecture.html](https://jupyter.readthedocs.io/en/latest/projects/architecture/content-architecture.html)

Jupyter Notebook 이야기가 나왔으니
Jupyter Notebook 모델 아키텍쳐를 잠시 살펴봅시다

사용자는 브라우저를 통해 Jupyter Notebook을 사용합니다.
Jupyter Notebook 서버는 커널 그리고 Notebook 파일과 연결되어 있습니다

Notebook 서버와 ipython 커널은 zeroMQ 소켓을 통해 json 메시지 형태로 통신을 하고
Notebook 서버는 브라우저와 HTTP Websocket 으로 다시 통신합니다

Notebook 서버는 Notebook 파일을 불러오고 저장하는데만 신경쓰고
커널은 Notebook 파일을 신경쓰지않고 주어진 코드를 실행후 결과값을 보내는 것만 관여합니다.

그런데 말입니다, 웹 브라우저와 JavaScript 로 간단한 파이썬 코드를 넘어 어떤 코드까지 돌릴 수 있을까요?
앞서 살펴본 Pyodide 는 NumPy, SciPy, Pandas 그리고 Matplotlib 비롯한 75종의 패키지를 사용할 수 있다고 합니다

다시 처음으로 돌아와서, 저희 개발팀이 만드려고 했던건 AI학습플랫폼,

그러니깐 간단한 Python 코드말고
Deep Learning 코드를 웹 브라우저에서 돌아가게 해야합니다.
브라우저에서 동작하는게 Python 코드가 아닌 Deep Learning 코드라면 어떻게 될까요?

대표적인 머신러닝 프레임워크 TensorFlow,
TensorFlow를 원활히 수행해보려면 GPU 카드가 탑재된 GPU 사용기기가 필요하고,

GPU 카드가 탑재된 노트북은? 게이밍 노트북?!
네 여러분이 상상한 그 게이밍 노트북 맞습니다.

![Deep Learning in LMS](https://www.dropbox.com/s/je26xfwoqzgl8yv/Screen%20Shot%202021-10-06%20at%2012.44.51%20AM.png?dl=1)

실제로 LMS의 초기버전은 교육생에게 미리 제공된 게이밍 노트북에
Jupyter Notebook 환경을 사전에 셋팅하게 했습니다.
게이밍 노트북 로컬환경의 Jupyter 엔드포인트를
LMS 브라우저와 연결해 딥러닝학습을 가능하게 구현했습니다.

게이밍 노트북이라는 강력한 무기를 사용해
LMS에서 딥러닝학습을 가능하게하는 시나리오는 달성했습니다.

하지만 무거워진 교육생들의 가방만큼이나
학습 전 환경셋팅은 아직 진입장벽으로 남아있었습니다.

![Deep Learning in LMS](https://www.dropbox.com/s/8fcyxjhdey8m407/Screen%20Shot%202021-10-06%20at%2012.46.18%20AM.png?dl=1)

[https://github.com/executablebooks/thebe](https://github.com/executablebooks/thebe)

바로 앞에서 간단히 로컬환경의 Jupyter 엔드포인트를
LMS 브라우저와 연결했다고 했습니다. 어떻게 연결했는지 좀 더 자세히 설명드리겠습니다.

앞서 파이썬 코드를 웹브라우저에서 동작시키는 여러 방법들을 살펴봤습니다.
하지만 LMS 브라우저와 로컬 Jupyter 엔드포인트를 연결하는데는 다른 방법이 필요했습니다.

오랜 검색끝에 우리가 찾은 방법은 Thebelab 입니다.
Thebelab은 정적인 HTML 페이지를 변환해 코드를 직접 수정하고
실행시킬 수 있게한 오픈소스입니다
kernelOptions - serverSettings 를 설정하면
Jupyter Notebook 서버를 Thebelab에 직접 연결할 수 있습니다.

이 Thebelab 오픈소스를 활용해 사용자의 로컬 Jupyter 노트북 환경과 LMS 브라우저를 연결했습니다.
Thebelab을 사용한 초기 버전에서는 사용자가 입력을 자유롭게 할 수 있고 코드 결과값을 바로 확인할 수 있는 장점이 있었습니다.

![좀 더 가볍고 빠른 방식으로 - Jupyterlab](https://www.dropbox.com/s/iqo74ycwseqjdyk/Screen%20Shot%202021-10-06%20at%2012.47.02%20AM.png?dl=1)

그러나 LMS에선 프론트 프레임워크로 React를 사용하고 있었습니다.
Thebelab을 커스텀해서 사용해야했고 커널과 세션의 상태를 지속적으로 체크해
사용자 이벤트에 반응하기 위해선 불가피하게 iframe으로 렌더링 해야만 했습니다.

이러한 한계로 사용자 입장에서는 LMS 사용시간이 늘어남에 따라 브라우저 속도 및 성능 저하가 나타났습니다.

사용자의 LMS 사용성과 브라우저 속도향상을 위해 초기 LMS 버전에 도입했던 Thebelab을 걷어내야했고
Jupyterlab의 패키지와 api를 직접 활용한 버전으로 프로젝트를 업그레이드 하였습니다.

![좀 더 가볍고 빠른 방식으로 - Docker & Cloud](https://www.dropbox.com/s/zlx6j76lwf4gkz2/Screen%20Shot%202021-10-06%20at%2012.47.55%20AM.png?dl=1)

여러 시도들을 통해 현재는 학습콘텐츠별 맵핑되는 도커이미지를
컨테이너 환경으로 직접 구성해
학습콘텐츠에서 필요로 하는 패키지 환경이 구축된 클라우드 엔드포인트와 LMS 브라우저를 직접 연결합니다

교육생은 더이상 로컬환경에 따로 딥러닝 학습환경을 설정하지않아도 되고
몇번의 클릭만으로 연결된 클라우드 환경에서 딥러닝 학습을 손쉽게 진행할 수 있습니다.

LMS 유저의 사용성이 높아진 것은 물론
덤으로 무거운 게이밍 노트북에서 해방되어.. 이제는 가벼운 일반노트북에서도 학습이 가능해졌습니다

# **학습플랫폼의 한 축 콘텐츠관리 시스템 (CMS)**

- Django Wagtail 패키지를 커스터마이즈해서 React와 API 통신

학습플랫폼을 처음 듣게되면 진도율 및 학습스케쥴 관리를 먼저 생각하게 됩니다.

우선 교육생 개인별로 서로 다른 학습 진행률이 기록되어야 하고
코스별로 서로 다른 스케쥴링이 되어야합니다.
해당 부분은 한번 셋업이 되면 스케쥴 로직 및 관리를 위한 코드변경이 잦지 않습니다.

실제로 학습플랫폼을 개발하고 운영해보니
학습관리만큼이나 학습콘텐츠를 동시에 편집/작업하고 주기적으로 업데이트하고 형상관리하는 콘텐츠관리가 중요한 부분이었습니다.

다음 주제는 학습플랫폼에서 콘텐츠관리를 어떻게 구현했는지에 대한 내용입니다

![Django Wagtail](https://www.dropbox.com/s/y1x1szub9dxxoym/Screen%20Shot%202021-06-30%20at%2012.39.30%20AM.png?dl=1)

Wagtail 은 
[https://github.com/wagtail/wagtail](https://github.com/wagtail/wagtail)

Django에서 제공하는 CMS(contents management system) 으로
콘텐츠를 블록형태로 쉽게 구성하고 자신이 원하는 블록들로 커스터마이즈하기 쉽습니다.
권한관리, 이미지 및 문서관리, 검색, 형상관리 등 CMS 의 대부분의 기능이 구현되어있습니다.

그리고 무엇보다 StreamField가 있는데요.
고정된 형태가 아니라 콘텐츠 형식에 맞게 다양한 블록타입을 혼합해서 사용가능한게 특징입니다.
다양한 블록들은 Wagtail의 admin 화면에서 손쉽게 추가 및 편집 가능합니다

이미지, 비디오, 지도, 차트, 코드 및 우리가 원하는 커스텀 블록을 생성 및 추가할 수 있습니다
streamfield의 블록들은 데이터베이스에 저장될때 json 형태로 저장되고 API 활용도 가능합니다

![Wagtail API 엔드포인트](https://www.dropbox.com/s/2onzzcvpi6jofn2/Screen%20Shot%202021-10-06%20at%2012.51.39%20AM.png?dl=1)

그렇다고 해도 Django 기반의 Wagtail 을 React에서 바로 사용하기는 쉽지않았습니다
Django 와 React는 기존에 rest api 방식으로 통신하고 있었는데
LMS 내에서도 학습컨텐츠를 조회하고 사용자의 입력을 저장하는 방식에 rest api 방식을 사용했습니다

Wagtail 은 기본적으로 api 엔드포인트를 제공합니다
여기 예제에서는
pages 는 PagesAPIViewSet을
images 는 ImagesAPIViewSet 을
documents 는 DocumentsAPIViewSet 을 활용해
api_router에 각각의 엔드포인트를 등록합니다

다음으로 [urls.py](http://urls.py/) 에서 api_router를 각각의 매칭하려는 url path에 입력하면
/api/v2/pages/ 에선 페이지를
/api/v2/images/ 에선 이미지를
/api/v2/documents/ 에선 도큐먼트를 api를 통해 조회할 수 있습니다

![Page모델 그리고 BaseStreamBlock](https://www.dropbox.com/s/qkot5vcvbq7y6f9/Screen%20Shot%202021-10-06%20at%2012.52.24%20AM.png?dl=1)

Page의 body 필드 내에 StreamField 를 적용하면
여러가지 타입의 블럭을 조합해 새로운 필드를 만들 수 있습니다
예시에서는 다음와 같이 선언해서 사용했습니다.

위 코드를 간단히 설명하면,
Page 모델을 상속한 BlogPage 모델은 컬럼으로 date, intro, body를 가지고 있습니다.
body 컬럼은 StreamField 타입으로 BaseStreamBlock을 지정했고 오른쪽에 있는 코드가 BaseStreamBlock class 입니다.

BaseStreamBlock class 안에는
heading, paragraph, image, block_quote, embed, jupyter, text, document 블럭 등이 있습니다.
Wagtail 어드민 화면에서 원하는 블럭을 선택후
블럭내용을 작성하면 해당하는 블럭 형태에 맞춰 컨텐츠가 나오게 됩니다

![React와 Rest Api로 통신하는 법](https://www.dropbox.com/s/u5lei498fb42dgp/Screen%20Shot%202021-10-06%20at%2012.52.57%20AM.png?dl=1)

LMS의 기술스택은 백엔드는 Django, 프론트는 React 로 구성되어있어
rest api 방식으로 학습콘텐츠 데이터를 주고받는다고 앞서 말씀드렸습니다.

Page 모델과 LMS에서 사용하는 학습콘텐츠를 맵핑하기 위해 slug를 활용해서 진행했습니다.
학습콘텐츠의 제목을 slugify 해서 슬러그를 자동으로 생성했고
프론트 화면에선 학습콘텐츠의 슬러그 정보를 이용해 콘텐츠 context를 json 형태로 가져옵니다.

오른쪽 위에 있는 body 부분을 자세히 보면 다양한 블럭들의 조합으로 구성되어있으며
프론트 화면에선 block의 type 으로 분기해서 다양한 블럭들을 렌더링하고 있습니다.
다양한 블럭들을 원하는 형태로 사용자 화면에 표기할 수 있습니다.

# **Django의 탄탄한 packages 없는게 없다**

- **바퀴를 재발명하지 말자**

Django packages 에는 없는게 없습니다.
초기 LMS를 개발할때부터 Django 패키지 도움을 많이 받았습니다.

![유용했던 Django Packages](https://www.dropbox.com/s/1cmhiz4jhcex8l3/Screen%20Shot%202021-06-30%20at%201.04.45%20AM.png?dl=1)

[https://djangopackages.org/](https://djangopackages.org/)

Django package 에서 auth, task job, filter, 검색, notification 등
사이트를 개발하면서 필요한 거의 대부분의 기능을 Django package에서  조회할 수 있었습니다.
그중에서도 특히 유용했던 몇가지 패키지들을 소개하는 것으로 마지막 주제에 대한 이야기를 해볼까 합니다

![유용했던 Django Packages](https://www.dropbox.com/s/8kriraorlwxa5xz/Screen%20Shot%202021-10-06%20at%2012.54.40%20AM.png?dl=1)

- Django REST framework
Django만으로도 사이트를 만들수 있지만 좀 더 완성도 높은 사이트를 개발하기 위해선 Django는 백엔드에 집중하고
React, Vue 와 같은 프론트 프레임워크를 별도로 구성해야합니다.
    
    프론트와 통신하기 위해선 django로 rest api 를 생성해야하고 Django rest framework는 이때 꼭 사용해야 할 패키지입니다.
    별도로 제공하는 브라우저 화면에서도 간단히 api 조회 및 생성, 수정을 할 수 있다는 점이 장점입니다.
    
- Django Pydenticon
identicon은 깃허브나 홈페이지 프로필에서 이미지가 등록되지 않은 사용자의 구별되는 식별 이미지를 말합니다.
LMS 에서도 사용자 프로필 기능이 있어서 사용자의 디폴트 이미지로 pydenticon이 유용하게 사용되었습니다

- Django Background Tasks
시스템을 운영할때에 비동기로 작업을 해야할 때가 있습니다.
LMS에서는 과정별 마감시한이 지나면 과정의 학습상태가 지연으로 변경되어야 하는데 이런 경우에 사용할 task queue가 필요했습니다.
Task queue로 CELERY가 유명하지만 background tasks는 설정이 쉬워 간단한 task를 등록해야한다면 background tasks를 추천합니다.

- Django Hijack
hijack 은 재미난 이름대로 관리자가 사용자 화면에 로그인없이 직접 접속할 수 있게 하는 기능입니다.
시스템 관련 이슈 리포트를 접하면 로그도로 확인할 수 있지만
로그로 확인되지 않는 specific한 이슈들은 같은 상황을 재현하기가 힘들었습니다.
hijack 기능을 사용하면 django-admin 화면에서 해당 유저를 찾아 우측 끝에 생성된 버튼을 클릭하면 해당 유저 view로 시스템 화면에 접속 가능합니다.

![Django survey customize](https://www.dropbox.com/s/sf28w509s28qf65/Screen%20Shot%202021-10-06%20at%2012.55.22%20AM.png?dl=1)

LMS에서는 학습이 끝날 때마다 학습 컨텐츠에 대한 설문평가를 제공합니다.
설문조사 기능을 직접 만들수 있었지만 생산성을 높이고자 django-survey-and-report 패키지를 사용하게 되었습니다.
django-admin 화면과 연동되어 설문내용을 입력하고 생성할 수 있습니다.
django survey 패키지를 예로 들어 custom 해서 사용했던 부분을 설명해보고자 합니다

django-survey-and-report 패키지 models에는 Answer 테이블이 있습니다
Answer 테이블을 custom 해서 사용하는 경우
먼저 프로젝트의 기존 사용중인 폴더 (여기서는 core) 하위에 survey 폴더를 직접생성합니다
그리고 나서  custom 이 필요한 파일들을 추가합니다.

예제에서는 [models.py](http://models.py/), [seriazliers.py](http://seriazliers.py/), [urls.py](http://urls.py/), [views.py](http://views.py/) 를 직접 생성해서 추가했습니다
Answer 테이블에 추가하려는 필드또는 메서드를 추가하고
이어서 각각의 파일을 차례대로 수정해서 Api 연결을 했습니다.
이후 시스템에서 survey를 사용할때에 django-survey-and-report패키지 그대로의 survey가 아닌
커스텀한 survey를 바라보게 되어 필요에 맞게 변경해서 사용할 수 있었습니다.

![모두의연구소 AIFFEL 개발팀](https://www.dropbox.com/s/7yyty06w53ddm5v/Screen%20Shot%202021-10-06%20at%2012.56.03%20AM.png?dl=1)

짧은 시간내 여러 내용을 담다 보니 이야기하지 못한 부분들이 있어 아쉽습니다
발표와 관련하여 궁금하신 부분은 문의주시면 답장드리겠습니다

모두의연구소 AIFFEL 개발팀은 저희가 가진 기술로 AI학습의 진입장벽을 낮추려고 합니다
AIFFEL AI학습플랫폼을 함께 만들어 갈 개발자분을 찾고 있습니다. 많은 관심 부탁드립니다

발표 들어주셔서 진심으로 감사드립니다




