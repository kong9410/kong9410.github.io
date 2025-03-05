---
title: 전략 패턴 (Strategy Pattern)
tags: design_pattern
layout: post
description: 알고리즘들의 패밀리를 정의하고, 각 패밀리를 별도의 클래스에 넣은 후 그들의 객체들을 상호교환할 수 있도록 하는 행동 디자인패턴
---

# 전략 패턴 (Strategy Pattern)

네비게이션 앱을 만든다 했을 때 목적지로 가는것을 차량경로로 만들 수도 있고 도보로 만들 수도 있고 자전거로 만들 수도 있다. 이러한 옵션은 알고리즘이 추가될때마다 메인 클래스의 크기가 두배로 늘어나고 유지보수하기가 어려워진다.

## 해결

전략 패턴은 특정 작업을 다양한 방식으로 수행하는 클래스를 선택한 후 모든 알고리즘을 전략이라는 별도의 클래스로 추출한다.

컨텍스트라는 클래스에는 전략 중 하나에 대한 참조를 저장하기 위한 필드가 있어야한다. 컨텍스트는 작업을 실행하는 대신 연결되어 있는 전략 객체에 위임한다.

컨텍스트는 작업에 적합한 알고리즘을 선택할 책임이 없고, 클라이언트가 원하는 전략을 컨텍스트에 전달한다. 이 컨텍스트는 인터페이스를 통해 모든 전략과 함께 작동하고 이 인터페이스는 선택된 전략 내의 알고리즘을 작동시킬 단일 메소드만 노출한다.

## 구조

![전략 디자인 패턴의 구조](https://refactoring.guru/images/patterns/diagrams/strategy/structure.png)

- 컨텍스트는 전략 중 하나에 대한 참조를 유지하고 전략 인터페이스를 통해서만 객체와 통신한다.
- 전략 인터페이스는 모든 구상 전략에 공통이고 컨텍스트가 전략을 실행하는 메소드를 선언한다.
- 구상 전략들은 컨텍스트가 사용하는 알고리즘의 다양한 변형들을 구현한다.
- 컨텍스트는 알고리즘을 실행해야할 때마다 연결된 전략 객체의 실행 메서드를 호출한다.
- 클라이언트는 특정 전략 객체를 만들어 컨텍스트에 전달한다.

## 코드

```kotlin
interface Strategy {
  fun execute(a: Int, b: Int)
}

class AddStrategy : Strategy {
  override fun execute(a: Int, b: Int) {
    return a + b
  }
}

class SubStrategy : Strategy {
  override fun execute(a: Int, b: Int) {
    return a - b
  }
}

class MultiplyStrategy : Strategy {
  override fun execute(a: Int, b: Int) {
    return a * b
  }
}

class Context(private var strategy: Strategy) {
  fun setStrategy(strategy: Strategy) {
    this.strategy = strategy
  }
  
  fun executeStrategy(a: Int, b: Int) {
    return strategy.execute(a, b)
  }
}

fun main() {
  val addStrategy = AddStrategy()
  val context = Context(addStrategy)
  
  val result1 = context.executeStrategy(1, 3)
  println(result1)
  
  context.setStrategy(SubStrategy())
  val result2 = context.executeStrategy(3, 1)
  println(result2)
  
  context.setStrategy(MultiplyStrategy())
  val result3 = context.executeStrategy(2, 2)
  println(result3)
}
```

## 적용

- 전략 패턴은 객체 내에서 한 알고리즘의 다양한 변형들을 사용하고 싶을 때
- 전략 패턴은 일부 행동을 실행하는 방식에서만 차이가 있는 유사한 클래스들이 많은 경우에 사용
- 전략 패턴을 사용하여 클래스의 비지니스 로직을 해당 로직의 컨텍스트에서 그리 중요하지 않을지도 몰는 알고리즘들의 구현 세부 사항들로부터 고립
- 이 패턴은 같은 알고리즘의 다른 변형들 사이를 전환하는 거대한 조건문이 클래스에 있는경우 사용

## 장단점

- 런타임에 내부 알고리즘을 교체
- 알고리즘 세부 구현 코드를 고립
- 상속을 합성으로 대체
- 개방폐쇄 원칙
- 알고리즘이 몇개 안된다면 지나치게 복잡할 수 있음
- 클라이언트들은 적절한 전략을 선택할 수 있도록 전략간의 차이점들을 알고있어야 한다

