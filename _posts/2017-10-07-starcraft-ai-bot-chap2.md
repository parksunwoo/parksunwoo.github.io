---
title: 스타크래프트 인공지능 봇을 만들어보자 chap2
categories:
  - Dev
tags:
  - api
  - bot
  - starcraft
  - deepmind
last_modified_at: 2017-10-07T09:00:00-00:00
---

 \- 튜토리얼 따라해보기
​
[앤타로막장](https://github.com/parksunwoo/makjangProject/tree/sw/JAVA/BasicBot)[BasicBot 주소](https://github.com/parksunwoo/makjangProject/tree/sw/JAVA/BasicBot) 
​
[사내위키](https://github.com/SamsungSDS-Contest/2017Guide/wiki/TutorialLevel0Bot_JAVA) 목차
​
(입문단계) 봇 프로그램 기본구조 살펴보기
​
(초급단계) BWAPI 사용해보기
​
(초급단계) 적군 정보 업데이트하기
​
(초급단계) 일꾼으로 자원 채취하기
​
(초급단계) 유닛 생산 및 건물 건설하기
​
주요 프로그램들을 설명해보면
​
Main.java 는 MyBotModule() 을 실행시키는 main 이다
​
MyBotModule.java는 스타크래프트 게임과 connection을 맺어서 봇을 중개해주는 역할과 게임내 각종 이벤트 처리를 담당했다
​
여기에는 대회측에서 설정한 게임패배조건들이 설정되어있었다
​
처음에는 우리가 그리 중요하게 생각하지 않았던 타임아웃 조건도 설정되어있어서 막판에는 우리를 애태우기도 했다
​
InformationManager.java 는
​
게임 상황정보 중 일부를 자체 자료구조 및 변수들에 저장하고 업데이트하는 class 이다
​
아군, 적군의 메인 기지와 점령지역, 첫번째 입구, 두번째 입구와 같은 지역정보를 저장하고 정찰을 통해 얻은 정보도 update를 하였다
​
ScoutManager.java 를 통해  초기 정찰유닛을 지정하고 정찰지역으로 보낼수있었다
​
우리가 설정한 예로 설명하면 1) 건물중에 첫번째 배럭이 올라가고 있으면  2) 배럭위치에서 가장 가까운 일꾼을 정찰유닛으로 지정한다
​
```java
for (Unit unit : MyBotModule.Broodwar.self().getUnits()) { if (unit.getType() == UnitType.Terran\_Barracks && unit.getType().isBuilding() == true && unit.getType().isResourceDepot() == false) { firstBuilding = unit; break; } } if (firstBuilding != null) { // grab the closest worker to the first building to send to scout 
Unit unit = SwWorkerManager.Instance().getClosestMineralWorkerTo(firstBuilding.getPosition()); // if we find a worker (which we should) add it to the scout units // 정찰 나갈 일꾼이 없으면, 아무것도 하지 않는다 
if (unit != null) { // set unit as scout unit 
currentScoutUnit = unit; WorkerManager.Instance().setScoutWorker(currentScoutUnit); } }
​```
  
정찰유닛은 맵에따라 설정되어있는 본진을 현재 우리 본진과의 거리를 계산해서 가장 가까운 본진부터 순차적으로 정찰을 시작한다
​
```java
for (BaseLocation startLocation : BWTA.getStartLocations()) { // if we haven't explored it yet (방문했었던 곳은 다시 가볼 필요 없음) 
if (MyBotModule.Broodwar.isExplored(startLocation.getTilePosition()) == false) { // GroundDistance 를 기준으로 가장 가까운 곳으로 선정 
tempDistance = (double)(InformationManager.Instance() .getMainBaseLocation(MyBotModule.Broodwar.self()).getGroundDistance(startLocation) + 0.5); if (tempDistance > 0 && tempDistance < closestDistance) { closestBaseLocation = startLocation; closestDistance = tempDistance; } } } if (closestBaseLocation != null) { // assign a scout to go scout it 
commandUtil.move(currentScoutUnit, closestBaseLocation.getPosition()); currentScoutTargetBaseLocation = closestBaseLocation; }
​```

<figure>
  <img src="/assets/images/starcraft1.png" alt="Trulli" style="width:650, height:516">
  <figcaption>가운데 게이트 건설시작 후 가까운 본진부터 정찰시작</figcaption>
</figure>

