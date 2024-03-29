---
title: 개발자들이 자주 접하는 오라클 에러 메시지
tags: oracle database
layout: post
description: 개발 과정에서 가장 빈번하게 일어나는 오라클 에러 메시지에 대한 내용을 정리
---

## ORA-00001: 유일성 제약조건에 위배됩니다.

가장 자주 접하는 에러다. PK나 UNIQUE INDEX가 있을때 중복되도록 INSERT하게 되면 발생한다. PK컬럼이나 UNIQUE INDEX 구성 컬럼은 업데이트 되지 않도록 하는 것을 원칙으로 정하고 쿼리 구분에서 아예 배제하는 것도 한 방편이다.

## ORA-00942: 테이블 또는 뷰가 존재하지 않습니다

개발자에 오타에 의한 경우도 있지만 실제 테이블이 생성됐는지 확인이 필요하다. 개발과 운영이 따로 관리하는 환경이라면 실제 해당 테이블 생성 유무를 착각할 수도 있다. 또한 권한이 없는 경우에도 발생한다.

대소문자 사용에 따른 문제일 수도 있다. 오라클은 테이블 생성 시 대소문자 구분은 없으며 자동으로 대문자로 생성되지만 따옴표로 감싸서 소문자로 생성 시 테이블 명은 소문자로 생성된다.

## ORA-00904: 열명이 부적합합니다

존재하지 않는 컬럼명을 쿼리 구문에 사용할 경우에 발생하는 에러 메시지다. 대부분은 오타가 원인이다. 간혹 컬럼을 실제로 존재하나 SELECT 절에 없는 컬럼을 ORDER BY 절에 사용해서 발생하는 경우도 있으므로 쿼리 작성 시 유의해야 한다.

## ORA-01017: 유효하지 않는 사용자/패스워드에 의한 접근을 제한합니다

오라클 접속 시 사용자ID나 패스워드가 일치하지 않아서 발생한다. 이런 경우가 아닌 대소문자 사용에 따라서 발생하기도 한다. 오라클 11g부터 대소문자를 구분하기 때문이다. 오라클 대소문자 구분 설정을 해제할 수도 있지만 보안상 이유때문에 통상적으로는 해제하지 않고 대소문자를 구분해 사용한다.

간혹 TNS(Transparent Network Substrate) 정보가 틀린 경우에도 발생하는데 이런 경우 에러 원인을 찾기 힘들다.

## ORA-01722: 수치가 부적합합니다

INSERT 혹은 UPDATE 시에 컬럼의 타입에 맞지 않는 값을 입력할 때 발생한다. 숫자 컬럼에 문자값을 입력하는 경우가 그러하다. 조건절에서도 마찬가지로 적용이 된다.

```sql
SELECT * FROM 상품
WHERE SUBSTR(상품코드, 1, 1) = 1
```

 위와 같이 사용시 상품코드가 문자열이라면 오류가 발생한다. 단, 상품코드가 숫자로 구성돼 있을 때는 에러가 발생하지 않는다.

## ORA-01555: 스냅샷이 너무 오래 됐습니다(롤백 세그먼트가 너무 작습니다)

사용자가 필요로 하는 롤백 세그먼트의 정보가 다른 트랜잭션에 의해 오버라이트돼 존재하지 않을 때 발생한다.

롤백 세그먼트가 너무 작다는 의미인데 가장 쉬운 방법은 롤백 세그먼트를 크게하는 것이다. 정말 작다면 늘리는게 맞는 방법이겠지만 다른 방법으로 조치가 가능하다.

대용량 처리시 빈번한 COMMIT 사용을 자제하거나, 처리를 한가한 시간대로 돌리는 것으로도 충분한 효과를 낼 수 있다. 이러한 에러메시지는 대부분 잘못된 무거운 쿼리에 기인하는 경우가 많다. 무거운 배치 쿼리의 실행에서 자주 발생한다. 이러한 경우에는 크기 조정보다는 튜닝을 하는 것이 더 우선책이 됄 수 있다.

## ORA-00911: 문자가 부적합합니다

쿼리 구문을 잘못 작성해서 발생하는 에러라는 생각이 들 수 있다. 쿼리 끝에 세미콜론을 사용해서 발생하는 경우도 있다.

## ORA-12541: 리스너가 존재하지 않습니다

오라클 리스너는 네트워크를 통해 클라이언트에서 오라클 서버로 접근하는 것을 관리하는 기능을 수행한다. 원격 데이터베이스 서버에 접근하기 위해서는 원격 서버에 리스너가 구동돼 있어야 한다. 주로 리스너가 구동돼 있지 않았을 때에 발생하는 에러메시지다.

다음 명령어로 리스너를 구동한다.

```sql
> Lsnrctl
> status
> stop
> start
```

리스너 로그 파일이 너무 커져서 문제가 발생하는 경우도 있다. 이러한 경우 리스너 로그 파일을 옮기고 새로 생성하거나 아예 만들지 않게 설정한다.

## ORA-03113: 통신 채널에 EOF가 있습니다

서버의 고장이나 네트워크가 불안정할 때 주로 발생하는데, 다량의 데이터를 INSERT하거나 UPDATE할 때 발생한다. 이 문제의 원인은 너무 포괄적이라 적절하게 대처하기 쉽지 않다.

이러한 에러를 만나면 네트워크 상태를 점검하기도 하고 방화벽을 의심해 보기도 한다. 아예 DB를 재가동해 해결하기도 한다. 만일 오렌지나 토드와 같은 툴에서는 잘 되는데 프로그램에서 잘 안된다면 쿼리의 길이를 줄여서 테스트를 해볼 필요도 있다. 시간이 지나면 자연스레 해결되는 경우도 있다.

## ORA-01476: 제수가 0 입니다

쿼리 구문의 나눗셈에서 분모가 0일 때 발생한다.

```SQL
select 0 / 1 from dual -- no error
select 1 / 0 from dual -- error occurred
```

해결방법은 다음과 같이 하면된다.

```sql
select case when 분모 = 0 then 0 else 분자 / 분모 end from ...
select decode(분모, 0, 0, 분자/분모) from ...
select nvl(분자 / decode(분모, 0, null, 분모), 0) from ...
```

