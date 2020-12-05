---
title: "어떤 툴을 어떻게 사용해야할까"
categories:
  - Agile
tags:
  - teams
  - tools
  - howto
last_modified_at: 2020-12-05T13:06:00-00:00
---

![영화 나를 찾아줘 포스터](https://www.dropbox.com/s/5b9ku6mnmdtnwa9/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-12-05%20%EC%98%A4%ED%9B%84%201.08.15.png?raw=1)

팀에게 맞는 툴을 찾아줘...

이 포스팅의 결론은 **팀에게 완벽한 툴은 없다** 이다. 

하지만 우리팀이 여러 툴을 직접 사용하며 겪은 일을 읽으면

반면교사 삼을 만한 부분도 있을 것이라 생각해 글을 적게되었다

툴은 잘 사용하면 팀의 생산성을 높이고 중요한 부분이지만 그보다 중요한 건 툴은 말 그대로 도구라는 것이다.  어느 툴을 쓸지 고민하는데 너무 시간을 많이 쓰는 것보단 익숙한 툴을 사용하면서 일이 진행되게 하는게 먼저라고 생각한다. 

---
<br><br><br>

## Google sheets

올해 초 프로젝트 개발팀이 꾸려졌고 우리가 가장 처음 사용한 툴은 Google sheets 였다.

![Google sheets Page](https://www.dropbox.com/s/3vabcknr5mvloq7/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-29%20%EC%98%A4%ED%9B%84%201.41.02.png?raw=1)

내가 3번째 팀원으로 합류하기 전부터 이미 WBS, 기획 문서를 비롯해 많은 문서가 Google Sheets에 정리되어 작성되고 있었다. Google Sheets 는 무엇보다 빠르게 만들수 있다는 장점이 있지만 아무래도 엑셀의 특성상 작성하는 사람의 스타일을 많이 따라간다. 정해진 표준 양식이 없다면 다른 사람이 작성한 문서를 이해하고 따라가는데 다소 시간이 소요된다.

 또 다른 불편했던 점은 새로운 문서를 생성해서 공유할 때에 권한을 신청하고 수락하는 프로세스가 더해진다는 것이다 . 이부분은 보안상 필요한 부분이면서 같이 일하는 팀원끼리 이런 작업이 반복되면 아무래도 빠른 공유와 피드백을 얻기가 힘들었다.

버전이 업데이트 될 때마다 문서에 새 스프레드 시트를 추가 생성하는 방식으로 관리하다보니 나중에는 너무 많은 시트로 인해   사용이 힘들어졌다.

스스로 생각하기에 Google sheets 는 개인 문서를 만들고 관리하기에는 용이하지만 

팀단위에서 사용하려면 양식을 표준화해서 관리하고 수정과 편집이 적은 기록용으로서의 데이터가 알맞을 것 같다.
<br><br>
## Gitlab

private repository로 Gitlab을 선택해 개발자가 아닌 다른 팀원들도 이슈관리 툴로써 함께 사용하기 시작했다. Gitlab은 개발자가 주로 사용하는 도구이다보니 개발자관점으로 많은 부분이 디자인 되어있었다.  이슈 등록과 관리가 편하고 이슈를 close 할 때에도 (커밋을 할때 이슈번호를 멘션 달아서 하면) 손쉽게 할 수 있다.  등록된 이슈를 List, Board, Milestones View로 구분해서 볼수 있다는 것도 장점이다.

![https://www.dropbox.com/s/27kjk0mw383inku/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-30%20%EC%98%A4%EC%A0%84%207.39.27.png?raw=1](https://www.dropbox.com/s/27kjk0mw383inku/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-30%20%EC%98%A4%EC%A0%84%207.39.27.png?raw=1)

![https://www.dropbox.com/s/ygdn4jn1sz9sdf3/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-30%20%EC%98%A4%EC%A0%84%207.41.04.png?raw=1](https://www.dropbox.com/s/ygdn4jn1sz9sdf3/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-30%20%EC%98%A4%EC%A0%84%207.41.04.png?raw=1)

![https://www.dropbox.com/s/3s9k6ivh61xx2cp/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-30%20%EC%98%A4%EC%A0%84%207.41.53.png?raw=1](https://www.dropbox.com/s/3s9k6ivh61xx2cp/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-30%20%EC%98%A4%EC%A0%84%207.41.53.png?raw=1)

위에서부터 차례로 List, Board, Milestones view

우리팀은 Gitlab을 Slack과 연동해서 이슈가 새롭게 등록되고 close 될때 알람이 올수 있게 했다. 이슈 관리 툴로써는 만족했지만 관리자 입장에서는 이슈가 정체되는 병목 지점을 파악하기가 힘들었다. 더욱이 이슈가 쌓이는 속도가 사라지는 속도보다 빨라지는 순간 쌓이는 이슈를 관리하기가 어려웠다.

이때는 프로젝트 초반 이라 정확한 애자일 프로세스가 지켜지기보다 계획과 일정에 기반한 업무처리 프로세스였다. CI/CD 연동과 back-end / front-end 코드 분리를 시도하면서 repository를 Github로 이동했고 자연스럽게 Gitlab 은 사용이 적어졌다.

애자일 프로세스가 익숙하고 이슈를 잘 정리할 수 있는 팀이라면 Gitlab은 잘 활용할수 있는 툴이 아닐까 싶다.
<br><br>
## Notion

전사적으로 애자일 프로세스를 도입하기 시작하는 한편 코로나로 인해 재택근무 시즌이 다시찾아왔다. 우리팀은 처음 시도해보는 애자일 프로세스를  노션에 스프린트 칸반페이지를 만들어서 진행했다. 그전까지는 노션을 개인적으로 그리고  팀 회의록 및 기록용 정도로 사용하고 있었다. 회사차원에서 협업툴로 본격적으로 사용하기 시작했던 것이다. 

![https://www.dropbox.com/s/c0sgbrs9bn20l1b/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-30%20%EC%98%A4%EC%A0%84%208.05.12.png?raw=1](https://www.dropbox.com/s/c0sgbrs9bn20l1b/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-30%20%EC%98%A4%EC%A0%84%208.05.12.png?raw=1)

![https://www.dropbox.com/s/mvmdazydog3fslj/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-30%20%EC%98%A4%EC%A0%84%208.06.19.png?raw=1](https://www.dropbox.com/s/mvmdazydog3fslj/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-30%20%EC%98%A4%EC%A0%84%208.06.19.png?raw=1)

컨텐츠팀과 개발팀의 스프린트 칸반보드 카테고리

노션은 사용이 직관적이고 아는 만큼 활용할 수 있는 툴이었다. 위 사진과 같은 보드 형태의 페이지를 만드는 것도 어렵지 않고 각 상태별로 카드를 추가하고 이동하는 것도 편리했다. 필터기능이 있어 원하는 선택 기준으로 필터링해서 볼수 있는 등 장점이 많았다.  애자일 프로세스를 계속 반복하면서 스프린트 및 프로젝트 소멸 차트를  엑셀 시트와 연동해 스프린트 보드 위에서 차트를 바로 조회할 수 있는 발전된 형태로 사용하게 되었다

![https://www.dropbox.com/s/zqsjuscxbifq7gs/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-30%20%EC%98%A4%EC%A0%84%208.14.57.png?raw=1](https://www.dropbox.com/s/zqsjuscxbifq7gs/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-11-30%20%EC%98%A4%EC%A0%84%208.14.57.png?raw=1)

스프린트 소멸차트가 더해진 스프린트 보드

노션은 애자일 프로세스와 같이 특정 목적으로 사용이 적합한것 같다. 
- 회사들의 노션 실제 사용기 ([https://www.notion.so/customers](https://www.notion.so/customers))

반면 팀내 문서들을 관리하기에는 개인마다 문서를 만드는 양식이 달라 어디에 어떤 문서가 저장되어있는지 검색하기가 어려웠다. 이 문제는 팀 단위의 공통적인 페이지 양식을 맞추고 미리 생성해둔 템플릿을 함께 쓰는 방법이 필요할 것 같다.

현재는 팀별로 나누어진 공통 페이지를 갖고 그곳에 문서를 추가하는 방식으로  진행되고 있다. 
<br><br>
## Teams

팀 플레이를 위한 최적의 툴. 회사에서 Microsoft office 365 를 사용하다보니 자연스레 Teams 도 찾아보게 되었다. Teams 는 이름 그대로 속해 있는 팀원들끼리 대화하고 일정을 함께 세우고 공유할 수 있다. 

![https://www.dropbox.com/s/jt339br9u7cuxu0/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-12-04%20%EC%98%A4%EC%A0%84%207.56.51.png?raw=1](https://www.dropbox.com/s/jt339br9u7cuxu0/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-12-04%20%EC%98%A4%EC%A0%84%207.56.51.png?raw=1)

Teams 의 팀 메뉴

팀 메뉴에선 원하는 팀을 만들 수 있고 팀별로 필요한 사람을 추가 할 수 있다. 팀에는 여러 개의 채널을 생성해서 필요한 목적에 맞게 사용할 수 있다. 기본적으로 채널별로 게시물 / 파일 / Wiki 가 제공되어서 우리 팀의 경우 계정 정보와 주요 설정 정보를 채널 Wiki에 정리를 했다. 

팀에 속해 있는 사람만 접근이 가능해 보안상 걱정이 없고 팀  별 링크 생성 및 이메일 전송도 가능하다 

![https://www.dropbox.com/s/dsnn6k5u9162muf/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-12-04%20%EC%98%A4%EC%A0%84%208.01.08.png?raw=1](https://www.dropbox.com/s/dsnn6k5u9162muf/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-12-04%20%EC%98%A4%EC%A0%84%208.01.08.png?raw=1)

또 유용했던 기능은 일정을 생성해두고 위에 보이는 일정을 선택하면 자동으로 생성된 회의방에 들어가 팀원들과 화상회의를 진행할 수 있다. 별거 아닌 기능일 수 도 있지만 Zoom 또는 Google Meet 의 경우 회의를 할 때마다 새로운 링크를 생성해서 팀원들에게 공유해야 한다. 하지만 Teams 의 경우 따로 전달하지 않아도 해당하는 일정의 모임으로 모이기만 하면 된다!

Teams의 또 다른 장점은 많은 다른 앱들과 연동을 해서 사용할 수 있다는 점 그리고 Microsoft office 사용자라면 기존 사용하던 도구와 연계해서 사용할 수 있다는 것이다. 

![https://www.dropbox.com/s/pkjf0c30rlbb6lm/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-12-04%20%EC%98%A4%EC%A0%84%208.05.15.png?raw=1](https://www.dropbox.com/s/pkjf0c30rlbb6lm/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202020-12-04%20%EC%98%A4%EC%A0%84%208.05.15.png?raw=1)

Teams의 앱 

<br><br><br>
---

현재 우리팀은 스프린트 칸반은 Notion을 통해서 진행중이고 Wiki 및 일정, 팀 회의는 Teams 를 활용해서 진행중이다. 팀에게 맞는 툴은 팀이 변하는 모습을 반영하고 있기도 하다. 더 나은 툴을 찾기 위한 우리의 여정은 계속 될 것이다.