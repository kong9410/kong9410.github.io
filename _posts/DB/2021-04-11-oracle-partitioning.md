---
title: 테이블 파티셔닝
tags: oracle database
---

# 파티셔닝

물리적 데이터를 논리적으로 나눈다는 뜻이다.

## 테이블 파티셔닝

논리적으로는 테이블로 접근하지만 물리적으로는 테이블 내의 각각의 파티션으로 접근한다. 각각의 파티션은 세그먼트에 해당한다. 세그먼트는 테이블과 1:1을 갖는다. 테이블 파티셔닝은 테이블 데이터를 일정 기준으로 나누어 저장하는 것이다. 예를 들어 서점에서 '오라클'관련 책을 찾기 위해 '데이터베이스' 코너로 가서 오라클을 찾을 수 있을것이다. 이 처럼 파티션은 데이터를 찾을 때 조금 더 쉽고 빠르게 찾을 수 있도록 도와주는 것이다. 단일 파티션으로 되어있는 경우에는 특정 영역에 해당하는 데이터를 삭제하려면 DELETE를 사용해야하지만 파티셔닝의 경우는 해당 파티션을 DDL로 삭제를 할 수 있다.

파티셔닝을 하는 방법은 크게 3가지가 있다.

- 레인지(Range) 파티션
- 리스트(List) 파티션
- 해시(Hash) 파티션

### 레인지 파티션

- 일반적으로 파티션을 나눌 때 가장 많이 쓰이는 형식
- 나누는 기준이 모호하거나 날짜별로 구분한다.
- 키 칼럼의 값이 범위를 가지고 있다
- LESS THAN에 해당하는 값은 포함하지 않는다

```sql
CREATE TABLE ord_range (
	ord_no NUMBER(10) NOT NULL
  , ord_dt VARCHAR2(8) NOT NULL
  , ...
)
PARTITION BY RANGE (ord_dt)
(
	PARTITION P201201 VALUES LESS THAN ('201202')
  , PARTITION P201202 VALUES LESS THAN ('201203')
  , PARTITION P201203 VALUES LESS THAN ('201204')
    ...
  , PARTITION P_DEFAULT VALUES LESS THAN (MAXVALUE)
)
```

각 파티션 명은 P201201과 같은 날짜별 이름이고 LESS THAN을 이용해서 구분지었다. 기준은 ord_dt다. 이 이외의 값은 P_DEFAULT에 저장하게 된다. 여기서 만일 1월 데이터만 조회를 한다면 1월에 해당하는 파티션만 읽게된다.

### 리스트 파티션

- 키 칼럼의 값이 설정한 값과 일치할 경우 사용한다. 
- 예를 들어 20120101 과 같이 일자가아닌 201201로 월로 끊어지고 월로 파티셔닝을 나눈다 할 때 사용할 수 있다.
- 구분되는 기준이 명확해야 데이터가 한쪽으로 쏠리지 않고 사용할 수 있다

```sql
CREATE TABLE ord_list (
	ord_dt VARCHAR2(6) NOT NULL
    ...
)
PARTITION BY RANGE (ord_dt)
(
	PARTITION P201201 VALUES ('201201')
  , PARTITION P201202 VALUES ('201202')
  , PARTITION P201203 VALUES ('201203')
    ...
  , PARTITION P_DEFAULT VALUES DEFAULT
)
```

ord_dt이 해당 년,월에 따라 파티션이 나뉘고 그 외의 값은 P_DEFAULT가 된다.

### 해시 파티션

- 키 칼럼의 값을 해시 함수를 이용해 지정한 파티션의 개수로 나누어 저장한다.
- 해시 파티션은 오라클이 해시 함수로 데이터를 분산시킨다.
- 키 칼럼을 저장할 때 데이터 분포를 고려해야한다
- 해시 함수는 같은 값이 입력되면 항상 같은 값이 반환된다.
- 같은 값이 많지 않은 칼럼을 사용해야 효과를 볼 수 있다
- 동치(=) 조건과 IN 조건으로만 사용할 수 있고, 부등호나 BETWEEN 조건으로 사용될 가능성이 있는 칼럼은 되도록 사용하지 않는다.

```sql
CREATE TABLE ord_hash (
	ord_no NUMBER(10) NOT NULL
  , ord_dt VARCHAR2(8)
)
PARTITION BY HASH (ord_no) PARTITIONS 8
```

해시 파티션은 파티션을 나누어 데이터를 나누어 담는 것외에는 의미가 없어보인다. 해시 조인을 사용할때 해시 파티션은 유리할 수가 있다.

#### 복합 파티셔닝

테이블을  파티셔닝을 할 계획이라면 복합 파티션을 우선 고려해 보는 게 좋다. 복합 파티션을 만들 때 삭제와 조회 기준으로 전략을 짜면 좋다

