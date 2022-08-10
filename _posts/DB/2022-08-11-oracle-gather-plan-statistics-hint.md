---
title: GATHER_PLAN_STATISTICS 힌트절
tags: database oracle
layout: post
description: gather_plan_statistics에 대해 알아보자
---

오라클 10g 부터 GATHER_PLAN_STATISTICS 힌트를 이용하면 SQL Trace를 수행하지 않고도 쿼리 실행계획을 단계별로 Get Block을 알 수 있다. 실행할 쿼리에 다음과 같이 `GATHER_PLAN_STATISTICS` 힌트절을 추가해 사용한다.

```sql
SELECT /*+ GATHER_PLAN_STATISTICS */ *
FROM 인사
WHERE 사용자명 = '이슬기';
```

앞의 대상 쿼리를 실행한 후에 다음 분석 쿼리를 곧바로 수행해야 한다.

```sql
SELECT *
FROM TABLE(dbms_xplan.display_cursor(null, null, 'ALLSTATS LAST'));
```

GATHER_PLAN_STATISTICS 힌트절을 추가해 실행한 가장 최근의 쿼리에 대한 수행 정보를 보여주는 분석 쿼리이다. 만일 'User has no SELECT privilege on V$SESSION'과 같은 메시지가 뜬다면 접근 계정에 대한 권한이 없으므로 DBA에게 요청해 권한을 부여 받아야 한다.

![column_img_1918.jpg](https://dataonair.or.kr/publishing/img/knowledge/column_img_1918.jpg)

각 헤더의 부분에 대해 알아보자

### Id, Operation, Name

자원에 대한 접근 순서와 접근 방법을 나타낸다. 접근 순서를 변경할 수 있는 힌트절은 ORDERED와 LEADING이 있고 접근 방법을 변경할 수 있는 힌트절은 USE_NL, USE_HASH, USE_MERGE가 있다.

### Starts

오퍼레이션을 수행한 횟수를 의미한다. Starts * E-Rows의 값이 A-Rows 값과 비슷하다면, 통계정보의 예측 로우 수와 실제 실행 결과에 따른 실제 로우 수가 유사함을 알 수 있다. 만일 값에 큰 차이가 있다면 통계정보가 실제 정보를 제대로 반영하지 못했다고 생각할 수가 있다. 이로 인해 오라클의 옵티마이저가 잘못된 실행계획을 수립할 수도 있음을 염두해 둬야 한다.

### E-Rows(Estimated Rows)

통계정보에 근거한 예측 로우 수를 의미한다. 통계정보를 갱신할 수록 매번 다른 수있다. 통계정보를 대부분 수시로 갱신하지 않으므로 큰 의미를 둘 필요는 없지만 E-Rows와 A-Rows값이 현저하게 차이 난다면 잘못된 plan을 세울수도 있음을 인지해야 하고 통계정보 생성을 검토해 보아야 한다.

### A-Rows(Actual Rows)

실행 결과에 따른 실제 로우 수를 의미한다. A-Rows에서 중요한 여러 정보를 추정할 수 있다.

### A-Time(Actual Elapsed Time)

쿼리 실행 결과에 따른 실제 수행 시간을 의미한다. 실행 시점의 여러 상황이 늘 가변적이고 메모리에 올라온 블록 수에 따라서 수행시간이 달라지므로 큰 의미를 둘 필요는 없다.

### Buffers(Logical Reads)

논리적인 Get Block 수를 의미한다. 오라클 옵티마이저가 일한 총량을 의미하므로 튜닝을 할때 중요한 요소 중 하나다.

### Reads(Physical Reads)

물리적인 Get Block 수를 의미한다. 동일한 쿼리를 여러 번 수행할 때 처음에는 값이 있으나, 처음이 아닌 경우에는 값이 0인 것을 보면 알 수 있듯이 메모리에서 읽어온 블록은 제외된다. 해당 값에 큰 의미를 둘 필요는 없다.

앞 헤더에서 튜닝 시 가장 중요하게 활용되는 부분은 Buffers와 A-Rows다. Buffers 값을 통해 Get Block의 총량을 알 수 있고 A-Rows를 통해 실행계획 단계별로 실제 로우 수를 알 수 있기 때문이다. 이제 동일 쿼리를 접근 순서를 달리해 실행 후 분석 쿼리를 성능상의 차이를 살펴보도록 한다.

```sql
SELECT /*+ GATHER_PLAN_STATISTICS LEADING(A B) */ *
FROM 인사 A, 실적 B
WHERE A.인사번호 = B.영업자번호
AND A.사용자명 = '이슬기'
AND B.영업일자 = '20150223'
```

![column_img_1919.jpg](https://dataonair.or.kr/publishing/img/knowledge/column_img_1919.jpg)

```sql
SELECT /*+ GATHER_PLAN_STATISTICS LEADING(B A) */ *
FROM 인사 A, 실적 B
WHERE A.인사번호 = B.영업자번호
AND A.사용자명 = '이슬기'
AND B.영업일자 = '20150223'
```

![column_img_1920.jpg](https://dataonair.or.kr/publishing/img/knowledge/column_img_1920.jpg)

동일 쿼리에 다른 힌트값을 줘서 접근 순서를 달리했다. 이를 실행 한 결과 다음과 같은 Buffers 값을 얻었다.

- 인사 -> 실적 : 14 Buffers
- 실적 -> 인사 : 393 Buffers

같은 쿼리라도 어떤 순서대로 테이블을 접근하느냐에 따라 일량이 28배 가량 차이가 났음을 알 수 있다.

이번에는 접근 방법을 달리해본다

```sql
SELECT /*+ GATHER_PLAN_STATISTICS LEADING(A B) USE_HASH(B) */ *
FROM 인사 A, 실적 B
WHERE A.인사번호 = B.영업자번호
AND A.사용자명 = '이슬기'
AND B.영업일자 = '20150223'
```

![column_img_1922.jpg](https://dataonair.or.kr/publishing/img/knowledge/column_img_1922.jpg)

다음과 같은 Buffers값을 얻었다.

- 인사 -> 실적, NL : 14 Buffers
- 실적 -> 인사, NL: 393 Buffers
- 인사 -> 실적, Hash: 29 Buffers

접근 방법에 따라서도 일량이 서로 다름을 알수가 있다.