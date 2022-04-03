---
title: 스프링 application context
tags: spring java
layout: post
---

## 원문 블로그

> [[Spring\] SpringBoot 소스 코드 분석하기, 애플리케이션 컨텍스트(Application Context)와 빈팩토리(BeanFactory) - (2) - MangKyu's Diary (tistory.com)](https://mangkyu.tistory.com/210)

스프링 Context 뿐만아니라 스프링 부트 어플리케이션 실행과정까지 상세히 분석을 해놓은 블로그다.

### 스프링 컨테이너

스프링 프레임워크는 스프링의 빈을 생성하고 관리하는 컨테이너를 가지고 있다. 이를 통해 IOC나 AOP에 대해서 관리한다.

### Bean Factory

스프링 설정파일에 등록된 Bean 객체를 생성하고 관리하는 기본적인 기능을 제공한다. 클라이언트 요청에 의해 Bean 객체가 사용되는 시점(Lazy Loading)에 객체를 생성한다.

### Application Context

BeanFactory를 상속받고 있다. Bean 객체를 생성하고 관리하는 기능을 가지고 있다. 거기에 더해 트랜잭션 관리, 메시지 기반 다국어 처리, AOP 처리 등 DI(Dependency Injection)과 IoC(Inverse of Conversion)외에 많은 부분을 지원하고 있다.

컨테이너가 구동되는 시점에 객체를 생성하는 Pre-Loading 방식을 사용하고 있다.

### Bean 요청 시 처리 과정

1. ApplicationContext는 @Configuration이 붙은 클래스들을 설정 저보로 등록해두고 @Bean이 붙은 메소드의 이름으로 빈 목록을 생성한다.
2. 클라이언트가 해당 Bean을 요청한다.
3. ApplicationContext는 자신의 Bean 목록에서 요청한 이름이 있는지 찾는다.
4. ApplicationContext는 설정 클래스로부터 빈 생성을 요청하고 생성된 빈을 돌려준다.

### Spring Context 장점

기존 싱글톤 단점

- private 생성자를 가지고 있어 상속이 불가능하다.
- 테스트하기 힘들다
- 서버 환경에서는 싱글톤이 1개만 생성됨을 보장하지 못한다.
- 전역 상태를 만들 수 있기 때문에 객체지향적이지 못한다.

스프링 싱글톤 장점

- 싱글톤 형태의 오브젝트를 만들고 관리하는 기능을 제공하는 싱글톤 레지스트리가 있다.
- static 메소드나 private 생성자 등을 사용하지 않아 객체지향적 개발을 할 수 있다.
- 테스트를 하기 편리하다.

### Application Context 종류

SpringBoot를 이용한다면 애플리케이션 종류에 따라 각기 다른 종류의 ApplicationContext가 내부에서 만들어진다.

- AnnotationConfigApplicationContext
  - 웹서버 없음
- AnnotationConfigServletWebServerApplicationContext
  - 톰캣
- AnnotationConfigReactiveWebServerApplicationContext
  - Reactor Netty

일반적으로 applicationContext 클래스는 spring-context 프로젝트에 존재하고 spring-core나 spring-bean 또는 spring-aop를 추가하면 같이 불러와진다. `AnnotationConfigServletWebServerApplicationContext` 이나 `AnnotationConfigReactiveWebServerApplicationContext`는 Spring Boot에서 추가된 클래스이므로 spring-boot-starter-web 또는 spring-boot-starter-webflux 같은 spring-boot 의존성을 추가해주어야 한다.

### DI 컨테이너와 ApplicationContext

애플리케이션 컨텍스트는 애플리케이션을 실행하기 위한 환경이며 동시에 DI 컨테이너라 불린다. ApplicationContext 상위에 Bean들을 생성하는 BeanFactory 인터페이스를 부모로 상속받고 있기 때문이다.

BeanFactory는 1개의 Bean을 찾기 위한 메소드를 가지고 있다. 스프링은 동일한 타입의 Bean이 여러개 존재할 때도 List로 Bean을 찾아 주입한다. 이는 ListableBeanFactory로 Bean 리스트를 처리할 수 있는 퍼블릭 인터페이스를 구현하기 때문이다.

ApplicationContext가 BeanFactory를 바로 상속 받는 것이 아닌 자식 인터페이스 ListableBeanFactory, HierachicalBeanFactory를 통해 상속받는다. HierarchicalBeanFactory는 여러 BeanFactory들 간의 계층 관계를 설정하기 위한 퍼블릭 인터페이스를 갖는다.

@Autowired를 처리하기 위한 AutowireCapableBeanFactory 등도 있다. ApplicationContext의 인터페이스를 보면 AutowireCapableBeanFactory를 상속받지는 않고 합성(Has-A) 관계를 가지고있다.

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory, MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
    /* ... */
    
    AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;
}
```

Spring Context가 BeanFactory, ListableBeanFactory, HierarchicalBeanFactory를 상속받아 applictionContext를 통해 Bean을 찾을 수 있다는 것을 알 수 있다.

그러나 Spring Bean들이 진짜 애플리케이션 컨텍스트에서 관리되는 것이 아니다. 스프링 부트가 만들어내는 3가지 Application Context는 모두 GenericApplicationContext라는 클래스를 부모로 가지고있다.

```java
public class GenericApplicationContext extends AbstractApplicationContext implements BeanDefinitionRegistry {
    private final DefaultListableBeanFactory beanFactory;
    
    public GenericApplicationContext() {
        this.beanFactory = new DefaultListableBeanFactory();
    }
    
    public GenericApplicationContext(DefaultListableBeanFActory beanFactory) {
        Assert.notNull(beanFactory, "BeanFactory must not be null");
        this.beanFactory = beanFactory;
    }
    
    /*...*/
}
```

해당 클래스를 살펴보면 진짜 Bean들을 등록하여 찾아서 관리해주는 DefaultListableBeanFactory를 생성하고 있음을 확인할 수 있다.

빈들을 관리하는 BeanFactory 구현체인 DefaultListableBeanFactory를 합성(Has-A)  관계로 내부에 가지고 있고 빈 처리 요청이 들어오면 BeanFactory로 이러한 이러한 요청을 위임하여 처리하게 된다.

또한 `@Autowired`를 처리하는 `getAutowireCapableBeanFactory` 메소드의 경우 해당 메소드를 호출하면 반환되는 빈 팩토리가 바로 `DefaultListableBeanFactory`이다.

### ConfigurableApplicationContext

거의 모든 ApplicationContext가 갖는 공통 애플리케이션 컨텍스트 인터페이스이다. ApplicationContext, Lifecycle, Closable 인터페이스를 상속받는다. 컨텍스트가 시작되고 종료될 때의 메소드들을 가지며 `AnnotationConfigApplicationContext`, `AnnotationConfigServletWebServerApplicationContext`, `AnnotationConfigReactiveWebServerApplicationContext`  모두 `ConfigurableApplicationContext`를 구현하고 있다. 스프링부트에서 `run` 메소드를 실행하면 반환 타입 역시 `ConfigurableApplicationContext`이다.