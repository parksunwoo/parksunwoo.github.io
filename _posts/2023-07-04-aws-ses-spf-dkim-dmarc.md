---
title: Amazon Simple Email Service 이메일 인증 SPF, DKIM, DMARC
categories:
  - Dev
tags:
  - AWS
  - SES
  - SPF
  - DKIM
  - DMARC
  - spam
last_modified_at: 2023-07-04T22:30:00-00:00
---

## 스팸함에 들어가는 메일을 구제하라!

---

Amazon SES는 인바운드 및 아웃바운드 이메일 서비스이다.

이메일 기능을 구현할때에 조금 더 쉽게 구현할 수 있게 한다.

기본적으로 고객에게 메일을 보내려면 

TO. 고객에게  보낼 FROM 메일주소가 필요하다. 

여기서 FROM 메일주소 는 도메인 또는 이메일 주소가 가능한데 여기서는 이메일 주소로 해서 진행해보겠다.

위에서 이야기한 도메인 또는 이메일 주소는 Amazon SES 에서 확인된 자격 증명 메뉴에서 확인할 수 있다.

![Identifier](/assets/images/identifier.png)

해당하는 자격증명, 여기서는 이메일 주소인 를 클릭해 상세화면으로 들어가면

![DKIM](/assets/images/dkim.png)

아래 "인증" 섹션에 **DomainKeys Identified Mail(DKIM) 를 확인할 수 있다.**

*DomainKeys Identified Mail(DKIM)* 은 발신자가 암호화 키를 사용하여 이메일 메시지에 서명할 수 있도록 허용하는 표준. 그런 다음 이메일 공급자는 이러한 서명을 사용하여 메시지가 전송 중에 타사에 의해 수정되지 않았는지 확인한다.

