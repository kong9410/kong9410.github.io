---
title: 오라클 힌트절 7가지
tags: database oracle
layout: post
description: 힌트절 7가지를 소개한다.
---

CBO(Cost Based Optimizer) 방식에는 주어진 환경아래 최적의 plan을 제공한다. 그러나 잘못된 sql이나 부정확한 통계정보로 엉뚱한 실행계획을 제공하는 경우도 있다. 이럴 때는 힌트절을 통해 바로 잡을 수 있다.

힌트절은 다음과 같이 주석문 앞에 표시한다.

```sql
SELECT /*+ (힌트절) */
```

주석으로 표시하기 때문에 잘못된 힌트절로 에러를 반환하는일은 없다.

힌트 종류는 매우 많지만 가장 많이 사용하는 7가지만 본다.

**접근 순서를 결정하는 힌트절**

- ORDERED: FROM 절에 나열된 테이블 순서대로 접근한다.
- LEADING: 테이블 접근 순서를 명시적으로 표시한다.

**접근 방법을 결정하는 힌트절**

- USE_NL: NESTED LOOP JOIN 방식으로 조인하도록 유도한다.
- USE_HASH: HASH JOIN 방식으로 조인하도록 유도한다.

**자원 사용을 결정하는 힌트절**

- INDEX: 인덱스를 통한 접근 경로를 유도한다.
- FULL: 테이블을 풀스캔한다.
- PARALLEL: 병렬 처리를 통해 성능을 높인다.



## 접근 순서를 결정하는 힌트절

ORDERED 힌트절과 LEADING 힌트절은 테이블 간 접근 순서를 결정한다.

