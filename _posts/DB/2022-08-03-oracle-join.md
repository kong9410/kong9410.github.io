---
title: 그림으로 풀어보는 오라클의 조인 방식
tags: database oracle
layout: post
description: 테이블 간에 어떤 방식으로 접근하는가에 대한 조인을 알아본다.
---

오라클에서 조인방식은 크게 3가지로 나뉜다.

- Nested Loop Join: 순차적 루프에 의한 접근 방식
- Sort Merge Join: 정렬을 통한 접근 방식
- Hash Join: 해시 함수를 이용한 접근 방식

Nested Loop Join은 택시와 같다. 소량의 데이터 처리에 적합한 조인방식이다.

Hash Join 방식은 버스와 같다. 대량의 데이터 처리에 효율적인 조인 방식이다.

Sort Merge Join은 거의 사용하지 않으므로 논외로 한다.

## Nested Loop Join

OLTP(Online Transaction Processing) 쿼리에서 가장 일반적이고 흔한 조인 방식이다. 소량의 데이터를 처리하거나 부분범위 처리에 적합한다. NL조인의 방식은 다음 For문으로 이해하면 이해하기 쉽다.

>FOR 고객.성명 = '홍길동' LOOP
>
>​    FOR 고객.주문번호 = 주문.주문번호 LOOP
>
>​        IF 주문.주문일자 = '20141201'
>
>​            ...
>
>​        END IF
>
>​    END LOOP
>
>END LOOP

NL 조인은 이러한 LOOP 문의 수행 방법과 동일한 것으로 테이블 간 조인을 순차적으로 수행한다. 테이블 간 접근 순서가 중요하다. 선행 테이블의 처리 범위가 작아야 하고, 조인절의 목적지 컬럼에 반드시 인덱스가 존재해야 한다.

```sql
SELECT *
FROM 고객 A, 주문 B
WHERE A.고객번호 = B.고객번호
AND A.고객명 = '홍길동'
AND B.주문일자 = '20141201'
```

NL 조인 처리 순서는 다음과 같다.

- 고객 테이블에서 이름이 '홍길동'인 고객을 구한다.(선행 테이블 결정)
- '홍길동'고객의 수만큼 순차적으로 주문 테이블을 고객번호 컬럼으로 접근한다. (순차적 접근)
- 주문 테이블에서 주문일자가 '20141201'인 정보만 필터한다.

## Sort Merge Join

성능이 더 좋은 Hash Join이 있어서 접할 가능성이 거의 없다. 이 조인 방식은 조인절에 인덱스가 없을 때 자주 발생한다. 소트가 발생하므로 대상 건수가 많을수록 소트 부하가 올라간다. 따라서 성능이 저하될 수 있다.

![sortmergejoin](https://user-images.githubusercontent.com/37204770/182624942-c1aee57b-21df-4549-8ecd-313c87a8975d.png)

소트 조인 순서는 다음과 같다.

- 고객 테이블에서 이름이 '홍길동'인 고객을 구한 후 고객번호 순으로 정렬한다. (sort)
- 주문 테이블에서 주문일자가 '20141201'인 주문을 구한 후 고객번호 순으로 정렬한다 (sort)
- 정렬된 고객 정보와 정렬된 주문 정보를 고객번호로 조인한다(merge)

## Hash Join

Hash Join은 대량의 데이터 처리에 유리하다. 메모리에 해시 테이블을 생성하고, 해시 함수를 이용해 연산 조인을 함에 따라 CPU 사용이 증가할 수 있다. 따라서 조회 빈도가 높은 온라인 프로그램에는 적합하지 않는 조인 방식이다.

![hashJoin](https://user-images.githubusercontent.com/37204770/182628359-b4a86c6c-6bfa-4fdb-88f2-8c6c38de5ad0.png)

위 그림은 1000쌍의 부부가 흩어져 있는 체육관에서 각자 배우자를 찾는 방법을 Hash Join으로 설명한 것이다. 규칙없이 배우자를 찾고자 한다면 많은 시간이 소요될 것이다. Hash Join은 그러한 시간을 줄일 수 있다. 남자들이 본인 성씨 구역으로 이동하면 여자들이 남자들의 성씨 구역으로 가서 남편을 찾는다

김씨 성을 가진 남편을 둔 여자들은 다른 여자들 보다 찾는 시간이 더 걸릴 것이다. 마찬가지로 Hash Join에서도 하나의 버킷(구역)에 많은 해시 키(김씨들)가 존재할 때, 그만큼 많이 액세스를 해야하므로 처리 시간이 늘어나서 성능이 떨어진다.

Hash Join 처리 순서는 다음과 같다.

- 조직 테이블에서 사업부가 '강원사업부'인 조직들을 구한 후, 조인절 컬럼인 조직코드를 해시 함수로 분류한다음, 해시 테이블을 생성한다. (해시 함수를 이용해 해시 테이블 생성)
- 집계 테이블에서 처리년월이 '201412'인 자료를 구한 후, 조인절 컬럼인 조직코드를 해시 함수로 변환 후 해시 테이블로 순차적으로 접근한다. (해시 함수를 통해 해시 테이블 탐색)

Hash Join은 작은 테이블을 먼저 접근하는 것이 성능 면에서 더 좋다. 해시 테이블 구성 작업에 부하가 많이 발생하기 때문이다. 작은 테이블에 접근해 해시 함수로 해시 테이블을 생성하고, 이후 큰 테이블에 접근해 해시 함수를 통해 순차적으로 해시 테이블로 접근한다. 이러한 조인 방식은 대량의 데이터를 다루는 배치성 프로그램에 유용하게 사용된다.

## 오라클 조인 방식 특징 비교

| Nested Loop Join | Sort Merge Join  | Hash Join      |
| ---------------- | ---------------- | -------------- |
| 부분 범위 처리에 적합     | 전체 범위 처리에 적합     | 전체 범위 처리에 적합   |
| 소량 데이터 처리에 적합    | 대량 데이터 처리에 적합    | 대량 데이터 처리에 적합  |
| 온라인 프로그램에 적합     | 사용하지 않는 것이 최선    | 배치 프로그램에 적합    |
| 일반적으로 흔하게 발생     | 조인절에 인덱스 없을 때 발생 | 대량 집계 작업 때 발생  |
| 온라인 쿼리 약 90% 분포  | 거의 볼 수 없음        | 배치 쿼리 약 50% 분포 |
| 테이블 접근 순서가 중요    | 소트 회피 가능 테이블 선행  | 작은 테이블로 해시 구성  |
| 조인절에 인덱스 필수      | 소트 부하가 가장 큼      | 메모리 크기에 영향     |
| 순차적 접근           | 소트 + 머지 접근       | 해시 함수 이용한 접근   |

## 조인 방식과 조인 순서 결정하기

주어진 작업의 성격과 DB 환경에 따라 조인 방식을 결정해야 한다. 조인 방식을 결정했다면 성능에 영향을 미치는 조인 순서도 결정해야 한다. 이런일은 주어진 정보를 바탕으로 직관적이고 종합적으로 판단해 능동적으로 수행해야 한다.