![How does DKIM work? - https://dmarcian.com/what-is-dkim/](https://dmarcian.com/wp-content/uploads/2022/05/Frame-1975-1-1024x555.png)

**DKIM** 작동 방식을 단순화한 버전은 다음과 같다.

1. **서명:** 아웃바운드 이메일 서버(이메일을 보내는 서버)가 메시지를 보낼 때 개인 키를 사용하여 해당 이메일에 대한 고유한 DKIM 서명을 생성합니다. 서명은 이메일 헤더에 삽입됩니다.
2. **게시:** DKIM 서명을 확인하는 데 사용할 해당 공개 키가 이메일을 보내는 도메인의 DNS 레코드에 게시됩니다.
3. **확인:** 인바운드 서버(이메일을 받는 서버)는 이메일 헤더에서 DKIM 서명을 보게 됩니다. 그런 다음 보낸 사람 도메인의 DNS 레코드에서 공개 키를 가져옵니다.
4. **비교:** 인바운드 서버는 이 공개 키를 사용하여 DKIM 서명을 해독하고 원본 메시지 다이제스트를 복구합니다. 또한 수신된 이메일 콘텐츠에서 새 메시지 다이제스트를 계산합니다. 이 두 다이제스트가 일치하면 이메일이 전송 중에 변경되지 않았으며 실제로 소유권이 주장된 도메인에서 온 것으로 확인됩니다.

Amazon SES 에서 DKIM 구성을 하는 것은 해당 안내서를 참고한다면 어렵지 않다

[send-email-authentication-dkim](https://docs.aws.amazon.com/ko_kr/ses/latest/dg/send-email-authentication-dkim.html)

보통은 메일 기능을 구현할때 여기까지 정도에서 끝나기마련인데

개발된 기능을 통해 메일을 보내는 테스트를 해보면 새로운 사실을 알게된다!!

![Spam Mails](/assets/images/spam-mails.png)

어렵게 구현한 기능인데, 메일은 보내졌지만 내가 보낸 메일들은 고객의 스팸함에 차곡차곡 쌓이게 된다.

(광고)와 같은 키워드가 붙어서 스팸메일로 분류되었을 수도 있지만

![Spam Mail Detail](/assets/images/spam-mail-detail.png)

고객입장에서는 해당메일을 확인하면서 스팸이 아님 이라는 버튼을 눌러 해당 메일을 구제해야한다.

해당 버튼을 누르지 않는다면 이제 저 메일주소로 보내는 메일들은 자연스럽게 스팸함에 쌓이게된다.

고객이 바로 확인하고 피드백을 주어야하는 경우에는 조금더 복잡해진다. 해결해야만한다..!


## 스팸함에 들어가는 메일을 구제하라!

chat GPT에게 어떻게 해야하는지 질문을 하니 여러가지 방법을 제시해준다.

1. **Authenticate Your Emails**:
2. **Maintain Good IP Reputation**:
3. **Maintain a Good Sending Reputation**:
4. **Content**:
5. **Personalization and Engagement**:
6. **Feedback Loops**:
7. **Provide Clear Unsubscribe Option**:
8. **Test Your Emails Before Sending**:

전용 IP를 구성하거나 평판을 관리하는 것을 안내하고 해당 사항들도 물론 중요하지만

가장 처음에 이야기한 **Authenticate Your Emails 부분을 조금더 자세히 뜯어보면**

**Authenticate Your Emails**: Email authentication helps servers verify that your emails aren't forged and shouldn't be marked as spam. Since you're using Amazon SES, you should set up DKIM (DomainKeys Identified Mail), `SPF (Sender Policy Framework), and DMARC (Domain-based Message Authentication, Reporting & Conformance)`. You mentioned you've confirmed DKIM, which is great, but also ensure SPF and DMARC are correctly configured.

DKIM 구성 이외에도 SPF, DMARC 구성이 나온다. SPF, DMARC 구성 은 무엇일까?
DKIM 구성가 어떤게 다를까?


*Sender Policy Framework*(SPF)는 이메일 스푸핑 방지를 위해 마련된 이메일 검증 표준입니다.

Amazon SES를 통해 보내는 메시지는 `amazonses.com`의 하위 도메인을 기존 MAIL FROM 도메인으로 자동 사용합니다. 기본 MAIL FROM 도메인은 이메일을 전송한 애플리케이션(이 경우 SES)과 일치하므로 SPF 인증은 이 메시지를 성공적으로 검증합니다.

다음은 SPF 프로세스의 기본 단계입니다.

1. **게시:** 도메인 소유자가 DNS에 SPF 레코드를 게시합니다. 이 레코드는 도메인을 대신하여 이메일을 보낼 권한이 있는 모든 서버의 IP 주소를 나열합니다.
2. **보내기:** 도메인에서 온 것처럼 서버에서 이메일을 보냅니다.
3. **확인:** 이메일이 수신되면 수신 메일 서버는 이메일의 반환 경로(이메일의 "보낸 사람" 주소와 일치해야 함)에 나열된 도메인의 DNS에서 SPF 레코드를 조회합니다. 머리글).
4. **확인:** 수신 메일 서버는 발신 서버의 IP 주소가 SPF 레코드에 있는지 확인합니다. 그렇다면 확인이 통과됩니다. 그렇지 않으면 확인에 실패합니다.

**이메일 스푸핑**은 발신자 주소를 위조하여 이메일 메시지를 생성하는 것입니다. 주된 목적은 수신자를 속여 메시지를 열고 응답하도록 하는 것입니다. 발신자 주소는 종종 수신자가 이메일을 신뢰할 수 있는지 여부를 결정할 때 가장 먼저 보는 항목이므로 피싱 및 사기 시도에 일반적으로 사용됩니다.

스푸핑 시나리오에서 이메일의 "보낸 사람" 필드는 실제 보낸 사람이 아닌 보낸 사람을 표시하도록 변경됩니다. 이것은 평판이 좋은 조직이거나 알려진 연락처일 수 있으며 수신자가 이메일 내용을 신뢰하도록 합니다. 스푸핑된 이메일은 피싱 시도, 맬웨어 확산 또는 사기 활동과 같은 악의적인 목적으로 사용될 수 있습니다.

## 확인된 도메인이 지정된 MAIL FROM 도메인을 사용하도록 구성하려면,

1. [Amazon SES](https://console.aws.amazon.com/ses/)에서 Amazon SES 콘솔을 엽니다.
2. 왼쪽 탐색 창의 **구성(Configuration)** 아래에서 **확인된 자격 증명(Verified identities)**을 선택합니다.
3. 자격 증명 목록에서 **자격 증명 유형(Identity type)**이 **도메인(Domain)**이고 **상태(Status)**가 *확인됨(Verified)*인 경우에 구성하려는 자격 증명을 선택합니다.
4. 화면의 맨 아래의 **사용자 지정 MAIL FROM 도메인(Custom MAIL FROM domain)** 창에서 **편집(Edit)**을 선택합니다.
5. **일반 세부 정보(General details)** 창에서 다음을 수행합니다.
    1. **사용자 지정 MAIL FROM 도메인 사용(Use a custom MAIL FROM domain)** 확인란을 선택합니다.
    2. **MAIL FROM 도메인**에 MAIL FROM 도메인으로 사용할 하위 도메인을 입력합니다.
    
    아래 그림에서는 mail. 서브도메인을 입력했다.

    
    ![Mail From Domain](/assets/images/mail-from-domain.png)
    
    1. **MX 장애 발생 시 동작(Behavior on MX failure)**에서 다음 옵션 중 하나를 선택합니다.
        - **기본 MAIL FROM 도메인 사용** – 사용자 지정 MAIL FROM 도메인의 MX 레코드가 올바르게 설정되지 않은 경우 Amazon SES가 `amazonses.com`의 하위 도메인을 사용합니다. 하위 도메인은 Amazon SES를 사용하는 AWS 리전에 따라 달라집니다.
    2. **변경 사항 저장(Save changes)**을 선택합니다. 이전 화면으로 돌아갑니다.
    
6. MX 및 SPF(TXT 형식) 레코드를 사용자 지정 MAIL FROM 도메인의 DNS 서버에 게시합니다.
    
    **사용자 지정 MAIL FROM 도메인(Custom MAIL FROM domain)** 창에서 이제 **DNS 레코드 게시(Publish DNS records)** 테이블에 도메인의 DNS 구성에 게시(추가)해야 하는 MX 및 SPF(TXT 형식) 레코드가 표시됩니다. 이 레코드는 다음 표에 표시된 형식을 사용합니다.
    
    | 이름 | 유형 | 값 |
    | --- | --- | --- |
    | subdomain.domain.com | MX | 10 feedback-smtp.region.amazonses.com |
    | subdomain.domain.com | TXT | "v=spf1 include:amazonses.com ~all" |

위의 경우에는 이름부분의 [subdomain.domain.com](http://subdomain.domain.com) 이 [mail.aiffel.io](http://mail.aiffel.io) 로 변경될 것이고

MX 와 TXT 2개의 유형의 레코드를 Route53에 입력하면 된다.

여기까지 진행하면 이메일 스푸핑 방지하기 위한 SPF 구성은 완료되었다.

Domain-based Message Authentication, Reporting and Conformance(DMARC)는 SPF와 DKIM 을 사용해 이메일 스푸핑을 찾아내는 이메일 인증 프로토콜.

위 SPF 구성보다는 조금 더 간결한데 Amazon Route 53 에 아래 레코드를 추가하면된다

| 이름 | 유형 | 값 |
| --- | --- | --- |
| _dmarc.example.com | TXT | "v=DMARC1;p=quarantine;pct=25;rua=mailto:dmarcreports@example.com" |

위의 경우에는 이름 부분의 `_dmarc.example.com` 이 _dmarc.aiffel.io 로 변경,

값 부분에서는 모든 메시지에 DMARC 정책이 적용되기 위해서는 pct 태그 부분

pct=25; 부분을 제거하고 mailto 부분에는 실제로 레포팅을 받게된 이메일 주소를 기재하였다.

여기까지 진행하면 DMARC 구성까지 완료되었다.

뭔가 여러개의 구성을 했는데 이렇게 하면 스팸함에 메일이 들어가는 걸 막을 수 있을까?

[https://www.learndmarc.com/](https://www.learndmarc.com/) 해당 사이트에서는 SPF, DKIM, DMARC 구성이 제대로 되었는지 웹사이트에서 하나씩 결과를 확인해볼 수 있다.

개발한 기능을 통해 아래와 같은 테스트 이메일 주소(ld- 로 시작하는 이메일)로 메일을 보내게 되면.

![learn dmarc 1](/assets/images/learn-dmarc-1.png)

메일을 수신한 이후부터 재미난 일이 벌어진다.

화면이 다이나믹하게 변경되면서 SPF, DKIM, DMARC 하나씩 인증 결과값을 확인할 수 있다.

![learn dmarc 2](/assets/images/learn-dmarc-2.png)

SPF 의 DMARC Alignment 부분이 서로 달라 DMARC 구성에서 SPF 가 FAIL로 나오긴 했지만

**DMARC 의 경우 SPF 또는 DKIM 중 한개만 구성되어 있어도 되기에**

결과적으로는 DMARC Result 가 PASS 로 나온다

**Final verdict:**

**DMARC does not take any specific action regarding message delivery. Generally, this means that the message will be successfully delivered. However, it's important to note that other factors like spam filters can still reject or quarantine a message.**

그리고 이제 시스템에서 보내는 메일이 스팸함이 아닌 받은편지함에서 확인할 수 있게된다.

이제 기술적으로는 스팸메일에서 벗어났지만 무엇보다 중요한건 메일에 담겨지는 내용에서 고객에게 외면받는다면 이제 손쓸 방법이 없다. 

GPT가 물고온 답변에 **Reputation, Content, Feedback 단어들이 있었다는 걸 잊지말자 - 끝**



## 참고자료

[send-email-authentication-dmarc](https://docs.aws.amazon.com/ko_kr/ses/latest/dg/send-email-authentication-dmarc.html)

[신뢰할 수 있는 이메일 발송 도메인](https://kdevkr.github.io/email-spf-dkim-dmarc/)

[https://www.learndmarc.com/](https://www.learndmarc.com/)