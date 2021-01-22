---
title: SELECT 공략
tags: database
---

# SELECT는 SQL의 심장부

## SELECT 본질

### SELECT 강점

데이터를 얻을 수 있는 유일한 수단이다. 데이터를 가져오는 일을 모두 의지하고 있는 것이다. 관계형 모델에서는 릴레이션이 연산의 단위이며 한 개 또는 여러 개의  릴레이션을 조합해 연산을 수행한다. 그 결과 릴레이션을 가진다.

SQL에서 릴레이션에 대응하는 것은 테이블이다. 그 테이블을 참조할 수 있는 것은 SELECT 뿐이다. 즉 RDB는 릴레이션의 연산에 해당하는 작업을 모두 SELECT로 수행한다.

## SELECT의 기본 구조

```sql
SELECT 칼럼의 목록
FROM 테이블의 목록
WHERE 검색 조건
```

칼럼의 목록은 사영(Projection), 테이블의 목록은 직적(Product) 검색 조건은 제한(Restrict)에 해당하는 작업이다. SELECT는 이와 같은 세 개의 릴레이션 연산을 동시에 수행하게 되어 있다. 논리 평가 순서는 다음과 같다

1. 테이블의 목록
2. 검색 조건
3. 칼럼의 목록

이것은 논리 평가 순서지 반드시 실행 순서가 된다는 것을 의마하는 것은 아니다. RDB 내부적으로 다르게 처리할 수 있으므로 유의하자

# SELECT의 다양한 모습

## 집계함수

### 함수의 유무만으로 의미가 바뀐다

구문이 완전히 같아도 SELECT에 칼럼의 목록 중에 집계함수가 포함돼 있으면 SELECT 결과 전체가 집계 결과가 된다.

```sql
SELECT COUNT(*)
FROM students
WHERE department = '관계형 모델'
```

이렇게 사용할 경우 결과 집합에 포함된 행은 한 개가 된다. 

이처럼 집계함수의 유무만으로 결과 행의 의미에 큰 변화가 생긴다.

### COUNT의 특수성

COUNT 연산시 WHERE 절의 조건에 일치하는 행이 없다면, 결과가 COUNT와 그 이외의 집계함수가 다르다. 예를 들어 존재하지 않는 행에 대한 집계연산은 어떻게 표현되는지 확인해보자

```SQL
SELECT AVG(age) FROM students WHERE grade = 5
```

위의 쿼리의 경우 5학년이 존재하지 않는 경우 AVG(age)의 결과는 NULL이 된다. 다른 집계함수도 결과가 같을 것이다. 그러나 COUNT함수의 경우 '공집합을 평가한 결과가 0이 된다'라는 동작을 한다. 

### GROUP BY를 이용한 집계의 서식

GROUP BY 절이 없다면 집계함수는 테이블 전체의 데이터를 대상으로 집계되지만 어떤 특정 항목별로 집계하고 싶다면 GROUP BY 절을 사용해야 한다.

GROUP BY 절을 사용하면 "그 SELECT가 집계를 나타내는 것이다"라는 사실을 쉽게 알 수 있다.

```sql
SELECT department, COUNT(*)
FROM students
GROUP BY department
```

이 쿼리는 학과별로 학생의 수를 집계하고 있다. 이때 소속 인원이 30명 이하인 학과의 목록과 학생수가 필요하다고 하면 쿼리는 다음과 같다

```SQL
SELECT department, COUNT(*)
FROM students
GROUP BY department
HAVING COUNT(*) <= 30
```

HAVING을 사용했다. 여기서 HAVING을 보면 집계 결과에 대한 연산을 수행한 것을 볼 수 있다.

WHERE와의 차이점은 집계결과에 대한 조건을 접근할 수 있는가 없는가의 차이가 있다. 다르게 말하면 HAVING 절에 사용할 수 있는 조건은 GROUP BY에 지정된 값과 집계함수의 결과 뿐이다. HAVING에 다른 칼럼에 대한 조건도 지정할 수 있지만 가능하면 그 방법은 피하는 것이 좋다.

