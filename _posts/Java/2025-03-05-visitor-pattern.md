---
title: 비지터 패턴(Visitor Pattern)
tags: design_pattern
layout: post
description: 알고리즘을 작동하는 객체들로부터 분리할 수 있도록하는 행동 디자인 패턴
---

# 비지터 패턴(Visitor Pattern)

## 구조

![비지터 디자인 패턴의 구조](https://refactoring.guru/images/patterns/diagrams/visitor/structure-ko.png)

- 비지터 인터페이스는 객체 구조의 구상요소들을 인수들로 사용할 수 있는 비지터 메서드들의 집합을 선언한다. 이러한 메서드들은 같은 이름을 가질 수 있지만 그들의 매개변수들의 유형은 달라야 한다.
- 구상 비지터는 다양한 구상 요소 클래스들에 맞춤으로 작성된 같은 행동들의 여러 버전을 구현한다
- 요소 인터페이스는 비지터를 수락하는 메서드를 선언한다
- 구상 요소는 수락 메서드를 구현한다. 이 메서드의 목적은 호출을 현재 요소 클래스에 해당하는 적절한 비지터 메서드로 리다이렉트 하는 것이다.
- 클라이언트는 컬렉션 또는 복잡한 객체를 나타낸다.

## 코드

```kotlin
interface Shape {
  fun move(x: Int, y: Int)
  fun accept(v: Visitor)
}

class Dot : Shape {
  override fun accept(v: Visitor) {
    v.visitDot(this)
  }
}

class Circle : Shape {
  override fun accept(v: Visitor) {
    v.visitCircle(this)
  }
}

interface Visitor {
  fun visitDot(d: Dot)
  fun visitCircle(c: Circle)
}

class XMLExportVisitor : Visitor {
  override fun visitDot(d: Dot) {
    TODO()
  }
  
  override fun visitCircle(c: Circle) {
    TODO()
  }
}

fun main() {
  val exportVisitor = XMLExportVisitor()
  val circle = Circle()
  val dot = Dot()
  val list = listOf(circle, dot)
  
  list.forEach {shape -> shape.accept(exportVisitor)}
}
```

## 적용

- 비지터 객체는 복잡한객체 구조의 모든 요소에 대해 작업을 수행하야 할때 사용한다
- 비지터 패턴을 사용하여 보조 행동들의 비지니스 로직을 정리한다
- 이 패턴은 행동이 클래스 계층구조의 일부 클래스들에서만 의미가 있고 다른 클래스들에서는 의미가 없을 때 사용

## 장단점

- 개방폐쇄원칙
- 단일책임원칙
- 다양한 객체와 작업하면서 유용한 정보를 축적할 수 있다
- 클래스가 요소 계층구조에 추가되거나 제거될때 비지터를 업데이트해야한다
- 비지터는 함께 작업해야하는 요소들의 비공개 필드들 및 메서드들에 접근하기 위해 필요한 권한이 부족할 수 있다