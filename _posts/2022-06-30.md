---
title: 어떻게 로깅할 것인가 feat. Elastic Beanstalk Cloud Watch Logging
categories:
  - Dev
tags:
  - logging
  - config
  - colorlog
  - elastic beanstalk
  - cloud watch
last_modified_at: 2022-01-30T22:00:00-00:00
---


실제로 서비스를 운영하기 시작하면 운영자로부터 다양한 요청들을 받기 시작한다.

필요한 데이터를 엑셀파일로 만들어 보내주거나 수정해야할 기능에 대한 개발요청을 받는다. 이런 요청들만 들어온다면 얼마나 좋겠냐만 때로는 이러한 요청들을 뒤로한채 긴급한 요청들도 들어온다

“A 유저가 결제를 진행했다고 하는데 구매목록에서는 확인되지 않는다고 합니다. 빨리 확인 부탁드려요!”

“B 유저가 클라우드 기능을 잘 사용하고 있었는데 3일전부터 클라우드 기능이 되지않습니다. 유저분이 답변을 기다리고 있어서 긴급이요!!!”

출처 : 네이버웹툰 ‘이말년씨리즈’

위와 같은 긴급상황에서는 신속하게 대응하는 것만큼 정확한 상황파악을 통해 무엇이 문제였는지를 확인하는 것도 중요하다

시스템의 주요정보들을 DB에 데이터로 저장하고 수정하지만 DB에 쌓이는 데이터들은최종 데이터들이고 데이터로 쌓이기까지 어떤 입력값이 있었고 어떻게 값이 변환되었는지를 상세히 들여다보려면 로깅이 필수이다.

개발단계가 지나 시스템 운영단계에서는 로깅을 중요하게 신경써야 하고 관리해야 한다.

더이상 마지막 결과값만으로 상황을 유추하지는 말자.

STEP1. 장고에서 로깅하기 + a
====================

