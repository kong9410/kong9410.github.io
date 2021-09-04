---
title: 영속성 컨텍스트
tags: jpa database
---

# 영속성 컨텍스트

- 논리적인 개념
- Entity를 영구 저장하는 환경

## 엔티티 생명 주기

- 비영속(new/transient)
  - 영속성 컨텍스트와 관계 없는 상태
  - 객체를 생성만 한 상태
- 영속(managed)
  - 영속성 컨텍스트에 저장된 상태
  - Entity가 영속성 컨텍스트에 의해 관리되는 상태
- 준영속(detached)
  - 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제(removed)
  - 실제 DB 삭제를 요청한 상태

## 사용 이유

### 1차 캐시

- 1차 캐시 = 영속성 컨텍스트
- `Map<Key, Value>`로 1차 저장
  - key: @Id로 선언한 필드값
  - value: 해당 Entity 자체
- 1차 캐시에 Entity가 있을 때의 이점
  - 조회
    - entityManager.find()를 하면 1차 캐시를 조회
    - 1차 캐시에 해당 Entity가 존재하면 바로 반환
    - 조회하고자 하는 Entity가 없으면 DB에서 조회후 1차 캐시에 저장
    - 그러나 Transaction유지 동안에만 유지됨

### 동일성 보장

- 같은 Entity를 2번 조회하면 1차 캐시에 의해 같은 Reference가 조회된다

### 엔티티 등록시 트랜잭션을 지원하는 쓰기 지연

- EntityManager는 데이터 변경시 트랜잭션을 시작해야한다.
- `persist`는 SQL을 쌓고 있는 상태
- `commit`으로 Transaction 커밋 (동시에 쿼리를 보낸다)
  - `flush()`는 쿼리를 DB에 날려서 DB와 싱크를 맞추는 역할을 한다
  - `flush()` 후에 실제 DB Transaction이 커밋된다.
- 버퍼링 기능이다.

### 엔티티 "수정"시 변경 감지(Dirty Checking)

- Entity 데이터 수정시 `update()`나 `persist()`로 해당 데이터를 업데이트 해달라고 알려줄 필요가 없다.
- `Entity`를 수정하고 `commit`하면 알아서 DB에 반영됨

#### 변경 감지(Dirty Checking)

- 1차 캐시
  - @Id, Entity, Snapshot
  - Snapshot: 영속성 컨텍스트에 최초로 값이 들어왔을 때의 상태값을 저장
- 변경 감지 매커니즘
  - `transaction.commit()`
    - `flush()`가 일어날때 엔티티와 스냅샷을 비교
    - 변경사항이 있으면 UPDATE QUERY생성
    - UPDATE QUERY를 지연 SQL 저장소에 넣는다
    - UPDATE QUERY를 DB에 반영 후 `commit`

### 엔티티 삭제

- Entity 수정에서의 메커니즘과 동일
- Transaction의 commit 시점에 DELETE Query가 나간다.

