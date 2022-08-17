---
title: 놓치기 아까운 오라클의 유용한 기능들
tags: database oracle
layout: post
description: flashback, 스케줄, sample, wm_concat, regexp_substr
---

## FLASHBACK

오라클에서는 갱신된 레코드는 `commit` 전에는  `rollback` 명령어를 사용하면 원복이 가능하지만 `commit`이후에는 불가능했다. 오라클 10g 이후로는 `flashback` 기능을 제공해 `commit` 이전 상태의 레코드도 조회할 수 있게 됐고, 레코드 원복도 가능해졌다. `flashback` 기능은 오라클 서버에 부하를 주므로 설정된 제한 시간만큼 지원된다.

오라클 파라미터 `db_flashback_retention_target`에서 설정된 시간을 확인할 수 있다.

```sql
SELECT *
FROM V$PARAMETER
WHERE name = 'db_flashback_retention_target'
```

조회된 value 값은 분 단위 시간을 의미한다. 제한된 설정 시간 내에 다음과 같은 쿼리를 사용해 과거 시점의 데이터를 조회하거나 원복 할 수 있다.

```sql
-- 1시간 전 시점
SELECT * FROM 고객 AS OF TIMESTAMP (SYSTIMESTAMP - INTERVAL '1' HOUR)

-- 20분 전
SELECT * FROM 고객 AS OF TIMESTAMP (SYSTIMESTAMP - INTERVAL '20' MINUITE)

-- 정해진 시간
SELECT * FROM 고객 AS OF TIMESTAMP TO_DATE('201501101020', 'YYYYMMDDHH24MI')

-- 30분 전
SELECT * FROM 고객 AS OF TIMESTAMP SYSDATE - 30 / (24 * 60)
```

20분 전에서 10분 전 사이에 삭제된 레코드를 원복 하고 싶다면 다음과 같이 사용하면 된다.

```sql
INSERT INTO 고객
SELECT * FROM 고객 AS OF TIMESTAMP (SYSTIMESTAMP - INTERVAL '20' MINUTE)
MINUS
SELECT * FROM 고객 AS OF TIMESTAMP (SYSTIMESTAMP - INTERVAL '10' MINUTE)
```

특정 시점 레코드로 테이블을 생성할 수 있다.

```sql
CREATE TABLE 고객_BACKUP
AS
SELECT * FROM 고객 AS OF TIMESTAMP TO_DATE('202208170000', 'YYYMMDDHH24MI')
```

## 스케줄

스케줄을 쓰기위해서는 SNP_PROCESS가 활성화 되어 있어야한다.

스케줄 수행 시간 단위는 다음과 같이 설정할 수 있다.

```sql
1분 간격 : SYSDATE + 1 / 24 /60
5분 간격 : SYSDATE + 5 / 24/ 60
1시간 간격 : SYSDATE + 1 / 24
매일 오전 1시 : TRUNC(SYSDATE) + 1 + 1 / 24
매일 오후 11시간: TRUNC(SYSDATE) + 23 / 24
매월 첫번째 일요일 01시 : TRUNC(NEXT_DAY(LAST_DAY(SYSDATE), '일')) + 1 / 24
매월 마지막일 오후 11시 : TRUNC(LAST_DAY(SYSDATE)) + 23 / 24
```

트래픽을 집계하는 `P_네트워크` 프로시저가 이미 생성되어 있다고 가정했을 때 다음과 같이 스케줄을 생성할 수 있다.

```SQL
DECLARE V_JOB_NO NUMBER; -- JOB_NO 1: 네트워크 스케줄 JOB 번호
BEGIN DBMS_JOB.SUBMIT(V_JOB_NO, 'P_네트워크;', SYSDATE, 'SYSDATE + 5 / 24 / 60');
END;
```

`P_동기화` 프로시저가 있다고 가정할 때 매일 오전 1시에 스케줄 작업을 진행한다면 다음과 같이 생성할 수 있다.

```sql
DECLARE V_JOB_NO NUMBER; -- JOB_NO 2: 동기화 스케줄 JOB 번호
BEGIN DBMS_JOB.SUBMIT(V_JOB_NO, 'P_동기화;', SYSDATE, 'TRUNC(SYSDATE) + 1 + 1 / 24');
END;
```

USER_JOBS 테이블에서 생성된 스케줄의 JOB번호를 구해 다음 명령어로 스케줄을 구동한다.

```sql
EXECUTE DBMS_JOB.RUN(1); -- JOB_NO 1 네트워크 스케줄 실행
EXECUTE DBMS_JOB.RUN(2); -- JOB_NO 2 동기화 스케줄 실행
```

스케줄을 중단하려면 다음과 같이 하면 된다.

```sql
EXECUTE DBMS_JOB.BROKEN(2, TRUE);
```

삭제하려면 다음과 같이 한다.

```sql
EXECUTE DBMS_JOB.REMOVE(2);
```

## SAMPLE, SAMPLE_SCAN

ORACLE 8.1 이후 부터는 샘플스캔 방식을 추가로 제공하고 있다. 샘플 스캔을 사용하면 데이터를 랜덤하게 샘플링할 수 있다.

