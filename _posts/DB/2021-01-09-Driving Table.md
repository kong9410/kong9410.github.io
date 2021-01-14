---
title: Driving Table
tags: database
---

# Driving Table

> JOIN시 먼저 액세스 되서 ACCESS PATH를 주도하는 테이블을 드라이빙테이블이라고 한다.

먼저 액세스 되는 테이블을 DRIVING TABLE, 나중에 액세스 되는 테이블을 INNER TABLE이라고 한다.



결정 규칙

1. JOIN 되는 칼럼의 한쪽에만 INDEX가 있는 경우, INDEX가 지정된 TABLE이 드라이빙 테이블이 된다.

   예)

   ```sql
   WHERE emp.deptno = dept.deptno
   ```

   dept.deptno에 인덱스가 있는 경우 dept 테이블이 드라이빙 테이블이 된다.

2. 연결되는 조건에 양쪽 모두 인덱스가 있는 경우, 상수 조건이 있는 테이블이 처리 범위를 줄이고 비용을 적게하기 때문에 상수 조건이 있는 테이블이 DRIVING TABLE이 된다.