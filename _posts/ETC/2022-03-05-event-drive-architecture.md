---
title: Event Driven Architecture(EDA)
layout: post
tags: msa eda
---

# Event Drive Architecture (EDA)

- 애플리케이션 설계를 위한 소프트웨어 아키텍쳐 및 모델이다
- 이벤트 아키텍처는 최소한의 결합을 지원하므로 현대적인 분산형 애플리케이션 아키텍처에 적합한 방법이다.

## Event

- 하드웨어 또는 소프트웨어 상태의 변화 또는 중대 사건의 발생을 의미
- 데이터 생성, 변경, 삭제를 보통 의미

## 이벤트 흐름 레이어

![Diagram of an event-driven architecture style](https://docs.microsoft.com/ko-kr/azure/architecture/guide/architecture-styles/images/event-driven.svg)

- 이벤트 프로듀서
  - 이벤트 메시지를 생성하는 이벤트 생성자이다.
  - 이메일 클라이언트
- 이벤트 채널
  - 이벤트 생성기에서 수집된 정보를 전파하는 메커니즘이다.
  - TCP/IP, 입력파일(XML, 전자메일 등)
- 이벤트 프로세싱 엔진
  - 이벤트 처리 엔진은 이벤트를 식별한 다음 적절한 반응을 선택하고 실행하는 논리 계층이다.
- 다운스트림 이벤트 기반 활동
  - 이벤트 결과가 표시되는 논리계층
  - 이메일이 사람에게 전송된다.

## 동작

- 이벤트 생성자와 이벤트 소비자로 구성되어 있다.
- 이벤트 생성자: 이벤트를 감지하고 메시지로 해당 이벤트를 나타낸다.
- 이벤트 채널을 통해 이벤트 생성자에서 이벤트 소비자로 전송된다.
- 이벤트 소비자: 이벤트 알람을 받고 이벤트를 처리해야한다.



## 이벤트 기반 아키텍처 모델

- 게시/구독 모델
  - 이벤트 스트림 구독 기반의 메시징 인프라
  - 이벤트 발생 후 알림을 받아야 하는 구독자에게 이벤트가 전송된다.
- 이벤트 스트리밍 모델
  - 이벤트가 로그에 기록된다.
  - 이벤트 소비자는 이벤트 스트림을 구독하지 않고 스트림의 모든 부분에서 읽기가 가능하고 언제든지 스트림에 참여할 수 있다.
    - 이벤트 스트림 처리: 카프카와 같이 스트리밍 플랫폼을 사용하여 이벤트를 수집하고 이벤트 스트림을 처리하거나 변환한다.
    - 단순 이벤트 처리: 이벤트가 이벤트 소비자에게 즉각적으로 동작을 트리거하는 경우
    - 복합 이벤트 처리: 이벤트 소비자가 패턴을 감지하기 위해 일련의 이벤트를 처리한다.



## 사용해야 하는 경우

- 여러 하위 시스템이 동일한 이벤트를 처리해야 하는 경우
- 최소 시간 지연의 실시간 처리
- 패턴 일치 또는 일정 기간의 집계와 같은 복합 이벤트 처리



## EDA 이점

- 생산자와 소비자가 분리됨
- 시스템에 새 소비자를 쉽게 추가할 수 있다.
- 확장성이 있다.
- 하위 시스템에서 독립적으로 확인할 수 있다.



## 출처

[이벤트 기반 아키텍처 스타일 - Azure Application Architecture Guide | Microsoft Docs](https://docs.microsoft.com/ko-kr/azure/architecture/guide/architecture-styles/event-driven)

[이벤트 기반 아키텍처란 무엇일까요? (redhat.com)](https://www.redhat.com/ko/topics/integration/what-is-event-driven-architecture)

[Event-driven architecture - Wikipedia](https://en.wikipedia.org/wiki/Event-driven_architecture)



