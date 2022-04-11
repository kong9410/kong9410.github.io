---
title: 카프카 커넥트(Kafka Connect)
layout: post
tags: kafka
describe: A서버의 DB에 저장한 데이터를 카프카 프로듀서/컨슈머로 B서버의 DB로도 보낼 수 있다. 이러한 파이프라인이 여러개면 매번 반복적으로 파이프라인을 구성해주어야 한다. Kafka Connect는 이러한 반복적인 파이프라인 구성을 쉽고 간편하게 만들 수 있게 만들어진 프로젝트 중 하나다.
---

## Kafka Connect

Kafka는 Producer와 Consumer로 Data Pipeline을 만들 수 있다. 만일 A에서 B로 보내는 Producer와 Consumer로 이루어진 pipeline이 있고 이러한 pipeline이 여러개면 반복적으로 만들어 줘야한다. 이러한 파이프라인 구성을 쉽고 간편하게 만들 수 있게 만들어진 Apache Kafka 프로젝트 중 하나다.

![kafka connect](https://user-images.githubusercontent.com/37204770/162602937-c15f747e-b0df-4ba7-8d58-3ae3201c42ba.png)

그림은 왼쪽의 DB데이터를 Connect와 Source Connector를 사용해 Kafka Broker로 보내고 Connect와 Sink Connector를 사용해 Kafka에 담긴 데이터를 DB에 저장한다.

- Connect: Connector를 동작하게 하는 프로세서(서버)
- Connector: Data Source(DB)의 데이터를 처리하는 소스가 들어있는 jar파일
- Source Connector: Data Source에 담긴 데이터를 topic에 담는 역할(Producer)을 하는 Connector
- Sink Connector: topic에 담긴 데이터를 특정 data source로 보내는 역할(Consumer)을 하는 connector



Connect는 단일 모드(Standalone)와 분산 모드(Distributed)로 이루어져 있다. 보통 단일모드는 개발 테스트 환경에서 이용하고 분산 모드는 운영환경에서 이용한다.

- 단일 모드(Standalone): 하나의 Connect만 사용하는 모드
- 분산 모드(Distributed): 여러개의 Connect를 한개의 클러스트로 묶어서 사용하는 모드



### 내부 구성

- 커넥터(Connector): 파이프라인에 대한 추상 객체. task들을 관리
- 테스크(Task): 카프카와의 메시지 복제에 대한 구현체. 실제 파이프라인 동작 요소
- 워커(Worker): 커넥터와 테스크를 실행하는 프로세스
- 컨버터(Converter): 커넥트와 외부 시스템 간 메시지를 변환하는 객체
- 트랜스폼(Transform): 커넥터를 통해 흘러가는 각 메시지에 대한 간단한 처리
- 데드 레터 큐(Dead Letter Queue): 커넥트가 커넥터의 에러를 처리하는 방식

구현된 커넥터를 이용하여 인스턴스를 생성하면, 커넥트 워커 프로세스 내부에 테스크들이 생성되고 파이프라인이 구동된다. 테스크 내부에는 외부 시스템 간 메시지를 변환하는 컨버터가 구성되고, 필요하면 트랜스폼을 통해 간단한 처리를 실시한다.



### 고려사항

커넥터를 고려하거나 구성할때 주의해야하는 점

- 외부 시스템을 지원하는 플러그인이 있는가?
- 플러그인의 라이센스는 어떻게 되어있는가?