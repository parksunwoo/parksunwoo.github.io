---
title: chatGPT를 slack 과 연동해서 멘션으로 대화하기 (feat. chatGPT)
categories:
  - Dev
tags:
  - chatGPT
  - GPT3
  - openai
  - slack
last_modified_at: 2023-01-29T23:00:00-00:00
---



## chatGPT를 조금더 편하게 사용할수는 없을까?
[https://chat.openai.com/](https://chat.openai.com/) 에 접속해 구글 로그인하고 궁금한 내용을 구글번역에서 영어로 변환한 후

질문하면 chatGPT는 친절하게 영어로 답변해준다.

이 복잡한 걸 한번에, 사용하고 있는 slack 메신저에서 chatGPT 봇을 멘션하고 궁긍한 거 한글로 물어보고 다시 바로 한글로 답변 받을수있다.

그럼 어떻게 시작해볼까?…

chatGPT에게 물어봤다.

"I want to get an answer when I ask a question by linking the chat gpt api with slack. Show me an example code"

---
> 
> 

그러자 잠시 기다리니 아주 친절하게 답변해준다.

> Here is an example of how you can use the Slack API to create a bot that responds to questions by calling the OpenAI API:
> 

```python
import openai_secret_manager
import openai
import json
import requests

# Use the openai_secret_manager to get your API key
secrets = openai_secret_manager.get_secrets("openai")
api_key = secrets["api_key"]

# Set the API key
openai.api_key = api_key

# Define the Slack bot's token
SLACK_BOT_TOKEN = "xoxb-your-token-here"

# Define the Slack bot's endpoint
SLACK_BOT_ENDPOINT = f"https://slack.com/api/chat.postMessage?token={SLACK_BOT_TOKEN}&channel=%s&text=%s"

# Define the Slack event listener
SLACK_EVENT_ENDPOINT = "https://slack.com/api/events.listen"

def handle_question(event_data):
    # Extract the user's question from the event data
    question = event_data["text"]

    # Use the GPT-3 model to generate an answer to the question
    response = openai.Completion.create(
        engine="text-davinci-002",
        prompt=question,
        max_tokens=3000
    )

    # Extract the answer from the response
    answer = response["choices"][0]["text"]

    # Send the answer back to the user via Slack
    channel = event_data["channel"]
    requests.get(SLACK_BOT_ENDPOINT % (channel, answer))

# Listen for events from Slack
while True:
    response = requests.post(SLACK_EVENT_ENDPOINT, json={"token": SLACK_BOT_TOKEN})
    event_data = json.loads(response.text)
    if "event" in event_data:
        event_type = event_data["event"]["type"]
        if event_type == "app_mention":
            handle_question(event_data["event"])
```

> This code uses the Slack API to create a bot that listens for mention events. When the bot is mentioned, it will extract the user's question from the event data and use the GPT-3 model to generate an answer to the question. The bot then sends the answer back to the user via Slack.
It's important to note that you need to have a valid API key from OpenAI and a valid Slack bot token to use this code.
Also, you will need to have 
> openai_secret_manager, openai, json, requests library installed.
> It's also important to note that this is an example and you need to adapt it to your particular use case and error handling.
> 

오호 그럼 chatGPT 가 던져준 내용을 가지고 하나씩 진행해보겠다

예시 코드를 찬찬히 살펴보니 몇가지 필요한 값들이 있다.

1) openai.api_key
2) SLACK_BOT_TOKEN

openai.api_key는 openai 의 여러 API들을 활용하기 위해 필요한 key 값이다.

