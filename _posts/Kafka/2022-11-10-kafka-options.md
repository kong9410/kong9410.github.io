---
title: 카프카 옵션
tags: kafka
layout: post
description: 카프카 주요 옵션과 관련된 개념들도 알아보자
---

## 카프카 컨슈머 옵션

- bootstrap.servers
  - 카프카 클러스터에 처음 연결을 하기 위한 호스트와 포트 정보로 구성된 리스트 정보를 나타낸다.
- fetch.min.bytes
  - 한번에 가져올 수 있는 데이터 사이즈
- fetch.max.bytes
  - 한번에 가져올 수 있는 최대 데이터 사이즈
- group.id
  - 컨슈머가 속한 컨슈머 그룹을 식별하는 식별자
- enable.auto.commit
  - 백그라운드로 주기적으로 오프셋을 커밋
- auto.offset.reset
  - 카프카 초기 오프셋이 없거나 현재 오프셋이 더 이상 존재하지 않은 경우에 다음 옵션으로 리셋한다.
  - earilest: 가장 초기의 오프셋 값으로 설정
  - latest: 가장 마지막의 오프셋값으로 설정
  - none: 이전 오프셋값을 찾지 못하면 에러 발생
- request.timeout.ms
  - 요청에 대해 응답을 기다리는 최대 시간
- session.timeout.ms
  - 컨슈머와 브로커사이의 세션 타임아웃 시간
- heartbeat.interval.ms
  - 그룹 코디네이터에게 얼마나 자주 KafkaConsumer poll() 메소드로 heartbeat를 보낼 것인지 조절한다. session.timeout.ms와 밀접한 관계가 있고, session.timeout.ms 보다 낮아야 한다. 일반적으로 3분의 1정도로 설정
  - 기본값 3초
- max.poll.records
  - 단일 호출 poll()에 대한 최대 레코드 수를 조절한다.
- max.poll.interval.ms
  - 컨슈머가 하트비트만 보내고 메시지를 가져오지 않을 경우 무한정 해당 파티션을 점유할 수 없도록 주기적으로 poll을 호출하지 않으면 장애라고 판단하여 컨슈머 그룹에서 제외한 후 다른 컨슈머가 해당 파티션을 가져갈 수 있도록 한다.
- auto.commit.interval.ms
  - 주기적으로 오프셋을 커밋하는 시간
- fetch.max.wait.ms
  - fetch.min.bytes에 의해 설정된 데이터보다 적은 경우 요청에 응답을 기다리는 최대 시간