<figure>
  <img src="/assets/images/starcraft2.png" alt="Trulli" style="width:650, height:516">
  <figcaption>6시 - 12시 방향 정찰 완료 후 적의 본진 위치 파악, 공격시작!</figcaption>
</figure>
​
제공된 소스에는 적본진을 발견했을 떄 정찰유닛의 상태를 변경하여 적 본진 가장자리를 따라 정찰을 할수도 있었다(정찰유닛이 죽을때까지?)
​
getScoutFleePositionFromEnemyRegionVertices () 메서드 참고
​
일꾼유닛을 적정한 미네랄에 지정해서 자원을 채취하게하는 것
​
건물을 건설하는 위치 선택과 같이 사람이 직접 게임을 플레이할때는 별로 고려하지 않아도 되는부분들이 인공지능 봇의 경우
​
그것을 미리 고려하여 일정조건을 부여해야 해서 고민이 깊어졌다
​
StrategyManager.java 는 상황을 판단하여, 정찰, 빌드, 공격, 방어 등을 수행하도록 총괄 지휘를 하는 class 이다
​
setInitialBuildOrder() 에서 초기에 우리가 구현하고자 하는 전략을 직접 구현을 하였다
​
건설할 유닛/ 건물목록을 저장하는 BuildOrderQueue.java 클래스에 우선순위에 따라 추가를 해놓으면 우선순위에 맞추어서 일꾼과 건물을 생산하였다
​
게임의 시작부터 종료때까지의 모든상황을 고려해서 입력을 해놓기에는 무리가 있기때문에 초반전략만 구현했다
​
executeCombat() 에는 공격유닛들의 공격조건을 구현했다
​
1)지어진 벙커가 있다면 벙커의 위치를 저장해서 2) 마린의 경우 벙커에 들어가있게하고 3) 메딕의 경우 마린 근처에 머물고 4) 탱크는 첫번째입구에서 시즈모드상태로 대기를 하도록 하였다
​
```java
for (Unit unit : MyBotModule.Broodwar.self().getUnits()) { // 벙커의 위치를 저장 
if (unit.getType() == UnitType.Terran\_Bunker && unit.isCompleted()) { bunker = unit; } // 마린의 경우 벙커에 들어가있거나 주위에 있도록 
if (unit.getType() == UnitType.Terran\_Marine && unit.isIdle()){ marine = unit; if (bunker != null && bunker.getSpaceRemaining() != 0){ commandUtil.rightClick(marine, bunker); }else { commandUtil.attackMove(marine, firstChokePoint.getPoint()); } // 메딕의 경우 마린근처에 머물도록
 }else if(unit.getType() == UnitType.Terran\_Medic && unit.isIdle()) { medic = unit; if(marine != null){ commandUtil.move(medic, marine.getPosition()); }else{ commandUtil.move(medic, firstChokePoint.getPoint()); } // 시즈탱크는 첫번째길목에서 시즈모드로 대기
  }else if(unit.getType() == UnitType.Terran\_Siege\_Tank\_Tank\_Mode){ tank = unit; commandUtil.attackMove(tank, tankPosition); if(tank.getPoint().getDistance(tankPosition) < 50){ tank.useTech(TechType.Tank\_Siege\_Mode); } } }
​```
  
초기 공격조건이 만족되면 공격유닛들이 적진으로 공격을 시작하는데
​
유닛들의 위치와 이동하는데 걸리는 시간을 따져보면 일렬로 가서 장렬히 전사하는 경우가 발생하기에
​
적군의 2번째 입구에서 일단 모인다음 일정유닛이 모이면 적의 본진으로 공격을 가게끔 구현하였다