[https://openai.com/api/](https://openai.com/api/) 사이트에 접속해 보니 GPT-3 뿐 아니라 
Codex, DALL’E 등 openai의 다양한 서비스들을 API 로 사용할수 있고 또 이를 이용해 멋진 앱을 만들어볼 수도 있다. 

구글 계정으로 openai 사이트에 회원가입후 로그인하면

해당 페이지를 볼 수 있다.

![OpenAI-Mainpage](/assets/images/openai1.png)

우측 상단 계정을 클릭하면 사이드 메뉴가 나오는데 거기서 View API keys를 클릭하면 메뉴로 이동한다

SECRET KEY를 생성한적이 없다면 새로 생성을 하고, 생성할때에는 꼭 해당 key값을 메모해두어야한다. 생성후에는 아래와 같이 나와서 더이상 해당키값을 알수없다.

![OpenAI-APIkeys](/assets/images/openai2.png)

SLACK_BOT_TOKEN 은 Slack API 문서와 event api bot 관련해 정리가 잘되어있는 깃허브를 참고했다.
[python-slack-events-api Git](https://github.com/slackapi/python-slack-events-api/blob/main/example/README.rst#example-slack-events-api-bot)

위 링크를 참고한다면 진행하는게 그렇게 어렵지 않을텐데 나는 몇가지 부분에서 조금 삽질을 했다.

![Slackbot-OAuth-and-Permissions](/assets/images/openai3.png)
![Slackbot-OAuth-and-Permissions2](/assets/images/openai4.png)

OAuth & Permissions 메뉴에선 Bot Token Scopes를 위와같이 설정했다.

슬랙앱 상에서 GPT 봇이 멘션했을때 대화내용을 읽을수 있어야 하고, 첨부파일이 있다면 해당 파일을 읽을수 있어야 한다.

무엇보다 봇도 대화하려면 chat 관련된 권한이 필요하다. 위 내용이 그러한 권한들이고 추가로 다른 작업을 한다면 그에 맞춰서 권한을 추가/수정하면 된다.




![Slackbot-Event-Subscriptions](/assets/images/openai5.png)
![Slackbot-Event-Subscriptions2](/assets/images/openai6.png)

내가 시간을 조금많이 쏟았던 부분은 Event Subscription 부분인데

이 부분은 쉽게 말하면 슬랙에서 봇을 멘션하는 특정 이벤트가 발생했을때 이 이벤트를 우리 서버와 연결하는 URL 설정부분이다. 
설정한 URL로 슬랙이 post 방식으로 이벤트 내용을 쏴주면 이 내용을 우리서버에서 parsing 해서

우리가 하고 싶은 동작과 연결하고 다시 slack 채널에 쏴주는 것이다.

![start-ngrok](/assets/images/openai7.png)

로컬에서 위 작업을 해보려면 ngrok을 사용해야한다. 
쉽게 이야기해서 ngrok은 로컬서버에 임의의 url을 부여해서 외부에서도 접속가능하게 해주는 도구이고 모바일 테스트에서도 사용가능한 것으로 알고 있다.

터미널에서 

```
ngrok http 3000
```

명령어를 치면 위 예시 그림상으로는 
`http://h7465j.ngrok.io -> localhost:9292` 로 연결된다.

그렇다는 말은 'localhost:9292' 로컬서버를 별도로 띄워놓아야한다는 것이다.

You'll need to have your server and ngrok running to complete your app's Event Subscription setup

```
python example.py
```

'example.py' 라는 python 파일을 실행시키면 python 파일에는 동작하고 싶은 코드내용이 작성되어 있을 거고 

포트는 9292 가 열려있어야 할것이다. ngrok 과 python 서버를 동시에 실행시켜두면

`http://h7465j.ngrok.io` URL에 접속하면 로컬서버 9292포트로 연결되는 것이다. 

다시 ngrok에 대한 설명을 잠시 마무리하고 slack Event Subscriptions 메뉴로 돌아오면

![verified-request-url](/assets/images/openai8.png)

**Request URL을 아래와 같이 설정해야한다**
 (your ngrok URL + `/slack/events`)

(your ngrok URL + `/slack/events`) URL로 접속시 slack에서 이벤트 연결 검증을 위해 challenge 파라미터에 값을 실어보내는데 challenge 값을 return 하는 코드를 작성해두어야한다.

We’ll send HTTP POST requests to this URL when events occur. As soon as you enter a URL, we’ll send a request with a `challenge`
 parameter, and your endpoint must respond with the challenge value.


아래 django 예시코드에서는     

elif "challenge" in event_data: 

해당부분이다.

```python
@csrf_exempt
@api_view(["POST"])
@permission_classes((AllowAny,))
def slack_action_test(request):
    event_data = request.data
    logger.info('1' * 100)
    logger.info(event_data)

    if "event" in event_data:
        event_type = event_data["event"]["type"]
        if event_type == "app_mention":
            handle_question(event_data["event"])
        return JsonResponse({"message": "ok"}, status=status.HTTP_200_OK)

    elif "challenge" in event_data:
        return JsonResponse({"challenge": event_data["challenge"]})
```

Request URL 연결에 성공하면 Verified 가 표시된다. 그럼 Event Subscriptions 연결이 완료되었다는 것이다.

슬랙채널에서 테스트삼아 우리가 생성한 bot을 멘션하면 동작하는걸 확인할 수 있을 것이다.

여기까지 따라왔다면 이제 절반이상은 완료된것이다.

다시 한번 짚어보면 우리는 chatGPT를 slack에서 slack bot을 이용해 멘션으로 편하게 사용하는 작업을 진행중이고 이 코드를 작성하기 위해서 2개 값이 필요해

1) openai.api_key
2) SLACK_BOT_TOKEN