장고는 문서화가 잘되어있는 웹 프레임워크이고 [장고 로깅에 대한 개요사항](https://docs.djangoproject.com/ko/4.0/topics/logging/)은 링크된 문서를 참고하면 된다

이 글에서는 [How to configure and use logging](https://docs.djangoproject.com/ko/4.0/howto/logging/#logging-how-to) 해당 문서를 같이 살펴보고 자신에게 알맞는 logging 설정을 해본다. 추가로 로깅을 시각적으로 구분할수있는 colorlog 를 어떻게 적용할수있는지도 살펴본다.

*   **Make a basic logging call**

```
import logging  
logger = logging.getLogger(\_\_name\_\_)
```

문서에서 처음 나오는 이야기는 getLogger에 대한 부분이다. loggers 안에도 여러개의 logger 를 선언할 수 있는데 getLogger에서 특정 logger를 지정해서 사용하기 보다 ‘\_\_name\_\_’ 으로 선언하면 몇가지 이점이 있다.

우선은 ‘\_\_name\_\_’ 이 파일이 속해 있는 경로를 가져오다보니 경로별로 서로 다른 로깅설정을 할 수 있다. 예를들면, 결제관련 모듈은 좀 더 자세한 로깅을 하고 일반 모듈은 심플한 로깅이 가능하다. 그리고 따로 경로별 로깅설정을 하지 않아도 loggers 에 디폴트 logger를 설정해두면 ‘\_\_name\_\_’ 만으로 디폴트 logger 설정이 가능하다

```
LOGGING = {  
    \[...\]  
    'loggers': {  
        '': {  
            'level': 'DEBUG',  
            'handlers': \['file'\],  
        },  
    },  
}
```

위 loggers 내부에 ‘’ 는 따로 이름이 명명되지않았고 이는 따로 로깅설정을 하지않은 경로라면 디폴트 logger를 사용하게 된다.

*   **Configure a formatter**

```
LOGGING = {  
    \[...\]  
    'formatters': {  
        'verbose': {  
            'format': '{name} {levelname} {asctime} {module}          {process:d} {thread:d} {message}',  
            'style': '{',  
        },  
        'simple': {  
            'format': '{levelname} {message}',  
            'style': '{',  
        },  
    },  
}
```

formatters 에서는 여러 format 타입을 선언할 수 있다. 위 예제에서 simple 은 디버그 레벨수준과 message 정도만 표기되는데 비해 verbose 는 시간, 모듈, 프로세스, 쓰레드와 같은 세부 정보를 추가로 표기한다. 이렇게 선언된 formatter 는 handler 안에서 formatter 를 지정하면 사용할 수 있다.

*   **Use logger namespacing**

앞에서 getLogger 에 ‘\_\_name\_\_’ 을 선언해서 사용할때 이점을 설명했는데, 예시를 보면서 좀 더 자세히 이야기해보겠다.

```
LOGGING = {  
    \[...\]  
    'loggers': {  
        'my\_app.views': {  
            ...  
        },  
    },  
}LOGGING = {  
    \[...\]  
    'loggers': {  
        'my\_app': {  
            ...  
        },  
    },  
}
```

loggers 에서 특정 경로를 지정하면 경로별로 로깅설정을 따로 지정할 수 있다. 위 예시에서는 my\_app.views와 my\_app 경로가 따로 설정되어있다.

my\_app.views 로깅설정은 my\_app.views 에만 적용되고 my\_app 로깅설정은 my\_app 하위에 있는 views 를 포함한 같은 위치의 모든 파일들에 적용된다.

그럼 my\_app.views 와 my\_app 두 로깅설정에 동시에 영향을 받는 my\_app.views 파일은 어떤 로깅설정을 따르는 걸까?

```
LOGGING = {  
    \[...\]  
    'loggers': {  
        'my\_app': {  
            \[...\]  
        },  
        'my\_app.views': {  
            \[...\]  
        },  
        'my\_app.views.private': {  
            \[...\]  
            'propagate': False,  
        },  
    },  
}
```

다시 2번째 예시에서는 my\_app, my\_app.views, my\_app\_views.private 3개 logger 가 나온다

my\_app.views.private 에는 propagate 설정이 있고 False 로 되어있다.

my\_app.views.private 입장에서 my\_app.views 와 my\_app 이 자신의 상위 폴더이고 부모가 된다.

따라서 my\_app.views.private는 my\_app.views 와 my\_app 두개 로깅설정 모두에 영향을 받게 되는데 propagate 를 False로 설정해서 부모의 로깅설정에 영향을 받지 않고 my\_app.views.private 자기 자신의 로깅설정만 영향을 미치게 된다.

*   **Configure responsive logging**

로깅설정 에는 level 을 지정할 수 있는데, DJANGO\_LOG\_LEVEL 환경변수를 사용해 여러 서버 환경에 맞게 적절한 level 수준을 맞춰 사용할 수 있다

```
'level': os.getenv('DJANGO\_LOG\_LEVEL', 'WARNING')
```

**STEP2. Log formatting with colors!**
======================================

pycharm을 사용하다보면 language 설정을 통해 작성한 코드의 글자색이 구분되어 가독성이 좋지만 까만 터미널 화면에서는 로그가 보통 같은 색으로만 표시된다

[colorlog](https://github.com/borntyping/python-colorlog) 를 사용하면 로깅 레벨 수준에 따라서 자신이 원하는 색깔로 구분해 표기할 수 있다.

```
pip install colorlog
```

pip 에서 colorlog 를 설치하고 verbose formatter 에 colorlog 설정을 적용해보겠다.

“()” 에 colorlog.ColoredFormatter 를 불러와서 colorlog 를 사용하는걸 명시하고

“format” 에는 {log\_color} 변수를 선언해 로깅시에 자신이 설정한 색이 나오게 한다.

“log\_colors” 에서는 로깅 레벨수준에 따라 각각 다른 색 설정을 할수 있다.

```
"formatters": {  
        "verbose": {  
            "()": "colorlog.ColoredFormatter",  
            "format": "{log\_color} {levelname} {asctime} {pathname} {lineno} {message}",  
            "log\_colors": {  
                "DEBUG": "cyan",  
                "INFO": "green",  
                "WARNING": "yellow",  
                "ERROR": "red",  
                "CRITICAL": "red,bg\_white",  
            },  
            "style": "{",  
        },  
        "simple": {  
            "format": "{levelname} {message}",  
            "style": "{",  
        },  
    },
```

위 설정을 적용해서 서버를 실행시켜보면 INFO는 아래와 같이 green 으로 나오고

좀 더 높은 레벨 수준의 정보는 아래와 같이 나온다.  
차례로 WARNING, ERROR, CRITICAL 을 나타낸다.

STEP3. Elastic Beanstalk 로깅 설정
==============================

이후 진행할 실습상황에 대한 배경을 간단히 설명하면  
[sentry](https://sentry.io/welcome/)를 사용하고 있어 application 단의 프론트, 백엔드 에러는 sentry 를 통해 확인할 수 있다. 하지만 에러가 발생한 경우에만 sentry로 확인할 수 있고 결제 등 중요 이벤트의 로깅을 볼수 없는 상황이다.

*   Goal

운영환경 주요 모듈단위에서 에러가 아닌 로그를 확인할수있게 하고 싶고 그 중에서도 backend app 에서 발생하는 로그를 확인하고 싶다.

AWS Elastic Beanstalk 에서 로그를 확인하는 방법은 크게 3가지가 있다

1.  AWS Elastic Beanstalk console logs 메뉴에서 직접 확인하거나
2.  Elastic Beanstalk 설정에서 rotated logs to S3를 선택하거나
3.  Setting Up Log Streaming to CloudWatch Logs 선택할 수 있다

[각 방법의 장단점](https://aws.plainenglish.io/aws-elastic-beanstalk-logging-i-show-3-techniques-with-pros-and-con-s-4380598c2039)은 링크된 블로그에서 좀 더 자세히 확인할 수 있다.

실시간으로 로그를 확인할수 있으며 필요한 로그정보를 찾기가 용이한  
Elastic Beanstalk + CloudWatch 방법을 선택했고  
Log groups 설정을 통해 기본 로그를 확인할 수 있다.

Elastic Beanstalk > Environments > Project Name > Configuration 메뉴에서

Instance log streaming to CloudWatch Logs 영역, Log streaming Enabled 를 체크하면

기본적인 ecs, docker-events 로그들만 확인할 수 있었고 application 과 web server(nginx) 의 로그 정보는 확인할 수 없었다

application 과 web server(nginx) 로그정보들을 확인하기 위해선 기본설정로그 이외에 custom log를 보기 위한 추가설정이 필요하다.

IAM Roles에서 elasticbeanstalk ec2와 관련된 aws-elasticbeanstalk-ec2-role 을 찾아서 아래 형태의 CloudwatchPolicy 를 추가로 생성해 Attach 해야한다.

```
{  
    "Version": "2012-10-17",  
    "Statement": \[  
        {  
            "Effect": "Allow",  
            "Action": \[  
                "logs:CreateLogGroup",  
                "logs:CreateLogStream",  
                "logs:PutLogEvents",  
                "logs:DescribeLogStreams"  
            \],  
            "Resource": \[  
                "arn:aws:logs:ap-northeast-2:999999999999:log-group:\*:\*"  
            \]  
        }  
    \]  
}
```

custom 로그들의 정확한 위치를 확인해야 하는데  
ec2 instance 에 직접 연결해 로그 파일의 저장위치를 확인하거나  
위 log groups에서 확인되는 기본 로그파일의 위치와 AWS Elastic Beanstalk Console 상 ( Elastic Beanstalk > Environments > Project Name > Logs ) Full logs 파일을 다운로드 받아서 custom 파일의 위치를 확인해 볼 수 있다.

아래는 Logs 메뉴에서 Full logs를 다운로드 받아 압축파일을 풀었을때의 모습이다.

기본 로그파일들이 아래 경로에 저장되고

[/aws/elasticbeanstalk/Project-name/var/log/](https://ap-northeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-2#logsV2:log-groups/log-group/$252Faws$252Felasticbeanstalk$252FDockerdjangoapp-prd-kdc$252Fvar$252Flog$252Fdocker-events.log)

다운로드 받은 Full Logs를 보면 우리가 확인해보고 싶은  
custom log가 containers 경로에 저장되어있음을 알 수 있다.

따라서 우리가 확인해야할 custom log의 경로는  
[/aws/elasticbeanstalk/Project-name/var/log/](https://ap-northeast-2.console.aws.amazon.com/cloudwatch/home?region=ap-northeast-2#logsV2:log-groups/log-group/$252Faws$252Felasticbeanstalk$252FDockerdjangoapp-prd-kdc$252Fvar$252Flog$252Fdocker-events.log)containers 이다.

STEP4. Elastic Beanstalk docker 운영환경에서 실시간 로깅설정
===============================================

Elastic Beanstalk 설정을 위해선 프로젝트의 root 위치에 **.**ebextensions 폴더를 생성하고 log-streaming.config 파일을 추가한다. 해당 파일은 아래와 같다

```
packages:  
  yum:  
    awslogs: \[\]  
option\_settings:  
  - namespace: aws:elasticbeanstalk:cloudwatch:logs  
    option\_name: StreamLogs  
    value: true  
  - namespace: aws:elasticbeanstalk:cloudwatch:logs  
    option\_name: DeleteOnTerminate  
    value: false  
  - namespace: aws:elasticbeanstalk:cloudwatch:logs  
    option\_name: RetentionInDays  
    value: 90  
files:  
  "/etc/awslogs/awscli.conf" :  
    mode: "000600"  
    owner: root  
    group: root  
    content: |  
      \[plugins\]  
      cwlogs = cwlogs  
      \[default\]  
      region = \`{"Ref":"AWS::Region"}\`  
"/etc/awslogs/config/logs.conf" :  
    mode: "000600"  
    owner: root  
    group: root  
    content: |  
      \[/var/log/containers/backend.log\]  
      log\_group\_name = \`{"Fn::Join":\["/", \["/aws/elasticbeanstalk", { "Ref":"AWSEBEnvironmentName" }, "var/log/containers/backend.log"\]\]}\`  
      log\_stream\_name = {instance\_id}  
      file = /var/log/containers/backend\*  
commands:  
  "01":  
    command: systemctl enable awslogsd.service  
  "02":  
    command: systemctl restart awslogsd
```

위와 같이 로깅관련 설정파일을 생성할수도 있지만 .ebextensions를 사용하지 않고 Dockerrun.aws.json 파일을 사용하고 있다면 로깅관련 설정은 조금 다르게 진행해야한다

```
{  
    "containerDefinitions": \[  
        {  
	    ,  
            "logConfiguration": {  
                "logDriver": "awslogs",  
                "options": {  
                    "awslogs-group": "backend",  
                    "awslogs-region": "us-west-2",  
                    "awslogs-create-group": "true",  
                    "awslogs-stream-prefix": "/aws/elasticbeanstalk/Dockerdjangoapp-dev"  
                }  
            }  
	},  
	{  
	    ,  
	"logConfiguration": {  
                "logDriver": "awslogs",  
                "options": {  
                    "awslogs-create-group": "true",  
                    "awslogs-group": "nginx",  
                    "awslogs-region": "ap-northeast-2",  
                    "awslogs-stream-prefix": "/aws/elasticbeanstalk/Dockerdjangoapp-dev"  
                }  
            }  
	}  
}
```

containerDefinitions 내에 logConfiguration 설정을 추가하고  
awslogs-group 에는 로그 드라이버가 로그 스트림을 전송할 로그 그룹을 지정,  
awslogs-region 에는 로그 드라이버가 Docker 로그를 전송할 리전을 지정,  
awslogs-stream-prefix 에는 해당 옵션을 사용하여 로그 스트림을 지정한 접두사, 컨테이너 이름 및 컨테이너가 속하는 Amazon ECS 태스크의 ID와 연결한다.

해당 설정적용 후 서버에 반영하게 되면

Log Group 에서 backend, nginx Log Group 을 찾을 수 있고

선택해서 들어가보면 설정했던 stream-prefix 에 맞춰서 아래 Log streams 부분에 새롭게 추가된 것을 확인할 수 있다.

Log streams 를 하나 선택해서 들어가보면 실시간으로 서버에서 생성되는 로그를 확인할 수 있다.

Django 로깅설정의 구성을 하나씩 살펴보는 것부터 시작해 실제로 운영하고 있는 Elastic Beanstalk 에서 실시간 로깅을 구축하는 것까지 살펴보았다.

개발 할때에 실제 운영상황을 염두에 두고 주요부분들에 로깅을 고려해서 진행한다면  
곤경에 처하는 상황을 줄일 수 있을 것이다. 화이팅!

로깅해서 이런 상황은 제발 피하자! (출처: [https://www.codemotion.com/magazine/ai-ml/big-data/logging-in-python-a-broad-gentle-introduction/](https://www.codemotion.com/magazine/ai-ml/big-data/logging-in-python-a-broad-gentle-introduction/))

_참고자료_

*   [https://www.youtube.com/watch?v=ziegOuE7M4A](https://www.youtube.com/watch?v=ziegOuE7M4A)
*   [https://lincolnloop.com/blog/django-logging-right-way/](https://lincolnloop.com/blog/django-logging-right-way/)
*   [https://aws.plainenglish.io/how-to-setup-aws-elasticbeanstalk-to-stream-your-custom-application-logs-to-cloudwatch-d5c877eaa242](https://aws.plainenglish.io/how-to-setup-aws-elasticbeanstalk-to-stream-your-custom-application-logs-to-cloudwatch-d5c877eaa242)
*   [https://docs.aws.amazon.com/ko\_kr/AmazonECS/latest/developerguide/using\_awslogs.html](https://docs.aws.amazon.com/ko_kr/AmazonECS/latest/developerguide/using_awslogs.html)
