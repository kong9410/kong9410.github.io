---
title: 메멘토 패턴(Memento Pattern)
tags: design_pattern
layout: post
description: 객체의 구현 세부 사항을 공개하지 않으면서 해당 객체의 이전 상태를 저장하고 복원할 수 있게 해주는 행동 디자인 패턴
---

# 메멘토 패턴(Memento Pattern)

## 문제

텍스트 에디터같은 것을 만들 때 작업을 하다가 이전 상태로 되돌려야 하는 경우가 있는데, 이럴경우 앱은 가장 최신 스냅샷을 가져와 모든 객체의 상태를 복원해야 한다. 스냅샷은 객체의 모든 필드를 보고 스토리지에 복사해야 할 것이다. 그러나 대부분의 객체들은 필드들이 private으로 되어 있을 것 이다. 이런 경우 복사를 할 수가 없다. 또한 스냅샷을 만드려면 값 수집 후 일종의 기록 컨테이너에 저장해야한다. 다른 객체들이 이 저장된 스냅샷을 읽고 불러올 수 있어야한 다면 스냅샷 필드는 public으로 되어야한다. 그러면 스냅샷에 발생하는 자그마한 변경에도 이를 참조하는 클래스들은 영향을 받을 것이다.

## 해결

이는 캡슐화의 실패로 인해 발생한다. 일부 객체들은 원래 해야 할 일보다 더 많은 일들을 수행하려 한다.

메멘토는 스냅샷들의 생성을 해당 상태의 실제 소유자인 originator 객체에 위임한다. 다른 객체들이 외부에서 상태를 복사하는 대신 자신의 상태에 대해 완전한 접근 권한을 갖는 클래스ㄹ가 자체적으로 스냅샷을 생성할 수 있다.

이 패턴은 메멘토라는 특수 객체에 객체 상태의 복사본을 저장한다. 메멘토의 내용에는 메멘토를 생성한 객체를 제외한 다른 어떤 객체도 접근할 수 없다. 다른 객체들은 메멘토들과 제한된 인터페이스를 사용해 상호작용해야 한다. 이러한 인터페이스는 스냅샷의 메타데이터에만 접근 가능하고, 스냅샷에 포함된 원래 객체의 상태는 가져오지 못한다.

이러한 제한 정책을 사용하면 일반적으로 케어테이커라고 하는 다른 객체들 안에 메멘토를 저장할 수 있다. 케어테이커는 제한된 인터페이스를 통해서만 메멘토와 작업하기 때문에 메멘토 내부에 저장된 상태를 변경할 수 없다. 동시에 originator는 메멘토 내부의 모든 필드에 접근할 수 있으므로 언제든지 자신의 이전 상태를 복원할 수 있다.

사용자가 실행 취소를 작동하면 기록 스택에서 가장 최근의 메멘토를 가져온 후 원래 객체를 메멘토에서 가져온 값들로 변경시킨다.

## 구조

### 구조1

![중첩된 클래스들에 기반한 메멘토](https://refactoring.guru/images/patterns/diagrams/memento/structure1.png)

- originator 클래스는 자신의 상태에 대한 스냅샷들을 생성할 수 있고, 필요시 스냅샷에서 자신의 상태를 복원할 수 있다.
- 메멘토는 originator의 상태의 스냅샷 역할을 하는 값 객체, 메멘토는 불변으로 만든 후 생성자를 통해서 한번만 전달한다.
- 케어테이커는 언제 그리고 왜 originator의 상태를 캡쳐해야 하는지알고 있다.
- 메멘토 클래스는 originator 내부에 중첩된다. originator가 메멘토의 필드들과 메서드들이 비공개로 되어있는 경우에도 접근할 수 있게 된다.

### 구조2

다른 클래스들이 오리지네이터의 상태를 메멘토를 통해 접근할 가능성을 완전히 제거하고자 할 때 사용한다

![엄격한 캡슐화를 사용한 메멘토](https://refactoring.guru/images/patterns/diagrams/memento/structure3.png)

- 여러유형의 오리지네이터와 메멘토를 보유할 수 있다. 각 오리지네이터는 그에 상응하는 메멘토 클래스와 함께 작동한다. 오리지네이터들과 메멘토들은 자신의 상태를 누구에게도 노출하지 않는다.
- 케어테이커들은 이제 메멘토들에 저장된 상태의 변경에 명시적인 제한을 받는다. 케어테이커 클래스는 복원 메서드가 이제 메멘토 클래스에 정의되어 있으므로 오리지네이터에게서 독립된다.
- 각 메멘토는 그것을 생성한 오리지네이터와 연결된다. 오리지네이터는 자신의 상태들을 메멘토의 생성자에 전달한다. 오리지네이터가 적절한 세터를 정의했을 때 자신의 오리지네이터의 상태를 복원할 수 있다.

## 예시

```kotlin
class Originator(var state: String) {
  fun setState(state: String) {
    this.state = state
  }
  
  fun toMoment(): Memento {
    return Memento(state)
  }
  
  fun setState(memento: Memento) {
    state = memento.getState()
  }
}

class Memento(var state: String) {
  fun getState(): String {
    this.state
  }
}

class CareTaker {
  var mementoList: List<Memento> = ArrayList<Memento>()
  
  fun add(memento: Memento) {
    mementoList.add(memento)
  }
  
  fun getMemento(index: Int): Memento {
    return mementoList[index]
  }
}

fun main() {
  val originator = Originator()
	originator("state1")

  val careTaker = CareTaker()
  careTaker.add(originator.toMoment())
  
  originator.setState("state2")
  careTaker.add(originator.toMoment())
  
  originator.setState("state3")
  println(originator.state)
  originator.setState(careTaker.getMemento(1))
  println(originator.state)
  originator.setState(careTaker.getMemento(0))
  println(originator.state)
}
```

## 적용

- 이전 상태를 복원할 수 있도록 객체의 상태의 스냅샷들을 생성하려는 경우에 사용
- 이 패턴은 객체의 필드들을 접근하는 것이 해당 객체의 캡슐화에 위배될 때 사용

## 장단점

- 캡슐화를 위반하지 않고 객체의 상태의 스냅샷을 생성할 수 있다.
- 케어테이커가 오리지네이터의 상태의 기록을 유지하도록 하여 오리지네이터의 코드를 단순화 할 수 있다
- 상태저장이 많을 수록 ram을 많이 소모할 수 있다
- 쓸모없는 메멘토를 파괴할 수 있도록 오리지네이터의 수명주기를 파악해야한다.
- 동적 프로그래밍 언어에서는 메멘토 내의 상태가 그대로 유지된다고 보장할 수 없다.