- 복합 파티션을 메인 파티션과 서브 파티션으로 구분한다.
- 메인으로는 RANGE와 LIST만 가능하다.
- 서브로는 RANGE, LIST, HASH 모두 가능하다.
- 삭제가 빈번하고 특정 컬럼과 조인을 자주한다면 [RANGE(삭제기준칼럼) - HASH(조인기준칼럼)], [LIST(삭제기준칼럼) - HASH(조인기준칼럼)] 전략이 좋다.

## 인덱스 파티셔닝

인덱스도 테이블 데이터에 종속적이기 때문에 인덱스를 나눌 때도 테이블을 나누는 전략을 잘 활용해야한다.

파티션 테이블에서 인덱스를 만들 때 세 가지로 나눌 수 있다.

### 비파티션 인덱스

- 테이블을 파티셔닝 할때 인덱스는 파티션을 하지 않는다.

- PK의 경우에는 비파티션 인덱스를 사용해야 완벽하게 무결성을 유지할 수 있다

### 로컬 파티션 인덱스

- 테이블 파티션과 같은 기준으로 인덱스를 나눈다.
- 테이블 파티션 1개당 인덱스 파티션도 1개 만들어진다
- 파티션을 삭제할 때ㅑ 해당 파티션의 로컬 인덱스도 동시에 삭제가 가능하다

### 글로벌 파티션 인덱스

- 테이블 파티션과는 다른 기준으로 인덱스를 나눔
- 테이블 파티션의 기준과 다른 키 컬럼을 적용하고자 할 때 사용한다.
- 예를 들어 테이블은 주문번호로 나누고, 인덱스는 주문날짜로 나누는 경우다.
- 제약 사항이 많아 글로벌 파티션 인덱스는 거의 사용하지 않는다

## 파티션 Pruning

테이블 전체를 읽지 않고, 필요한 파티션만을 읽을 수 있도록 하는 기능이다.  몇가지 예를들어 설명한다.

```sql
SELECT COUNT(*)
FROM ord_list
WHERE ord_ym = '201201'
```

ord_list의 경우에는 ord_ym 기준으로 list 파티셔닝이 되어있다. 이 경우 실행계획을 보면 table access full scan이 되겠지만 실제로는 partition list single로 ord_ym에 해당하는 파티션 하나만을 스캔함을 확인할 수 있다.

```sql
SELECT COUNT(*)
FROM ord_range
WHERE ord_ym = '201201'
```

range 테이블의 경우는 filter를 한번 거치는데 ord_ym의 값이 한 개뿐이라는 것을 알 수 없기 때문에 필터를 한번 거친다.

```sql
/*
인덱스(로컬 파티션) ORD_LIST_X01 = ORD_DT + ORD_HMS
*/
SELECT COUNT(*)
FROM ORD_LIST
WHERE ORD_YM = '201201'
AND ORD_DT BETWEEN '20120101' AND '20120110'

  Operation				NAME
0 SELECT STATEMENT
1 SORT ARREGATE
2 PARTITION LIST SINGLE
3 INDEX RANGE SCAN		ORD_LIST_X01

3 - access("ORD_DT" >= '20120101' AND "ORD_DT" <= '20120110'
```

ORD_YM의 필터 없이 테이블로의 랜덤 액세스 없이 인덱스만 읽었다.

```sql
/*
인덱스(로컬 파티션) ORD_RANGE_X01 = ORD_DT + ORD_HMS
*/
SELECT COUNT(*)
FROM ORD_RANGE
WHERE ORD_YM = '201201'
AND ORD_DT BETWEEN '20120101' AND '20120110'

  operation
0 SELECT
1 SORT
2 PARTITION RANGE SINGLE
3 TABLE ACCESS BY LOCAL INDEX ROWID	ORD_RANGE
4 INDEX RANGE SCAN					ORD_RANGE_X01

3 - filter("ORD_YM"='201201')
4 - access("ORD_DT">='20120101' AND "ORD_DT"<='20120110')
```

인덱스를 액세스한 모든 로우에서 테이블로 접근이 있었다. ORD_YM 조건 때문에 파티션 Pruning이 일어났지만 ORD_YM의 값이 하나 뿐이라 보장하지 못하므로 테이블로의 액세스가 발생한다.

이 외에도 여러가지 pruning 방법이 있다.

리스트 파티션을 만들어도 경우에 따라 BETWEEN과 같은 범위 조건을 사용했을 경우 해당 파티션만 액세스 할 수 있다.

파티션 키 컬럼도 인덱스 칼럼과 마찬가지로 가공하면 오라클이 어느 파티션으로 액세스 해야할지 모른다. 필요하다면 값을 가공해야한다.

MERGE나 UPDATE문의 경우에도 Pruning을 사용하는 것이 좋다.

