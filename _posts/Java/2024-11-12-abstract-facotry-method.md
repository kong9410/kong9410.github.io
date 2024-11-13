---
title: 추상 팩토리 패턴
tag: design_pattern
layout: post
description: 디자인 패턴의 생성 패턴 방식 중 하나인 추상 팩토리 패턴에 대해 설명
---

## 추상 팩토리 패턴

추상 팩토리는 관련 객체들의 구상 클래스들을 지정하지 않고도 관련 객체들의 모음을 생성할 수 있도록 하는 생성패턴이다.

## 문제

Chair + Sofa + CoffeeTable로 형성된 패밀리 제품군이 있다고 가정한다.

각 가구들은 Modern, Victorian, ArtDeco와 같은 양식으로 제공된다.

![제품 패밀리들과 그들의 변형들](https://refactoring.guru/images/patterns/diagrams/abstract-factory/problem-ko.png)

가구별로 객체를 생성했을때 다른 가구 객체들과 양식이 일치하지 않도록 할 방법이 필요하다. 가구 공급업체들은 카탈로그를 자주 변경하기 때문에 제품군을 추가할 때마다 기존 코드를 변경해야 하는 번거로움을 피하고 싶을 것 이다.

## 해결

첫번째 방안은 각 제품군에 해당하는 개별적인 인터페이스를 명시하는 것이다. 예를들어 VictorianChair가 있고 ModernChair가 있으면 이 둘을 추상화한 인터페이스 Chair가 있는 것이다. 마찬 가지로 다른 제품군에도 이와 같은 방식을 적용시킨다.

![Chairs 클래스 계층](https://refactoring.guru/images/patterns/diagrams/abstract-factory/solution1.png)

다음 방법은 **추상 팩토리 패턴**을 선언하는 것이다. 추상 팩토리 패턴은 개별 제품들의 생성 메소드들이 목록화되어 있는 인터페이스다. 예를들어 VictorianFurnitureFactory 클래스에는 각각 createChair, createCoffeeTable, createSofa 메소드가 있으며 이것의 반환값은 각각 VictorianChair, VictorianCoffeeTable, VictorianSofa가 되는것이다.

## 구조

![추상 팩토리 디자인 패턴](https://refactoring.guru/images/patterns/diagrams/abstract-factory/structure.png)

1. 추상 제품들은 개별 제품들 집합에 대한 인터페이스를 선언한다
2. 구성 제품들은 그룹화된 추상 제품들의 다양한 구현체이다.
3. 추상 패곹리 인터페이스는 각각의 추상 제품들을 생성하기 위해 여러 메서드 집합을 선언
4. 구상 팩토리는 추상 팩토리를 구현한다. 각각 자신의 제품군에 알맞게 생성한다.
5. 구상 팩토리는 구상 제품들을 인스턴스하거나 그 제품들의 생성 메서드들의 추상 제품을 반환해야한다.

## 코드

```java

```

## 장단점

- 구상 제품들과 클라이언트 코드 사이의 단단한 결합을 피할 수 있다.
- 단일 책임 원칙, 제품 생성 코드를 한곳으로 추출하여 코드를 더 쉽게 유지보수 할 수 있다.
- 개방/폐쇄 원칙. 기존 클라이언트 코드를 훼손하지 않고 제품의 새로운 변형들을 생성할 수 있다.
- 패턴과 함께 새로운 인터페이스들과 클래스들이 많이 도입된다.