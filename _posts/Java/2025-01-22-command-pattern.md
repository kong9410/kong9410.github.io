---
title: 커맨드 패턴(Command Pattern)
tags: design_pattern
layout: post
description: 요청에 대한 모든 정보가 포함된 독립실행형 객체로 변환하는 디자인 패턴
---

# 커맨드 패턴(Command Pattern)

![그래픽 사용자 인터페이스 객체들은 비즈니스 논리 객체들을 직접 액세스할 수 있습니다](https://refactoring.guru/images/patterns/diagrams/command/solution1-ko.png)

어떤 기능을 실행하기 위한 버튼이 있고 이 버튼이 각각 독립적인 비지니스 로직을 호출한다고 한다. 그렇다면 매번 이러한 비지니스로직이 포함된 버튼들을 무한정만들어야 할 수도 있다. 커맨드 패턴은 이러한 요청을 작동시키는 단일 메서드를 가진 별도의 커맨드 클래스로 추출할 수 있다.

커맨드 객체는 그래픽 사용자 인터페이스 객체들과 비지니스 논리 객체들 간의 링크 역할을 한다. 그래픽 사용자 인터페이스 객체는 어떤 비지니스 논리 객체가 요청을 받을지와 이 요청을 어떻게 처리할지 알 필요가 없다. 그래픽 사용자는 그저 커맨드를 작동할 뿐이다.

![커맨드를 통해 비즈니스 논리 레이어를 액세스합니다.](https://refactoring.guru/images/patterns/diagrams/command/solution2-ko.png)

다음 단계는 이 커맨드 인터페이스를 구현하는 일이다. 일반적으로 커맨드는 매개변수를 받지 않고 수행이된다. 버튼에서 커맨드를 미리 설정해두거나 자체적으로 커맨드를 가져올 수 있어야한다.

![그래픽 사용자 인터페이스 객체들은 작업을 커맨드들에 위임합니다](https://refactoring.guru/images/patterns/diagrams/command/solution3-ko.png)

Button에 필요한 커맨드를 필드로 가지고 있고 버튼을 클릭하면 해당 command가 동작하도록 하게 할 수 있다. 결론적으로 커맨드 패턴은 사용자 인터페이스와 비지니스 간의 결합도를 낮춰준다.

## 구조

![커맨드 디자인 패턴의 구조](https://refactoring.guru/images/patterns/diagrams/command/structure.png)

- 발송자(Invoker) 클래스는 요청을 시작하는 역할을 한다. 이 클래스에는 커맨드 객체에 대한 참조를 저장하기 위한 필드가 있어야한다.
- 커맨드 인터페이스는 커맨드를 실행하기 위한 단일 메서드만 선언한다.
- 구상 커맨드는 다양한 유형의 요청을 구현한다.
- 수신자 클래스에는 일부 비지니스 로직이 포함되어 있다. 대부분의 커맨드들은 요청이 수신자에게 전달되는 방법만있다
- 클라이언트는 구상 커맨드를 만들고 생성한다.

## 코드

```kotlin
interface Command {
  fun execute() 
}

class CopyCommand : Command {
  override fun execute() {}
}

class CutCommand : Command {
  override fun execute() {}
}

class PasteCommand : Command {
  override fun execute() {}
}

class CopyButton {
  val command: CopyCommand()
  
  fun copy() {
    command.execute()
  }
}

class CutButton {
  val command: CutCommand()
  
  fun cut() {
    command.execute()
  }
}

class PasteButton {
  val command: PasteCommand()
  
  fun paste() {
    command.execute()
  }
}

class CommandHistory {
  val history : List<Command>
  
  fun push(val command: Command)
  
  fun pop(): Command
}

fun main() {
  val commandHistory = CommandHistory()
  
  val copy = CopyButton()
  copy.copy()
  commandHistory.push(copy.command)
  
  val cut = CutButton()
  cut.cut()
  commandHistory.push(cut.command)
  
  val paste = PasteButton()
  paste.paste()
  commandHistory.push(paste.command)
  
  commandHistory.pop()
}
```

## 적용

- 객체를 매개변수화 하려는 경우 커맨드 패턴을 사용한다
- 커맨드 패턴은 작업들의 실행을 예약하거나 작업들을 대기열에 넣거나 작업들을 원격으로 실행하려는 경우 사용한다
- 커맨드 패턴은 되돌릴 수 있는 작업을 구현하려고 할 때 사용한다

## 장단점

- 단일 책임 원칙
- 개방 폐쇄 원칙
- 실행 취소/다시 실행을 구현
- 작업들의 지연된 실행을 구현가능
- 간단한 커맨드들의 집합을 복잡한 커맨드로 조합
- 발송자와 수신자사이에 새로운 레이어를 도입하기 때문에 코드가 복잡해질 수 있다.