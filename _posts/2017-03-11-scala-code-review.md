---
title: Scala Code Review- foldLeft and foldRight (번역)
categories:
  - Dev
tags:
  - scala
  - flodleft
last_modified_at: 2017-03-11T09:00:00-00:00
---

스칼라 문제를 풀다보면 foldLeft와 foldRight 가 문제를 푸는데 많이 활용된다

헷갈리는 개념을 좀 더 명확히 하고자 영문블로그 내용을 번역, 요약해서 정리해본다

원문은 [https://oldfashionedsoftware.com/2009/07/10/scala-code-review-foldleft-and-foldright/](https://oldfashionedsoftware.com/2009/07/10/scala-code-review-foldleft-and-foldright/) 이다

이것은 foldLeft 함수의 시그니처로 List\[A\]는 type A들을 포함하는 List형태이다

```scala
def foldLeft\[B\](z: B)(f: (B, A) => B): B
```

이 함수는 단지 2개 괄호속에 있는 2개의 인자( z와 f)를 가진다

첫번째 인자 z는 B 타입이며, 이것은 list 타입과는 다르다

두번째 인자 f 는 B와 list를 구성하는 A를 파라미터로 가진다 그리고 결과값으로 B를 출력한다

따라서 f 함수의 목적은 B값을 취하여 리스트를 이용하여 그 값을 다시 B형태로 

리스트의 첫번째 요소는 첫번째 인자인 z이며, z는 f 함수의 첫번쨰 인자로 사용된다

두번째 요소로는 f함수의 첫번째 호출결과값이 B 형태의 인자로 사용된다

예를들어 설명하면 우리는 1,2,3으로 된 리스트가 있다

우리가 foldLeft("X")((b,a) => b+a)를 호출하면

첫번째 요소인 1은 우리가 정의한 X에 더해져서 "X1"을 출력한다

그리고 두번째 요소인 2는 함수에서 문자열 "X1"에 더해져 "X12"를 출력한다

그리고 마지막 요소인 3은 "X12"에 더해서 "X123"을 출력한다

추가로 다른 예는..

`list``.foldLeft(``0``)((b,a)` `=``> b``+``a)`

`list``.foldLeft(``1``)((b,a)` `=``> b``*``a)`

`list``.foldLeft(``List``[``Int``]())((b,a)` `=``> a :: b)`

첫번째 예제는 단순하다. z 값은 이전의 문자열 "X"를 대신해서 숫자 0이다

이 fold 결합은 이전과는 달리 더하기 연산이다. 따라서 여기서 fold 는 list에 있는 모든값의 합계가 된다

2번째 예제는 곱셈결합이다. 여기서 우리는 이 경우에 z값이 1인것을 봐야한다

3번째 예제는 조금 복잡한데, 비어있는 숫자배열에서 출발해서 각각의 요소들을 accumulator에 더한다

(우리는 함수에서  b 인자를 accumulator라 부르는데 이것은 list의 data들을 누적해서 더하기때문에 )

여기선 머리부터 시작해서 accumulator 리스트의 시작부터 원본 리스트의 마지막 까지 더해가기때문에

결국 원본 리스트의 역순을 출력한다.

foldRight 함수는 foldLeft와 같은 방식으로 동작한다. 차이점은 리스트의 끝부터 시작해서 머리까지 작업을 한다는 것이다.