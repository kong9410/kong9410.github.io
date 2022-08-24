---
title: 오라클 랜덤 함수와 사용자 정의 함수
tags: oracle database
layout: post
description: 오라클에서 기본적으로 제공하는 함수가 있다. 사용자가 필요에 따라 함수를 정의해서 사용할 수 있다.
---

## 오라클 DBMS_RANDOM 패키지

오라클이 제공하는 DBMS_RANDOM 패키지는 랜덤값을 제공하는 함수 기능을 포함하고 있다.

### VALUE

```sql
SELECT DBMS_RANDOM.VALUE(1, 10) FROM DUAL
==============================
7.64451979980338966929483536410663788138
```

소수점 이하 값을 포함하는  1에서 10사이의 값을 반환한다.

```sql
SELECT ROUND(DBMS_RANDOM.VALUE(100000, 999999), 0) FROM DUAL
==============================
965606
```

ROUND 함수를 이용해 소수점 이하 값을 제외한 6자리 숫자를 랜덤하게 리턴하게 할 수 있다.

```sql
SELECT DBMS_RANDOM.STRING('U', 10) FROM DUAL -- 대문자 10자리
SELECT DBMS_RANDOM.STRING('L', 10) FROM DUAL -- 소문자 10자리
SELECT DBMS_RANDOM.STRING('A', 10) FROM DUAL -- 대소문자 10자리
SELECT DBMS_RANDOM.STRING('X', 10) FROM DUAL -- 대문자 및 숫자 10자리
SELECT DBMS_RANDOM.STRING('P', 10) FROM DUAL -- 대소문자 및 특수문자
```

랜덤 문자는 STRING 함수를 사용하면 된다.

## 문자열 역순으로 리턴하는 REVERSE 함수

오라클에서 제공하는 함수 중에서 문자열을 역순으로 리턴하는 함수가 있다.

```sql
SELECT REVERSE('ABCD') FROM DUAL
===================
DCBA
```

문자열의 경우는 정상적으로 리턴하지만 숫자형인 경우 오라클 에러가 발생한다.

```sql
SELECT REVERSE('12345') FROM DUAL -- 54321
SELECT REVERSE(12345) FROM DUAL -- ERROR
```

한글이나 한글 자모의 경우에도 에러가 발생한다.

```sql
SELECT REVERSE('대한민국만세') FROM DUAL -- ERROR
SELECT REVERSE('ㄷㅎㅁㄱㅁㅅ') FROM DUAL -- ERROR
```

결국 오라클에서 기본적으로 제공하는 내장 함수인 REVERSE는 한글을 제외한 문자형에서만 역순으로 반환 가능함을 알 수 있다. MAX 함수인 경우 문자형, 숫자형, 날짜형 모두 가능하지만 REVERSE 함수는 다름을 알 수 있다.

## 사용자 정의 함수 만들어 쓰기

오라클 내장 함수는 오라클에서 자체적으로 제공하는 함수다. 반면 사용자가 필요에 따라 직접 만든 함수는 사용자 정의 함수라 한다. 기본적으로 제공하지 않지만 꼭 필요한 함수라면 개발자가 스스로 만들줄 알아야한다.

사용자 정의 함수는 별도의 실행 엔진에서 구동된다고 한다. 곧 부하가 있다는 얘기다. DB를 안정적으로 유지, 관리해야 하는 DBA 입장에서는 조심스러울 수밖에 없다.

## 사용자 정의 함수 ISNUMERIC

예제를 통해 간단학데 사용자 함수를 만드는 방법을 알아보자.

숫자이면 1을 리턴하고 아니면 0을 리턴하는 함수를 만들어보자.

```sql
CREATE OR REPLACE FUNCTION ISNUMERIC (P_NUM IN VARCHAR2) RETURN NUMBER
AS
    V_NUM NUMBER;
BEGIN
    V_NUM := TO_NUMBER(P_NUM);
    RETURN 1;
EXCEPTION
    WHEN OTHERS THEN
        RETURN 0;
END;
```

