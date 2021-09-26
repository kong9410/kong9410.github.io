---
title: Object 클래스
tags: java
---

```java
class Universe {}
```

위 코드는 다음과 같다고 볼 수 있다.

```java
class Universe extends Object {}
```

이처럼 자바의 모든 클래스는 Object를 상속받아 사용하고 있다. 그러한 이유는 모든 클래스가 공통으로 포함하고 있어야 하는 기능을 제공하기 위해서다.

Object 클래스가 가지고 있는 메소드는 다음과 같다.

| 반환타입             | 메소드 명                                    |
| ---------------- | ---------------------------------------- |
| protected Object | clone()<br />오브젝트의 복사를 만든다               |
| boolean          | equals(Object obj)<br />이 오브젝트와 같은지 알려준다 |
| protected void   | finalize()<br />가비지 컬렉터가 참조를 하지 않는 오브젝트라고 판단했을 때 호출된다 |
| Class<?>         | getClass()<br />이 오브젝트의 런타임 클래스를 반환한다    |
| int              | hashCode()<br />오브젝트의 해쉬 코드 값을 반환한다      |
| void             | notify()<br />잠들어 있던 스레드 중 하나를 골라 깨운다    |
| void             | notifyAll()<br />잠들어 있던 스레드를 모두 깨운다      |
| String           | toString<br />오브젝트를 스트링으로 나타내는 값을 반환한다   |
| void             | wait()<br />갖고 있는 고유 락을 해제하고 스레드를 잠들게 한다 |
| void             | wait(long timeout)                       |
| void             | wait(long timeout, int nanos)            |

