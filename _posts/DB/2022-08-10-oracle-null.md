---
title: 개발자들의 영원한 숙제 NULL
tags: database oracle
layout: post
description: 오라클에서 null을 다루는 방법을 알아보자
---

## 사칙연산에서 NULL

```sql
SELECT NULL + 3 FROM DUAL
-- 결과: NULL
SELECT NULL - 3 FROM DUAL
-- 결과: NULL
SELECT NULL * 3 FROM DUAL
-- 결과: NULL
SELECT NULL / 3 FROM DUAL
-- 결과: NULL
```

사칙연산에서 NULL 계산 결과는 항상 NULL 이다

## 비교연산에서 NULL

| 번호   | 기준금액 | 목표금액 |
| ---- | ---- | ---- |
| 1    | 100  | 200  |
| 2    | 100  | NULL |
| 3    | NULL | 200  |
| 4    | NULL | NULL |

```sql
SELECT 번호
  FROM 실적
 WHERE 기준금액 = 목표금액
-- 결과 없음
SELECT 번호
FROM 실적
WHERE 기준금액 > 목표금액
-- 결과 없음
SELECT 번호
FROM 실적
WHERE 기준금액 < 목표금액
-- 결과: 1
SELECT 번호
FROM 실적
WHERE 기준금액 <> 목표금액
-- 결과: 1
```

알 수 없는 값인 NULL은 어떤 비교를 하던 NULL이다. NULL과 NULL의 비교도 무의미 함을 알 수 있다.

## 집계함수에서 NULL

| 번호   | 주문금액 |
| ---- | ---- |
| 1    | 200  |
| 2    | 400  |
| 3    | NULL |

```sql
SELECT SUM(주문금액) FROM 주문 -- 결과: 600
SELECT AVG(주문금액) FROM 주문 -- 결과: 300
SELECT MAX(주문금액) FROM 주문 -- 결과: 400
SELECT MIN(주문금액) FROM 주문 -- 결과: 200
```

사칙연산과 다르게 SUM과 같은 집계함수는 NULL을 제외하고 집계 결과를 구한다.

## 문자열 결합에서 NULL

표준 SQL에서는 NULL과 문자열의 결합은 NULL 이다. 하지만 오라클 문자열 결합에서 NULL은 특별한 규칙을 따른다. 문자가 0인 문자열과 동일하게 인식한다.

```sql
SELECT NULL || 'ABC' FROM DUAL -- 결과 'ABC'
SELECT NULL || 3 FROM DUAL -- 결과 3
SELECT NULL || 'ABC' || 3 FROM DUAL -- 결과 'ABC3'
```

## 논리연산에서의 NULL

```sql
NULL AND TRUE -- NULL
NULL AND FALSE -- FALSE
NULL OR TRUE -- TRUE
NULL OR FALSE -- NULL
NOT(NULL) -- NULL
```

논리연산자에서 AND는 모든 조건을 만족해야 참이고 하나라도 거짓이면 거짓이다. OR은 모든 조건을 만족하지 않아야 거짓이고 하나라도 참이어도 참이다.



지금까지의 내용을 정리하면 다음과 같다.

1. 사칙연산에서 NULL 계산 결과는 의미가 없다.
2. 비교연산에서 NULL 비교 결과는 의미가 없다.
3. 집계함수에서 NULL 집계 결과는 의미가 있다.
4. 문자열 결합에서의 NULL 결합 결과는 의미가 있다.
5. 논리연산에서의 NULL 논리 결과는 의미가 있다/없다.

## 인덱스에서 NULL

인덱스가 있는 컬럼이라도 조건절에 `IS NULL` 혹은 `IS NOT NULL` 구문 사용 시 인덱스를 사용하지 못하고 풀스캔하게 된다. 이유는 인덱스는 기본적으로 NULL 정보를 보관하지 않기 때문이다.

주문테이블이 있다고 가정하다 최초 주문 데이터가 insert될때 배송일자 컬럼은 null이 된다. 배송이 완료되면 배송일자 컬럼에 char(8) 값을 덥데이트 한다. 그렇다면 다음 쿼리로 배송 못한 데이터를 조회할 수 있다.

```sql
SELECT * FROM 주문 WHERE 주문일자 IS NULL
```

앞서 말한바와 같이 IS NULL은 인덱스를 사용하지 못한다. 그렇기 때문에 다음과 같은 방법을 생각해 볼 수 있다.

1. NULL 회피 전략

   배송일자에 null이 아닌 '99991231'과 같은 우리가 전혀 접해볼 수 없는 미래의 날짜를 사용하는 것이다. 이렇게하면 인덱스를 사용할 수 있다.