GROUP BY의 까다로운 점은 WHERE에 해당하는 행이 없을 때 그 항목에 관해서는 결과가 표시되지 않는다는 점이다. 해당하는 행이 없는 항목에 관해서는 집계를 수행할때 서브쿼리를 이용하는 방법이 있다

```sql
SELECT department, (
	SELECT COUNT(*)
    FROM students
    WHERE department = t1.department
    AND grade in (1, 2)
) AS COUNT
FROM (
	SELECT DISTINCT department
    FROM students) t1
WHERE COUNT <= 30
```

WHERE 절의 검색 조건에 일치하지 않으면 행에 대한 집계도 필요할 때는 GROUP BY를 사용할 수 없다는 점이 까다롭다. GROUP BY의 제한 사항을 제대로 파악하고, COUNT() 이외의 집계함수는 NULL 대책도 필요하므로 잊지 않도록 하자.

만일 집계된 것을 수치 순으로 정렬 시키고 싶으면 다음과 같이 ORDER BY를 사용할 수 있다

```SQL
SELECT department, COUNT(*)
FROM students
GROUP BY department
HAVING COUNT(*) <= 30
ORDER BY COUNT(*) ASC
```

GROUP BY, HAVING, ORDER BY 세 개의 절은 GROUP BY, HAVING, ORDER BY 순으로 써야한다. 이는 각절의 논리적인 평가 순서로 되어있다. "GROUP BY 절에 지정된 칼럼의 값별로 집계하고 그 결과를 HAVING 절의 조건으로 필터링한 다음에 ORDER BY 절의 조건으로 정렬한다"라는 의미다.

### 서브쿼리

서브쿼리의 외형은 SELECT이지만 쿼리 결과는 스칼라, 행, 테이블과 같은 형태로 자유롭게 변화할 수 있다.

#### 테이블 서브쿼리

테이블 서브 쿼리는 서브 쿼리의 결과가 테이블 형태이며 세 개의 종류가 있다.

IN, ANY (SOME), ALL 구에 따라서 사용된다. IN 절 등은 대부분 서브쿼리의 결과가 한 열이 되는 경우가 많지만, 여러 칼럼을 한번에 비교할 수 있다.

```sql
SELECT COUNT(*)
FROM course_registration
WHERE (department, course) IN (
    SELECT department, course
    FROM courses
    WHERE minimum_grade >= 2)
```



테이블 서브쿼리는 FROM 절의 서브쿼리로 사용하는 경우다. 이경우 결과를 FROM 절에서 일반 테이블로 다루고 SELECT에 따라서 추가 연산을 하거나 JOIN하는 식으로 사용한다.

```SQL
SELECT AVG(c)
FROM (
	SELECT COUNT(*) AS c
	FROM students
	GROUP BY department)
```



EXISTS 서브쿼리는 IN, ANY, ALL 등과 같은 용도로 사용되지만, 평가되는 것은 서비 쿼리를 평가한 결과, 행이 한 개라도 존재하는지 아닌지다. 1개 이상의 행이 존재한다면 TRUE다.

서브쿼리가 반환한 결과의 내용에 대해서는 아무런 문제가 없지만, 서브쿼리의 결과에 행과 칼럼이 몇 개 포함돼 있을 수 있으므로 구조상 테이블이 된다. WHERE 절에서 사용되는 경우가 많지만, SELECT LIST나 HAVING 절에도 사용할 수 있다.

```sql
SELECT name, department
FROM students
WHERE NOT EXISTS (
	SELECT *
	FROM course_registration
	WHERE student_name = students.name)
```

#### 스칼라 서브쿼리

스칼라 서브쿼리는 스칼라값이 나오는 다양한 곳에서 사용할 수 있다. 예를 들어 스칼라 서브쿼리는 WHERE 절이나 HAVING 절 등에서 스칼라값과 비교하거나 SELECT LIST에서 스칼라값을 구하는 등의 목적으로 사용한다.

