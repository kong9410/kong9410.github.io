---
title: Filter, Interceptor, AOP의 차이점
tags: web spring
---

# Filter, Interceptor, AOP의 차이점

## Filter

- 서블릿 2.3 규약에 새롭게 추가되었음
- HTTP 요청과 응답을 변경할 수 있는 재사용 가능한 코드
- 객체의 형태로 존재
- 클라이언트로 부터 오는 요청과 최종자원(서블릿/JSP 등) 사이에 위치



### Filter Chain

- 여러개의 필터가 존재
- 필터를 거쳐서 나오는 요청은 다음 필터의 요청으로 본다
- 응답도 마찬가지로 필터를 거친 응답이 다음 필터에서 처리된다



## Interceptor

- 컨트롤러에 들어오는 HttpRequest와 응답하는 HttpResponse를 가로채는 역할
- 쿠키, 세션 등의 용도로 사용한다



## AOP

- 관점 지향 프로그래밍을 말함(Aspect Oriented Programming)
- 객체지향 프로그래밍에서 중복부분을 줄이기위해 종단면 관점에서 바라보면서 처리
- 메소드의 실행전, 실행후, 실행중 등 시점에 따라 수행함



## 셋의 차이

| 유형    | Filter                    | Interceptor                     | AOP                 |
| ----- | ------------------------- | ------------------------------- | ------------------- |
| 호출 시점 | DispatcherServlet이 실행되기 전 | DispatcherServlet이 컨트롤러를 호출하기 전 | 특정 메소드 시점 전후에 사용    |
| 구현 방식 | 보통 web.xml에 등록            | servlet-context에 등록             | servlet-context에 등록 |
| 용도    | 인코딩 변환, XSS 방어 등          | 로그인 체크, 권한 체크, 실행시간 계산작업 등      | 로깅, 트랜잭션, 에러처리 등    |



## 출처

[갓대희의 작은공간 :: [Spring\] Filter, Interceptor, AOP 차이 및 정리 (tistory.com)](https://goddaehee.tistory.com/154)

[[Spring\] Interceptor (1) - 개념 및 예제 :: victolee (tistory.com)](https://victorydntmd.tistory.com/176)

[[Spring\]필터(Filter)란 무엇인가 | 두발로걷는개 (twofootdog.github.io)](https://twofootdog.github.io/Spring-%ED%95%84%ED%84%B0(Filter)%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80/)