2개 값을 설정하는 작업을 진행했다.

그럼 다시 chatGPT가 보여준 예시코드를 처음부터 읽어보자.

```python
import openai_secret_manager
import openai
import json
import requests

# Use the openai_secret_manager to get your API key
secrets = openai_secret_manager.get_secrets("openai")
api_key = secrets["api_key"]

# Set the API key
openai.api_key = api_key

# Define the Slack bot's token
SLACK_BOT_TOKEN = "xoxb-your-token-here"

# Define the Slack bot's endpoint
SLACK_BOT_ENDPOINT = f"https://slack.com/api/chat.postMessage?token={SLACK_BOT_TOKEN}&channel=%s&text=%s"

# Define the Slack event listener
SLACK_EVENT_ENDPOINT = "https://slack.com/api/events.listen"

def handle_question(event_data):
    # Extract the user's question from the event data
    question = event_data["text"]

    # Use the GPT-3 model to generate an answer to the question
    response = openai.Completion.create(
        engine="text-davinci-002",
        prompt=question,
        max_tokens=3000
    )

    # Extract the answer from the response
    answer = response["choices"][0]["text"]

    # Send the answer back to the user via Slack
    channel = event_data["channel"]
    requests.get(SLACK_BOT_ENDPOINT % (channel, answer))

# Listen for events from Slack
while True:
    response = requests.post(SLACK_EVENT_ENDPOINT, json={"token": SLACK_BOT_TOKEN})
    event_data = json.loads(response.text)
    if "event" in event_data:
        event_type = event_data["event"]["type"]
        if event_type == "app_mention":
            handle_question(event_data["event"])
```

가장 아래부분 while True를 보면

1) 슬랙 이벤트를 수신하고 있다가 event type이 app_mention 인 경우 

2) event_data[”event”] 를 handle_question 에 실어서 보낸다

handle_question 메서드는

3) event_data 에서 text 를 읽어서 (text 가 곧 우리가 슬랙채널에서 봇에게 던지는 질문같다)

4) 질문을 openai.Completion.create 메서드에 실어 보내면

5) response 응답에 GPT-3 가 보내는 답변이 온다

GPT-3 가 보내는 답변을 SLACK_BOT_ENDPOINT 에 보내면 봇이 우리가 보낸 질문에 답을 하는 방식이 되는 것 같다. 오호.

실제로 위 코드를 수행해보았을때 몇가지 수정해야할 부분이 있었다.

openai_secret_manager 는 따로 없는 듯하고 

slack bot이 GPT-3 가 보내온 답변을 다시 Slack 채널에 보내는 부분도 API가 다소 변경되어있었다. 

다소 수정한 최종 코드는 아래와 같다.

