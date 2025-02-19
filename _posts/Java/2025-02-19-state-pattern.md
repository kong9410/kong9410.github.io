---
title: 상태 패턴(State Pattern)
tags: design_pattern
layout: post
description: 객체의 내부 상태가 변경될 때 해당 객체가 행동을 변경할 수 있도록 하는 행동 디자인 패턴이다.
---

# 상태 패턴(State Pattern)

## 문제

오토마타와 유사하다

이 패턴의 주요 개념은 모든 순간에 프로그램이 속ㄷ할 수 있는 상태들의 수가 유한하다는 것이다. 어떤 상태에서든 프로그램은 다르게 행동하며, 한 상태에서 다른 상태로 즉시 전환될 수 있다. 현재의 상태에 따라 다른 상태로 전환되거나 전환 되지 않을 수 있다. 이러한 전환 규칙들을 천이(trasition)라고 한다. 이러한 규칙들은 유한하다

문서에 대해서 draft하고 moderation하고 publish하는 과정이 있다고 가정하자. 다음과 같은 코드가 나올 것이다.

```kotlin
class Document {
  fun publish() {
    if (state == "draft") {
      ...
    } else if (state == "moderation") {
      ...
    } else if (state == "published") {
      ...
    }
  }
}
```

조건문 기반의 상태머신의 단점은 상태가 추가된다면 조건문이 거대해진다. 이러한 코드는 유지관리하기가 어렵다.

## 해결

객체의 모든 가능한 상태들에 대해 새 클래스를 만들고 모든 상태별 행동들을 이러한 클래스들로 추출한다. context라는 객첸는 모든 행동을 자체적으로 구현하는 대신 현재 상태를 나타내는 상태 객체 중 하나에 대한 참조를 저장하고 모든 상태와 관련된 작업을 그 객체에 위임한다. context를 다른 상태로 전환하려면 활성 상태 객체를 다른 객체로 바꾸면 된다.

## 구조

![상태 디자인 패턴 구조](https://refactoring.guru/images/patterns/diagrams/state/structure-ko.png)

- context는 구상 상태중 하나에 대한 참조를 저장하고 모든 상태별 작업을 위임한다. context는 상태 인터페이스를 통해 상태 객체와 통신하고 새로운 상태 전달을 위해 setter를 노출한다.
- 상태 인터페이스는 상태별 메서드를 선언한다.
- 구상상태들은 상태별 메서드들에 대한 자체적인 구현을 한다.
- context와 구상 상태 모두 context의 다음 상태를 설정할 수 있고, context에 연결된 상태 객체들을 교체하여 실제 상태를 변경할 수 있다.

## 예시

```kotlin
class Player(var state: State) {
  fun changeState(state: State) {
    this.state = state
  }
}

abstract class State(var player: Player) {
  abstract fun onLock(): String
  abstract fun onPlay(): String
}

class LockedState : State {
  constructor(player: Player): this(player)
  
  override fun onLock(): String {
    if (player.isPlaying()) {
      player.changeState(ReadyState(player))
      return "stop playing"
    } else {
      return "locked"
    }
  }
  override fun onPlay(): String {
    player.changeState(ReadyState(player))
    return "Ready"
  }
}

class ReadyState: State {
  override fun onLock(): String {
    player.changeState(LockedState(player))
    return "Locked"
  }
  
  override fun onPlay(): String {
    val action = player.startPlayback()
    player.changeState(PlayingState(player))
    return action
  }
}

class PlayingState: State {
  ...
}

fun main() {
  val player = Player(ReadyState())
  player.onLock() // Locked
  player.onPlay() // Ready
  player.onPlay() // "some action"
  player.onLock() // Locked
}
```

## 적용

- 현재 상태에 따라 다르게 행동하는 객체가 있을 때 상태들의 수가 많을 때 상태별 코드가 자주 변경될 때 사용
- 클래스 필드들의 현재 값들에 따라 클래스가 행동하는 방식을 변경하는 거대한 조건문들로 오염된 클래스가 있을 경우에 사용
- 상태 패턴은 유사한 상태들에 중복 코드와 조건문-기반 상태 머신의 천이가 많을 때 사용

## 장단점

- 단일책임원칙
- 개방/폐쇄원칙
- 거대한 상태 머신 조건문을 제거
- 상태 머신에 몇가지 상태만 있거나 머신이 거의 변경되지 않을 때 사용하는 것은 과할 수 있다

