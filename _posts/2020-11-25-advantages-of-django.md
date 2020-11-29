---
title: "Django가 좋은 백 한가지 이유"
categories:
  - Django
tags:
  - package
  - admin
  - documentation
last_modified_at: 2020-11-25T08:29:00-00:00
---

Django를 이용해 웹 어플리케이션을 개발하면서 기존에 사용했던 Java 와 다른 장점들을 느낄 수 있었다. Django 의 장점이 많지만 그 중에 몇가지 정도만 설명해보려 한다

---
<br><br>
## Django는 다양한 Package 들을 제공한다.

![djangopackages.org](https://www.dropbox.com/s/mlli1pqdbzdxwbw/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-17%20%EC%98%A4%EC%A0%84%207.05.17.png?raw=1)

[https://djangopackages.org/](https://djangopackages.org/)

Django Package 들은 모듈화 되어 있어서 기존 프로젝트에 쉽게 추가할 수 있다. 만들고자 하는 기능들을 직접 밑바닥부터 구현할 수도 있지만 package 들을 잘 활용하면 개발 시간을 단축할 수 있음은 물론 생산성도 높일 수 있다. 위에 보이는 djangopackages 사이트에서 간단히 카테고리를 선택하면 해당 카테고리에 대한 설명이 나오고 카테고리에 속한 패키지들이 차림판 처럼 나온다. 패키지 별로 star 와 repo, fork 수 그리고 주요 특징, 패키지를 체험해 볼 수 있는 데모 사이트를 확인해보고 자신이 원하는 패키지를 선택하면 된다.

프로젝트에 사용했던 주요 Package들을 정리해보면 아래와 같다.
<br><br>
### wagtail

![wagtail site](https://www.dropbox.com/s/acjxh9hbj9m0r9o/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-18%20%EC%98%A4%EC%A0%84%206.51.55.png?raw=1)

LMS(학습관리시스템 : Learning Management System) 를 구현할때 무엇보다 CMS(컨텐츠관리시스템 : Contents Management System) 를 어떻게 구현할지가 가장 큰 고민이었다. wagtail을 활용하여 블로그를 만든 사례는 있었지만 다른 시스템과 연계해서 만든 사례는 찾기 힘들었다. 

 그러나 wagtail 을 꼼꼼히 확인해 본 결과 잘 사용한다면 LMS와 연계 시킬 수 있겠다는 계산이 나왔다. wagtail과 연계하여 개발한 부분은 이후 다른 블로그에서 설명하고자 한다.

wagtail 은 Django-Admin 화면 과는 별도의 wagtail Admin 화면을 제공한다. 

![wagtail Admin](https://www.dropbox.com/s/xqo4t9l1w6rnceo/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-18%20%EC%98%A4%EC%A0%84%206.59.16.png?raw=1)

기본으로 제공하는 Page 모델을 상속받아 사용자 페이지 모델을 정의할수 있고 Page의  body 는 StreamField 를 적용하면 여러가지 타입의 블럭을 조합해 새로운 필드를 만들 수 있다. 진행했던 프로젝트의 경우에는  아래와 같이 선언해서 사용했다.

```python
# model.py
class BlogPage(Page):
    date = models.DateField(verbose_name = ("Post date"), default=datetime.date.today)
    intro = models.CharField(max_length=250, default="-")
    body = StreamField(
        BaseStreamBlock(), verbose_name="Page body", blank=True
    )

# 위 body의 StreamField 를 구성하는 BaseStreamBlock의 상세구조
class BaseStreamBlock(StreamBlock):
    """
    Define the custom blocks that `StreamField` will utilize
    """
    heading_block = HeadingBlock()
    paragraph_block = RichTextBlock(
        icon="fa-paragraph",
        template="blocks/paragraph_block.html"
    )
    image_block = ImageBlock()
    block_quote = BlockQuote()
    embed_block = EmbedBlock(
        help_text='Insert an embed URL e.g https://www.youtube.com/embed/SGJFWirQ3ks',
        icon="fa-s15",
        template="blocks/embed_block.html")
    jupyter_block = JupyterCode(label='Code')
    form_block = WagtailFormBlock()
    markdown = MarkdownBlock(icon="code")
    gallery_block = ThumbnailGalleryBlock()
    equation_block = TextBlock()
    document_block = DocumentChooserBlock()
```

위 코드를 간단히 설명하면,

Page 모델을 상속한 BlogPage 모델은 컬럼으로 date, intro, body를 갖고있다.
body 컬럼은 StreamField 타입으로 BaseStreamBlock을 지정했고 그 아래 코드가 BaseStreamBlock class 이다. heading, paragraph, image, block_quote, embed, jupyter, form, markdown, gallery, equation, document 블럭이 있다. wagtail 에디터 화면에서 원하는 블럭을 선택후 내용을 작성하면 해당하는 블럭 형태에 맞춰 컨텐츠가 나온다.

![wagtail BaseStreamBlock](https://www.dropbox.com/s/7knssksr35rhpo4/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-22%20%EC%98%A4%ED%9B%84%2011.59.10.png?raw=1)

PAGE BODY 부분에서 다양한 블럭을 선택할 수 있다

wagtail 의 장점은 작성한 컨텐츠를 쉽게 검색 및 탐색할 수 있게 구성되어있고 작성한 컨텐츠를 미리보기하거나 발행, 임시 저장 할 수도 있다. 또한 권한 설정이나 검색, 사이트 구성과 같은 관리자 기능도 충실하다.  SEO 설정하는 부분도 있다. Django 로만 사이트를 만든다면 wagtail 을 사용해 그 기능을 100% 활용할 수 있을 것이다.  (진행했던 프로젝트의 경우 Django&React 로 진행해 wagtail의 코드를 그대로 사용하지 못해 아쉬움이 남았다)


<br><br>
### Django REST framework

Awesome web-browsable Web APIs.

![Django REST framework](https://camo.githubusercontent.com/9fdda978b33bd03453d9c7198cbfcb7b25df622e583fe1a3756b7edfbda7730a/68747470733a2f2f7777772e646a616e676f2d726573742d6672616d65776f726b2e6f72672f696d672f717569636b73746172742e706e67)

Django REST framework User List 예제

Django로 rest api 를 생성한다면 반드시 사용해야 할 패키지. 브라우저 화면에서도 간단히 조회 및 생성, 수정을 할 수 있다는 점이 좋다. 우측 상단에 보이는 options 를 선택해서 표시되는 타입 형태를 변경해서 볼 수도 있다.
<br><br>
### Django Pydenticon

Django 어플리케이션에서 사용 할 수 있는 identicon 생성 패키지.

![identicon - Google Search](https://www.dropbox.com/s/m5cwd6oys6u93tz/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-25%20%EC%98%A4%EC%A0%84%208.38.07.png?raw=1)

identicon은  깃허브나 홈페이지 프로필에서 사용자 이미지  등록 전에 생성되는 식별 이미지를 말한다. LMS 에서도 사용자의 프로필 사진을 수정할 수 있지만 대부분의 사용자들이 디폴트 이미지를 그대로 지정하다보니 pydenticon이 유용하게 사용되었다.
<br><br>
### Django survey

LMS에서는 학습이 끝날 때마다 학습 컨텐츠에 대한 설문평가를 제공한다. 설문조사 기능을 직접 만들수 있었지만 생산성을 높이고자 django-survey-and-report 패키지를 사용하게 되었다.

![django-survey-and-report](https://www.dropbox.com/s/zqj179qy3dj92a0/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-19%20%EC%98%A4%EC%A0%84%207.09.42.png?raw=1)

django-admin 화면과 연동되어서 설문 내용을 입력하고 생성할 수 있다. 현재는 모든 과정이 끝날 때 동일한 내용의 설문이 보여지지만 이후에 과정별로 다른 설문이 나가야할 경우에도 쉽게 대응할 수 있을 것 같다.
<br><br>
### Django Background Tasks

비동기로 작업을 해야할 때가 있다. LMS에서는 과정별 마감시한이 지나면 과정의 상태가 변경되어야 하는데 이런 경우에 사용할 task queue 중 하나가 background tasks 이다. CELERY가 유명하지만 background tasks 가 설정이 쉽고 등록할 task가 간단한 내용이라 background tasks로 진행했다. 이후 시스템이 더 고도화 된다면 그 때는 CELERY를 사용해야 할 것 같다.
<br><br>
### Django Hijack

hijack 은 재미난 이름대로 관리자가 사용자 화면에  로그인없이 직접 접속할 수 있게 하는 기능이다. LMS 관련 이슈 리포트가 되면 처음에는 stage서버에서 해당 사용자의 비번을 초기화한 후에 직접 로그인 해서 확인했다. 요렇게 되면 번거롭기도 하고 시간이 많이 소요된다. hijack 기능을 사용하면 아래보이는 django-admin 화면에서 해당 유저를 찾아 옆에 생성된 버튼을 클릭하면 해당 유저 접속이 가능하다.

![django-hijack](https://www.dropbox.com/s/h0dpfyt2m82wt3m/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-19%20%EC%98%A4%EC%A0%84%207.23.09.png?raw=1)

추가로 Django 의 훌륭한 package들을 더 구경하고 싶다면 아래의 사이트도 참고할만하다
**awesome-django** : [https://github.com/wsvincent/awesome-django](https://github.com/wsvincent/awesome-django)


---
<br><br>
## Django 는 Admin 화면을 제공한다.

![django-admin](https://www.dropbox.com/s/vsglfr5lbj70gf7/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-20%20%EC%98%A4%ED%9B%84%206.54.30.png?raw=1)

Django Admin 로그인 화면

Django 는 다른 언어와 다르게 별도의 Admin화면을 제공한다. Admin 화면을 따로 구현하지 않아서 좋고,  Admin 화면을 통해서 실제로 사용할 데이터들을 직접 입력해 볼 수 있다. 이렇게 되면 동작하는 사이트를 준비 → 확인해 볼수 있는 시간을 줄일 수 있다.  

바로 위 로그인 화면에서 오른쪽에 있는 검은색 패널은 Django Debug Toolbar 인데 여러 기능을 가지고 있다.  간단히 현재 실행한 Django 버전과  설정정보, static files, cache 등의 정보를 확인할 수 있다. 

![django-debug-toolbar](https://www.dropbox.com/s/kl7zuf5ctf0o7ls/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-22%20%EC%98%A4%ED%9B%84%2011.05.35.png?raw=1)

Django Debug Toolbar  SQL 탭

무엇보다 SQL 탭을 선택하면 위 SQL queries.. 화면이 나오고 현재 실행되고 있는 쿼리를 확인할 수 있다. 서비스 이후에 어플리케이션 속도가 느려지는 것은 많은 부분 잘못된 쿼리에서 비롯되는 것 같다. 중복되는 쿼리나 속도가 느린 쿼리를 간단히 확인할 수있는 건 큰 장점이다.

```python
# admin.py

from django.contrib import admin
from forum.models import *

class TagsAdmin(admin.ModelAdmin):
    pass

admin.site.register(Tags, TagsAdmin)
```

<b>admin.py</b> 에서 Admin 화면에 등록해서 사용할 테이블을 선언할 수 있다.

선언한 테이블은 테이블 데이터를 리스트 조회하거나 상세조회, 삭제, 수정이 가능하다!

Admin 화면의 기본 템플릿을 수정하면 데이터를 그래프 형태로 재가공해서 볼 수도 있고 (아래)

![django-admin graph](https://www.dropbox.com/s/z1cl9unbjlet7d9/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-22%20%EC%98%A4%ED%9B%84%2011.29.43.png?raw=1)

사용자 필터를 설정하면 리스트 화면에서 원하는 컬럼 기준으로 데이터를 필터링해서 조회할 수 있다. 테이블에 있는 원래 컬럼 뿐 아니라 관계 테이블의 컬럼도 필터링 할  수 있는 것은 유용한 기능이다. 

물론 Admin 화면의 사용대상을 데이터 구조를 잘 모르거나 일반 사용자로 생각한다면 별도 Admin 화면을 구현해야 하겠지만 어느 정도 사이트의 구조를 파악하고 있는 사이트의 운영자로 고려한다면 Django Admin 기능은 100% 활용할 수 있는 기능이다.

---
<br><br>
## Django 는 Documentation이 훌륭하다

Django 의 또다른 강점은 모든 설명이 다 나와있는 documentation 이다

[Django documentation](https://docs.djangoproject.com/en/3.1/)

documentation 의 첫 부분은 초보자도 따라해 볼 수 있는 tutorial 이다. 설치부터 app 을 만들고 하나씩 기능을 추가해보면서 Django를 배워볼 수 있다. tutorial 만 2-3번 정도 반복해보면 기본적인 구성을 파악할 수 있을 것이다. 

그리고 그 아래 부분에는 documentation 이 어떻게 구성되어 있는지 model / view / template / form / admin 등 Django의 주요 구성 요소별로 안내가 나와있다.

documentation 에 대해 여러 언어를 지원하고 한국어도 포함하고 있지만 모든 내용을 지원하는 것은 아니어서 영어로 확인해야하는 부분도 있다는 건 참고하면 좋을 것이다. 

 개발을 시작하면 숱한 에러를 만나게 되는데 구글링을 하는 것도 좋지만 documentation의 기본 내용을 다시 확인해보면서 잘못된 코드를 수정했던 기억이 있다. 
documentation이 잘 되어있다는 것은 버전관리와 그에 따른 업데이트가 잘 반영된다는 것이기도 하다.

<br>
끝으로 Django 를 공부하기 시작하면서 가장 도움이 되었던 사이트 및 책을 소개하며 이번 포스팅을 마치려한다.

아래 자료들을 발판삼아 직접 프로젝트를 구현해보면서 예전보다 Django에 대해 더 넓게 알게되었다.
이 글을 읽고 Django 를 처음 시작하시는 분께도 도움이 되길바란다.

**Django Girls Tutorial** : [https://tutorial.djangogirls.org/ko/](https://tutorial.djangogirls.org/ko/)

**Ask Company** : [https://www.askcompany.kr/](https://www.askcompany.kr/)

**Two Scoops of Django** : [https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=88857020](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=88857020)