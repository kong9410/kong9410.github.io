---
title: 템플릿 메서드 (Template Method)
tags: design_pattern
layout: post
description: 부모 클래스에서 알고리즘의 골격을 정의하지만 알고리즘 구조를 변경하지 않고 자식클래스들이 알고리즘의 특정 단계를 재정의 하는 행동 디자인패턴
---

# 템플릿 메서드 (Template Method)

어떤 파일을 읽어서 저장하는 로직이 있다고 가정한다. DOC파일일수도 있고 CSV파일일 수도있고 PDF파일일 수가 있다. 각 확장자별로 파일을 읽는 로직이 존재하고 저장하는 로직은 모두 동일하다. 이럴 경우 골격은 같기때문에 읽는 알고리즘만 변화하면 된다

## 해결책

알고리즘을 일련의 단계들로 나누고 이러한 단계들을 메서드로 분리한뒤 단일 템플릿 메서드 내부에서 일련의 호출들을 넣는다. 이런거는 abstract나 디폴트 구현을 가질 수 있다.

## 구조

![템플릿 메서드 디자인 패턴 구조](https://refactoring.guru/images/patterns/diagrams/template-method/structure.png)

- 추상클래스는 알고리즘의 단계들의 역할을 하는 메서드들을 선언한다.
- 구상클래스들은 모든 단계들을 오버라이드 할 수 있다

## 코드

```kotlin
abstract class GameAI {
  fun turn() {
    collectResources()
    buildStructures()
    buildUnits()
    attack()
  }
  
  fun collectResources() {
    TODO()
  }
  
  abstract fun buildStructures()
  abstract fun buildUnits()
  
  fun attack() {
    TODO()
  }
}

fun OrcAI : GameAI() {
  override fun buildStructures() {
    TODO()
  }
  
  override fun buildUnits() {
    TODO()
  }
}

fun MonstersAI : GameAI {
  override fun buildStructures() {
    // do nothing
  }
  
  override fun buildUnits() {
    // do nothing
  }
}
```

## 적용

- 클라이언트들이 알고리즘의 특정 단계를 확장할 수 있도록할때 적용한다
- 거의 같은 알고리즘을 포함하는 여러 클래스가 있는 경우에 사용

## 장단점

- 알고리즘의 특정부분만 오버라이드해 변경에 영향을 덜 받도록 한다
- 중복 코드를 부모 클래스로 가져올 수 있다
- 일부 클라이언트들은 알고리즘을 수정할 수 없을 수 있다
- 디폴트 구현을 억제하여 리스코프 치환 원칙을 위반할 수 있다
- 템플릿 메서드들은 단계들이 더 많을수록 유지가 더 어려운 경향이 있다

