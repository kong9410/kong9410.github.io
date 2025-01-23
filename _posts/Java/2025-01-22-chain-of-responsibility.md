---
title: 책임 연쇄 패턴 (Chain of Responsibility)
tags: design_pattern
layout: post
description: 핸들러의 체인을 따라 요청을 전달할 수 있게 해주는 행동 디자인 패턴이다.
---

# 책임 연쇄 패턴(Chain of Responsibility)

온라인 주문 시스템을 개발하고 있다고 가정한다. 인증된 사용자만 주문을 생성할 수 있고, 관리 권한 있는 사용자들에게는 모든 주문에 대해 전체 접근 권한을 부여하려고 한다.

이런 검사들은 차례대로 수행되어야 하고, 앱의 사용자들은 자격 증명이 포함된 요청을 받을 때마다 사용자 인증을 시도한다. 이러한 자격 증명은 인증에 실패하면 다른 검사들을 진행할 이유가 없다.

![새로운 검사를 추가할 때마다 코드가 더 커지고 더 복잡해지고 더 추악해졌습니다](https://refactoring.guru/images/patterns/diagrams/chain-of-responsibility/problem2-ko.png)

인증+권한부여+검증+캐싱... 등 새로운 기능이 추가할 때마다 더욱 크게 부풀어 오르고, 하나의 검사 코드를 바꾸면 다른 검사 코드가 영향을 받기도 했다. 더 심각한 문제는 시스템의 다른 컴포넌트들을 보호하기 위해 검사를 재사용하려고 할 때 해당 컴포넌트들에 일부 코드를 복제해야 했다.

## 해결

책임 연쇄 패턴은 특정행동들을 핸들러라는 독립 실행형 객체들로 변환한다. 이 앱의 경우 각 검사를 수행하는 단일 메서드가 있는 자체 클래스로 추출되어야 한다. 요청은 이제 데이터와 함께 이 메서드에 인수로 전달된다.

이 패턴은 핸들러들을 체인으로 연결하도록 한다. 각 핸들러에는 체인의 다음 핸들러에 대한 참조를 저장하기 위한 필드가 있다. 요청을 처리하는 것 외에도 핸들러들은 체인을 따라 더 멀리 전달하기도 하고, 모든 핸들러가 요청을 처리할 기회를 가질 때 까지 체인들 따라 이동한다.

![핸들러들이 하나씩 줄지어 체인을 형성합니다.](https://refactoring.guru/images/patterns/diagrams/chain-of-responsibility/solution1-ko.png)

이 방식에서는 핸들러가 요청을 받으면 핸들러는 요청을 처리할 수 있는지 판단한다. 처리가 가능한경우 핸들러는 이 요청을 더이상 전달하지 않는다. 요청을 처리하는 핸들러는 하나뿐이거나 아무 핸들러도 요청을 처리하지 않는다. 이 접근 방식은 그래픽 사용자 인터페이스 내에서 요소들의 스택에서 이벤트들을 처리할 때 일반적이다.

![체인은 객체 트리의 가지에서 형성될 수 있습니다](https://refactoring.guru/images/patterns/diagrams/chain-of-responsibility/solution2-ko.png)

사용자가 버튼을 클릭하면 그래픽 사용자 인터페이스 요소 체인을 통해 전파된다. 이 체인은 버튼으로 시작하여 해당 컨테이너들을 따라 이동후 메인 애플리케이션 창으로 끝난다.

## 구조

![책임 연쇄 디자인 패턴 구조](https://refactoring.guru/images/patterns/diagrams/chain-of-responsibility/structure.png)

- 핸들러는 모든 구상 핸들러에 공통적인 인터페이스를 선언한다.
- 베이스 핸들러는 선택적인 클래스고 여기에 모든 핸들러 클래스들에 공통적인 코드를 넣을 수 있다
- 구상 핸들러는 요청을 처리하기 위한 실제 코드가 포함되어 있다. 각 핸들러는 요청을 받으면 이 요청을 처리할지와 함께 체인을 따라 전달할지 결정해야 한다.
- 클라이언트는 앱의 논리에 따라 체인들을 한 번만 구성하거나 동적으로 구성할 수 있다. 요청은 체인의 모든 핸들러에 보내지게 될 수 있고 첫번째 핸들러가 아니어도 된다.

## 코드

```kotlin
interface ComponentWithContextualHelp {
  fun showHelp()
}

abstract class Component : ComponentWithContextualHelp {
  protected var container: Container? = null
  
  // default
  override fun showHelp() {
    
  }
}

abstract class Container : Component {
  protected val children: List<Component>
  
  fun add(child: Component) {
    children.add(child)
    child.container = this
  }
}

class Button : Component {
}

class Panel : Component {
  override fun showHelp() {
  }
}

class Dialog : Container {
  override fun showHelp() {
  }
}

fun main() {
  val dialog = Dialog()
  val panel = Panel()
  val cancel = Button()
  val okButton = Button()
  
  panel.add(ok)
  panel.add(cancel)
  dialog.panel(panel)
}
```

## 적용

- 프로그램이 다양한 방식으로 다양한 종류의 요청들을 처리할 것으로 예상되지만 정확한 요청 유형들과 순서들을 미리 알 수 없는 경우 사용한다.
- 이 패턴은 특정 순서로 여러 핸들러를 실행해야 할 때 사용한다.
- 책임 연쇄 패턴은 핸들러들의 집합과 순서가 런타임에 변경되어야 할 때 사용한다.

## 장단점

- 요청 처리 순서를 제어할 수 있다
- 단일 책임 원칙
- 개방 폐쇄 원칙
- 일부 요청들은 처리되지 않을 수 있다.

## 예시

스프링의 HttpMessageConverters