​```java
if (unit.canAttack() || unit.canRepair()) { if(unit.isIdle()){ if(!isOkAttackBaseLocation){ commandUtil.attackMove(unit, secondChokePointEnemey.getPoint()); if(MyBotModule.Broodwar.getUnitsInRadius(secondChokePointEnemey.getPoint(), 250).size() > 12){ isOkAttackBaseLocation = true; System.out.println("isOkAttackBaseLocation = true"); }else{ isOkAttackBaseLocation = false; System.out.println("isOkAttackBaseLocation = false"); commandUtil.attackMove(unit, secondChokePointEnemey.getPoint()); } }else{ commandUtil.attackMove(unit, targetBaseLocation.getPoint()); } } }
```
​
글로 적으니 간단하지만 시즈탱크의 적절한 위치를 잡는게 어려워서 입구를 막아 다른유닛들이 해당위치로 가지못하는 경우도 발생하고
​
적의 본진으로 밀고 들어가야하는데 적정한 수를 설정하지못하여 공격유닛이 분산되는 경우도 발생했다ㅠㅠ
​
정말 하나를 구현하면 하나가 막히고 쉽지않은 과정이었다 
​
우리의 테란전략이 잘 먹히지 않는데 팀원들이 공감하게되었고.. 구현하지 못한 우리들의 책임이겠지만...
​
우리는 테란의 컨트롤을 감당할수없어서 프로토스의 필살기 전략을 구현하는데 모든 노력을 쏟기로했다
​
결국 8강전에 올라간 팀들의 경기를 보았을때는 현란한 시즈탱크 모드 변환을 구현한 팀을 볼수가 있었다 ^^;;
​
게임의 맵은 헌터, 로템, 투혼 3개 맵이었고 우리는 맵의 중앙에 2개 게이트를 짓고 질럿 물량으로 승부하는 전진2게이트 전략을 구현하기 시작했다
​
우선 맵별로 전략을 구분해서 구현했다 (3개 맵의 특성이 워낙 달라서)
​
2개 게이트를 지을 맵의 중앙위치를 직접 설정하였고
​
<figure>
  <img src="/assets/images/starcraft3.png" alt="Trulli" style="width:650, height:516">
  <figcaption></figcaption>
</figure>

​
mainBaseDefence() 에서는 본진에 들어온 적유닛을 제거, 저그의 초반러쉬를 막기위한 전략을 구현했다
​
상태값을 구분하여 넥서스 근처에 접근한 적유닛을 일꾼으로 때려잡는 부분이다

​```java
public void mainBaseDefence() { // mineralMoveCount가 0 보다 크면 적군에게서 가장 먼 미네랄로 모인다
 if (mineralMoveCount > 0) { moveEnemy\_NearMainBase\_UsingProbe(targetMineral, 300); mineralMoveCount--; return; // mineralMoveCount가 0이 되면 공격 
 } else if (mineralMoveCount == 0) { attackEnemy\_NearMainBase\_UsingProbe(enemyPosition, 300); mineralMoveCount--; return; // mineralMoveCount가 -1이면 공격 중이라는 뜻이다
 } else if (mineralMoveCount == -1) { Unit enemy = getEnemy\_NearMainBase(300); if (enemy != null) { attackEnemy\_NearMainBase\_UsingProbe(enemy.getPosition(), 300); return; } else { mineralMoveCount--; } //mineralMoveCount가 -2이면 공격이 끝났으니 다시 복귀
 } else if (mineralMoveCount == -2) { targetMineral = null; enemyPosition = null; stopProbe\_NearMainBase(500); mineralMoveCount--; // -3이 평시 상태. 계속 적이 공격오지는 않았는지 체크한다. 
 } else { if (MyBotModule.Broodwar.getFrameCount() % 5 != 0) return; if (getUnderAttackedUnit\_NearMainBase(300) != null) { Unit enemy = getEnemy\_NearMainBase(300); if (enemy != null) { enemyPosition = enemy.getPosition(); targetMineral = getMineral\_MostFarFrom\_Enemy(enemy); mineralMoveCount = 60; // 50프레임동안 유닛을 모은다. 
 } } } }
​```