```python
# Define the Slack bot's endpoint
SLACK_BOT_ENDPOINT = "https://slack.com/api/chat.postMessage"

# Define the Slack event listener
SLACK_EVENT_ENDPOINT = "https://slack.com/api/events.listen"

def handle_question(event_data):
    # Extract the user's question from the event data
    question = event_data["text"]
    logger.info('2' * 100)
    logger.info(question)

    # Use the GPT-3 model to generate an answer to the question
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=question,
        max_tokens=1000
    )

    # Extract the answer from the response
    answer = response["choices"][0]["text"]

    # Send the answer back to the user via Slack
    channel = event_data["channel"]

    # res = requests.post(SLACK_BOT_ENDPOINT % (channel, answer))
    response = requests.post("https://slack.com/api/chat.postMessage",
        headers={"Authorization": "Bearer "+SLACK_BOT_TOKEN},
        data={"channel": channel, "text": answer}
    )
    logger.info('3' * 100)
    logger.info(response.status_code)
    logger.info(response.text)

@csrf_exempt
@api_view(["POST"])
@permission_classes((AllowAny,))
def slack_action_test(request):
    event_data = request.data
    logger.info('1' * 100)
    logger.info(event_data)

    if "event" in event_data:
        event_type = event_data["event"]["type"]
        if event_type == "app_mention":
            handle_question(event_data["event"])
        return JsonResponse({"message": "ok"}, status=status.HTTP_200_OK)

    elif "challenge" in event_data:
        return JsonResponse({"challenge": event_data["challenge"]})

    else:
        return JsonResponse({})
```

Event Subscriptions 에 연결해두었던 Request URL 은 ngrok이 제공하는 URL에서 

실제 개발서버 URL로 변경했다.

'http://test-dev.xxx.com/slack/actions'

개발서버 URL이 위와 같다면 

'http://test-dev.xxx.com/slack/actions' 가 call 되면 위 최종 djnago 코드에서

def slack_action_test(request): 

메서드가 수행된다

그럼 event 내용을 parsing 해서 handle_question 이 수행되는 것이다.

SLACK_BOT_ENDPOINT 가 조금 변경되었고 해당 엔드포인트를 호출하는 방식을 get 에서 post 방식으로 변경하였다.

그럼 이제 한번 해볼까?

![mention-slack-bot1](/assets/images/openai9.png)

slack 봇을 멘션해보니 조금씩 이야기를 시작한다.

근데.. 너무 말이 많다.. 신기하지만 쉬지않고 계속이야기를 해서 질문을 한번 하기가 무섭다.

![mention-slack-bot2](/assets/images/openai10.png)

아래 부분에서 max_tokens 을 1000 으로 수정했는데도 조금 이야기가 길다.

open ai documentation 상으로 max_tokens 파라미터는 따로 지정하지 않으면 디폴트 값은 16이라고 한다.

적절한 값을 지정하면 조금더 간결하게 이야기 나눌수있을 것 같다.

그리고 engine은 조금더 최신엔진인 `text-davinci-003` 으로 변경했다.

```python
    # Use the GPT-3 model to generate an answer to the question
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=question,
        max_tokens=1000
    )
```

max_tokens | integer | Optional | Defaults to 16

The maximum number of **[tokens](https://beta.openai.com/tokenizer)** to generate in the completion.

The token count of your prompt plus **`max_tokens`** cannot exceed the model's context length. Most models have a context length of 2048 tokens (except for the newest models, which support 4096).

슬랙 메신저안으로 들어온 GPT3는 우리가 사용하기에 조금더 편리해지고 접근성도 낮아졌다.

아직 조금 부족한 부분이 있지만 그래도 처음시작하기 막막한 작업을 진행할때 가이드라인을 잡을수 있는 것 같아 훌륭하다

openai 에서는 3달동안 사용할수 있는 $18 크레딧을 제공한다. 해당 크레딧은 개인이 사용하기에는 적절한 것 같다. 

다양한 실험을 설계하고 맘껏 실행하시라. 

GPT3는 물건이다.