```SQL
SELECT name, age
FROM students s1
WHERE age = (
	SELECT MAX(age)
	FROM students s2)
	
SELECT course, COUNT(*) AS COUNT
FROM course_registration
GROUP BY course
HAVING COUNT(*) > (
	SELECT AVG(c)
	FROM (
    	SELECT COUNT(*) AS c
    	FROM course_registration
    	GROUP BY course) AS t)
    	
SELECT (
	SELECT AVG(age) FROM students s
	WHERE s.department = d.department) AS age
	FROM department d
```

#### 행 서브쿼리

행 서브쿼리는 서브쿼리를 평가한 결과가 1행이고 열이 여러 개일 때다. 예를 들어 WHERE 절에서 (COL1, COL2) = (VAL1, VAL2) 같이 비교할 수 있다. 그러나 SELECT LIST 내에 행 서브쿼리를 사용할 수 없다. SELECT LIST 내에는 값이 스칼라여야 한다. 따라서 행 서브쿼리는 반드시 1행만 반환된다고 알고 있을 때 사용 가능하다 이는 스칼라 서브쿼리도 같다.

### 뷰

뷰를 사용하는 목적의 하나는 복잡성을 숨기기 위해서다. 관계형 모델은 일반 테이블과 뷰를 구분하지 않는다. 양쪽 모두 릴레이션을 나타내고 같은 연산을 사용한다. 복잡한 쿼리로 뷰를 정의하면 뷰에 대한 쿼리를 깔끔하게 표현할 수 있다.

그러나 뷰는 백그라운드에서 어떻게 처리가 되는지 보이지 않는다. 간단한 쿼리지만 끝나지 않는 뷰를 열어보면 복잡한 뷰일 경우가 있다. 성능 문제가 발생하므로 뷰를 다룰때 주의해야 한다.

### UNION

sql은 사양 상 결과 집합에 포함된 칼럼 수가 같다면 두 개의 select를 union으로 더할 수 있다. 주의해야 할 점은 UNION으로 더한 두 개의 SELECT는 다른 테이블을 참고하고 있거나 전혀 다른 실행 계획이 있다는 점이다. 두 개의 SELECT에 공통점은 출력 형태가 비슷하다는 것 뿐이다.

## 관계형이 아닌 조작

SELECT를 깊이 이해하기 위해 다른 측면에서 SELECT를 봐야 한다. 그것은 처리가 관계형인가 아닌가다.

SELECT의 기본형은 곱집합, 제한, 사영이라는 릴레이션 연산을 사용하지만 관계형 모델의 법칙을 따르지 않는 다양한 조작도 지원하고 있다.

### 관계형 조작의 복습

앞서 말한 IN, ANY, EXISTS 서브쿼리는 JOIN과 DISTINCT를 사용해 바꿀 수 있다. 이러한 서브쿼리는 결합 중에 사람이 조금 더 이해하기 쉬운 버전이다라고 할 수 있다. 그 외의 서브쿼리는 SELECT로 표현력을 비약적으로 높이는 효과가 있다. FROM절의 서브쿼리가 집약된 경우처럼 서브쿼리는 반드시 릴레이션 연산에 대응하는 것은 아니다

| 릴레이션의 연산 | SELECT로 표현             |
| --------------- | ------------------------- |
| 제한            | WHERE                     |
| 사영            | SELECT LIST               |
| 곱집합          | FROM                      |
| 결합            | FROM                      |
| 곱              | FROM                      |
| 합              | UNION                     |
| 차              | NOT EXIST 서브쿼리, MINUS |
| 속성명 변경     | SELECT LIST               |
| 확장            | SELECT LIST               |

### 정렬(sort)

