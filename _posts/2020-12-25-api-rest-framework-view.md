---
title: "API 그리고 Django REST framework View들 비교하기"
categories:
  - Django
tags:
  - api
  - restframework
  - apiview
last_modified_at: 2020-12-25T08:06:00-00:00
---
API를 알게되고 개발에 직접 사용하게 된 것은 얼마되지 않았다. <br>
API는 백엔드와 프론트가 분리되어 개발되는 현재 우리팀에서도 물론 필요하고 아래 그림처럼 API를 사용하는 End Consumers 가 다양해지는 앞으로의 환경속에서는 그 사용이 더 늘어날 것이다. 

---
이번 포스트에서는 API 그리고 REST API에 대해 간단히 정리해보고 Django에서 Web API를 생성하는  Django REST framework 패키지의 View 들을 비교해보았다.
<br>
![https://www.dropbox.com/s/df942yj9rcvjrat/Screen%20Shot%202020-12-22%20at%208.11.33%20AM.png?raw=1](https://www.dropbox.com/s/df942yj9rcvjrat/Screen%20Shot%202020-12-22%20at%208.11.33%20AM.png?raw=1)
[출처 : https://www.aloi.io/docs/installation/]

<br>

# 우리는 왜 API 를 사용하는 걸까?

한 개의 개발서버에서만 프로그램을 띄워 실행한다면 다른 시스템과의 통신을 생각할 필요는 없다. 하지만 시스템 규모가 커짐에 따라 내부의 다른 시스템과도 통신해야하고 외부 시스템과도 통신을 해야한다. 우리는 다른 시스템 (Consumers)와 통신하기 위한 여러 방법중에 하나로 API 방식을 사용하는 것이다.

![https://www.dropbox.com/s/kktdtoa1kcgy0bh/Screen%20Shot%202020-12-22%20at%208.59.24%20AM.png?raw=1](https://www.dropbox.com/s/kktdtoa1kcgy0bh/Screen%20Shot%202020-12-22%20at%208.59.24%20AM.png?raw=1)

- API는 통신하는 방식이다 [ 출처 :https://news.hmgjournal.com/TALK/Human/Hyundai-conversation-technique]
<br><br>

물론 통신하는데 API만 있는 것은 아니다.

예전 근무했던 금융권에서는 내부 시스템 간 통신에 연계 솔루션 방식*을 사용했다.

*연계 솔루션 방식 : EAI 서버와 송수신 시스템에 설치되는 클라이언트를 이용하는 방식, EAI는 송수신 데이터를 식별하기 위해 송수신 처리 및 진행 현황을 모니터링하고 통제하는 시스템*

보안이 중요하기도 하고 여러 시스템간 통신을 편리하게 관리하기 위함이였다. 정해진 틀이 있다보니 틀에서 조금이라도 벗어난 경우에는 통신이 불가하였고 그러다보니 개발자가 원하는 대로 변경은 어렵고 개발 자유도도 낮았다. 그러던 중 네이버와 개발 프로젝트를 진행하게되어 협력하게 되었는데 이때 API로 개발을 진행하게 되었다. 그 전에는 API를 단어로만 알고있었지만 직접 개발해보니 API가 가진 장점들을 알게되었다. 

API는 애플리케이션 소프트웨어를 구축하고 통합하기 위한 정의 및 프로토콜 세트로, 애플리케이션 프로그래밍 인터페이스 (Application Programming Interface)를 말한다.

API는 소프트웨어 개발을 단순하게 만들고 그 이면의 로직을 알 필요가없다. 따라서 이미 잘 만들어진 API를 활용하는 것만으로 우리는 우리의 소프트웨어를 빠르게 확장할 수 있다. 

API중에서도 REST (Representational State Transfer) 아키텍쳐의 제약 조건을 준수하는 웹 API를 RESTful API 라고 한다. REST는 아키텍쳐 스타일이라 RESTful 웹 API에는 공식적인 표준이 없다. 6가지 주요 제약 조건은 위키피디아 링크에서 상세내용 확인이 가능하다. [위키피디아 : RESTful API](https://ko.wikipedia.org/wiki/REST#REST_%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98%EC%97%90_%EC%A0%81%EC%9A%A9%EB%90%98%EB%8A%94_6%EA%B0%80%EC%A7%80_%EC%A0%9C%ED%95%9C_%EC%A1%B0%EA%B1%B4)

Django에서 외부와 통신하는 WEB API 를 생성하는 패키지로 많이사용하는게 이제부터 좀더 자세히 살펴볼 Django REST framework (이하 REST framework) 이다.

---
<br>
<br>

# REST framework View 비교해보기

블로그나 사이트에서 Django에 대한 설명을 하는 예제를 보면 보통은 일반적인 CRUD로 구현되는 아래와 같은 예제들이 대부분이다.

```python
from rest_framework.generics import (
    ListAPIView,
    RetrieveAPIView,
    CreateAPIView,
    DestroyAPIView,
    UpdateAPIView
)

# Article 리스트 조회
class ArticleListView(ListAPIView):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
    permission_classes = (permissions.AllowAny, )

# Article 상세 조회
class ArticleDetailView(RetrieveAPIView):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
    permission_classes = (permissions.AllowAny, )

# Article 생성
class ArticleCreateView(CreateAPIView):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
    permission_classes = (permissions.IsAuthenticated, )

# Article 수정
class ArticleUpdateView(UpdateAPIView):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
    permission_classes = (permissions.IsAuthenticated, )

# Article 삭제
class ArticleDeleteView(DestroyAPIView):
    queryset = Article.objects.all()
    serializer_class = ArticleSerializer
    permission_classes = (permissions.IsAuthenticated, )
```

반복되는 CRUD를 다시 만들 필요없이  REST framework 에서 기본적으로 제공하는  GenericView 들을 활용하는 것인데 이런 GenericView 들을 모아 더 간단히 사용하는 것이 Viewset 이다

정말 간단하지 않은가? 20줄 가까이 되던 로직이 5줄로 줄어드는 순간이다.

 

```python
from rest_framework import viewsets

class ArticleViewSet(viewsets.ModelViewSet):
    serializer_class = ArticleSerializer
    queryset = Article.objects.all()
	 permission_classes = (permissions.IsAuthenticated, )
```

RSET framework 사이트에선 ViewSet이 View 보다 나은 장점을 2가지 정도 소개한다

- 반복되는 로직이 한 개 클래스에 결합된다. 예를 들어 queryset 을 한번 지정하면 클래스 안의 다양한 views에서 사용이 가능하다. 위의 코드에서는 ListView, DetailView 등에서 동일한 queryset 을 매번 선언했는데 viewset은 모든 view를 통일한 것이라 한번만 선언되어 있다. <br>
- routers를 사용하면 더 이상 URL conf 설정에 신경쓸 필요가 없다. ListView, DetailView 등으로 만들면 해당하는 View에 맞게 URL conf 설정을 해야하지만 viewset으로 만들면 viewset 을 router에 담으면 자동으로 URL conf 설정이 된다! <br>

하지만 미리 잘 만들어진 View 에서는 custom 로직을 구현하기가 어렵다.

예를 들면 Article 정보 뿐 아니라 특정 조건의 다양한 정보까지 함께 출력하고 싶다면 Viewset 과 GenericView를 사용하기가 힘들다. 이 때는 GenericView 의 원형인 APIView 를 사용한다.

`from rest_framework.views import APIView`

Django를 처음 사용할 때에 APIView 를 사용해서 원하는 결과값을 어떻게 출력할지 헤딩을 많이 했는데 해답은 간단했다. 

`from rest_framework.response import Response`

에 아래 코드와 같이 dict 의 형태로 담아 출력하면 된다. 

```python
from rest_framework.views import APIView
from rest_framework.response import Response

class UserProfileView(APIView):
    def get(self, request, *args, **kwargs):
        pk = kwargs.get('pk', None)
        user = get_object_or_404(User, pk=pk)
        user_serializer = UserSerializer(user)

        challenge_qs = UserChallenge.objects.filter(user=user)
        challenge_serializer = UserChallengeSerializer(challenge_qs, many=True)

        return Response({
            'user_info': user_serializer.data,
            'challenge_list': challenge_serializer.data
        })
```

위의 코드를 GenericView 를 사용해서 구현했다면  serializer 로 UserSerializer가 설정되어 user 정보만 보낼수 있었겠지만

APIView를 사용해 출력값을 dict 형태를 선언해  'user_info' 에 UserSerializer 를 담고 
'challenge_list'에 UserChallengeSerializer 를 담으면 관련 정보들을 한번에 출력할 수 있다.

REST framework 가 제공하는 APIView 는 View 와는 아래의 부분들에서 다르다고 한다.

- Django의 HttpRequest 가 아닌 REST framework 의 Request를 사용한다
- 리턴하는 값 역시 HttpResponse 가 아닌 REST framework의 Response 를 출력한다. 출력되는 콘텐츠를 변형하거나 적절한 renderer 를 설정할 수 있다 (JSON, HTMLForm, MultiPart 등)
- APIException 예외 처리를 할 수 있다.
- 들어오는 request 를 인증 및 권한을 체크해 throttle 관리할 수 있다

<br><br>

위에서 View 와 ViewSet, GenericView 그리고 APIView를 비교해보았는데,

정리해보면 간단하고 빠르게 기본적인 구성을 갖춰 개발할때에는 GenericView / ViewSet 을,

custom 로직을 사용하여 다양한 시도를 해보고싶을 때는 APIView 로 구현해 볼 수 있을 것 같다.

<br><br>