elimination() 은 적의 건물을 다 제거하였는데도 숨겨진 건물을 발견하지 못하여 패배되는걸 막기위해
​
질럿의 수가 50마리 이상인데 유닛들이 공격상태가 아니면 flag를 통해 지도상의 모든곳을 랜덤하게 공격포인트로 찍어서 이동하게 하였다
​
후에 발견된 문제점은 해당 메서드가 시행되는 간격이 너무 길었고 발견된다고 해도 1,2마리가 이동하여 건물을 파괴하기에는 시간이 많이걸렸다
​
타임아웃을 우려하여 시간간격을 설정하였던걸 감안하면 지도 전체를 랜덤하게 가기보다는 적진부터 시작해서 가까운 곳으로 퍼져나가는 전략이 더 유효하지않았을까 생각해본다
​
그리고 이동 도중에 건물을 발견하면 다른 유닛들이 함께와서 공격을 하도록 했으면하는 아쉬운 생각이 든다
​
```java
public void elimination() { if (MyBotModule.Broodwar.getFrameCount() % 2701 != 0) return; if (!isElimination) { if (InformationManager.Instance().getUnitData(InformationManager.Instance().selfPlayer) .getNumUnits("Protoss\_Zealot") >= 50) { boolean noAttack = true; for (Unit u : MyBotModule.Broodwar.self().getUnits()) { if (u.isAttacking()) { noAttack = false; break; } } if (noAttack) { isElimination = true; } } } if (isElimination) { Random r = new Random(); int height = MyBotModule.Broodwar.mapHeight(); int width = MyBotModule.Broodwar.mapWidth(); int count = 0; for (Unit u : MyBotModule.Broodwar.self().getUnits()) { if (u.getType() == UnitType.Protoss\_Zealot) { count++; commandUtil.attackMove(u, new TilePosition(r.nextInt(width), r.nextInt(height)).toPosition()); if (count >= 30) return; } } } }
​```

toggleAttackMode() 임의로 설정한 toggleAttackMode\_stopAttack, toggleAttackMode\_startAttack 조건을 체크해서
​
공격유닛이 대기상태이나 일정수를 넘어서면 공격모드로 전환하고, 전투후 유닛들이 파괴되어 일정수 아래로 내려가면 다시 대기모드로 공수전환을 구현했다
​
```java
public void toggleAttackMode() { if (MyBotModule.Broodwar.getFrameCount() % 23 != 0) return; if (!isFirstExpantion) return; int idle = 0; for (Unit u : MyBotModule.Broodwar.self().getUnits()) { if (u.getType() == UnitType.Protoss\_Zealot && !u.isAttacking() && !u.isUnderAttack()) { if (BWTA.getGroundDistance(u.getTilePosition(), InformationManager.Instance() .getSecondChokePoint(InformationManager.Instance().selfPlayer).getPoint().toTilePosition()) <= 1000) idle++; } } if (idle <= toggleAttackMode\_stopAttack) isReadyAttack = false; else if (idle >= toggleAttackMode\_startAttack) isReadyAttack = true; } 
​```

<figure>
  <img src="/assets/images/starcraft4.png" alt="Trulli" style="width:650, height:516">
  <figcaption>일정 유닛수 도달 전까지 첫번째 입구에서 대기모드</figcaption>
</figure>

<figure>
  <img src="/assets/images/starcraft5.png" alt="Trulli" style="width:650, height:516">
  <figcaption>유닛 수 도달과 동시에 적진으로 공격시작</figcaption>
</figure>

​
unlimitedExpantion() 에서는 무한확장전략을 구현했다
​
1)미네랄이 350이상이며 2) 첫번째 확장을 했으며 3) 현재 넥서스를 짓고있지 않은상태이면
​
아군 메인과 적군 메인 그리고 아군의 첫번째 확장지역을 제외한 곳에 넥서스를 짓도록 하였다
​
<figure>
  <img src="/assets/images/starcraft6.png" alt="Trulli" style="width:650, height:516">
  <figcaption>위 화면은 앞마당 첫번째 확장시작 모습</figcaption>
</figure>

<figure>
  <img src="/assets/images/starcraft7.png" alt="Trulli" style="width:650, height:516">
  <figcaption>앞마당 확장과 동시에 질럿의 대기장소도 두번째 입구로 이동~</figcaption>
</figure>

