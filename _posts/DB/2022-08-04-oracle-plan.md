---
title: 쿼리를 작성한 후에 실행계획을 확인하라
tags: database oracle
layout: post
description: 오라클 옵티마이저가 완벽한 plan을 제공하기 어려우므로 DBA는 물론 개발자도 실행계획을 볼줄 알아야 한다.
---

CBO 방식에서 옵티마이저는 주어진 환경(통계정보, SQL 문) 하에서 최적의 실행계획(Plan)을 우리에게 제공한다. 어떤 경로로 테이블을 접근하는지 어떤 방식으로 조인하는지 인덱스 자원을 사용하는지 등에 대한 최적화된 실행 계획을 알려준다.

## 실행 계획을 늘 확인하자

### 예시1

```sql
SELECT *
FROM 고객

Execution Plan
-------------------------
SELECT STATEMENT Optimizer=ALL ROWS
TABLE ACCESS (FULL) OF '고객' (TABLE) (Cost=633K Card=42M Bytes=15G)
```

- ACCESS FULL: 고객 테이블 풀 스캔(전체 접근)
- Cost=633K: 633,000비용발생(논리적비용=IO+메모리+CPU+네트워크+...)
- Card=42M: 42,000,000건(접근하는 레코드 수)
- Bytes=15G: 15,000,000,000(42,000,000+1 로우의 총 길이)

ACCESS FULL은 고객 테이블을 풀 스캔한다는 의미다. 어떠한 인덱스를 통하지 않고 접근한다는 뜻이다. 실행계획에서 이 용어가 보인다는 것은 다음 3가지 경우에 해당한다.

- 해당 쿼리에 적절한 인덱스가 존재하지 않은 경우로 필요한 인덱스를 생성함으로써 해결 가능
- 인덱스는 존재하지만 부정확한 통계정보로 인해 인덱스를 타지 않는 경우다. 최신의 통계 정보를 구성하거나 힌트절을 사용해 해결할 수 있다.
- 테이블 풀스캔이 인덱스를 통한 랜덤 액세스보다 유리한 경우다. 데이터 조회 범위가 커서 인덱스를 사용하는 것이 별로 효용성이 없을 때다.

Cost는 비용으로, 해당 쿼리가 동작했을 때 소요되는 비용을 말한다. 여기서 비용은 논리적 비용을 의미한다. 논리적 비용은 직접적이고 구체적인 수치에 의해 명확하게 알 수 없는 비용이다.

Card(Cardinality)는 쿼리 조건에 맞는 레코드 건수를 의미한다. 위의 쿼리는 42M이므로 데이터 건수가 4200만임을 알 수 있다.

Bytes는 쿼리 실행 시 발생하는 네트워크 트래픽을 의미한다. 즉 IO 발생량이다. 1로우를 구성하는 컬럼의 길이 총합을 구한 후 Card 값을 곱하면 된다.

위 실행계획을 종합하자면 고객 테이블을 풀스캔으로 접근해 4200만건의 데이터를 읽어온다. 이때 15,000,000,000바이트의 네트워크 트래픽을 유발하고 비용은 633K이다.

### 예시2

```sql
SELECT * FROM 고객
ORDER BY 고객명

Execution Plan
----------------------------
3 SELECT STATEMENT Optimizer=ALL ROWS
2    SORT (ORDER BY) (Cost=6M Card=42M Bytes=15G)
1        TABLE ACCESS (FULL) of '고객' (TABLE) (Cost=633K Card=42M Bytes=15G)
```

실행 계획을 해석하는 순서는 다음과 같다.

1. 레벨(깊이)이 다른 경우 안쪽 레벨부터 해석한다.
2. 레벨(깊이)이 같은 경우에는 위에서 아래로 해석한다.

고객 테이블을 풀스캔하는 Cost는 633K인데 반해, SORT하는 Cost는 6M임을 알 수가 있다. 결국 4200만건 데이터를 가져오는 것보다 가져온 데이터를 SORT하는 비용이 10배 가량 높음을 알 수 있다.

