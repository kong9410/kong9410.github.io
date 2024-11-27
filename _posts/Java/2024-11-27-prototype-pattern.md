---
title: 프로토타입 패턴
tags: design_pattern
layout: post
description: 객체를 복사하는 방법에 대해 알아보자
---

객체의 정확한 복사본을 만들고 싶다면 새 객체를 만들고 필드를 복사하면 된다고 생각하지만 일부 객체는 private 일 수도 있다. 또한 객체의 복제본을 생성하려면 객체의 클래스를 알아야하므로 해당 클래스가 복사하는 클래스에 의존하게 된다.

## 해결책

프로토타입의 패턴은 실제로 복제되는 객체들에 복제 프로세스를 위임한다. 패턴은 복제를 지원하는 모든 객체에 대한 공통 인터페이스를 선언한다. 일반적으로 이러한 인터페이스에는 단일 clone 메서드에만 포함된다.

객체들이 같은 클래스에 속한 다른 객체의 비공개 필드들에 접근할 수 있도록 하므로 비공개 필드들을 복사하는것도 가능하다.

복제를 지원하는 객체를 프로토타입이라고 한다. 객체들에 수십 개의 필드와 수백개의 가능한 설정들이 있는 경우 서블클래싱의 대안이 될 수 있다.

## 구조

![프로토타입 디자인 패턴의 구조](https://refactoring.guru/images/patterns/diagrams/prototype/structure.png)

## 코드

```kotlin
abstract class Shape {
    val x: Int
    val y: Int
    val color: String

    constructor(shape: Shape) {
        x = shape.x
        y = shape.y
        color = shape.color
    }

    abstract fun clone(): Shape
}

class Rectangle : Shape {
    val width: Int
    val height: Int

    constructor(source: Rectangle) : this(source) {
        this.width = source.width
        this.height = source.height
    }

    fun clone(): Shape {
        return Rectangle(this)
    }
}

class Circle : Shape {
    val radius: Int

    constructor Circle(source: Circle) : this(source) {
        this.radius = source.radius
    }

    fun clone(): Shape {
        return Shape(this)
    }
}

```

## 언제사용하는가

- 복사해야하는 구상 클래스들에 코드가 의존하면 안될 때 사용한다.

- 프로토타입 패턴은 복제를 지원하는 모든 객체와 작업할 수 있도록 인터페이스를 제공해야한다.

- 각자의 객체를 초기화하는 방식만 다른 자식 클래스들의 수를 줄이고 싶을 때 사용한다.

  ​

## 장단점

- 객체들을 그 구상 클래스들에 결합하지 않고 복제할 수 있다
- 반복되는 초기화 코드를 제거하고 그 대신 미리 만들어진 프로토타입을 복제하는 방법을 사용할 수 있다
- 복잡한 객체들을 더 쉽게 생성할 수 있다
- 복잡한 객체들에 대한 사전 설정들을 처리할 때 상속 대신 사용할 수 있다
- 순환 참조가 있는 복잡한 객체들을 복제하는 것은 매우 까다로울 수 있다.