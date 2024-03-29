---
title: 스타크래프트 인공지능 봇을 만들어보자 chap1
categories:
  - Dev
tags:
  - bwapi
  - bwta
  - starcraft
  - ai
last_modified_at: 2017-10-07T09:00:00-00:00
---

 - 막장을 위하여! 위대한 여정의 시작

 회사에서 개발자 우대문화를 만든다는 취지아래 해마다 알고리즘(?) 대회를 해왔는데 올해 종목은 스타크래프트였다

스타크래프트를 잘 하는건 아니지만, 친구들과 함께하던 추억이 있는 게임이기도 하고 올해는 한번 참여해보고 싶은 마음이 들었다

사내대회이기에 주변에 관심있을만한 동기들을 조금씩 찾아보기 시작했다

다행히도 개발을 잘하고 관심있어하는 형이 흔쾌히 함께하겠다고 해서 순조로운 시작이었다

2명에서 시작하는거지만 그래도 시간을 내어 자주 만나 앞으로의 계획을 세우는것부터 시작했다

하지만 처음부터 우리가 난관에 부딪힌건 어떤 종족을 선택하느냐였다..

[어느 종족을 하는 게 좋을까요?](https://starcraft.com/ko-kr/articles/20961942) 

스타크래프트 리마스터 페이지에도 이런 글이 남겨져있을정도로 스타크래프트는 어느 종족을 선택하느냐에 따라

운영방식이 많이 달라지기에 우리는 신중할수밖에 없었다 

결론적으로 우리는 처음에는 저그를 선택했다가 다시 테란으로 바꿨다가 결국에는 프로토스로 대회에 참가했지만..

우리가 처음 저그를 선택한 이유는 단순했다 우리가 직접 컨트롤 하는게 아니니 

최대한 컨트롤 영향이 적은 저그를 선택해서 물량으로 밀어붙이자.

지금 생각해보면 처음대로 밀고나갔어도 재미있을뻔했지만ㅋㅋ

혼자하는 프로젝트가 아니다보니 협업을 할 공간을 마련하는것도 중요했다

처음 고려했던 곳은 깃헙이었으나 깃헙에선 우리의 전략이 노출될 우려가 있어서 패스하기로 하고

사적인 공간인 깃랩에서 진행하기로 결정!

깃랩은 처음 사용하기도 했지만 이슈보드나 마일드스톤같이 프로젝트를 진행하기에 큰 어려움이 없게 잘 구성되어있었다

우리가 할일들을 간단히 등록해놓고 서로 공유할수도 있었다

물론 우리가 주로 이야기한건 카톡으로 이야기를 많이 했지만 ^^;;

깃랩은 깃허브와 연동이 되어 로그인이 가능하니 참고하면 좋을것같다

아 우리 팀명은 "막장의 전설" 이었다

어릴적 스타에 관심있는 사람이면 한번쯤 들어보았을 법한 쌈장 SSamzang을 기억하며 우리도 그때의 영광을 다시 한번 살려보겠다고 지었다

팀원 한명이 늘어나서 팀명은 "앤타로 막장"으로 (막장을 위하여?) 최종 확정되었다

[코넷으로 접속하라 쌈장의 전설 마침표...](http://www.whitepaper.co.kr/news/articleView.html?idxno=66249) 기사보기

\- 개발환경 설정, 설정이 반이다

깃을 평소잘 사용하지 않았던 우리들은 환경설정에 꽤나 시간을 쏟아야했다

초기에는 다행히도 소스수정이 많지 않아서 깃랩에서 직접 수정을 하는 노가다를 하기도 했지만

이래선 안되겠다는데 공감한 이후로 이클립스에서 깃과 연동하는 환경설정을 하고 간단히 커밋, 푸쉬로 변경된 소스를 반영할수 있었다

깃은 사용하면 사용할수록 정말 개발자들을 위해 잘 만들어졌다는 생각을 하게되었다

 대회에서는 봇을 구현할수있도록 기본적인 기능들이 구현된 소스를 제공했다

우리는 주어진 소스에서 충돌을 방지하기 위해 최대한 변경을 자제하고 

우리가 구현할 소스들을 새롭게 생성한 프로그램에 작성하는 방식으로 시작하였다

[알고리즘 경진대회 공식 깃헙](https://github.com/SamsungSDS-Contest/2017Guide/wiki) 에서 좀 더 상세한 환경설정 부분을 참고할수있다

스타크래프트 버전은 1.16.1 버전 이었으며 회사에서 제공한 스타크래프트를 설치한 후

BWAPI 라이브러리를 추가 설치해야한다 

BWAPI (Brood War Application Programming Interface) 는 스타크래프트 (Starcraft: Broodwar)  
게임과 상호작용하는 무료 오픈소스 라이브러리이다

BWTA (Brood War Terrain Analyzer) 는 BWAPI 를 사용하여 게임 맵을 분석하고 분석 결과를 반환하는 함수들을 포함하는  
add-on 라이브러리이다

<figure>
  <img src="/assets/images/starcraft.png" alt="Trulli" style="width:900, height:506">
  <figcaption>Starcraft play in IDE</figcaption>
</figure>


설치가 완료되면 이클립스로 작성한 봇 프로그램을 Run하고  Chaoslauncher로 게임을 실행하면 작성한 봇을 실행시킬수있다!

원래 제공된 봇 소스가 C++로 되있는걸 자바로 변환한 거라 2가지 버전으로 제공되고 있었다

 [uAlbertaBot](https://github.com/davechurchill/ualbertabot), [Atlantis](https://github.com/Ravaelles/Atlantis), [BWSAL](https://github.com/Fobbah/bwsal)  3개의 오픈소스를 참고하여 작성되었다고 한다

위에 작성되어있는 깃헙주소에서 환경설정 부분을 참고하시고 궁금한 사항은 글을 남겨주시면 확인되는대로 답변을 드리겠다