SORT를 없애는 것도 튜닝에서 중요한 부분임을 알 수 있다.

### 예시3

```sql
SELECT * FROM 고객
WHERE ROWNUM <= 1
Execution Plan
------------------------
3 SELECT STATEMENT Optimizer=ALL ROWS
2    COUNT (STOPKEY)
1        TABLE ACCESS (FULL) OF '고객' (TABLE) (Cost=2 Card=1 Bytes=356)
```

```sql
SELECT * FROM 고객
WHERE ROWNUM <= 2
Execution Plan
------------------------
3 SELECT STATEMENT Optimizer=ALL ROWS
2    COUNT (STOPKEY)
1        TABLE ACCESS (FULL) OF '고객' (TABLE) (Cost=2 Card=2 Bytes712)
```

위 두개의 쿼리의 차이점은 `ROWNUM <= 1` 과 `ROWNUM <= 2`이다. 테이블을 풀스캔하기 위해 접근하지만 COUNT(STOPKEY) 부분에서 레코드 건수가 1혹은 2가 됐을 때 스캔을 중지하고 빠져 나옴을 알 수 있다. 2개의 실행계획에서 ROWNUM 값이 1혹은 2에 따라서 Card값과 Bytes 값이 배수가 됨을 알 수 있다. 그런데 Cost가 같다.

그것은 고객테이블의 첫번째 레코드와 두번째 레코드가 동일 블럭에 있기 때문이다. 오라클은 최소 운반 단위인 블록 단위로 데이터를 운반한다.

### 예시4

```sql
SELECT *
FROM 주문 A, 고객 B
WHERE A.고객번호 = B.고객번호
AND A.주문번호 = '1501120003'

Execution Plan
-------------------------
6 SELECT STATEMENT Optimizer=ALL ROWS
5    NESTED LOOPS (Cost=6 Card=1 Bytes=560)
2        TABLE ACCESS (BY INDEX ROWID) OF '주문' (TABLE) (Cost=3 Card=1 Bytes=204)
1            INDEX(UNIQUE SCAN) OF '주문_주문번호_PK' (INDEX(UNIQUE)) (Cost=2 Card=1)
4        TABLE ACCESS (BY INDEX ROWID) OF '고객' (TABLE) (Cost=3 Card=1 Bytes=356)
3            INDEX(UNIQUE SCAN) OF '고객_주문번호_PK' (INDEX(UNIQUE)) (Cost=2 Card=1)
```

실행계획 내용에서 UNIQUE SCAN임을 알 수 있고, 두 테이블의 조인 방식은 NESTED LOOP JOIN임을 알 수 있다. 즉 순차적 루프에 의한 접근 방식이다. 실행계획의  해석 순서를 그림으로 변환하면 인덱스 생성도와 동일하다는 것을 알 수 있다(주문번호 인덱스 -> 주문 테이블 -> 고객번호 인덱스 -> 고객 테이블)

### 예시5

```sql
SELECT *
FROM 주문 A, 고객 B
WHERE A.고객번호 = B.고객번호
AND A.주문번호 = '1501120003'

Execution Plan
--------------------------
5 SELECT STATEMENT Optimizer=ALL ROWS
4    NESTED LOOPS (Cost=633K Card=1 Bytes=560)
2        TABLE ACCESS (BY INDEX ROWID) OF '주문' (TABLE) (Cost=3 Card=1 Bytes=204)
1            INDEX(UNIQUE SCAN) OF '주문_주문번호_PK' (INDEX(UNIQUE)) (Cost=2 Card=1)
3        TABLE ACCESS (FULL) OF '고객' (TABLE) (Cost=633K Card=1 Bytes=356)
```

고객 테이블에서 풀스캔이 발생하고 있다. 실제 리턴 결과 건수는 1이지만, 인덱스가 존재하지 않음에 따라 고객 테이블의 전체 데이터를 풀스캔하고 있다. 여기에서 우리는 Card=1임에 주목할 필요 있다. 비록 인덱스는 없지만 고객 테이블에 통계정보가 구성돼 있음을 유추할 수 있고, 고객번호는 UNIQUE함을 추정할 수 있다. 따라서 고객번호 컬럼을 인덱스로 생성해야 한다.

