---
title: 옵저버 패턴 (Observer Pattern)
tags: design_pattern
layout: post
description: 여러 객체에 관찰 중인 객체에 발생하는 모든 이벤트에 대하여 구독 메커니즘을 정의할 수 있도록 하는 행동 디자인 패턴이다.
---

# 옵저버 패턴 (Observer Pattern)

## 문제

신제품이 출시되었다는 소식을 고객에게 알려야할 때 관심없는 사용자에게 까지 알림을 가는 것은 과할 수 있다. 해당 카테고리에 관심있는 사람만 알림을 받게 해야한다.

## 해결

주기적으로 신제품을 출시하는 주체를 subject라고 한다. 그리고 이 신제품에 관한 소식을 구독할 수 있는 subscriber가 있다. subscriber는 언제든 subject를 구독하거나 구독취소할 수있다. 이제 중요한 이벤트가 발생할 때 마다 subscriber리스트를 참조하고 그들에게 특정 알림 메서드를 호출한다.

![알림 메서드들](https://refactoring.guru/images/patterns/diagrams/observer/solution2-ko.png?id=c91ea0905f2ea3ebdce959d975b19ab8)

## 구조

![옵서버 디자인 패턴 구조](https://refactoring.guru/images/patterns/diagrams/observer/structure.png?id=365b7e2b8fbecc8948f34b9f8f16f33c)

- publisher는 다른 객체들에게 이벤트를 발행한다.
- 새 이벤트가 발생하면 publisher는 subscriber 리스트를 보고 알람을 보낸다
- subscriber 인터페이스는 알림 인터페이스를 선언하며 대부분 단일 update 메서드로 구성된다
- 구상 subscriber는 publisher가 보낸 알림들ㅇ에 대한 응답으로 몇가지 작업을 수행한다
- 일반적으로 subscriber 업데이트를 올바르게 처리하기 위해 컨텍스트 정보가 어느정도 필요하다. 그러므로 context 데이터를 알림 메서드의 인수들로 전달한다
- 클라이언트는 publisher는 subscriber들은 별도로 생성한 후 subscriber들은 publisher 업데이트에 등록한다

## 예시

```kotlin
interface Listener {
  fun update(val filename: String)
}

class LoggingListener: Listener {
  override fun update(val filename: String) {
    println("logging ${filename}")
  }
}

class Publisher {
  private var listenerList: List<Listener> = arrayList()
  
  fun subscribe(listener: Listener) {
    listenerList.add(listener)
  }
  
  fun unscribe(listener: Listener) {
    listenerList.remove(listener)
  }
  
  fun notify(data: String) {
    listenerList.forEach {listener -> listener.update(data)}
  }
}

fun main() {
  val publisher = Publisher()
  
  val listener = LoggingListener()
  
  publisher.subscribe(listener)
  
  publisher.notify("Hello")
}
```

## 적용

- 한 객체의 상태가 변경되어 다른 객체들이 변경되어야할 때
- 이 패턴은 앱의 일부 객체들이 제한된 시간 동안 또는 특정 경우에만 다른객체들을 관찰해야 할 때 사용

## 장단점

- 개방폐쇄원칙
- 런타임에 객체 간의 관계들을 형성할 수 있다
- 구독자들은 무작위로 알림을 받는다