2. 함수 기반 인덱스(Function Based Index)

   컬럼에 함수까지 포함시켜서 인덱스를 생성하면 된다. 인덱스는 NULL값을 보관하지 않지만 함수 기반 인덱스는 함수를 이용해 변환된 값을 보관하기 때문에 가능하다. 이 경우 배송일자 컬럼에 NULL값이 있어도 되므로 유용하다.

   ```sql
   SELECT * FROM 주문 WHERE NVL2(배송일자, 배송일자, '99991231') = '99991231'
   ```

만일 IS NOT NULL 일경우 어떻게 되는지 알아보자

```sql
SELECT * FROM 주문 WHERE 배송일자 IS NOT NULL
```

만일 배송일자 값이 IS NOT NULL인 경우가 대부분이라면 어떠한 방법도 없어 풀스캔이 더 효율적일 것이다. 만약 전체 데이터 건수에서 배송일자 값이 IS NOT NULL 인 경우가 적다면, 다음과 같이 BETWEEN 구문을 이용해 NULL 사용을 대체할 수 있다.

```sql
SELECT * FROM 주문 WHERE 배송일자 BETWEEN '00010101' AND '99991231'
```

이때는 인덱스를 사용할 수 있게 된다. 조회 구간 범위가 적어서 테이블 풀스캔보다 인덱스를 통한 접근이 효율적이다.

## 검색에서의 NULL

SQL 문 작성시 검색 조건에서 NULL과 관련하여 조회하는 경우가 많다. NULL 검색은 `IS NULL` 혹은 `IS NOT NULL`로 조건을 걸어야 한다.

```SQL
WHERE 컬럼 IS NULL -- 올바른 NULL 조건
WHERE 컬럼 IS NOT NULL -- 올바른 NULL 조건
WHERE 컬럼 = NULL -- 잘못된 NULL 조건
WHERE 컬럼 <> NULL -- 잘못된 NULL 조건
WHERE 컬럼 = 'NULL' -- 문자열 NULL 검색
WHERE 컬럼 LIKE '%NULL%' -- 문자열 NULL 검색
WHERE 컬럼 IN (NULL) -- 잘못된 NULL 검색
WHERE 컬럼 IN ('NULL') -- 문자열 NULL 검색
```

## 함수에서의 NULL

`NVL`, `NVL2`는 대표적인 NULL 관련 함수다. 해당 컬럼의 값이 NULL이면 특정 컬럼 값으로 치환해야 하는 경우 사용한다.

```SQL
NVL(컬럼, NULL이면 치환할 값)
NVL2(컬럼, NULL이 아니면 치환할 값, NULL이면 치환할 값)
```

NVL함수 사용시 주의해야 할 점이 있다.

| 번호   | 주문금액 |
| ---- | ---- |
| 1    | 200  |
| 2    | 400  |
| 3    | NULL |

위 주문금액 컬럼에는 NULL이 존재한다. 집계함수 사용시 NULL은 제외하고 결과를 구한다. 이때 NVL함수와 SUM함수를 동시에 사용한다음 SQL을 살펴본다.

```sql
SELECT SUM(주문금액) -- 결과 600
SELECT SUM(NVL(주문금액, 0)) -- 결과 600
SELECT NVL(SUM(주문금액), 0) -- 결과 600
```

첫번째 SQL문은 올바르게 사용했으나 집계할 레코드가 없는 경우 NULL을 리턴한다.

두번째 SQL문은 집계함수에서 NULL은 제외하고 사용하기 때문에 NVL은 의미가 없다. 오히려 NVL 함수를 호출함에 따라 부하가 더 발생한다.

세번째 SQL문은 집계할 데이터가 없다면 0이 리턴된다.

```sql
SELECT AVG(주문금액) -- 결과 300
SELECT AVG(NVL(주문금액, 0)) -- 결과 200
SELECT NVL(AVG(주문금액), 0) -- 결과 300
```

첫번째 SQL문은 올바르게 사용했으나 집계할 레코드가 없는 경우 NULL을 리턴한다.

두 번째 SQL문은 NULL값이 있는 레코드가 0으로 치환되어 분자의 값은 변함이 없으나 분모의 값을 크게 만들어서 평균값이 낮아지는 결과를 초래한다.

세 번째 SQL문은 올바른 방법이다. 집계할 레코드가 없을 경우 0을 리턴한다.

## 조인에서의 NULL

OUTER JOIN에서 연결되지 않는 레코드의 컬럼 값은 NULL 이다.

```sql
SELECT * FROM 주문 A, 고객 B WHERE A.고객번호 = B.고객번호(+)
SELECT * FROM 주문 A LEFT OUTER JOIN 고객 B ON A.고객번호 = B.고객번호
```

둘중 어떤 형식을 사용하던지 연결되지 않는 레코드 값은 NULL이다.

## NULL 회피 전략

NULL에 대해 충분히 이해하고 사용함에 전혀 부족함이 없다면 NULL을 사용하면 된다. 그렇지 않다면 NULL을 회피할 수도 있다. 테이블 생성시 NOT NULL 컬럼으로 규정하거나 NULL 대신 의미 있는 값을 사용하면 된다.