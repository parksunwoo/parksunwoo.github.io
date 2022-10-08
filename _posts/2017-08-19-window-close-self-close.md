---
title: window.close 와 self.close
categories:
  - Dev
tags:
  - web
last_modified_at: 2017-08-19T09:00:00-00:00
---

팝업창을 닫아야 하는데 ie에서는 잘 작동되는 소스가 ipone 웹, 앱에서는 작동되지않는 문제가 발생다른 방법으로 해결하기는 했으나 close 메서드가 다양하게 있는걸 알고 각각의 특징을 정리해두고자 한다  
  
window.close();  
현재창을 닫거나 호출된 창을 닫는다이 메서드는 window.open() 메서드를 사용해서 열린 창들에게만 적용된다스크립트로 호출된 창이 아니면 다음과 같은 에러가 나오기도 한다Scripts may not close windows that were not opened by script.

사용예

1\. window.open 으로 열린 창을 닫을때

var openedWindow;

function openWindow() {

  openedWindow = window.open('moreinfo.htm');

}

function closeOpenedWindow() {

  openedWindow.close();

}

2\. 현재 창을 닫을때

과거에는 window 인스턴스의 close()를 호출하기보다 window 오브젝트의 close() 메서드를 직접 호출하곤 했다

script로 생성된 창이거나 그렇지 않거나 가장 위에 표시된 창을 닫았다

하지만 보안상의 이유로 script로 열린 창이 아니면 닫히지 않는다

(Firefox 46.0.1: scripts can not close windows, they had not opened)

\\

function closeCurrentWindow() {

  window.close();

}

self.close();

window.clsoe()가 현재 창을 닫는것이라 할때 self.close()는 자기 자신 창을 닫는다고 한다 뭔가 의미가 모호하다...

아래 블로그 내용으로 봤을때는 ie 6.0  이전에서 사용하는 메서드로 정리해둔다

참고 사이트

https://developer.mozilla.org/ko/docs/Web/API/Window/close

[ie버전에 따른 윈도우창 닫기](http://qkrrjsgh.tistory.com/4) 

[팝업창 관련 자주 사용하는 스크립트 모음](http://jace.tistory.com/4)