​
정말 형들과 함께 진행하는 동안 푹 빠져서 살았던것같다 퇴근후에 시청 근처에 모여서도 하고
​
주말에도 시간을 내어 카페에서 이야기하면서 진행하는게 참 재미있었다
​
소스 제출일자가 다가오면서 우리집에 모여서도 하고 또 막차를 탈때까지 피씨방에서도 같이 구현을 했었다
​
우리의 코드가 예선전을 통과하지는 못했지만 구현하는 동안 
​
직장생활에 찌들어있던 우리들은 잠시나마 우리가 처음 프로그래밍을 공부하던 그때처럼 코드를 구현해보는 즐거움 속에 살았다
​
결과가 발표되고 우리끼리 자그마하게 뒷풀이를 하는 시간을 가졌다
​
결승전의 이야기를 잠시나누고 그동안의 즐거움에 대해 이야기를 나누었다 잊지못할 시간들이었다
​
\- 딥마인드의 다음종목은 스타크래프트
​
[스타크래프트2 api를 공개합니다](http://kr.battle.net/sc2/ko/blog/20944009/%EC%8A%A4%ED%83%80%ED%81%AC%EB%9E%98%ED%94%84%ED%8A%B8-ii-api%EB%A5%BC-%EA%B3%B5%EA%B0%9C%ED%95%A9%EB%8B%88%EB%8B%A4-2017-08-11)  : 기사보기
​
지난 8월, 블리자드 사는 스타크래프트 인공지능 api를 공개하였다
​
알파고를 만들었던 딥마인드는 이제 좀 더 높은 기술 수준을 요구하는 스타크래프트를 도전종목으로 택하고 
​
블리자드사와의 협업을 통해 인공지능 연구를 심화시키고 있다
​
머신러닝과 강화학습 부분을 공부하다보니 딥마인드의 머신러닝 분야에서 활약상을 새삼다시 느끼게되었다
​
단순히 알파고를 구현한줄 알았는데 논문을 바탕으로 이론을 직접 구현한 성과물이 바로 알파고였다
​
기존 한계를 몇가지 간단한 아이디어로 뛰어넘었다는 사실!
​
추가적으로 재미있는 사실들을 적어보면
​
1) 구글이 공정한 스타 2 대결을 위해 ‘로봇 팔’을 훈련한다는 이야기도 있다. 구글 머신러닝 연구팀을 이끄는 제프 딘은 지난해 3월 “현 시점에서 인공지능을 스타에 직접 접목할 경우 인간이 이기기는 매우 어려운데, 이는 컴퓨터가 인간보다 훨씬 많은 일을 동시에 소화하기 때문”이라며 “구글은 로봇 팔이라는 사람 팔 형태의 로봇을 만들어 훈련 중”이라고 밝혔다. 순다 피차이 구글 최고경영자(CEO)도 같은 해 5월 미국 캘리포니아주 마운틴뷰 쇼라인 엠피시어터에서 “구글은 머신러닝과 딥러닝을 활용해 로봇팔을 학습시키고 있다”고 말했다.
​
 굳이 로봇 팔을 쓰는 이유는 ‘컨트롤’ 유·불리 때문이다. 한 차례씩 공평히 돌 놓을 기회를 주고받는 턴제 게임인 바둑과는 달리, 스타는 컨트롤 면에서 인간이 훨씬 불리하다. 인간은 아무리 마우스를 빠르게 휘둘러도 서로 다른 두 지점을 한 번에 클릭하는 게 불가능하다. 하지만 컴퓨터는 내부 데이터 처리로 수십 군데를 동시에 클릭할 수 있다. 이 때문에 인간 프로게이머 분당 명령 횟수(APM)가 대개 300대 안팎인 반면, AI는 그 60~70배인 2만대에 달한다. 하지만 구글 딥마인드는 이와 같은 유리함을 버리고, 지성(知性)에 기반을 둔 전략과 전술만으로 인간을 상대하려 드는 것이다. 상당한 패기와 자신감이 엿보이는 대목이다.  
  
출처 : http://news.chosun.com/site/data/html\_dir/2017/05/24/2017052401387.html  
  
​
기사에서 딥마인드의 자신감이 묻어나온다 이기는게 당연하기에 인간에 맞추어 로봇팔을 만들어서 게임을 진행하겠다는 생각..
​
2) 사내 대회에서 우승한 팀은 2017 블리즈컨에 참가할수있는 비행기 티켓을 거머쥐게되었는데
​
평소 관심없었던 행사이지만 이번계기로 알게되어 우리팀의 굉장한 부러움을 샀다
​
 블리즈컨은 블리자드 엔터테인먼트가 주요 게임들을 홍보하기위해 반정기적으로 주최하는 컨벤션이다. 2005년 10월에 애너하임 컨벤션 센터에서 처음 열린 이후 계속 같은 장소에서 개최되고 있다. 컨벤션에서는 게임 관련 발표, 개발중인 신작과 게임 컨텐츠 미리보기, 블리자드 개발자와의 문답시간 등이 포함되어 있으며 다양한 블리자드 게임들을 체험할 수 있다. [위키백과](https://ko.wikipedia.org/wiki/%EB%B8%94%EB%A6%AC%EC%A6%88%EC%BB%A8)