---
layout: post
title: 파티셔닝
tags: Database
---

## Partitioning

하나의 DBMS가 수많은 테이블을 관리하다보니까 성능이 저하되는 문제가 생겨 이런 문제를 해결하기 위한 방법 중 하나인 파티셔닝이 있다. 파티셔닝은 큰 Table이나 인덱스를 관리하기 쉬운 단위로 분리하는 방법을 말한다.

### 이점

- 가용성

   물리적 파티셔닝으로 인해 데이터 훼손 가능성 낮아지고 가용성 향상

- 관리용이성

  큰 테이블을 제거하여 쉽게 관리할 수 있음

- 성능

  특정 DML과 Query의 성능을 향상시킴, 주로 대용량 데이터 Write 환경에서 효율적

  많은 Insert가 있는 OLTP 시스템에서 Insert 작업들을 분리된 파티션들로 분산시켜 경합을 최소화

## 단점

- Join 비용이 증가
- 테이블과 인덱스를 같이 파티셔닝 해야함



## Partitioning 범위

### Range Partitioning

연속적인 숫자, 날짜를 기준으로 Partitioning 한다.

날짜 => [1월, 2월, 3월 ... 12월]

손쉬운 방법으로, 관리 시간을 단축시킬 수 있다.

### List Partitioning

특정 Partition에 저장될 Data에 대한 명시적 제어 가능

분포도가 비슷하며, 많은 SQL에서 해당 Column의 조건이 많이 들어오는 경우에 유용

Multi-Column Partition Key 제공이 어려움

[한국, 중국, 일본 -> 아시아]

### Composite partitioning

Composite Partition은 Partition의 Sub-Partitioning을 말함

큰 파티션에 대한 I/O 요청을 여러 partition으로 분산

파티셔닝 결과 생성된 파티션이 너무 클때 유용함

Range-list, Range-Hash

### Hash Partitioning

Partition Key의 Hash 값에 의한 Partitioning (균등 분할 가능)

select시 조건과 무관하게 병렬 Degree 제공 (질의 성능 향상)

특정 Data가 어느 Hash Partition에 있는지 판단 불가

Hash Partition은 파티션을 위한 범위가 없는 데이터에 적합

## Partitioning 방법

### Horizontal Partitioning

데이터 개수를 기준으로 파티셔닝

데이터 개수가 작아지고 index개수도 작아져 자연스럽게 성능이 향상된다

서버간 연결과정이 많아진다

찾는 과정이 기존보다 복잡하기 때문에 지연시간이 증가

하나의 서버가 고장나게 되면 데이터의 무결성이 깨질 수 있음

### Vertical Partitioning

테이블의 칼럼을 기준으로 나누어 Partitioning

자주 사용하는 칼럼등을 분리시켜 성능 향상

Vertical Partitioning은 이미 정규화된 Data를 분리하는 과정



## Sharding

같은 테이블 스키마를 가진 데이터를 다수의 데이터베이스에 분산하여 저장하는 방법

Horizontal Partitioning으로 볼 수 있음



## Sharding 적용 전

가능하면 샤딩을 피하거나 지연시키는 방법을 찾는게 우선시 되어야함

- Scale-in
  - 하드웨어 스펙이 더 좋은 컴퓨터 사용
- Read 부하
  - Cache나 Replication 적용
- 일부 컬럼만 자주 사용
  - Vertically Partition 사용
  - Data를 Hot, Warm, Cold Data로 분리



## Sharding 방법

> Shard Key를 어떻게 정의하느냐에 따라 데이터를 효율적으로 분산시키는 것이 결정됨

### Hash Sharding

Shard Key : Database id를 Hashing 하여 결정
- Hash 크기는 Cluster안에 있는 Node 개수로 결정
Node의 개수가 변경된다면 ReSharding이 필요
공간에 대한 효율이 고려되지 않음 (예: 짝수번째에만 큰 데이터가 들어감)

### Dynamic Sharding

Naming을 Dynamic으로 바꿀 수 있음

Locator Service를 통해 Shard Key를 획득

Node개수가 늘어난 다면 Locator Servce에 Shard Key를 추가만 하면 됨

Data Relocate를 한다면 Locator Service의 Shard Key Table도 일치시켜줘야함

Locator가 성능을 위해 Cache하거나 Replication을 하면 잘못된 Routing을 통해 Data를 찾지 못하고 Error가 발생

Locator에 의존할수밖에 없음

### Entity Group

Hash Sharding과 Dynamic Sharding은 Key-Value 형태를 지원하기 위해 나온 방법

하나의 물리적인 Shard에 쿼리를 진행한다면 효율적

하나의 Shard에서 강한 응집도를 가질 수 있음

데이터는 자연스럽게 사용자별로 분리되어 저장

사용자가 늘어남에 따라 확장성이 좋은 Partitioning

Cross-partition 쿼리는 Single-partition 쿼리보다 consistency의 보장과 성능을 잃음

### Pitfall

> Logical Shard는 반드시 Sigle Node안에 있어야함

Dynamic Sharding을 진행하게 된다면 작업량을 효과적으로 줄일 수 있음

HotSpot을 찾고 Sharding을 진행

지속적으로 Sharding을 진행하게 된다면 오른쪽 Node만 Write를 진행하게 됨

나머지 Node들은 Read Performance 향상



## Replication

Application Server -> Master Database

단순한 데이터베이스를 구성할 때는 위와 같은 방식으로 만들수 있다. 그러나 많은 Query를 처리하기에는 힘든 상황이다. Query의 대부분을 차지하는 SELECT를 해결하기 위해 Replication을 사용한다.

두 개 이상의 DBMS 시스템을 Master/Slave로 나누어서 동일한 데이터를 저장하는 방식이다.

App -> Master Database
 |              |
Slave         |
Database ┘

Master DB는 수정사항만 바영하고 Slave에 실제 데이터를 복사한다.

### 복제 방법

#### 로그기반 복제

- Statement Based: SQL을 복사하여 진행 (Timestamp등에 따라 결과가 달라질 수 있음)
- RowBased: SQL에 따라 변경된 Row만 기록 (데이터가 많이 변경된 경우 데이터가 커짐)
- Mixed: 기본적으로 Statement Based로 진행하면서 필요에 따라 Row Based를 사용

### 장점

대부분의 Query가 Select이다. 때문에 Read(Select) 성능 향상 효과를 얻을 수 있다

Master Database 영향없이 로그를 분석할 수 있다