---
title: 오라클 딕셔너리 기반 DB 툴 프로그램 'FreeSQL'
tags: database oracle
layout: post
description: DBMS 관리정보를 갖고 있는 테이블 집합인 딕셔너리의 중요성
---

## 오라클 딕셔너리

DBMS를 관리하기 위한 정보를 갖고 있는 테이블의 집합을 딕셔너리(DICTIONARY)라 한다. 딕셔너리는 오라클 시스템에 대한 정보의 보고이며 데이터 사전이라 부르기도 한다. 사용자 소유의 정보를 알고자 한다면 다음과 같이 조회하면 된다.

```sql
SELECT * FROM USER_OBJECTS;
SELECT * FROM USER_TABLES;
SELECT * FROM V$SESSIONS
```

이와 같이 DB 시스템 정보를 제공하는 수많은 딕셔너리는 다음과 같이 알 수 있다.

```SQL
SELECT *
FROM DICTIONARY
-- 혹은
SELECT *
FROM DICT
```

| TABLE_NAME    | COMMENTS                                 |
| ------------- | ---------------------------------------- |
| USER_CUBES    | OLAP Cubes owned by the user in the database ... |
| USER_TABLES   | Description of ther user's own relational tables ... |
| USER_TRIGGERS | Triggers having FOLLOWS or PRECEDES ordering owned by the user ... |

현재 오라클은 딕셔너리를 통해 700여개 이상의 시스템 정보를 제공할 수 있다.

오라클 딕셔너리는 크게 다음과 같은 접두어로 분류해 볼 수 있다.

- ALL_*: 현재 오라클에 접속한 사용자가 접근 가능한 모든 정보에 대한 딕셔너리
- USER_*: 현재 오라클에 접속한 사용자가 소유하고 있는 모든 정보에 대한 딕셔너리
- DBA_*: 관리자 계정으로 접속한 사용자가 조회 가능한 모든 정보에 대한 딕셔너리
- V$: Dynamic Performance View라고도 하며, DBA의 모니터링에 많이 이용된다.

