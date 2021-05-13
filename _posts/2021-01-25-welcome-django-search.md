---
title: "어서와, Django 검색은 처음이지?"
categories:
  - Django
tags:
  - haystack
  - whoosh
  - drf_haystack
last_modified_at: 2021-01-25T08:06:00-00:00
---
**django-haystack + Whoosh  + drf_haystack** <br>
Django 검색관련하여 여러 글들을 확인했지만 해당 패키지 tutorial 예제를 그대로 가져와서 번역한 정도였다. 실제 서비스에 검색을 적용하려고 보니 몇 걸음 더 나아간 설정이 필요했고 그 과정에서 알게된 것들을 이번 포스팅에 정리해 보았다.
---
<br><br><br>

## 1단계 django-haystack + Whoosh 셋팅하기

Haystack 은 장고에서 사용하기 편한 검색 모듈이다. 코드 변경없이 다양한 검색 백엔드 (Solr, Elasticsearch, Whoosh, Xapian )로 변경할수 있다는 점이 장점이다

검색을 처음 구현하다 보니 가장 간단히 시도해볼수 있는 Whoosh 를 선택해서 진행해보았다

```shell
pip install django-haystack
```

```pyhton
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',

    # Added.
    'haystack',
]
```

django-haystack 을 설치하고 *settings.py* INSTALLED_APPS 에 'haystack'를 추가한다.

```python
import os

WHOOSH_INDEX = os.path.join(BASE_DIR, 'whoosh')
HAYSTACK_CONNECTIONS = {
    'default': {
        'ENGINE': 'haystack.backends.whoosh_backend.WhooshEngine',
        'PATH': WHOOSH_INDEX,
    },
}
```

*settings.py* 에 HAYSTACK_CONNECTIONS 을 추가, ENGINE 과 PATH를 넣는다.

PATH는 Whoosh 인덱스가 위치한 경로를 넣는데 이를 구분하기 위해 WHOOSH_INDEX로 따로 선언했다.

이정도로 간단히 설정이 완료된다면 좋겠지만 검색은 고려해야할 부분이 좀 있다.

검색엔진에는 색인(Index) 라는게 있다. 새로운 검색어가 나왔을때 모든 테이블을 찾는다면

한번 검색할때마다 시간이 많이 걸릴 것이다.