ORDER BY 절을 이용한 집합의 정렬은 관계형 모델상의 연산은 없다. 관계형 모델은 집합 논리를 바탕으로 하는 모델이며 릴레이션은 집합이다. 중요한 점은 각 요소에 순서는 없다. 그 때문에 SQL에서 ORDER BY는 SELECT 자신이 아닌 커서의 조작이라고 되어 있어 까다롭다.

관계형 모델에서는 적합하지 않지만 응용 프로그램 개발에 있어서 꼭 필요한 조작이다.

### 명시적으로 정의되지 않은 칼럼

RDB에 따라 행번호를 나타내는 ROWNUM 같은 암묵적인 칼럼을 사용할 수 있지만 이를 사용하면 관계형 모델을 벗어나게 되므로 주의해야한다

### 스토어드 함수(사용자 정의 함수)

스토어드 함수의 로직은 절차적으로 작성된 것에 따라서 문제가 발생한다.

SELECT에 스토어드 함수가 포함돼 있으면 절차형으로 처리된다. 이와 같은 경우 옵티마이저는 스토어드 함수의 실행에 드는 비용을 예측할 수 없고 최적화할 수 없다. 스토어드 함수가 포함된 쿼리는 실행 코스트가 높아지기 쉽다. 그런 이유로 스토어드 프로시저로 로직을 구현하는 것도 해서는 안 된다.

SQL은 선언형 프로그래밍 언어다. 절차형 프로그래밍 언어에서 쓰는 루프를 집어 넢으면 로직이 파괴되기 쉽다.

### 관계형이 아닌 조작의 취급법

SELECT는 관계형인 조작과 관계형이 아닌 조작의 복합체다. SQL이 관계형 모델에 완전히 충실한다면 문제는 아주 간단하지만 현실은 그렇지 않다.

RDB를 사용한 응용프로그램 개발에서는 관계형 모델에 따른 조작을 기본으로 하면서도 관계형이 아닌 조작도 필요하다. 따라서 SELECT는 양쪽의 성질을 가진 조작을 겸비하게 되었다.

관계형 조작과 그렇지 않은 조작을 명확하게 구별해야 한다. 다음과 같은 지침에 따라 구축하면 좋다.

- 관계형 모델의 범위에서 가능한 것은 절대로 관계형이 아닌 조작으로 구현하지 않는다.
- 관계형 모델의 범위에서 작성할 수 없다면 DB설계를 검토한다.
- 관계형 모델이 아닌 조작이 필요할 때는 관계형 조작에 대한 로직을 먼저 실시한다.
- 가능한 한 관계형 모델의 범위에서 처리를 작성하거나 관계형 모델에 따라서 먼저 처리되게 하는 것이 중요하다. 옵티마이저에 의한 최적화는 관계형 모델의 범위에서 가장 큰 위력을 발휘하기 때문이다.

## 들여쓰기로 SELECT 문장을 읽기 쉽게

DB 응용 프로그램을 개발하거나 유지보수를 할 때 자신이 작성하지 않은 SELECT를 매일 보게 된다. SELECT는 매우 유연하므로 복잡한 SELECT의 구조를 파악하기는 굉장히 어려운 작업이다. 이 경우에 쉽게 구조를 파악하는 방법은 적절하게 들여쓰기를 하는 것이다.

### 들여쓰기 규칙

- SELECT 절은 들여쓰기하지 않는다.
- UNION 절은 그다음 SELECT와 함께 작성한다
- 칼럼은 한 줄마다 작성한다
- 칼럼의 리스트는 네 개의 공백으로 들여쓰기 한다
- FROM 절, WHERE 절은 두 개의 공백으로 들여쓰기 한다.
- FROM 절의 테이블 목록, WHERE 절의 검색 조건 목록은 한 줄씩 작성한다.
- 서브쿼리의 괄호는 각각 한 줄로 작성한다.

```SQL
SELECT
  department,
  (
    SELECT
      COUNT(*)
    FROM
      students
    WHERE
      department = t1.department
  ) AS count
FROM
  (
    SELECT
      DISTINCT department
    FROM students
  )
```

