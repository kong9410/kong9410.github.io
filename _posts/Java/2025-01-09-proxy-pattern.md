---
title: 프록시 패턴(Proxy Pattern)
tags: design_pattern
layout: post
description: 다른 객체에 대한 대체 또는 자리표시자를 제공할 수 있는 구조 디자인 패턴이다.
---

# 프록시 패턴(Proxy Pattern)

필요할 때만 객체를 만들어서 지연된 초기화를 구현할 수 있다. 그러면 객체의 모든 클라이언트들은 어떤 지연된 초기화 코드를 실행해야 하는데, 이것은 아마도 많은 중복 코드를 초래할 것이다.

## 해결책

프록시 패턴은 원래 서비스 객체와 같은 인터페이스로 새 프록시 클래스를 생성하라고 제안한다. 그러면 프록시 객체를 원래 객체의 모든 클라이언트들에 전달하도록 앱을 전달할 수 있다. 클라이언트로부터 이 요청을 받으면 프록시는 실제 서비스 객체를 생성하고 모든 작업을 이 객체에 위임한다.

클래스의 메인 로직 이전이나 이후에 무언가를 실행해야 하는 경우 프록시는 해당 클래스를 변경하지 않고도 이 무언가를 수행할 수 있도록 한다. 프록시는 원래 클래스와 같은 인터페이스를 구현하므로 실제 서비스 객체를 기대하는 모든 클라이언트에 전달될 수 있다.

## 구조

![프록시 디자인 패턴의 구조](https://refactoring.guru/images/patterns/diagrams/proxy/structure.png)

- 서비스 인터페이스는 서비스의 인터페이스를 선언한다. 프록시가 서비스 객체로 위장할 수 있으려면 이 인터페이스를 따라야한다
- 서비스는 어떤 유용한 비지니스 로직을 제공한다
- 프록시 클래스에는 서비스 객체를 가리키는 참조 필드가 있다. 요청의 처리를 완료하면 그 후 처리된 요청을 서비스 객체에 전달한다.
- 클라이언트는 같은 인터페이스를 통해 서비스들 및 프로시들과 함께 작동해야한다.

## 예시

```kotlin
interface ThirdPartyYoutubeLib {
  fun listVideos()
  fun getVideoInfo(id: String)
  fun downloadVideo(id: String)
}

class ThirdPartyYoutube : ThirdPartyYoutube {
  override fun listVideos()
  override fun getVideoInfo(id: String)
  override fun downloadVideo(id: String)
}

class CachedYoutube : ThirdPartyYoutube {
  private var service: ThirdPartyYoutubeLib
  private var listCache
  private var videoCache
  val needReset
  
  override fun listVideo() {
    if (listCache == null || needReset) {
      listCache = service.listVideos()
    }
    return listCache
  }
  
  override fun getVideoInfo(id: String) {
    if (videoCache == null || needReset) {
      videoCache = service.getVideoInfo(id)
    }
    return videoCache
  }
  
  override fun downloadVideo(id: String) {
    if(!downloadExists(id) || needReset) {
      service.downloadVideo(id)
    }
  }
}

class YoutubeManager(val service: ThirdPartyYoutubeLib) {
  fun renderVideoPage(id: String) {
    service.getVideoInfo(id)
  }
  
  fun renderListPanel() {
    service.listVideos()
  }
  
  fun reactOnUserInput() {
    renderVideoPage()
    renderListPanel()
  }
}

class Application {
  fun main() {
    val youtubeService = ThirdPartyYoutube()
    val youtubeProxy = CachedYoutubeClass(youtubeService)
    val manager = YoutubeManager(youtubeProxy)
    manager.reactOnUserInput()
  }
}
```

## 적용

지연된 초기화, 어쩌다 필요한 무거운 객체가 항상 가동되어있어 시스템 자원들을 낭비하는 경우에 사용할 수 있다

특정 클라이언트들만 서비스 객체를 사용할 수 있도록 하려는 경우에 사용할 수 있다.

원격 서비스의 로컬 실행. 서비스 객체가 원격 서버에 있는 경우

요청들의 로깅. 서비스 객체에 대한 요청들의 기록을 유지하려는 경우

요청 결과들의 캐싱. 이것은 클라이언트 요청들의 결과들을 캐시하고 이 캐시들의 수명 주기를 관리해야할 때, 특히 결과들이 상당히 큰 경우에 사용된다.

## 장단점

클라이언트들이 알지 못하는 상태에서 서비스 객체를 제어할 수 있다

클라이언트들이 신경 쓰지 않을 때 서비스 객체의 수명 주기를 관리할 수 있다

프록시는 서비스 객체가 준비되지 않았거나 사용할 수 없는 경우에도 작동한다

개방 폐쇄 원칙

새로운 클래스를 도입해야 해서 코드가 복잡해질 수 있다

응답이 늦어질수있다