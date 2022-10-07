---
title: 공변성/반공변성(Variance)
categories:
  - Dev
tags:
  - scala
  - variance
last_modified_at: 2017-03-04T09:00:00-00:00
---

공변성/반공변성(Variance)
스칼라의 타입 시스템은 다형성 뿐 아니라 클래스 계층관계도 처리해야만 한다. 클래스의 계층은 상/하위 타입 관계를 만든다. OO와 다형성이 합쳐지면서 생기는 핵심문제는 바로 “

T’

가 

T

의 하위클래스일 때, 

Container[T’]

가 

Container[T]

의 하위클래스인가?”하는 문제이다. 프로그래머는 클래스 계층과 다형성 타입에서 다음과 같은 관계를 공변성 표기를 통해 표시할 수 있다.


의미	스칼라 표기
공변성(covariant)	C[T’]는 C[T]의 하위 클래스이다	[+T]
반공변성(contravariant)	C[T]는 C[T’]의 하위 클래스이다	[-T]
무공변성(invariant)	C[T]와 C[T’]는 아무 관계가 없다	[T]
하위타입 관계가 실제 의미하는 것은 “어떤 타입 T에 대해 T’이 하위 타입이라면, T’으로 T를 대치할 수 있는가?” 하는 문제이다(역주: 이를 리스코프치환원칙(Liskov Substitution Principle)이라 한다. 이는 상위타입이 쓰이는 곳에는 언제나 하위 타입의 인스턴스를 넣어도 이상이 없이 동작해야 한다는 의미이다. 이를 위해 하위 타입은 상위 타입의 인터페이스(자바의 인터페이스가 아니라 외부와의 접속을 위해 노출시키는 인터페이스)를 모두 지원하고, 상위타입에서 가지고 있는 가정을 어겨서는 안된다).

```scala

scala> class Covariant[+A] defined class Covariant  scala> val cv: Covariant[AnyRef] = new Covariant[String] cv: Covariant[AnyRef] = Covariant@4035acf6  scala> val cv: Covariant[String] = new Covariant[AnyRef] <console>:6: error: type mismatch;  found   : Covariant[AnyRef]  required: Covariant[String]        val cv: Covariant[String] = new Covariant[AnyRef]                                    ^ 
scala> class Contravariant[-A] defined class Contravariant  scala> val cv: Contravariant[String] = new Contravariant[AnyRef] cv: Contravariant[AnyRef] = Contravariant@49fa7ba  scala> val fail: Contravariant[AnyRef] = new Contravariant[String] <console>:6: error: type mismatch;  found   : Contravariant[String]  required: Contravariant[AnyRef]        val fail: Contravariant[AnyRef] = new Contravariant[String]                                      ^ v
```

반공변성은 이상해 보일지 모른다. 언제 반공변성이 사용될까? 약간 놀랄지 모르겠다!

```scala
trait Function1 [-T1, +R] extends AnyRef 
```

이를 치환의 관점에서 본다면, 수긍이 된다. 우선 간단한 클래스 계층을 만들어 보자.

```scala
scala> class Animal { val sound = "rustle" } defined class Animal  scala> class Bird extends Animal { override val sound = "call" } defined class Bird  scala> class Chicken extends Bird { override val sound = "cluck" } defined class Chicken 
```

이제 Bird를 매개변수로 받는 함수를 만든다.

```scala
scala> val getTweet: (Bird => String) = // TODO 
```

표준 동물 라이브러리에도 이런 일을 하는 함수가 있기는 하다. 하지만, 그 함수는 Animal을 매개변수로 받는다. 보통은 “난 ___가 필요한데, ___의 하위 클래스밖에 없네”라는 상황이라도 문제가 되지 않는다. 하지만, 함수 매개변수는 반공변성이다. Bird를 인자로 받는 함수가 필요한 상황에서, Chicken을 받는 함수를 사용했다고 가정하자. 이 경우 인자로 Duck이 들어온다면 문제가 생길 것이다. 하지만, 거기에 Animal을 인자로 받는 함수를 사용한다면 아무 문제가 없을 것이다.
(역주: 함수 Bird=>String가 있다면 이 함수 내에서는 Bird의 인터페이스를 사용해 무언가를 수행하리라는 것을 예상할 수 있다. 이제 이런 함수를 받아 무언가 동작을 하는 f: (Bird=>String)=>String이라는 타입의 함수가 있고, f의 몸체에서는 Bird=>String함수에 Duck 타입의 값을 전달한다고 생각해 보자. 지금까지는 아무 문제가 없다.
이제 f에 Chicken=>String 타입의 함수 g를 넘긴다고 생각해 보면 문제가 왜 생기는지 알 수 있다. 이런 경우 g에 Chicken이 아닌 Duck이 전달될 것이다. 하지만, g 안에서는 닭만이 할 수 있는 짓(닭짓?)을 하게 되고, 돼지는 닭짓이 불가능하다. 따라서, 프로그램은 오류가 발행할 수 밖에 없다. 
반면, f에 Animal=>String 타입의 함수 h를 넘긴다면, h에 Duck이 전달될 것이다. 바람직하게도, h 안에서는 모든 동물이 할 수 있는 짓만을 하기 때문에, 오리도 아무 문제 없이 사용될 수 있다.)

```scala
scala> val getTweet: (Bird => String) = ((a: Animal) => a.sound ) getTweet: Bird => String = <function1> 
```

함수의 반환 값은 공변성이있다. Bird를 반환하는 함수가 필요한데, Chicken을 반환하는 함수가 있다면 아무런 문제가 없다.

```scala
scala> val hatch: (() => Bird) = (() => new Chicken ) hatch: () => Bird = <function0>
```

출처 : https://twitter.github.io/scala_school/ko/type-basics.html



++ 코세라 수업을 듣기시작한지 5주차에 접어들고있다

코세라 수업은 지금까지의 강의와는 다르게 반응형이라

강의 중간중간에 퀴즈들이 계속해서 나오고

매주 강의가 끝나면 과제가 나와서 소스를 완성해서 제출해야하는 압박이 있다



하지만 강의를 듣다보면 왜 그렇게 되는지? 1 + 1 방식으로 

기초개념들을 하나하나 설명해가면서 하다보니 많은 도움이 되는듯하다



숙제들을 보면 강의내용들을 종합해서 커다란 문제를

조각내어 하나씩 해결해나가는 재미도 느낄수있다



영어로 된 강의라 지레 겁먹을 수도 있지만 어려운 개념들을 원어 그대로 배우다보니 

좀더 쉽게 이해된다는 장점도 있다



Scala를 처음 공부하는 분들에게는 강추하는 강의라고 감히 말씀드린다

공부하며 참고를 한 사이트는 위의 자료의 출처인 스칼라 학교 이다



