---
title: 중재자 패턴 (Mediator Pattern)
tags: design_pattern
layout: post
description: 객체 간의 혼란스러운 의존 관계들을 줄일 수 있는 행동 디자인 패턴
---

# 중재자 패턴 (Mediator Pattern)

## 문제

프로필을 만들고 편집하는 대화 상자가 있다고 가정하자. 이 대화 상자는 텍스트필드와 여러 버튼 및 체크 박스같은 것들로 구성되어 있다. 일부 컴포넌트들의 경우 다른 컴포넌트들과 상호 작용할 수 있다. 예를 들어 어떤 버튼을 클릭하면 새로운 텍스트란이 나타난다던가 패스워드 일치 여부 등을 들 수 있다.

이러한 양식 코드들을 코드 내에서 직접 구현하려면 클래스들 앱의 다른 양식에서 재사용하기 어려워진다.

## 해결

서로 독립적으로 작동해야하는 컴포넌트 간의 모든 통신을 막고, 중재자를 통해서만 상호작용을 하는 것이다. 컴포넌트는 다른 컴포넌트를 의존하는 대신 중재자를 의존하게 된다.

![UI 요소들은 중재자를 통해 통신해야 합니다.](https://refactoring.guru/images/patterns/diagrams/mediator/solution1-ko.png)

## 구조

![중재자 디자인 패턴 구조](https://refactoring.guru/images/patterns/diagrams/mediator/structure.png)

- 컴포넌트들은 비지니스 로직을 포함한 다양한 클래스들이다. 각 컴포넌트에 중재자에 대한 참조가 있다.
- 중재자 인터페이스는 일반적으로 단일 알람 메서드만을 포함하는 컴포넌트들과의 통신 메소드를 선언한다
- 구상 중재자들은 다양한 컴포넌트 간의 관계를 캡슐화 한다
- 컴포넌트들은 다른 컴포넌트들을 인식하지 않아야 한다. 다른 컴포넌트에 중요한 일이 일어나면 컴포넌트는 이를 중재자에게만 알려야한다.

## 예시

```kotlin
interface Mediator {
  fun notify(val sender: Component, val event: String)
}
class AuthenticationDialog : Mediator {
  override fun notify(val sender: Component, val event: String) {
    if (sender == loginOrRegisterChkBx && event == "check") {
      if (loginOrRegisterChkBx.checked) {
        title = "login"
      } else {
        title = "register"
      }
    }
    
    if (sender == okBtn && event == "click") {
      if (loginOrRegister.checked) {
        ...
      }
    }
  }
}
open class Component(val dialot: Mediator) {
  fun click() {
    dialog.notify(this, "click")
  }
  fun keypress() {
    dialog.notify(this, "keypress")
  }
}
class Button : Component {
}
class TextBox: Component {
}
...
```



## 적용

- 중재자패턴은 일부 클래스들이 다른 클래스들과 단단하게 결합하여 변경하기 어려울 때 사용한다
- 이 패턴은 타 컴포넌트들에 너무 의존하기 때문에 다른 프로그램에서 컴포넌트를 재사용할 수 없는 경우 사용한다
- 중재자 패턴은 몇가지 기본 행동을 다양한 컨텍스트들에서 재사용하기 위해 수많은 컴포넌트 자식클래스들을 만들고 있는 경우에 사용한다

## 장단점

- 단일 책임 원칙
- 개방 폐쇄 원칙
- 컴포넌트간의 결합도를 줄인다
- 개별 컴포넌트들을 좀더 쉽게 재사용할 수 있다
- 중재자가 전지전능한 객체가 될 수 있다.