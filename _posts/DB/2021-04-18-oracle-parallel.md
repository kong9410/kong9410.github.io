---
title: 오라클 병렬처리
tags: database oracle
---

## SQL 명령문의 병렬 실행 과정

- 사용자 세션 또는 쉐도우 프로세스는 쿼리 코디네이터(QC)라는 역할을 수행한다.
- 쿼리 코디네이터는 병렬 서버 수를 가져온다.
- SQL문은 작업 시퀀스로 실행된다.  병렬 실행 서버는 가능하면 병렬로 각 작업을 수행한다.
- 병렬 서버가 명령문 실행이 완료되면 쿼리 코디네이터는 병렬로 실행할 수 없는 작업의 모든 부분을 수행한다. 예를 들어 작업이 있는 병렬 쿼리는 각 병렬 처리에서 계산한  개별 하위 합계를 추가해야한다.
- 마지막으로 쿼리 코디네이터는 결과를 사용자에게 반환한다.

## DOP(Degree Of Parallelism)

DOP(Degree Of Parallelism)는 병렬처리 할 때 병렬 프로세스를 몇개 띄울 지를 의미한다. DOP가 20이면 20개의 병렬 프로세스를 띄워서 작업한다는 의미이다.

DB서버의 자원(CPU, Memory, Disk I/O)을 최대한 사용해서 작업을 빠르게 끝낼 수 있는 아주 유용한 기능이다.

## 오라클에서 병렬처리를 사용하는 방법

```sql
SELECT /*+ parallel */ *
FROM some_table
...
```

```sql
SELECT /*+ parallel(10) */ *
FROM some_table
...
```

```sql
SELECT /*+ parallel(some_table, 10) */ *
FROM some_table
...
```

```sql
ALTER SESSION SET parallel_degree_policy = limited;
ALTER TABLE emp parallel (degree default);
```

- 병렬 프로세스 갯수를 지정할 수 있고 안할 수도 있다.
- 테이블을 지정할 수도 있다.
- 병렬 프로세스 갯수를 지정하지 않고 사용할 경우 시스템 디폴트 만큼 가동될 수 있다.
- 왠만하면 병렬 프로세스 갯수를 지정해서 사용하는 것이 좋다.
- 갯수를 지정한다하여도 반드시 지정된 갯수만큼 작동되는 것은 아니고, 시스템 리소스 상황에 따라 자동으로 조정된다.
- Trace나 Realtime SQL Monitoring을 활용해 병렬프로세스가 얼마나 뜨는지 확인하는 것이 좋다.
- 1로 세팅할시에 병렬처리를 하지 않는다.

## Producer와 Consumer 운영

![Description of Figure 8-2 follows](https://docs.oracle.com/cd/E11882_01/server.112/e25523/img/vldbg013.gif)

- 데이터를 읽는 병렬 프로세서들을 Producer라고 하고, 이 읽은 데이터를 받아서 sorting, DML, DDL, Join 등을 수행하는 작업을 하는 병렬 프로세서들을 Consumer라고 한다.
- 작업된 결과를 통합하는 것을 QC(Query Coordinator)라고 한다.
- 각 프로세스간 통신이 발생하는 비 효율 요소가 있다.



## 병렬 실행 서버 통신 하는 방법

![Description of Figure 8-3 follows](https://docs.oracle.com/cd/E11882_01/server.112/e25523/img/vldbg015.gif)

- 오라클 데이터베이스는 쿼리를 병렬로 실행하기 위해 생산자 병렬 실행 서버 집합과 소비자 병렬 실행 서버 집합을 만든다.
- 생산자 서버는 테이블 행을 검색한다.
- 소비자 서버는 이러한 행에서 조인, 정렬, DML 및 DDL과 같은 작업을 수행한다.
- 생산자 집합의 각 서버에는 소비자 집합의 각 서버에 대한 연결이 있다.
- 병렬 실행 서버간의 가상 연결 수는 병렬 연결 정도의 제곱으로 증가한다.
- 단일 인스턴스 환경은 각 통신 채널에 대해 대부분의 3개의 버퍼에서 사용한다.
- 오라클 애플리케이션 클러스터에서는 각 채널에 대해 대부분 4개 버퍼를 사용한다.
- 동일한 인스턴스의 두 프로세스 사이에 연결이 있는 경우 서버는 버퍼를 메모리 앞뒤로 전달하여 통신한다.
- 서로 다른 인스턴스의 프로세스 간 연결이 있는 경우, 상호 연결을 통해 외부 고속 네트워크 프로토콜을 사용하여 메시지가 전송된다.



## 참고

[How Parallel Execution Works (oracle.com)](https://docs.oracle.com/cd/E11882_01/server.112/e25523/parallel002.htm)

[07. 병렬 처리 - [종료\]구루비 DB 스터디 - 개발자, DBA가 함께 만들어가는 구루비 지식창고! (gurubee.net)](http://wiki.gurubee.net/pages/viewpage.action?pageId=29065839)

[오라클 병렬처리 Parallel DOP (Degree of Parallelism) (tistory.com)](https://jack-of-all-trades.tistory.com/96)

[오라클 병렬처리(Parallel Processing) 개념 및 용어 정리, 종합페이지 (tistory.com)](https://jack-of-all-trades.tistory.com/198)