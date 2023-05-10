---
title: 스프링 트랜잭션(Spring Transaction)
tags: database spring
layout: post
description: 스프링 프레임워크에서 트랜잭션을 사용하는 방법을 알아보자
---

### 트랜잭션이란

### 스프링이 제공하는 트랜잭션 기술

#### 트랜잭션 동기화

트랜잭션을 시작하기 위한 Connection 객체를 Connection 특정 저장소에 보관해두고 필요할 때 꺼내서 쓸 수 있도록 하는 기술이다.

트랜잭션 동기화 저장소는 작업 스레드마다 Connection 객체를 독립적으로 관리하기 때문에 멀티스레드 환경에서도 충돌이 발생하지 않는다.

![img](https://velog.velcdn.com/images/dyko/post/77f157da-e6bd-4b73-94be-e737cf9b8dc0/image.png)

예를들어 update를 세번하는 UserService의 upgradeLevels 메소드가 하나의 트랜잭션으로 묶여있다 가정한다. UserService는 Connection을 생성해 트랜잭션 동기화 저장소에 저장하고 트랜잭션을 시작시킨다.

update가 일어날때마다 connection을 꺼내와 update sql을 실행한다. 마지막 update가 끝났을 때 최종적으로 connection의 commit을 호출하여 트랜잭션을 완료시키고 트랜잭션 저장소에서 완료한 커넥션을 제거한다.

#### 트랜잭션 추상화

스프링은 트랜잭션 기술의 공통점을 담은 트랜잭션 추상화 기술을 제공하고 있다. 이를 이용해 애플리케이션에 각 기술마다 JDBC, JPA, Hibernate 등 종속적인 코드를 이용하지 않고도 일관되게 트랜잭션 처리를 할 수 있도록 해주고 있다.
스프링이 제공하는 트랜잭션 경계 설정을 위한 인터페이스는 `PlatformTransactionManager`가 있다. 만일 JDBC의 로컬 트랜잭션을 이용하려 한다면 DataSourceTxManager를 사용하면 된다.
이제 사용하는 기술과 무관하기 `PlatformTransactionManager`를 통해 다음의 코드와 같이 트랜잭션을 공유하고 커밋하고 롤백할 수 있게 되었다.

```java
public Object invoke(MethodInvoation invoation) throws Throwable {
	TransactionStatus status = this.transactionManager.getTransaction(new DefaultTransactionDefinition());
	
	try {
		Object ret = invoation.proceed();
		this.transactionManager.commit(status);
		return ret;
	} catch (Exception e) {
		this.transactionManager.rollback(status);
		throw e;
	}
}
```

#### AOP를 이용한 트랜잭션 분리

스프링에서는 트랜잭션 코드를 직접적으로 작성하는 것이 아닌 코드가 존재하지 않는 것처럼 보이기위해 로직을 클래스 밖으로 꺼내서 별도의 모듈로 만드는 AOP(Aspect Oriented Programming)을 고안하게 되었다. 이를 적용한 트랜잭션 어노테이션(`@Transactional`)을 지원하게 됐다.

```java
@Transactional
public class UserService {
  private final UserMapper userMapper;
  /*...*/
  public void addUsers(List<User> userList) {
    userList.stream().forEach(user -> {
      userMapper.insert(user);
    });
  }
}
```

### 스프링 트랜잭션 설정

#### 트랜잭션 전파(Propagation)

트랜잭션 전파는 트랜잭션 경계에서 이미 진행중인 트랜잭션이 있거나 없을때 어떻게 동작할 것인지를 정의한다. A작업의 트랜잭션이 진행중이고 B작업이 시작됄때 B의 트랜잭션은 어떻게 처리할지에 대한 것이다.

- REQUIRED
  새로운 트랜잭션을 만들지 않고 기존 진행중인 트랜잭션에 참여하게 된다. 같은 트랜잭션이기 때문에 어느 하나 실패하면 중간에 참여한 트랜잭션도 같이 롤백이 된다.
  디폴트설정이다.
- SUPPORTS
  이미 시작된 트랜잭션이 있으면 참여하고 그렇지 않으면 트랜잭션 없이 진행한다.
- MANDATORY
  항상 새로운 트랜잭션을 시작해야하는 경우에 사용한다. 이미 진행중인 트랜잭션이 있으면 잠시 보류하고 새로운 트랜잭션을 만들어 사용한다.
- REQUIRED_NEW
  B의 트랜잭션을 A와 무관하게 동작한다. B가 수행되는 순간 COMMIT이 이루어진다. A가 롤백되어도 B에게 영향을 주지 못한다.
- NOT_SUPPORTED
  트랜잭션을 걸지 않는다. 이미 진행중인 트랜잭션이 있으면 이를 보류하고 트랜잭션을 사용하지 않도록 한다.
- NEVER
  이미 진행중인 트랜잭션이 있으면 예외를 발생시키고 트랜잭션을 사용하지 않도록 강제한다.
- NESTED
  이미 진행중인 트랜잭션이 있으면 중첩트랜잭션을 시작한다. 중첩 트랜잭션은 트랜잭션 안에 다시 트랜잭션을 만드는 것으로 부모 트랜잭션의 커밋과 롤백에는 영향을 받지만 자식의 트랜잭션은 부모에게 영향을 주지 않는다.

#### 읽기전용(readOnly)

- 읽기 전용으로 설정함으로써 성능을 최적화
- 쓰기 작업이 일어나는 것을 의도적으로 방지

읽기전용으로 설정되어 있으면 트랜잭션 매니저에게 이러한 정보가 전달되어 적절한 작업을 수행한다.

#### 격리수준

디비 트랜잭션은 격리수준을 가지고 있어야한다. 서버에서 여러개의 트랜잭션이 동시에 진행될 수 있는데 모든 트랜잭션을 독립적으로 만들 수는 없다. 따라서 적절하게 격리시켜 동시성을 향상시키면서 문제가 발생하지 않아야한다. jdbc나 DataSource등에서 설정할 수 있다. 기본적으로 DB나 DataSource에 설정된 기본 격리 수준을 따르는 것이 좋다.

#### 제한시간

트랜잭션을 수행하는 제한시간을 설정할 수 있다. 제한시간의 설정은 트랜잭션을 직접 시작하는 REQUIRED나 REQUIRED_NEW의 경우에 사용해야 의미있다.

#### 읽기전용

읽기전용으로 설정해두면 트랜잭션 내에서 데이터를 조작하는 시도를 막아주고 데이터 액세스 기술에 따라 성능이 향상될 수 있다.

