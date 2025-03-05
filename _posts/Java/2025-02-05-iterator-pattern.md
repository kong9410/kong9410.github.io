---
title: 반복자 패턴 (Iterator Pattern)
tags: design_pattern
layout: post
description: 컬렉션의 요소들의 기본 표현을 노출하지 않고 하나씩 순회할 수 있도록 하는 행동 디자인 패턴
---

# 반복자 패턴 (Iterator Pattern)

## 문제

컬렉션이 어떻게 구성되어 있는지에 따라 컬렉션은 그 요소들에 접근할 수 있는 방법을 제공해야한다. 순회하는 방법이 있을 것이다. 어떤 경우엔 깊이우선탐색이 되야할 수도 있고 다른 경우엔 너비우선탐색이 되어야 할 수도 있다. 컬렉션에 이러한 순회 알고리즘을 추가할 수록 컬렉션의 주요 책임이 명확해지지 않게 될 수 있다.

## 해결

반복자라는 별도의 객체로 추출하는 방법이다. 반복자 객체는 알고리즘 자체를 구현하는 것 외에 모든 순회 세부 정보를 캡슐화하여, 여러 반복자들이 서로 독립적으로 동시에 같은 컬렉션을 순회할 수 있다.

일반적으로 반복자는 하나의 주 메소드만 있고, 아무것도 반환하지 않을 때까지 계속 실행되는 형태이다.

컬렉션을 순회하는 특별한 방법이 필요한다면 컬렉션의 변화 없이 새 반복자 클래스를 만들면 된다.

## 구조

![반복자 디자인 패턴 구조](https://refactoring.guru/images/patterns/diagrams/iterator/structure.png)

- 반복자 인터페이스는 컬렉션의 순회에 필요한 작업들을 선언한다.
- 구상 반복자들은 컬렉션 순회를 위한 특정 알고리즘을 구현한다.
- 컬렉션 인터페이스는 컬렉션과 호환되는 반복자들을 가져오기 위한 하나 이상의 메소드들을 선언한다.
- 구상 컬렉션은 클라이언트가 요청할 때마다 특정 구상 반복자 클래스의 인스턴스를 반환한다.
- 클라이언트는 반복자들과 컬렉션들의 인터페이스를 통해 작동한다. 클라이언트가 구상 클래스들에 결합하지 않으므로 클라이언트 코드로 다양한 컬렉션들과 반복자들을 사용할 수 있도록 한다.

## 예시

```kotlin
interface SocialNetwork {
  fun createFriendsIterator(val profileId: String): ProfileIterator
  fun createCoworkerIterator(val profileId: String): ProfileIterator
}

class Facebook : SocialNetwork {
  override fun createFridendsIterator(val profileId: String): ProfileIterator {
    return FacebookIterator(this, profileId, "friends")
  }
  override fun createCoworkerIterator(val profileId: String): ProfileIterator {
    return FacebookIterator(this, profileId, "coworkers")
  }
}

interface ProfileIterator {
  fun getNext(): Profile
  fun hasMore(): Boolean
}

class FacebookIterator(
  private val facebook: Facebook
  private val profileId: String
  private val type: String
) : ProfileIterator {
  private var currentPosition: Int? = null
  private var cache: List<Profile> = null
  
  private fun lazyInit() {
    if (cache == null) {
      cache = facebook.socialGraphRequest(profileId, type)
    }
  }
  
  override fun getNext(): Profile {
    if (hasMore()) {
      return cache[currentPosition++]
    }
  }
  override fun hasMore(): Boolean {
    return currentPosition < cache.length
  }
}

class SocialSpammer {
  fun send(val iterator: ProfileIterator, message: String) {
    while(iterator.hasMore()) {
      profile = iterator.getNext()
      println(profile.getEmail(), message)
    }
  }
}

fun main() {
  val network = Facebook()
  val spammer = SocialSpammer()
  
  val iterator = network.createFriendIterator(profile.getId())
  spammer.send(iterator, "the message")
}
```

## 적용

- 컬렉션이 내부에 복잡한 데이터 구조가 있지만 이 구조의 복잡성을 보안이나 편의성의 이유로 클라이언트에게 숨기고 싶을 때 사용
- 반복자 패턴을 사용해 앱에서 순회 코드의 중복을 줄인다
- 반복자 패턴은 코드가 다른 데이터 구조들을 순회할 수 있기를 원할 때 또는 이러한 구조들의 유형을 미리 알 수 없을 때 사용

## 장단점

- 단일 책임 원칙
- 개방폐쇄 원칙
- 같은 컬렉션을 병렬로 순회 가능
- 순회를 지연하고 필요할 때 계속할 수 있다
- 단순 컬렉션을 이용하는 경우 과할 수 있다
- 특수 컬렉션들의 요소들을 직접 탐색하는 것보다 덜 효율적일 수 있다