### 예시6

```sql
SELECT *
FROM 고객 A, 주문 B
WHERE A.고객번호 = B.고객번호
AND A.고객명 = '홍길동'
AND B.주문일자 = '20150112'

Execution Plan
---------------------------------
8 SELECT STATEMENT Optimizer=ALL ROWS
7    MERGE JOIN (Cost=52 Card=32 Bytes=17K)
3        SORT(JOIN) (Cost=14 Card=32 Bytes=11K)
2            TABLE ACCESS (BY INDEX ROWID) OF '고객' (TABLE) (Cost=10 Card=32 Bytes=11K)
1                INDEX(RANGE SCAN) OF '고객_고객명_IDX' (INDEX) (Cost=6 Card=32)
6        SORT(JOIN) (Cost=36 Card=852 Bytes=173K)
5            TABLE ACCESS (BY INDEX ROWID) OF '주문' (TABLE) (Cost=32 Card=852 Bytes=173K)
4                INDEX(RANGE SCAN) OF '주문_주문일자_IDX' (INDEX) (Cost=24 Card=852)
```

이 쿼리의 문제점은 조인절 양쪽 모두에 인덱스가 없다는 것이다. 두 테이블간 조인방식은 예전에는 Sort Merge Join 방식으로 풀리는 경우가 많았다. 하지만 성능상 문제가 많은 조인 방식이므로 Nested Loop Join 방식으로 실행계획이 풀리게 해야 한다.

업무상 인덱스를 생성하기 어렵다면 힌트절을 추가해 Hash Join 방식으로 접근하는 것도 좋은 방식이다. 대부분 Hash Join 방식이 Sort Merge Join 방식보다 좋다.

### 예시7

```sql
SELECT /*+ USE_HASH(B) */ *
FROM 고객 A, 주문 B
WHERE A.고객번호 = B.고객번호
AND A.고객명 = '홍길동'
AND B.주문일자 = '20150112'

Execution Plan
------------------------------
SELECT STATEMENT Optimizer=ALL ROWS
    HASH JOIN (Cost=52 Card=32 Bytes=17K)
        TABLE ACCESS (BY INDEX ROWID) OF '고객' (TABLE) (Cost=10 Card=32 Bytes=11K)
           INDEX(RANGE SCAN) OF '고객_고객명_IDX' (INDEX) (Cost=6 Card=32)
        TABLE ACCESS (BY INDEX ROWID) OF '주문' (TABLE) (Cost=32 Card=852 Bytes=173K)
           INDEX(RANGE SCAN) OF '주문_주문일자_IDX' (INDEX) (Cost=24 Card=852)
```

Hash Join 방식의 plan이다. 대량의 데이터 처리에 효율적인 방식이다. 처리 방법은 다음과 같다

- 고객 테이블에서 고객명이 '홍길동'인 고객을 구한 후, 조인절 컬럼인 고객번호를 해시 함수로 분류해 해시 테이블을 생성한다.(해시 함수를 이용해 해시 테이블 생성)
- 주문 테이블에서 주문일자가 '20140112'인 주문을 구한 후, 조인절 컬럼인 고객번호를 해시 함수로 변환해 해시 테이블로 순차적으로 접근한다(해시 함수를 통해 해시 테이블 탐색)

메모리에 해시 테이블을 생성하고 해시 함수를 이용해 연산 조인을 하기 때문에 CPU 사용이 증가할 수 있다. 따라서 조회 빈도가 높은 온라인 프로그램에는 적합하지 않는 조인 방식이다.

### 예시8

```sql
SELECT 고객ID, 고객명 FROM 고객 WHERE 우편번호 = '760010'
UNION ALL
SELECT 인사ID, 인사명 FROM 인사 WHERE 조직코드 = '0102'

Execution Plan
SELECT STATEMENT Optimizer=ALL ROWS
    UNION-ALL
        TABLE ACCESS (FULL) OF '고객' (TABLE) (Cost=633k Card=1K Bytes=356K)
            TABLE ACCESS (FULL) OF '인사' (TABLE) (Cost=108 Card=50 Bytes=7K)
```

