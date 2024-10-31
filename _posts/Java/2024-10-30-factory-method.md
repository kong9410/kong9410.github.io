---
title: 팩토리 메소드 패턴
tags: design_pattern
layout: post
---

# 팩토리 메소드 패턴(Factory Method)

물류 관리 앱을 만들다고 가정했을 때, 물류를 전달하는 트럭(Truck) 클래스가 존재하고, 이 트럭은 배송(deliver) 역할을 한다. 모든 물류가 트럭으로 관리된다고 하면은 문제가 없지만 선박, 비행 등의 종류가 추가된다면 코드가 전체적으로 변화되어야 할 것이다. 많은 조건문이 존재하고 클래스에 따라 앱의 로직이 바뀌는 매우 길고 복잡한 로직이 작성될 것이다.

## 해결책

팩토리 메소드로 해결하는 간단한 방법은 운송수단(Transport)를 생성하는 추상화된 공통의 클래스를 상속하는 클래스를 만드는 것이다. 단순히 생성자가 팩토리 메소드 안으로 들어간 것 뿐이라고 볼 수 있는데, 팩토리 메소드를 오바리이딩하고 그 메소드에 의해 생성되는 제품들의 클래스를 변경할 수 있게 되었다.

다만 인터페이스 상속클래스의 경우 하나의 공통된 반환값이 존재해야하기 때문에 운송수단 클래스도 하나의 추상화된 클래스에 상속받아 만들어져야 한다. 또한 공통된 기능을 사용해야한다. 예를들어 `Truck`, `Shipment`, `Plane` 클래스가 생성되고 상속받은 `deliver` 메소드는 각각 다르게 오버라이딩 하는것이다.

팩토리 메소드를 사용하는 코드를 *클라이언트 코드*라고 부르고 클라이언트 코드는 자식 클래스들에서 실제로 반환되는 여러 제품 간의 차이에 대해 알지 못한다. 클라이언트 코드는 모든 제품을 추상 Transport(운송체계)로 간주한다.

## 구성

![image](https://private-user-images.githubusercontent.com/37204770/381597809-c760a3df-9693-475a-bf1f-64d9d82aefcb.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MzAyOTk0MjksIm5iZiI6MTczMDI5OTEyOSwicGF0aCI6Ii8zNzIwNDc3MC8zODE1OTc4MDktYzc2MGEzZGYtOTY5My00NzVhLWJmMWYtNjRkOWQ4MmFlZmNiLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDEwMzAlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQxMDMwVDE0Mzg0OVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTk2NTBlMGU1ODQ0NDU4M2MzZTZlZDQ2YjllOTNlM2I2MjI4YzNhZWEyOTdmOTdiOWU3MjU2YmZmN2VhZmIyMGEmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.IuYXRNZh808WoNWRvhMVLV81SF6GsMphtGnFIEBukiY)

1. 제품 인터페이스 선언
2. 제품 인터페이스의 구현체
3. 크리에이터는 새로운 제품 객체를 반환하는 팩토리 메소드를 선언한다. 메소드의 반환 유형은 제품 인터페이스다. 중요한건 **크리에이터의 주 책임은 제품을 생성하는 것이 아니다.** 크리에이터 클래스에는 제품과 관련된 비지니스 로직이 있고, 팩토리 메소드는 이 로직을 제품 클래스들로부터 디커플링(분리)하는데 도움만 준다.
4. 구상 크리에이터는 기초 팩토리 메소드를 오버라이드하여 다른 유형의 제품을 반환하게 한다.

## 예시코드

```kotlin
// Creator
interface TransportCreator {
  fun create(): Transport
}

// Concrete Creator
class TruckCreator : TransportCreator {
  override fun create(): Transport {
    return Truck()
  }
}

class ShipmentCreator : TransportCreator {
  override fun create(): Transport {
    return Shipment()
  }
}

class AirplaneCreator : TransportCreator {
  override fun create(): Transport {
    return Airplane()
  }
}

interface Transport {
  fun deliver()
}

class Truck : Transport {
  override fun deliver() {
    // 트럭 운송 로직
  }
}

class Shipment : Transport {
  override fun deliver() {
    // 배 운송 로직
  }
}

class Airplane : Transport {
  override fun deliver() {
    // 비행기 운송 로직
  }
}

fun main(args: Array) {
  val type = args[0]
  
  val transport = makeTransport(type)
  transport.deliver()
}

fun makeTransport(type: String): Transport {
  var transportCreator
  if (type == "Road") {
    transportCreator = TruckCreator()
  } else if (type == "Seaway") {
    transportCreator = ShipmentCreator()
  } else if (type == "Air Road") {
    transportCreator = AirplaneCreator()
  } else {
    throw IllegalArgumentException("illegal type")
  }
  
  return transportCreator.create()
}
```

## 언제 사용하는가

- 팩토리 메소드는 제품 생성 코드를 제품 사용 코드와 분리한다. 그러면 제품 생성코드는 확장하기 쉬워진다.
- 라이브러리 사용자들에게 내부 컴포넌트를 확장하는 방법을 제공하고 싶을 때 사용한다.
- 팩토리 메소드는 기존 객체들을 재구축하는 대신 이들을 재사용하여 시스템 리소스를 절약하고 싶을 때 사용한다. 이러한 것들은 DB연결, 파일입출력 등을 처리할 때 자주 발생한다.



## 장단점

- 단일 책임 원칙에 따라 제품 생성 코드를 한 위치로 옮겨 더 쉽게 관리할 수 있다
- 개방 폐쇄 원칙에 따라 클라이언트 코드를 훼손하지 않고 새로운 유형의 생성 코드를 도입할 수 있다
- 자식 클래스가 많아져 복잡해 질 수 있다
