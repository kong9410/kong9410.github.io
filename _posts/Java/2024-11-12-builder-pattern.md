---
title: 빌더 패턴
tags: design_pattern
layout: post
description: 디자인 패턴의 생성 패턴 중 빌더 패턴에 대해 설명
---

## 빌더 패턴

빌더는 복잡한 객체들은 단계별로 생성할 수 있도록 하는 디자인 패턴이다.

## 문제

많은 필드와 중첩된 객체들을 힘들게 단계별로 초기화해야하는 복잡한 객체를 생성해야 한다면, 일반적으로 많은 매개변수가 있는 커다란 생성자를 사용해야 할 것이다. 여기서 `setter`는 가변 객체이기때문에 생각하지 않는다.집을 만들어야 한다고 가정할때 창문, 문, 방, 마당 등의 정보들이 필요할 것이다. 이러한 집의 필드들은 거대한 생성자를 만들게 된다.

![점층적 생성자](https://refactoring.guru/images/patterns/diagrams/builder/problem2.png)

## 해결책

빌더 패턴은 자신의 클래스에서 객체 생성 코드를 추출하여 builders라는 별도의 객체들로 만들도록 한다. 이 패턴은 객체 생성을 일련의 단계들로 정리하며, 객체 생성하고 싶으면 위 단계를 builder 객체에 실행하면 된다. 중요한 점은 모든 단계를 호출할 필요 없다는 것으로, 객체의 특정 설정을 제작하는데 필요한 단계들만 호출하면 된다.

### 디렉터 (관리자)

더 나아가 제품을 생성하는 데 사용하는 빌더 단계들에 대한 일련의 호출을 디렉터라는 별도의 클래스로 추출할 수 있다. 디렉터 클래스는 제작 단계들을 실행하는 순서를 정의하는 반면 빌더는 이러한 단계들에 대한 구현을 제공한다.

디렉터 클래스는 필수사항은 아니다. 디렉터 클래스는 다양한 생성 루틴들을 배치하여 프로그램 전체에서 재사용할 수 있는 좋은 클래스가 될 수 있다.

## 구조

![빌더 디자인 패턴 구조](https://refactoring.guru/images/patterns/diagrams/builder/structure.png)

1. 빌더 인터페이스는 모든 유형의 빌더들에 공통적인 제품 생성 단계들을 선언
2. 구상 빌더들은 생성 단계들의 다양한 구현을 제공
3. 제품들은 그 결과로 나온 객체들이다.
4. 디렉터 클래스는 생성단계들을 호출하는 순서를 정의한다
5. 클라이언트는 빌더 객체들 중 하나를 디렉터와 연결해야한다.

## 코드

## 장단점

- 객체들을 단계별로 생성하거나 생성 단계들을 연기하거나 재귀적으로 단계를 실행할 수 있다
- 단일 책임 원칙. 제품의 비지니스 로직에서 복잡한 생성 코드를 고립시킬 수 있다
- 패턴이 여러개의 새 클래스들을 생성해야한다.