---
title: Referees disappeared in Baseball Game - What happened?
categories:
  - ML
tags:
  - idea
  - baseball
  - referee
  - ai
last_modified_at: 2018-03-17T09:00:00-00:00
---

\# Project ideas (Question and Data)

Referees disappeared in Baseball Game: What happened?

In the baseball game, sometimes the game flow or match results changes depending on the wrong judgment of the referee. Using the data of the baseball live video of Naver, i will try to implement baseball referee AI which makes an accurate judgment in the baseball game Live.

Major League Baseball's (MLB) Statcast is measures 27 types of data such as the ball thrown by the pitcher, the ball hit by the batter, runner and defensive movement. A part of the data measured by Statcast is open to the public from 2015 and is also used for broadcasting etc. Statcast also have some limitations. First of all, the ball may not be recognized in some cases. Next, the strike zone is a Three-dimensional space that takes into account the stance of the batter, the moving batting form. However, using the batter's height information only, measure the vertical length of the strike zone and draw a plane rectangle to determine the zone.

<figure>
  <img src="/assets/images/referee-baseball.png" alt="Trulli" style="width:580, height:535">
  <figcaption></figcaption>
</figure>

In related work, there is research on <3D Object Manipulation in a Single Photograph using Stock 3D Models> at Carnegie Mellon University. Free photograph software created based on research, OM3D recognizes the shape of the selected object and connects the existing 3D image with the data. Thus, general 3D models already exist can be 3D modeled smoothly via software. However, it may be difficult to implement for images that have personal characteristics such as face, hands and so on.

 Therefore, i propose project ideas for implementing Baseball Referee AI, using baseball video data of Naver Sports (2007-2017 season) and KBO competition rules / player data. 1) Perform strike zone 3D modeling work of each batter in image of baseball game  2) Compare strike zone 3D images created in advance and the position of the ball on the baseball broadcast screen to make a strike-ball decision 3) Implement baseball referee AI by enlarging the range of judgment

I will expect statistics and data generated through the research to be released to the Naver Sports Site and lead to the expansion of open data on Korean baseball.

\### Reference Sites

[http://www.cs.cmu.edu/~om3d/](http://www.cs.cmu.edu/~om3d/)

[http://sports.news.naver.com/index.nhn](http://sports.news.naver.com/index.nhn)

[https://www.koreabaseball.com/](https://www.koreabaseball.com/)