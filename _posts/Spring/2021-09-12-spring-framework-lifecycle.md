---
title: 스프링 라이프사이클
tags: java spring
---

![spring-request-lifecycle](https://user-images.githubusercontent.com/37204770/132982003-935cdfce-a7ac-4c09-bcde-1f302e2037e9.jpg)

## MVC 처리 과정

1. Filter를 통해 요청정보를 변경한다
2. Dispatcher Servlet은 요청을 HandlerMapping으로 적절한 요청을 처리할 핸들러를 찾는다
3. DispatcherServlet은 컨트롤러의 비지니스 로직을 실행하는 HandlerAdapter를 실행하는 작업을 한다
4. HandlerAdapter는 컨트롤러의 비지니스 논리 프로세스를 호출한다
5. 컨트롤러는 비지니스 로직을 실행하고 Model에서 처리 결과를 설정, HandlerAdapter에 뷰의 논리적 이름을 반환한다
6. Dispatcher Servlet은 렌더링 프로세스를 반환된 뷰로 디스패치한다
7. 뷰는 모델 데이터를 렌더링하고 응답을 반환한다
8. Filter를 통해 응답정보를 변경한다



### 내부 동작

![3](https://user-images.githubusercontent.com/37204770/132981995-883b9b89-d648-4c74-ac02-d344e5282fa6.png)

1. Locale
   - Locale 결정, 브라우저가 전달해주는 헤더정보로 Locale값을 결정한다
   - `LocaleResolver` 지역 정보를 결정해주는 전략 오브젝트
   - 디폴트인 `AcceptHeaderLocalResolver`는 HTTP 헤더의 정보를 보고 결정
2. RequestContextHolder
   - RequestContextHolder에 요청을 저장
   - Thread Local 객체
   - `RequestContextHolder`는 일반 빈에서 `HttpServletRequest`, `HttpServletResponse`, `HttpSession`등을 사용할 수 있도록 함. 일반 빈에서 사용하게 되면 Web에 종속적일 수 있음
3. FlashMap
   - 스프링3에서 추가된 기능
   - 리다이렉트로 값을 전달할 때 사용되는 기능
   - 리다이렉트 될때 값을 한번 유지시킬 수 있음
   - `FlashMapManager` FlashMap 객체를 조회, 저장을 하기 위한 인터페이스
   - `RedirectAttributes`의 `addFlashAttribute` 메소드를 이용해서 저장
   - 리다이렉트 후 조회를 하면 정보는 바로 삭제됨
4. MultiPart
   - 미디어가 들어오면 Request를 MultiPartResolver가 결정하게된다
5. 핸들러 결정 및 실행



### 내부 동작 - 요청 전달

![5](https://user-images.githubusercontent.com/37204770/132981998-4e5e1b77-4bfe-4f2f-a558-41bf18999014.png)

1. HandlerMapping으로 HandlerExecutionChain 결정
   - HandlerMapping 구현체는 어떤 핸들러가 요청을 처리할지에 대한 정보
   - 디폴트 핸들러매핑은 `BeanNameHandlerMapping`과 `DefaultAnnotationHandlerMapping` 두가지가 있다
2. HandlerExecutionChain
   - HandlerExecutionChain 구현체는 실제로 호출된 핸들러에 대한 참조를 가지고있음. 즉, 무엇이 실행되어야 할지를 알고 있는 객체 핸들러 실행 전과 실행 후에 수행될 `HandlerInterceptor`도 참조하고 있음
3. HandlerExecutionChain을 발견하지 못하면 Http 404 전달
4. HandlerAdapter
   - 실제 핸들러를 실행하는 역할을 담당한다
   - 핸들러 어댑터는 선택된 핸들러를 실행하는 방법과 응답을 ModelAndView로 변화하는 방법에 대해 알고있음
   - 디폴트 어댑터는 `HttpRequestHandlerAdapter`, `SimpleControllerHandlerAdapter`, `AnnotationMethodHandlerAdapter`
   - 어노테이션으로 정의되는 컨트롤러는 `DefaultAnnotationHandlerMapping`의해 핸들러가 결정되고 그에 대응되는 `AnnotationMethodHandlerAdapter`에 의해 호출이 일어남
5. 존재하면 HandlerAdapter 결정
6. HandlerAdapter 발견하지 못하면 ServletException 발생
7. 존재하면 요청처리



### 내부 동작 - 요청 처리

![5](https://user-images.githubusercontent.com/37204770/132981998-4e5e1b77-4bfe-4f2f-a558-41bf18999014.png)

1. 사용 가능한 인터셉터인지 확인
2. 있으면 `preHandler`호출해 처리
3. 핸들러 실행
4. `ModelAndView`를 리턴하는지 확인
   - `Controller`의 처리 결과를 보여줄 view와 view에서 사용할 값을 전달하는 클래스
5. 반환하지 않는다면 인터셉터의 `postHandler`를 호출해 요청 처리
6. `ModelAndView`가 뷰를 갖는지 확인
7. `RequestToViewNameTransfer`
   - 컨트롤러에서 뷰 이름이나 뷰 오브젝트를 제공해주지 않았을 경우 URL과 같은 요청정보를 참고해서 자동으로 뷰 이름을 생성해주는 전략 오브젝트이다. 디폴트는 `DefaultRequestToViewNameTranslator`이다



### 내부 동작 - 예외처리

![6](https://user-images.githubusercontent.com/37204770/132981999-110da339-d6b6-43b2-a944-a7b39b167724.png)

1. 예외 발생
2. `HandlerExceptionResolver`에 문의
   - `DispatcherServlet`이 `DefaultHandlerExceptionResolver`를 등록
   - 예외가 던져졌을 때 어떤 핸들러를 실행할 것인지에 대한 정보 제공
3. `ModelAndView`를 리턴하면 요청처리 재개
4. 리턴하지 않으면 다시 예외를 던진다



### 내부 동작 - 뷰 렌더링 과정

![7](https://user-images.githubusercontent.com/37204770/132982000-63511b9f-76e8-4a3c-933c-b37e39a431e2.png)

1. 뷰 렌더링 요청
2. 뷰가 String을 참조하면 `ViewResolver`로 `view` 구현체를 찾음
   - 컨트롤러가 리턴한 뷰 이름을 참고해서 적절한 오브젝트를 찾아주는 로직을 가진 전략 오브젝트. 뷰의 종류에 따라서 적절한 리졸버를 추가설정 필요
3. `View` 구현체가 존재하지 않으면 `ServletException` 던짐
4. 있으면 `View` 구현체로 렌더링
5. 요청처리 재개



### 내부 동작 - 요청 처리 종료

![8](https://user-images.githubusercontent.com/37204770/132982001-efce9a0e-0fc7-4c40-a992-5673202e9354.png)

1. `HandlerExecutionChain`이 존재하면 인터셉터의 `afterCompletion` 메소드 실행
2. `RequestHandledEvent` 발생
3. 요청 처리됨



# 스프링 MVC 구성 요소

## 1. Filter

- Web Application의 전역적인 로직을 담당
- 전체적인 Filter를 설정하는 곳이다.
- DispatcherServlet에 들어가기 전 Web Application 단에서 실행



## 2. DispatcherServlet

- 프론트 컨트롤러(Front Controller)
- 들어오는 모든 Request를 우선적으로 받아 처리해주는 서블릿
- HandlerMapping에게 Request에 대해 매핑할 Controller 검색을 요청
- HandlerMapping으로부터 Controller 정보를 반환받아 Controller와 매핑



## 3. HandlerMapping

- DispatcherServlet으로부터 검색을 요청받은 Controller를 찾아 정보를 리턴



## 4. HandlerInterceptor

- Request가 Controller에 매핑되기전 앞단에서 부가적인 로직을 추가할 수 있음
- 주로 세션, 쿠키, 권한 인증 로직에 많이 사용된다.



## 5. Controller

- Request와 매핑되는 곳
- Request에 대해 어떤 로직으로 처리할 것인지 결정하고 그에 맞는 Service를 호출
- Service Bean을 스프링 컨테이너로부터 주입받아야 함. Service Bean의 메소드를 호출해야 한다.



## 6. Service

- 데이터 처리 및 가공을 위한 비즈니스 로직을 수행
- Request에 대한 실질적인 로직을 수행하기 때문에 Spring MVC Request Lifecycle의 심장.
- Repository를 통해 DB에 접근하여 CRUD 처리



## 7, Repository(DAO)

- DB에 접근하는 객체



## 8. ViewResolver

- Controller에서 리턴한 View 이름을 DispatcherServlet으로 부터 넘겨받고 해당 View를 렌더링
- 렌더링한 View는 DispatcherServlet으로 리턴, DispatcherServlet에서는 해당 View 화면을 Response



## 출처

[2.2. 봄 MVC 아키텍처 개요 — TERASOLUNA 글로벌 프레임워크 개발 가이드라인 1.0.1.RELEASE 문서 (terasolunaorg.github.io)](https://terasolunaorg.github.io/guideline/1.0.1.RELEASE/en/Overview/SpringMVCOverview.html)

[[Spring\] Spring MVC Request Lifecycle (velog.io)](https://velog.io/@damiano1027/Spring-Spring-MVC-Request-Lifecycle)

[웹 프로그래밍(풀스택) > 2) Spring MVC구성요소-2 : 부스트코스 (boostcourse.org)](https://www.boostcourse.org/web316/lecture/254347#15236)