![column_img_1656.png](https://dataonair.or.kr/publishing/img/knowledge/column_img_1656.png)

위 그림에서는 1번과 3번에만 인덱스가 존재하므로 테이블 접근순서는 고객 -> 주문이 된다.

![column_img_1657.png](https://dataonair.or.kr/publishing/img/knowledge/column_img_1657.png)

마찬가지로 위 그림도 인덱스 위치가 명확하므로  오라클 옵티마이저가 접근 순서를 잘못 판단할 가능성은 거의 없고 2번과 4번 컬럼에만 인덱스가 존재하므로 테이블의 접근 순서는 주문 -> 고객이다.

![column_img_1658.png](https://dataonair.or.kr/publishing/img/knowledge/column_img_1658.png)

그러나 위의 그림과 같이 접근 하는 모든 컬럼에 인덱스가 있다면 테이블 접근 순서가 명확하지 않게 된다. 옵티마이저는 통계정보 기반으로 최적의 방향을 결정하려 할것이다. 그러나 만일 우리가 판단하는 접근 방향과 옵티마이저가 판단하는 접근 방향이 다르다면 우리는 힌트절을 통해 테이블 접근 방향을 변경할 수 있다. ORDERED 힌트절은 FROM 절에 나열된 테이블 순서대로 접근하고자 할 때 사용한다. 

```sql
SELECT /*+ ORDERED */ *
  FROM 고객 A, 주문 B
 WHERE A.고객번호 = B.고객번호
   AND A.고객명 = ?
   AND B.주문일자 = ?
```

ORDERED 힌트절로 인해 옵티마이저는 고객 -> 주문 방향으로 접근을 진행한다. ORDERED 힌트절은 FROM 절에 나열된 테이블 순으로 접근을 유도하지만 LEADING 힌트절은 테이블 접근 순서를 명시적으로 표시할 수 있다. ORDERED 힌트절보다 훨신더 개선된 힌트절인데, LEADING 힌트절은 FROM 절에 종속적이지 않기 때문이다.

```sql
SELECT /*+ LEADING(B A) */ *
  FROM 고객 A, 고객 B
 WHERE A.고객번호 = B.고객번호
   AND A.고객명 = ?
   AND B.주문일자 = ?
```

LEADING(B A)로 인해 주문 -> 고객 방향으로 접근을 수행할 것이다.

![column_img_1661.png](https://dataonair.or.kr/publishing/img/knowledge/column_img_1661.png)

위와 같은 상황에서는 가 -> 나 -> 다 -> 라 혹은 라 -> 다 -> 나 -> 가로 테이블 접근방향을 결정할 수 있다. LEADING 힌트로 명시하면 끝이다.



![column_img_1662.png](https://dataonair.or.kr/publishing/img/knowledge/column_img_1662.png)

위 그림과 같은 경우는 만일 조건 1로 진입할 때 그 다음 접근 순서는 나, 다, 라 테이블 중 선택해야 할 것이다. 만일 조건2로 접근하면 나 다음 가가 될것이고 그다음 다와 라 중에 선택해야 할 것이다. 접근 경로가 많다면 옵티마이저가 잘못된 판단을 할 가능성이 존재한다. 그러한 상황이 발생한다면 LEADING 힌트절을 통해 접근 순서를 변경하면 된다.

테이블 접근 순서 기준은 다음 3가지다

- 진입형 테이블을 결정 : 조건 중에서 조회 범위가 작은 테이블을 우선
- 연결 확장형보다는 연결 축소형 테이블을 우선: 조회 범위가 줄어드는 JOIN을 우선
- OUTER JOIN 보다는 INNER JOIN을 우선: INNER JOIN은 조회 범위 축소가 가능



## 접근 방법을 결정하는 힌트절: USE_NL과 USE_HASH

USE_NL과 USE_HASH는 테이블간 접근방법을 결정하는 힌트절이다.

![column_img_1663.png](https://dataonair.or.kr/publishing/img/knowledge/column_img_1663.png)

1. 고객 테이블에서 고객명이 '홍길동'인 고객을 구한다(선행 테이블 결정)
2. '홍길동' 고객의 수만큼 순차적으로 주문 테이블을 고객번호 컬럼으로 접근한다.(순차적 접근)
3. 주문 테이블에서 주문일자가 '20141201'인 정보만 필터한다.

고객 테이블에서 주문 테이블로 순차적으로 접근한다는 의미다. 진행 방향인 1번 컬럼과 3번 컬럼엔 인덱스가 반드시 존재해야 한다.

![column_img_1664.png](https://dataonair.or.kr/publishing/img/knowledge/column_img_1664.png)

위 그림에서 USE_HASH에 의한 HASH JOIN은 다음과 같다.

1. 조직 테이블에서 '강원사업부'인 조직들을 구한 후, 조인절 컬럼인 조직코드를 해시 함수로 분류한 다음, 해시 테이블을 생성한다. (해시 함수를 이용해 해시 테이블 생성)
2. 집계 테이블에서 집계년월이 '201412'인 자료를 구한 후, 조인절 컬럼인 조직코드를 해시 함수로 변환후 해시 테이블로 순차 접근한다. (해쉬 함수를 통해 해시 테이블을 탐색)

조회 조건 컬럼인 1번 4번 인덱스는 사용되고 있고 2번 3번은 존재하더라도 사용되지 않는다. Hash Join에서는 작은 테이블을 먼저 접근하는 것이 성능 면에서 더 좋다. 해시 테이블 구성 작업에 부하가 많이 발생하기 때문이다. 이후 큰 테이블을 접근해 해시 함수를 이용해 순차적으로 해시 테이블에 접근한다. 이는 대량의 데이터를 처리하는 배치성 프로그램에 적합한 조인 방식이다.

![column_img_1665.png](https://dataonair.or.kr/publishing/img/knowledge/column_img_1665.png)

위 그림은 복잡한 쿼리를 인덱스 생성도로 단순화 한것이다. 만일 옵티마이저가 개발자가 의도한 방향으로 실행계획을 만들지 않는다면 어떤 힌트절을 주어야할까?

- 오라클 옵티마이저 접근순서: 가 -> 다 -> 마 -> 바 -> 라 -> 나 (전구간 USE_NL)
- 개발자 접근순서: 가 -> 라 -> 다 -> 마 -> 바 -> 나 (가~라는 USE_HASH, 나머지는 USE_NL)

오라클 옵티마이저가 제공하는 실행계획에서는 사용하는 인덱스가 가 -> 다 -> 마 -> 바 -> 라 -> 나다. 진행 방향의 목적지 컬럼에 인덱스가 있음을 알 수 있다.

반면 개발자가 생각하는 실행계획에서 사용하는 인덱스는 가 -> 라 -> 다 -> 마 -> 바 -> 나 이다. 힌트절은 다음과 같이 접근 순서와 접근 방법에 대해 복합적으로 적용하면 된다.

```sql
/*+ LEADING(가 라 다 마 바 나) USE_HASH(라) USE_NL(다 마 바 나) */
```

## 자원 사용을 결정하는 힌트절 INDEX, FULL, PARALLEL

![column_img_1666.png](https://dataonair.or.kr/publishing/img/knowledge/column_img_1666.png)

위 그림은 두 개의 조건절에 모두 인덱스가 존재할 때 옵티마이저는 통계정보에 근거한 최소 비용이 소요되는 인덱스를 선택할 것이다. 하지만 실제 정보를 반영하지 못해 옵티마이저가 잘못된 선택을 하던가 우리가 원하는 인덱스가 아닐 때는 힌트절을 명시적으로 인덱스를 지정할 수 있다.

다음과 같은 인덱스 관련 힌트절이 있다

- INDEX_SS : 결합인덱스의 선행 컬럼 조건이 입력되지 않을 때 사용된다. (INDEX SKIP SCAN)
- INDEX_FFS : 인덱스만을 빠르게 전체 스캔한다. (INDEX FAST FULL SCAN)
- INDEX_DESC : 인덱스를 통해 데이터를 역순으로 스캔한다.

![column_img_1667.png](https://dataonair.or.kr/publishing/img/knowledge/column_img_1667.png)

위 그림은 조건절로 구간 조회를 할 수 있다. 테이블 전체 수에 비해 구간 건수가 너무 적다면 인덱스를 통해 접근하는 것이 빠를것이다. 하지만 구간 조회 범위가 넓다면 액세스 부하가 클것이다. 이 때는 인덱스 랜덤 액세스보다 테이블을 직접 풀스캔하는 것이 더 빠를 수 있다.

어떻게 정하면 좋을까? 책의 저자는 1% 범위를 기준으로 한다. 조회 건수가 1% 미만일때 인덱스를 사용하게 하고, 1% 이상일 때 풀스캔 힌트절 사용을 고려한다.

![column_img_1668.png](https://dataonair.or.kr/publishing/img/knowledge/column_img_1668.png)

PARALLEL의 경우는 FULL 힌트절과 같이 사용된다. 병렬 처리를 위한 힌트절이므로 처리 성능은 매우 좋으나 자원을 독점적으로 사용하므로 멀티 유저 환경에서 주의해야한다. 만일 수치를 1로 주면 FULL 힌트절만 작동한다. 수치값을 주지않는다면 사용가능한 모든 자원을 사용하기 때문에 주의해야 한다.

## 배치 튜닝의 마법사 같은 힌트절 삼총사: USE_HASH, FULL, PARALLEL

USE_HASH, FULL, PARALLEL 3가지 힌트절은 배치성 쿼리에서 가장 많이 사용된다. 대용량 데이터 처리와 조회에 빈번하게 사용하는 힌트절이다. 대용량 데이터 처리와 조회에 빈번하게 사용된다.

힌트절을 사용함에 있어서 다음의 3가지로 나누어 사용여부를 검토해야한다.

- USE_HASH 힌트절만 사용해서 조회 가능한지 검토: 적당히 무거운 쿼리에 사용
- 조회 범위가 크다면 FULL 힌트절 추가 사용을 검토: 대개 이 단계에서 튜닝 완료
- 대용량 데이터의 빠른 처리가 요구 될 때 PARALLEL 힌트절 사용: 제한적 사용

![column_img_1669.png](https://dataonair.or.kr/publishing/img/knowledge/column_img_1669.png)