```sql
SELECT 고객ID, 고객명 FROM 고객 WHERE 우편번호 = '760010'
UNION
SELECT 인사ID, 인사명 FROM 인사 WHERE 조직코드 = '0102'

Execution Plan
SELECT STATEMENT Optimizer=ALL ROWS
    SORT (UNIQUE) (Cost=633K Card=1K Bytes=363K)
        UNION-ALL
            TABLE ACCESS (FULL) OF '고객' (TABLE) (Cost=633k Card=1K Bytes=356K)
                TABLE ACCESS (FULL) OF '인사' (TABLE) (Cost=108 Card=50 Bytes=7K)
```

쿼리에서 UNION ALL 구문을 사용하면 중복되는 데이터를 있는 그대로 모두 보여준다.  두 실행계획의 차이점 SORT(UNIQUE) 부분이다.  이것은 데이터를 정렬한 후에 중복된 데이터를 제거하고 유니크하게 보여주는 데에 있다.

소트를 하는 이유는 중복된 데이터를 제거하는 가장 단순하고 쉬운 방법이기 때문에 한다. 소트 이후에 연속된 값을 제거하면 되기 때문이다.

## 오라클 옵티마이저의 실행계획과 개발자 실행계획

오라클 옵티마이저의 실행계획은 통계정보(과거+현재)를 기반으로 한다. 개발자는 과거+현재+미래로 정보를 기반으로 한다. 때문에 개발자 정보가 더 정확하다. 옵티마이저 실행계획과 개발자 실행계획이 일치하면 좋겠지만 항상 일치하지 않는다. 다음과 같은 이유로 일치하지 않는다.

- 통계정보 구성이 실제 데이터를 반영하지 못하거나 없는 경우
- 적절한 인덱스가 존재하지 않거나 부적절한 경우
- 쿼리가 최적화 돼 있지 않는 경우나 잘못 사용된 경우
- 오라클 옵티마이저의 알고리즘이 완벽하지 않다는 현실적인 문제



개발자는 쿼리를 작성할 때 실행계획의 내용을 쿼리에 포함시키는 방법을 사용할 수 있다. 바로 공정쿼리다. 인덱스 생성도에서 테이블 접근 순서 및 인덱스 생성 위치를 알 수 있다. 공정 쿼리 작성법에 의해 작성된 쿼리에서도 테이블 접근 순서 및 인덱스 생성 위치를 쉽게 알 수 있다. 작성된 쿼리에 개발자가 생각하는 실행계획이 내포되어 있어서, 오라클 옵티마이저가 제시하는 실행계획과의 비교 작업이 한결 쉬워진다.

## 바인드 변수와 하드 파싱

대부분 개발자 튜닝 시 실행계획을 상수값으로 테스트하지만 실제로 바인드 변수로 운영되는 경우에는 실행 계획이 다를 수 있다.

```SQL
SELECT * FROM 고객 WHERE 고객명 = '홍길동' --- 상수값
SELECT * FROM 고객 WHERE 고객명 = :NAME   --- 바인드 변수
```

이와 같은 바인드 변수를 사용하면 간혹 실행계획이 다른 경우도 있다. 쿼리가 어떤 방식으로 운영되는지에 따라 실행계획을 구분, 확인해야 한다.

바인드 변수는 하드 파싱을 줄이기 위한 수단으로 사용된다. 오라클  옵티마이저는 상수값이 다르면 서로 다른 쿼리로 인식해 파싱을 새로한다. 특히 실행 횟수가 많고 많은 컬럼의 distinct값이 크다면 하드 파싱이 자주 발생해 전반적으로 비용이 높을것이다. 반면에 바인드 변수를 사용한다면 옵티마이저는 동일한 쿼리로 인식하므로 파싱을 매번 하지 않는다. OLTP 프로그램에서는 DB 성능을 고려해 바인드 변수 사용을 권고한다.