---
title: 스프링 Transactional
tags: db spring java
layout: post
description: 
---

## @Transactional

스프링의 `@Transactional` 어노테이션을 사용하면 트랜잭션 처리를 할 수가 있다. 이러한 스프링 트랜잭션은 AOP를 통해 이루어진다.

### Transaction Management

스프링은 트랜잭션 처리를 TransactionManager 객체를 통해 관리를 한다. 구현체는 갈아끼울 수 있게 인터페이스인 PlatformTransactionManager가 주입되어 사용된다. 구현체는 `JtaTransactionManager`, `JpaTransactionManager`, `DataSourceTransactionManager`, `HibernateTransactionManager` 등이 있다. 기본 구현체는 `DataSourceTxManager`이고 Spring JPA를 사용시 `JpaTxManager`로 설정이 바뀐다.

트랜잭션을 직접사용하는 방법은 다음처럼 `TransactionTemplate`를 사용하는 방법이다.

```java
@Service
public class UserService {
  @Autowired
  private TransactionTemplate template;
  
  public Long registerUser(User user) {
    Long id = template.execute(status -> {
      // execute SQL
      return id;
    })
  }
}
```

위 코드는 다음과 같은 장점이 있다.

1. IoC를 통해 설정영역과 DB실행 코드가 분리되었다.
2. 데이터베이스 커넥션을 개발자가 개발하지 않는다.
3. 저수준의 SQLExceptions을 스프링에서 잡아 추상화된 런타임 예외로 포장해준다.

### @Transactional 어노테이션

대부분의 개발자들이 사용하는 방식으로 메소드에 어노테이션을 사용하는 방식이다. `@Transactional`이 사용된 메소드는 트랜잭션이 사용된다.

```java
@Service
public class UserService {
  @Transactional
  public PlatformTransactionManager txManager() {
    return yourTxManager;
  }
  
  @Transactional
  public void insert() {
    
  }
}
```

#### 동작 원리

Transactional은 Spring의 AOP를 통해 동작한다. `@Transactional`이 명시된 메소드를 호출을 하면 해당 클래스의 타켓 메서드를 직접호출 하는 것이 아닌 AOP Proxy객체가 생성되어 대신 실행이된다. AOP Proxy에서 타깃 메시지가 실행이 되기전에 트랜잭션이 생성되고 타깃메시지가 끝나고나서 커밋되거나 롤백된다.

트랜잭션 코드는 Proxy에서 직접 실행하는 것이 아닌 TransactionManager 객체 에게 위임하여 처리한다.

#### Spring이 @Transactional을 찾는 법

- @Transactional은 컴포넌트 스캔 대상이 아니다. 

- @EnableTransactionManagement기점으로 하위패키지를 탐색한다. 

- ```java
  @Import({TransactionManagementConfigurationSelector.class})
  public @interface EnableTransactionManagement {
    /* ... */
  }
  ```

- `@Import({TransactionManagementConfigurationSelector.class})` Selector를 스프링 빈으로 등록한다.

- Selector는 3가지 종류의 @Transactional 어노테이션을 분류하는 기능을 가지고 있다.

- @EnableTransactionManagement는 PlatformTxManager 타입으로 등록된 빈을 찾아서 사용한다. 

- 이후 @Transactional이 적용된 클래스를 프록시로 바꿔 PlatformTxManager를 주입한다.

### Transactional Propagation(전파 속성)

- REQUIRE: 부모 트랜잭션이 존재한다면 그에 포함되어 동작. 부모 트랜잭션이 존재하지 않을 경우 트랜잭션 생성(DEFAULT)
- SUPPORTS: 부모 트랜잭션이 존재하면 그에 포함되어 동작. 부모 트랜잭션이 없다면 트랜잭션 없이 동작.
- MANDATORY: 부모 트랜잭션이 존재하면 그에 포함되어 동작. 부모 트랜잭션이 없다면 예외 발생
- REQUIRES_NEW: 부모 트랜잭션 존재 여부와 상관없이 트랜잭션을 새로 생성
- NOT_SUPPORTED: 트랜잭션을 사용하지 않는다. 부모 트랜잭션이 존재하면 보류시킨다.
- NEVER: 트랜잭션을 사용하지 않도록 강제한다. 부모 트랜잭션이 존재할 경우 예외를 발생시킨다.
- NESTED: 부모 트랜잭션이 존재하면 부모 트랜잭션 안에 트랜잭션을 만든다. 부모 트랜잭션의 커밋과 롤백에 영향을 받지만 자신의 커밋과 롤백은 부모 트랜잭션에 영향을 주지 않는다.

### Transactional Isolation

- READ_UNCOMMITED: commit되지 않은 데이터를 읽는다.
- READ_COMMITED: commit된 데이터만 읽는다.
- REPEATABLE_READ: 자신의 트랜잭션이 생성되기 이전의 트랜잭션의 커밋된 데이터만 읽는다.
- SERIALIZABLE: LOCK을 걸고 사용
- DEFAULT: 사용하는 DB 기본 설정을 따른다.(Oracle은 READ_COMMITED, MySql은 REPEATABLE_READ)



## 참고

[@Transactional 의 동작원리, 트랜잭션 매니저 (tistory.com)](https://jiwondev.tistory.com/154#head6)

[@Transactional Propagation (전파속성), Isolation (격리수준레벨) 그리고 synchronized (tistory.com)](https://developyo.tistory.com/250)