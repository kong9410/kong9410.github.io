---
title: Java Reflection
tags: java
layout: post
description: 자바의 Reflection은 JVM에서 실행되는 애플리케이션의 런타임 동작을 검사하거나 수정할 수 있는 기능이 필요한 프로그램에서 사용된다.
---

## Java Reflection이란?

Java Reflection은 자바에서 기본적으로 제공하는 API이다. 클래스, 인터페이스, 메소드들을 찾을 수 있다. 객체를 생성하거나 변수를 변경할 수 있고 메소드를 호출 할 수 있다. TC를 작성하기위해 private 변수를 변경할 때 리플렉션을 사용할 수 있다.

Reflection은 다음과 같은 정보를 가져올 수 있다.

- Class
- Constructor
- Method
- Field

## 어떻게 가능한가?

자바는 JVM이 실행되면 코드가 컴파일되어 바이트 코드로 변환되어 static 영역에 저장된다. Reflection API는 이 정보를 활용한다. 클래스 이름만 알고있다면 static 영역을 찾아서 정보를 가져올 수 있다.

### 단점

- 런타임에 동적으로 타입을 분석하고 정보를 가져오기 때문에 JVM을 최적화 할 수 없다.
- private 변수, 메소드에 접근하기 때문에 내부를 노출하면서 추상화가 깨진다.

## 언제 사용하는가?

애플리케이션 개발 보다는 프레임워크나 라이브러리에서 많이 사용된다. Spring Framework에서도 Spring Container의 BeanFactory가 Reflection API를 사용한다. Bean은 애플리케이션이 실행된 후 객체가 호출될 때 동적으로 객체의 인스턴스를 생성하는데 이때 Spring Container의 Bean Factory에서 리플렉션을 사용한다.

MyBatis나 JPA도 기본생성자가 필요로 하는 이유가 동적으로 객체 생성시 Reflection API를 사용하기 때문이다.



## 사용예시

### 테스트를 위한 객체

![User](https://user-images.githubusercontent.com/37204770/167883575-b05597c6-7a45-40a9-808c-32cdc52927ff.PNG)

### 생성자

![Constructor](https://user-images.githubusercontent.com/37204770/167883572-a487a270-589a-4f43-a5c0-30eea50d7c0d.PNG)

### 생성자를 이용한 인스턴스 생성

![Instance](https://user-images.githubusercontent.com/37204770/167883567-4dc9c9d9-3b95-4fb8-9ccd-2d0b68f345f6.PNG)

### 필드

![Field](https://user-images.githubusercontent.com/37204770/167883563-b5becd0e-7154-48fe-b0f5-8c9989397a6b.PNG)

### 필드 세팅

![FieldSet](https://user-images.githubusercontent.com/37204770/167883553-595b5621-36a5-4c4d-a730-11510886eeec.PNG)

### 메소드

![Method](https://user-images.githubusercontent.com/37204770/167883558-b3666e80-7f59-4f3c-85af-2cdedf3b2741.PNG)

### 메소드 실행

![InvokeMethod](https://user-images.githubusercontent.com/37204770/167883544-17209062-db85-4f81-81c7-b38642a2a31f.PNG)



## 참고

[Reflection API 간단히 알아보자. (techcourse.co.kr)](https://tecoble.techcourse.co.kr/post/2020-07-16-reflection-api/)

[Guide to Java Reflection | Baeldung](https://www.baeldung.com/java-reflection)

[Java Reflection 개념 및 사용법 (tistory.com)](https://gyrfalcon.tistory.com/entry/Java-Reflection)