```sql
SELECT * FROM 고객 SAMPLE BLOCK(10) WHERE 지역 = '인제' ORDER BY 고객명
```

SAMPLE 구간에 사용 가능한 값은 0에서 100사이다. 0.000001 보다 크거나 같고 100보다 작아야한다. `SAMPLE_BLOCK(10)`은 블록 단위로 주어진 값의 비율만큼 읽어 오는 것을 의미한다.

```sql
SELECT * FROM 고객 SAMPLE(10) WHERE 지역 = '인제' ORDER BY 고객명
```

`SAMPLE(10)`은 레코드 단위로 주어진 값의 비율만큼 읽어 오는 것을 의미한다.  일반적인 실행계획에서 Cost 값은 SAMPLE보다 SAMPLE BLOCK이 더 낮다. Get Block 값도 SAMPLE보다 SAMPLE BLOCK이 더 낮다. 두 가지 중 비용 측면을 더 고려해 사용한다면 SAMPLE BLOCK 사용을 권한다.

규모가 작은 테이블보다는 큰 테이블에서 더 정확한 샘플링을 할 수 있다. 작은 테이블에서의 사용은 무의미 할 수 있다.

```sql
SELECT COUNT(*) * 100 FROM 고객 SAMPLE BLOCK(1)
```

1% 샘플링을 구해 카운터 값에 100을 해서 전체 레코드와 유사한 결과값을 얻어낼 수 있다.

## WM_CONCAT

`WM_CONCAT`은 종으로 된 컬럼 값을 횡으로 구현하는 기능을 가지고 있다.

| 번호   | 국가   | 도시   | 인구(단위: 만) |
| ---- | ---- | ---- | --------- |
| 1    | 한국   | 서울   | 1010      |
| 2    | 중국   | 베이징  | 1961      |
| 3    | 한국   | 부산   | 351       |
| 4    | 일본   | 도쿄   | 1329      |
| 5    | 한국   | 안동   | 16        |

```sql
SELECT 국가, WM_CONCAT(도시) AS 도시들, SUM(인구) AS 총인구
FROM 도시인구현황
WHERE 인구 > 10
GROUP BY 국가
ORDER BY 총인구 DESC
```

RESULT:

| 국가   | 도시들        | 총 인구(단위: 만) |
| ---- | ---------- | ----------- |
| 중국   | 베이징,       | 1961        |
| 한국   | 서울, 부산, 안동 | 1377        |
| 일본   | 도쿄         | 1329        |

## REGEXP_SUBSTR

이 함수는 횡으로 된 값을 종으로 구현할 수 있다.

| 번호   | 이름   | 취미리스트  | 우수고객 |
| ---- | ---- | ------ | ---- |
| 1    | 심인술  | 축구,영화  | Y    |
| 2    | 김윤호  | 자전거,낚시 | N    |
| 3    | 김의석  | 여행,스키  | Y    |

```sql
SELECT 이름, REGEXP_SUBSTR(취미리스트, '[^,]+', 1, LEVEL) AS 취미
FROM 고객
WHERE 우수고객 = 'Y'
CONNECT BY REGEXP_SUBSTR(취미리스트, '[^,]+', 1, LEVEL) IS NOT NULL
GROUP BY 이름, 취미리스트, LEVEL
ORDER BY 이름, 취미
```

RESULT:

| 이름   | 취미   |
| ---- | ---- |
| 김의석  | 스키   |
| 김의석  | 여행   |
| 심인술  | 영화   |
| 심인술  | 축구   |

REGEXP_SUBSTR 함수는 횡을 종으로 구현하는 용도로도 사용하지만 일반적으로 다음과 같이 문자열을 분리하는 용도로 많이 사용된다.

| 번호   | 이름   | 전화번호        | 주소                |
| ---- | ---- | ----------- | ----------------- |
| 1    | 김상미  | 02-274-3328 | 서울 동대문구 신설동 123번지 |
| 2    | 이도윤  | 02-272-2723 | 서울 강동구 강일동 272번지  |
| 3    | 허은미  | 02-392-8989 | 서울 서초구 우면동 237번지  |

```sql
SELECT 이름
, REGEXP_SUBSTR(전화번호, '[^-]+', 1, 1) AS 지역
, REGEXP_SUBSTR(전화번호, '[^-]+', 1, 2) AS 국번
, REGEXP_SUBSTR(전화번호, '[^-]+', 1, 3) AS 전화번호
, REGEXP_SUBSTR(주소, '[^ ]+', 1, 1) AS 시도명
, REGEXP_SUBSTR(주소, '[^ ]+', 1, 2) AS 시군구명
, REGEXP_SUBSTR(주소, '[^ ]+', 1, 3) AS 읍면도명
FROM 고객
```

RESULT:

| 이름   | 지역   | 국번   | 전화번호 | 시도명  | 시군구명 | 읍면도명 |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 김상미  | 02   | 274  | 3328 | 서울   | 동대문구 | 신설동  |
| 이도윤  | 02   | 272  | 2723 | 서울   | 강동구  | 강일동  |
| 허은미  | 02   | 392  | 8989 | 서울   | 서초구  | 우면동  |

