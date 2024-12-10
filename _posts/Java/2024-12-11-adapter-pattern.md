---
title: 어댑터 패턴
tags: design_pattern
layout: post
description: 어댑터는 호환되지 않는 인터페이스를 가진 객체들이 협업할 수 있도록 하는 구조적 디자인 패턴이다.
---

# 어댑터 패턴

어댑터는 호환되지 않는 인터페이스를 가진 객체들이 협업할 수 있도록 하는 구조적 디자인 패턴이다.

## 문제

모니터링 앱을 만들고있고 데이터를 XML 형식으로 다운로드하여 사용자에게 차트와 다이어그램을 표현한다고 가정하자. 어느 시점에 타사의 스마트 분석 라이브러리를 통합하여 앱을 개선하기로 결정했다. 이 분석 라이브러리는 JSON 형식의 데이터로만 동작한다.

이 라이브러리를 xml과 작동하도록 변경할 수 있으나, 라이브러리에 의존하는 일부 코드가 손상될 수 있다. 또 처음부터 타사의 라이브러리 소스 코드에 접근하지 못할 수도 있다.

## 해결책

어댑터를 만들면 한 객체의 인터페이스를 다른 객체가 이해할 수 있도록 변환하는 특별한 객체이다.

어댑터는 데이터를 다양한 형식으로 변환할 수 있을 뿐만 아니라 다른 인터페이스를 가진 객체들이 협업하는데에도 도움을 줄 수 있다.

1. 어댑터는 기존에 있던 객체 중 하나와 호환되는 인터페이스를 받는다.
2. 이 인터페이스를 사용하면 기존 객체는 어댑터의 메서드들을 안전하게 호출할 수 있다.
3. 호출을 수신하면 어댑터는 이 요청을 두 번째 객체에 해당 객체가 예상하는 형식과 순서대로 전달한다.

## 구조

### 객체 어댑터

![어댑터 디자인 패턴의 구조 (객체 어댑터)](https://refactoring.guru/images/patterns/diagrams/adapter/structure-object-adapter.png)

- 클라이언트는 프로그램의 기존 비지니스 로직을 포함하는 클래스이다.
- 클라이언트 인터페이스는 다른 클래스들이 클라이언트 코드와 공동 작업할 수 있도록 따라야 하는 프로토콜을 뜻한다.
- 서비스는 일반적으로 타사또는 유용한 클래스를 말한다. 클라이언트는 서비스 클래스를 직접 사용할 수 없다. 왜냐면 서비스 클래스는 호환되지 않는 인터페이스를 가지고 있기 때문이다.
- 어댑터는 클라이언트와 서비스 양쪽에서 동작할 수 있는 클래스로, 서비스 객체를 래핑하는 동안 클라이언트 인터페이스를 구현한다. 어댑터는 인터페이스를 통해 클라이언트로부터 수신한 후 래핑된 서비스 객체가 이해할 수 있는 형식의 호출들로 변환한다.
- 클라이언트 코드는 클라이언트 인터페이스를 통해 어댑터와 작동하는 한 구상 어댑터 클래스와 결합하지 않는다. 기존 클라이언트 코드를 손상하지 않고 새로운 유형의 어댑터들을 프로그램에 도입할 수 있다.

### 클래스 어댑터

![어댑터 디자인 패턴 (클래스 어댑터)](https://refactoring.guru/images/patterns/diagrams/adapter/structure-class-adapter.png)

이는 c++과 같은 다중 상속을 지원하는 경우에만 사용할 수 있다.

- 클래스 어댑터는 객체를 래핑할 필요가 없다. 이유는 클라이언트와 서비스 양쪽에서 행동들을 상속받기 때문이다. 위 어댑터는 기존 클라이언트 클래스 대신 사용할 수있다.



## 예시

![img](https://velog.velcdn.com/images/gyomni/post/28d2d634-1eed-4ecc-9941-5b747dd56687/image.png)

User 클래스는 Fruit를 사용하고 Fruit는 인터페이스로 apple, banana등을 사용한다. mango의 경우는 외부 코드라 수정이 불가하다. 이럴때 MangoAdapter가 Fruit를 상속받으면서 Mango를 필드로 가지고있게 된다. 이렇게 되면 변경할 수 없는 Mango 클래스도 사용할 수 있게 된다. Fruit는 특정 서비스를 수행한다고 보면 된다.



```kotlin
fun main() {
    val apple = Apple(10)
    val banana = Banana(20)
    val mango = Mango(30)
    val mangoAdapter = MangoAdapter(mango)

    val list = listOf(apple, banana, mangoAdapter)
    list.forEach { println(it.price()) }
}

interface Fruit {
    fun price(): Int
}

class Apple(private val price: Int) : Fruit {
    override fun price(): Int {
        return price
    }
}

class Banana(private val price: Int) : Fruit {
    override fun price(): Int {
        return price
    }
}

// 인터페이스가 다른 클래스
class Mango(val price: Int)

// 어댑터로 래핑한다
class MangoAdapter(private val mango: Mango) : Fruit {
    override fun price(): Int {
        return mango.price
    }
}
```

## 장단점

- 단일책임원칙을 가진다
- 개방 폐쇄 원칙을 지킨다
- 다수의 새로운 인터페이스와 클래스들을 도입해 복잡성이 증가한다