![책이 분류되어 정리된 도서관의 모습 ](https://www.dropbox.com/s/v4r4kgr81a9kuwj/Screen%20Shot%202021-01-20%20at%208.00.15%20AM.png?raw=1)

책들이 잘 정리된 도서관의 모습, (출처:[https://www.eroun.net/news/articleView.html?idxno=3146](https://www.eroun.net/news/articleView.html?idxno=3146))

예를 들면 도서관에 갔을떄 우리가 책을 쉽게 찾을 수 있는건 책들이 이미 분류체계에 맞춰 정해진 자리에 위치해 있기 때문이다.

우리도 모델의 데이터를 미리 인덱싱해두면 검색을 빠르고 효율적으로 할수 있다.

```python
class Topic(models.Model):
    topicker = models.ForeignKey(settings.AUTH_USER_MODEL, related_name='+', null=True, on_delete=models.SET_NULL)

    title = models.CharField(max_length=255)
    slug = models.CharField(max_length=255)
    first_post = models.ForeignKey(
        "forum.Post",
        related_name="+",
        null=True,
        blank=True,
        on_delete=models.SET_NULL,
    )
    tags = models.ManyToManyField(Tags, blank=True)
    categories = models.ManyToManyField(Category, blank=True)
    created_at = models.DateTimeField(db_index=True, auto_now_add=True)
    updated_at = models.DateTimeField(db_index=True, auto_now=True)
```

여기서 Topic은 임의로 선언한 질문을 나타내는 모델이다.  Topic에서 어떤 필드를 검색되게 할 것인지 

선언하는 작업이 SearchIndexes 를 만드는 작업이다.

```python
# search_indexes.py

from haystack import indexes
from forum.models import Topic

class TopicIndex(indexes.SearchIndex, indexes.Indexable):
    text = indexes.CharField(document=True, use_template=True, template_name='search/indexes/topic_text.txt')
    author = indexes.CharField(model_attr='topicker')
    pub_date = indexes.DateTimeField(model_attr='created_at')

    def get_model(self):
        return Topic

    def index_queryset(self, using=None):
        """Used when the entire index for model is updated."""
        return self.get_model().objects.all()

```

```json

#search/indexes/topic_text.txt

{{ object.title }}
{{ object.first_post.body }}

```


먼저 text 필드는 검색에서 가장 우선순위가 높은 필드이다. 다른 필드와는 다르게  `document=True`

옵션이 들어가 있어 구분할수있고  자세한 필드내용은 `template_name`에  적힌 경로에 따로 표기한다. topic_text.txt 내용은 Topic 의 title 과 first_post.body 필드를 검색에 주로 사용하겠다는 걸 의미한다. topic_text.txt 는 정해진 경로에 생성되어 있어야한다.

그리고 추가로 author, pub_date 필드는 Topic의 topicker, created_at 필드에 각각 대응되어지고 추가적으로 검색 필터링 역할을 할 수 있다. 

설정이 완료되고 검색을 테스트해보면.. 검색이 안된다?!

DB에 있는 데이터를 search index에 붓는 작업이 남았다.

`./manage.py update_index` 또는 `./manage.py rebuild_index`

를 사용하면 인덱스 업데이트를 할수 있고 나의 경우에는 

`./manage.py update_index —age=24`

task job을 걸어두면 하루에 1번 자동으로 인덱스 업데이트가 된다.

<br><br><br>
## 2단계 한글검색이 이상하다?!

검색 페이지를 간단히 구현해서 (참고 [링크](https://django-haystack.readthedocs.io/en/master/tutorial.html#search-template)) 아래와 같은 검색 페이지를 만들었다.

처음 영문으로 검색했을때 제목과 본문내용 기준으로 원하는 검색결과가 나온다.

![영문 제목, 본문 검색](https://www.dropbox.com/s/fjvgxzvctt56oy0/Screen%20Shot%202021-01-26%20at%207.39.48%20AM.png?raw=1)
-영문검색시 제목과, 본문 검색이 잘 된다

그런데 한글로 검색을 해보니 제목으로는 검색결과가 나오는데 본문내용은 검색이 안되는 상황이 발생했다.

![한글 제목 검색](https://www.dropbox.com/s/ddgzjnamdw0uqa2/Screen%20Shot%202021-01-26%20at%207.41.29%20AM.png?raw=1)
-"블로그"로 제목 검색에 성공한 결과

![한글 본문 검색](https://www.dropbox.com/s/gohwdbh6e24r5v4/Screen%20Shot%202021-01-26%20at%207.43.57%20AM.png?raw=1)
-"활용"을 검색한 결과 : 없음
<br><br>

```json
{
            "id": 5,
            "title": "요새 복습하는 내용 정리해보려고 무작정 블로그를 만들었는데",
            "first_post": {
                "id": 5,
                "poster": {
                    "id": 145,
                    "username": "kang",
                    "photo_url": "/identicon/image/kang.png"
                },
                "post_number": 1,
                "body": "역시나 귀찮고,, 어떻게 활용해야 잘 했다고 소문이 날지도 궁금하고,, 블로그 고수님들의 아무 팁이나 구걸해봅니다 ",
                "created_at": "2020-12-16T17:05:46.950796+09:00",
                "updated_at": "2020-12-16T17:05:46.950815+09:00",
                "is_like": false,
                "like_count": 0,
                "is_poster": true,
                "reply_to_post_number": null,
                "reply_count": 0
            },
        },
```

json 파일을 보면 본문내용에 "활용"을 포함한 질문 실제 데이터는 있지만 검색 되지않는다.

원인을 찾기 위해 좀 더 실험을 해보니 "활용해야"는 검색이 되었다. 즉 있는 단어 그대로는 검색이 되지만 검색하는 "활용" 이라는 단어 기준으로는 검색이 안되는 것이다. 텀이 아닌 단어의 일부만 가지고 검색을 하는 것이 보통 일반적인 검색 방식이고 이런 사용을 위해 검색 텀의 일부만 미리 분리해서 저장을 할 수 있다. 이렇게 단어의 일부를 나눈 부위를 NGram 이라고 한다. 

'어떻게 활용해야 잘 했다고 소문이 날지도 궁금하고' 라는 질문 내용을 기준으로 설명하면
기존 CharField 는 위 문장을 단어 단위로 구분하지만

어떻게 / 활용해야 / 잘 / 했다고 / 소문이 / 날지도 / 궁금하고 

NgramField 2자리 방식이면 문장을 아래와 같이 구분해서 저장한다

어떻/떻게 / 활용/용해/해야 / 잘 / 했다/다고/ 소문/문이/ 날지/지도/ 궁금/금하/하고

그러나 이렇게 잘게 쪼개면 검색을 할 수는 있지만 저장되는 텀의 갯수도 기하급수적으로 늘어나고

보통은 단어의 맨 앞에서부터 검색하는 경우가 많기에 Edge NGram 을 사용한다

옵션을 min_gram : 2 , max_gram : 4 로 설정하면

어떻/어떻게 / 활용/활용해/활용해야 / 잘 / 했다/했다고 / 소문/소문이/ 날지/날지도/ 궁금/궁금하/궁금하고 

와 같은 방식으로 텀이 저장된다.

1단계에서 생성했던 search_indexes.py 파일로 돌아가

```python
class TopicIndex(indexes.SearchIndex, indexes.Indexable):
    text = indexes.EdgeNgramField(document=True, use_template=True, template_name='search/indexes/topic_text.txt')
    author = indexes.CharField(model_attr='topicker')
    pub_date = indexes.DateTimeField(model_attr='created_at')
```

Edge NGram 적용을 위해 text 필드의 타입을 CharField 에서 EdgeNgramField 로 변경했다.

이제 한글로 본문의 단어를 검색했을때 검색결과가 나온다.

<br><br><br>
## 3단계 Django REST Framework 에서 Haystack 사용

Django REST Framework를 사용하고 있다면

검색결과를 원래 사용하던 API 형식에 맞춰 전달하고 싶을 것이다.

이 경우에는 [drf-haystack](https://github.com/rhblind/drf-haystack) 을 활용하면 된다.

```shell
$ pip install drf-haystack
```

간단히 drf-haystack을 설치하고  아래와 같이 serializer 와 ViewSet 을 설정하면 된다

```python
from drf_haystack.serializers import HaystackSerializer
from drf_haystack.viewsets import HaystackViewSet

from forum.search_indexes import TopicIndex

# Serializer
class TopicSearchSerializer(HaystackSerializer):
    class Meta:
        index_classes = [TopicIndex]
        fields = ["text", "author", "pub_date"]

# ViewSet
class TopicSearchViewSet(HaystackViewSet):
    index_models = [Topic]
    serializer_class = TopicSearchSerializer
```

여기서 우리는 TopicSearchSerializer fields 들이 Topic 모델의 속성이 아닌 TopicIndex 의 속성들임에 주목해야 한다.

이렇게 되면 검색결과가 TopicSearchSerializer에 따라 text, author, put_date 가 나오게된다

그래서 Topic 모델의 형태로 결과를 얻고 싶다면 아래와 같이 변형을 해야한다

```python
from drf_haystack.serializers import HaystackSerializerMixin

class TopicSearchSerializer(HaystackSerializerMixin, TopicSerializer):
    class Meta(TopicSerializer.Meta):
        search_fields = ("text", )
```

HaystackSerializerMixin 과 원래의 TopicSerializer 를 사용해서 Topic 모델의 속성을 그대로 사용할 수 있게 한 것이다.

이제 처음에 원했던 Topic 모델  포맷에 맞게 검색결과가 나온다.

<br><br>
## Json 형태의 본문내용을 어떻게 검색 및 요약할 것인가?

```json
"first_post": {
                "id": 33,
                "poster": {
                    "id": 48,
                    "username": "taxfree9090",
                    "photo_url": "https://aiffelstaticdev.blob.core.windows.net/media/core/user/2020/08/07/taxfree9090.jpeg"
                },
                "post_number": 1,
                "body": "{\"entityMap\":{},\"blocks\":[{\"key\":\"8oskj\",\"text\":\"이거 수정이 되나 모르겠네요\",\"type\":\"unstyled\",\"depth\":0,\"inlineStyleRanges\":[],\"entityRanges\":[],\"data\":{}},{\"key\":\"1foif\",\"text\":\"\",\"type\":\"unstyled\",\"depth\":0,\"inlineStyleRanges\":[],\"entityRanges\":[],\"data\":{}},{\"key\":\"6jjlp\",\"text\":\"흠 \",\"type\":\"unstyled\",\"depth\":0,\"inlineStyleRanges\":[],\"entityRanges\":[],\"data\":{}}]}",
                "raw": "이거 수정이 되나 모르겠네요 흠",
                "created_at": "2021-01-24T12:28:21.775106+09:00",
                "updated_at": "2021-01-24T12:28:21.775135+09:00",
                "is_like": false,
                "like_count": 0,
                "is_poster": false,
                "reply_to_post_number": null,
                "reply_count": 0
            },
```

검색 및 본문요약을 처리하려고 하다보니 당면한 문제는 
프론트단에서 [Draft.js](https://draftjs.org/) 를 사용하다보니
위 body 필드를 보면 json 형태로 본문 내용이 작성되어있었다. 

1차적으로는 json에서 사용하는 키값 (entityMap, blocks, key, text, type) 들을 검색어에서 제외하려고 하였으나 일부 단어들은 충분히 검색가능한 단어들이었다. 고민끝에 이 문제를 해결하였는데, body 바로 아래에 raw 필드를 추가해서 순수 텍스트만 해당 필드에 따로 저장하는 것으로 하였다.

이렇게 되면 검색 인덱스를 생성하는 것도 더 수월해지고 일부 화면에서 본문내용을 요약해서 보내야하는 경우도 있어 요약을 하기에도 편해졌다.

<br><br>
지금까지 Django 에서 검색을 구현해보면서 부딪혔던 문제들을 하나씩 정리해보았다.

실제로 사용해봐야 하겠지만 검색량이 많아지면 검색엔진을 whoosh 에서 다른 엔진으로 바꿔야할 수 도 있을 것 같다.

Django 에서 검색을 구현하는 이에게  이 글이 시간을 단축할 수 있는 참고자료가 되기를 바란다.

<br><br>

