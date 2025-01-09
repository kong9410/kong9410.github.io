---
title: 퍼사드 패턴(Facade Pattern)
tags: design_pattern
layout: post
description: 라이브러리에 대한 프레임워크에 대한 또는 다른 클래스들의 복잡한 집합에 대한 단순화된 인터페이스를 제공하는 구조적 디자인 패턴이다.
---

# 퍼사드 패턴(Facade Pattern)

라이브러리나 프레임워크에 속하는 객체들을 나의 코드에 작동하게 만들어야한다고 생각해보자. 일반적으로 이러한 객체들을 모두 초기화하고 올바른 순서로 실행하는 등의 작업을 수행해야한다.

결과적으로 비지니스 로직이 타 클래스들의 구현 세부 사항들과 밀접하게 결합하여 코드를 이해하고 유지 관리하기가 어려워진다.

## 해결책

퍼사드는 복잡한 하위 시스템에 대한 간단한 인터페이스를 제공하는 클래스다. 퍼사드는 제한된 기능을 제공한다. 그러나 퍼사드는 클라이언트들이 정말로 중요하게 생각하는 기능들만 포함된다.

## 구조

![퍼사드 디자인 패턴 구조](https://refactoring.guru/images/patterns/diagrams/facade/structure.png)

- 퍼사드 패턴을 사용하면 하위 시스템 기능들의 특정 부분에 편리하게 접근할 수 있다.
- 추가적인 퍼사드 클래스를 생성하여 하나의 퍼사드를 관련 없는 기능들로 오염시켜 복잡한 구조를 만드는 것을 방지할 수 있다.
- 복잡한 하위 시스템은 수십개의 다양한 객체들로 구성된다.
- 클라이언트는 하위 시스템 객체들을 직접 호출하는 대신 퍼사드를 사용한다.

## 예시

```kotlin
class VideoFile

class OggCompressionCodec

class MPEG4CompressionCodec

class CodecFactory

class BitrateReader

class AudioMixer

class VideoConverter {
  fun convert(filename: String, format: String): File {
    val file = VideoFile(filename)
    val sourceCodec = CodecFactory().extract(file)
    
    var destinationCodec: CompressionCodec? = null
    if (format == "mp4") {
      destinationCodec = new MPEG4CompressionCodec()
    } else {
      destinationCodec = new OggCompressionCodec()
    }
    
    val buffer = BitrateReader.read(filename, sourceCodec)
    var result = BitrateReader.convert(buffer, destinationCodec)
    result = AudioMixer().fix(result)
    return File(result)
  }
}

class Application {
  fun main() {
    val convertor = VideoConverter()
    val mp4 = convertor.convert("dog.ogg", "mp4")
    mp4.save()
  }
}
```

## 적용

퍼사드 패턴은 복잡한 하위 시스템에 대한 제한적이지만 간단한 인터페이스가 필요할 때 사용한다

하위 시스템을 계층들로 구성하려는 경우 사용한다

## 장단점

복잡한 하위 시스템에서 코드를 별도로 분리할 수 있다

퍼사드는 앱의 모든 클래스에 결합된 전지전능한 객체가 될 수 있다.
