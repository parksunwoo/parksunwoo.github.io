---
title: Python 비동기 예제
categories:
  - Dev
tags:
  - python
  - asyncio
  - redis
  - celery
  - interview
last_modified_at: 2023-05-05T20:58:00-00:00
---

python에서 비동기 처리는 언제 필요할까? 간단한 코드 예제로 asyncio를 사용한 비동기처리 그리고 redis celery를 사용한 비동기처리를 살펴보자

---

## 비동기 함수를 사용할 때 고려해야할 사항

비동기 함수는 API 호출 또는 데이터베이스 읽기 및 쓰기와 같은 I/O 바인딩 작업을 처리할 때

python 애플리케이션의 성능과 확장성을 개선하는데 매우 유용할 수 있다

여러 작업을 동시에 효율적으로 실행할 수 있도록 함으로써 비동기 기능은 작업을 완료하는 데 걸리는 시간을

크게 줄이고 애플리케이션의 전반적인 응답성을 향상시킬 수 있다.

그러나 python 응용 프로그램의 모든 함수가 비동기화되어 이점을 얻는 것은 아니라는 점에 유의

실제로 비동기 함수를 사용할지 여부를 결정학 ㅣ전에 고려해야 하는 몇 가지 단점이 있다.

첫째, 동기 함수를 비동기 함수로 변환하려면 추가 노력이 필요하며 특히 함수가 데이터베이스나 파일과 같은 공유 리소스와 상호 작용하는 경우 코드가 더 복잡해질 수 있다. 여러 작업이 동시에 실행 중일 때 실행 흐름을 추적하기가 더 어려울 수 있으므로 비동기 코드 디버깅도 더 어려울 수 있다.

또한 비동기 함수는 I/O 바인딩 작업의 성능과 확장성을 향상시킬 수 있지만 상당한 계산이 필요한 CPU 바인딩 작업에 항상 최선의 선택은 아니다. 경우에 따라 비동기 작업을 관리하는 오버헤드로 인해 실제로 이러한 유형의 함수 실행 속도가 느려질 수 있다.

## 예제코드로 살펴보는 비동기 함수

파이썬에서 **`async`** 키워드는 비동기 함수를 정의하는 데 사용됩니다. 

비동기 함수는 다른 작업이 실행될 수 있도록 실행을 일시 중단했다가 해당 작업이 완료되면 실행을 재개할 수 있는 함수입니다. 

이는 동시 실행 또는 입출력 작업에 의존하는 애플리케이션의 성능과 확장성을 개선하는 데 유용합니다.

다음은 파이썬에서 비동기 함수의 예시입니다:

```python
import asyncio

async def fetch_data(url):
    # Suspend execution to allow other tasks to run
    await asyncio.sleep(1)

    # Fetch data from the given URL
    data = await some_library.fetch(url)

    # Return the fetched data
    return data
```

이 예제에서 **`fetch_data`** 함수는 async 키워드를 사용하여 비동기식으로 정의됩니다. 

**`await`** 키워드는 함수 실행을 일시 중단하고 다른 작업이 실행될 수 있도록 하는 데 사용됩니다. 

await 키워드가 사용되면 함수는 일시 중단된 함수 실행을 나타내는 `asyncio.Task` 객체를 반환합니다. 

**`await`** 키워드는 이 예제의 **`fetch`** 함수와 같은 다른 비동기 함수나 연산을 호출하는 데에도 사용됩니다.

전반적으로 `async` 키워드는 파이썬에서 비동기 프로그래밍의 중요한 부분이며, 

동시 실행 또는 입출력 연산에 의존하는 애플리케이션의 성능과 확장성을 향상시키는 데 널리 사용됩니다.

## Redis와 Celery를 사용한 비동기처리

Redis와 Celery를 사용하여 장고에서 이메일 전송을 비동기적으로 처리하고 싶다.

다음은 Redis와 함께 Celery를 사용하여 Django 애플리케이션에서 이메일을 비동기적으로 전송하는 방법에 대한 예제입니다:

먼저, Celery와 Celery용 Redis 패키지를 설치해야 합니다:

```python
pip install celery redis
```

Django 프로젝트의 [settings.py](http://settings.py/) 파일에 다음 구성을 추가합니다:

```python
# settings.py
import os
from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

app = Celery('myproject')
app.config_from_object('django.conf:settings', namespace='CELERY')
app.autodiscover_tasks()
```

그런 다음 프로젝트 디렉터리에 다음 내용으로 [celery.py](http://celery.py/) 파일을 생성합니다:

```python
# celery.py
from __future__ import absolute_import, unicode_literals
from celery import shared_task
from django.core.mail import send_mail

@shared_task
def send_email_task(subject, message, from_email, recipient_list, **kwargs):
    send_mail(subject, message, from_email, recipient_list, **kwargs)
```

views.py에서 이 함수를 비동기적으로 호출할 수 있습니다.

```python
#views.py
from .celery import send_email_task

def send_email_view(request):
    subject = 'Hello'
    message = 'Test message'
    from_email = 'from@example.com'
    recipient_list = ['to@example.com']
    send_email_task.delay(subject, message, from_email, recipient_list)
    return HttpResponse('Email sent successfully')
```

마지막으로 다음 명령을 실행하여 백그라운드에서 Celery 워커를 시작합니다:

```python
celery -A myproject worker --loglevel=info
```

celery beat를 실행하여 작업을 예약할 수도 있습니다.

```python
celery -A myproject beat --loglevel=info
```

그러면 celery 워커가 시작되고 이메일 전송 작업이 비동기적으로 처리되기 시작합니다.
또한 컴퓨터에서 Redis 서버가 실행 중이어야 합니다.

아래 다이어그램을 참고한다면 아래와 같은 순서로 진행될 것이다.

send_email_view()가 호출되고 send_email_task.delay() 가 실행

django 에서는 broker인 redis 에 새로운 task를 등록

celery worker 에서 등록된 새로운 task (send_email_task)를 가져와 작업시작

작업완료 후 backend인 redis에 상태값 업데이트

![testdriven 사이트 django-celery-flow 설명글](https://testdriven.io/static/images/blog/django/django-celery/django-celery-flow.png)

## 참고자료

[https://testdriven.io/blog/django-and-celery/](https://testdriven.io/blog/django-and-celery/)