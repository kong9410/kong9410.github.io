---
title: 싱글턴 패턴(Singleton pattern)
tags: java design-pattern
---

# 싱글턴 패턴

## 무엇인가

클래스를 사용하기 위해서는 `User user = new User()` 와같은 방식으로 인스턴스를 생성해서 사용하여야 한다. 그러나 제한된 자원에 맞춰서 사용할때 불가피하게 단 하나의 인스턴스만 공유해서 사용해야 하는 경우가 있다. 예를들어 JDBC를 사용해 DB에 연결할 때 매번 계정명과 비밀번호, url등을 입력해 디비 연결이 일어난다면 반복되는 코드들도 많아질 뿐만 아니라 db와 연결도 반복해서 일어날 것이다. 이렇듯 단 한번의 초기값으로만 인스턴스를 사용해야한다면 싱글턴 패턴을 적용할 수 있다.

싱글턴 패턴은 처음 사용할때 단 한번만 인스턴스를 사용하고 그 이후에는 모두 동일한 인스턴스를 사용하는 개발 패턴이다.

## 정적 클래스와의 차이점

정적 클래스 역시 클래스가 로드될때 초기값을 세팅해 사용할 수가 있다. 이는 싱글턴 패턴이 이루고자 하는 목표와 비슷할 수 있다. 하지만 인터페이스를 상속해서 사용해야 하는 경우엔 정적 클래스를 사용할 수가 없다. 왜냐면 인터페이스에서는 정적 메소드를 사용할 수 없기 때문이다.

```java
public interface User {
  static void sayHello(String userName); // 사용할 수 없다
}
```

이렇든 싱글턴 패턴은 정적 클래스가 할수없는 구현 여부의 차이가 있다. 만일 구현이 필요없는 단일 클래스라면 싱글턴 패턴을 적용시켜 개발하는 것보다 정적 클래스가 오히려 더 깔끔한 코드가 될것이다.

## 코드

단 한번만 인스턴스를 생성하기 위해서는 외부에서 인스턴스를 생성하는 코드를 막아야한다. 그렇기 때문에 생성자 접근제한자를 `private`으로 선언해야한다. 그리고 인스턴스는 한개 만들어야하는 상황이므로 이 인스턴스를 한번만 만들어줄 외부에 노출되는 메소드를 만들어야한다.

```java
public interface User {
  void sayHello(String userName);
}
```

```java
public class ServiceUser implements User {
  private static ServiceUser serviceUser = null;
  
  private ServiceUser() {}
  
  public static ServiceUser getServiceUser() {
    if (serviceUser == null) {
      serviceUser = new ServiceUser();
    }
    
    return serviceUser;
  }
  
  @Override
  public void sayHello(String userName) {
    System.out.println("Hello, My Name Is " + userName);
  }
}
```

주목할만한 점은 인스턴스를 할당하기 위한 내부 변수와 생성 메소드가 `static`으로 선언되었다는 것인데, 정적 변수, 메소드는 인스턴스에 속하는 영역이 아니고 클래스 자체에 속하는 의미이다. 그렇기 때문에 클래스의 인스턴스를 통하지 않고도 정적 메소드를 사용할 수가 있다.

## 문제점

멀티 스레드를 사용하는 환경에서는 위와 같은 코드는 문제가 될 수 있다. 다음과 같은 경우를 생각해보자

1. serviceUser가 아직 생성되지 않았을 때 스레드 1이 `getServiceUser()`메소드를 호출한다. if에서 serviceUser가 null인지까지 체크를 한다.
2. 스레드1이 인스턴스를 생성하지 않은 상황에서 스레드2가 `getServiceUser()`메소드를 호출한다. 아직 serviceUser가 null이 므로 serviceUser 인스턴스를 생성한다.
3. 스레드1과 스레드2 둘다 인스턴스를 생성하게 되고 결과적으로 `ServiceUser` 클래스의 인스턴스가 2개가 생성된다.

다음 코드로 예시를 들어보겠다.

```java
public class ServiceUser implements User {
  private static ServiceUser serviceUser = null;
  
  private ServiceUser() {}
  
  public static ServiceUser getServiceUser() {
    if (serviceUser == null) {
      try {
        Thread.sleep(1);
      } catch (InterruptedException e) {}
      serviceUser = new ServiceUser();
    }
    
    return serviceUser;
  }
  
  @Override
  public void sayHello(String userName) {
    System.out.println("Hello, My Name Is " + userName);
  }
}
```

```java
public class UserThread extends Thread {
  public UserThread(String name) {
    super(name);
  }
  
  @Override
    public void run() {
        User user = User.getServiceUser();
        user.sayHello(Thread.currentThread().getName() + " : " + user.toString());
    }
}
```

```java
public class Main {
  public static void main(String[] args) {
    UserThread[] threads = new UserThread[5];
    for (int i = 0; i < 5; i++) {
      threads[i] = new UserThread((i + 1) + "-thread");
      threads[i].start();
    }
  }
}
```

위와 같이 null 체크 후 스레드를 잠시 멈췄다가 이후에 `serviceUser`를 생성하게하고, `for`로 매번 다른 스레드에서 실행되도록 만들었다.

```
Hello, my name is 2-thread : project.personal.design.pattern.singleton.failure.User@53c2c2eb
Hello, my name is 1-thread : project.personal.design.pattern.singleton.failure.User@3c9493ab
Hello, my name is 4-thread : project.personal.design.pattern.singleton.failure.User@6a93c46f
Hello, my name is 5-thread : project.personal.design.pattern.singleton.failure.User@27cec074
Hello, my name is 3-thread : project.personal.design.pattern.singleton.failure.User@3ed5f4d8
```

결과는 매번 다른 인스턴스를 생성하게 될것이다. 이렇게 된다면 본래의 목적인 인스턴스를 단한번만 생성한다는 것이 실패한것이다.

## 해결법

두가지 방법이있다.

1. 정적 변수를 바로 초기화 한다.
2. 동기화를 사용한다.



첫번째 방법은 `getServiceUser()`를 사용할 때 `null`체크를 할 필요가 없어진다. 클래스를 로드함과 동시에 초기값으로 인스턴스가 바로 생성되기 때문이다.

```java
public class ServiceUser interface User {
  private static final ServiceUser serviceUser = new ServiceUser();
  
  public static ServiceUser getServiceUser() {
    return serviceUser;
  }
}
```

이러한 방법은 복잡한 코드를 사용하지 않아도 되지만 해당 객체가 사용하지 않을때도 메모리를 차지하게 된다는 단점이 있다.

두번째 방법은 `synchronized`를 사용하는 것이다. 이는 위에서 말한 단점을 해결하기위해 Lazy Initialization하는 방식이다. Lazy Initialization은 컴파일 시점이 아닌 인스턴스가 필요한 시점에서 생성하는 방식이다. 이것은 멀티 스레드 환경에서 동시에 여러 스레드가 해당 코드에 접근하는 것을 방지한다.

```java
public class ServiceUser interface User {
  private static ServiceUser serviceUser = null;
  
  public synchronized static ServiceUser getServiceUser() {
    if (serviceUser == null) {
      serviceUser = new ServiceUser();
    }
    
    return serviceUser;
  }
  
  /* ... */
}
```

다만 이럴경우에 모든 스레드가 `getServiceUser()`에 접근할때 동기화에 걸리게되므로 해당 메소드의 사용이 빈번할 때는 성능의 저하가 일어날 수 있다. 그 때문에 메소드 전체를 `synchronized`를 하는것이 아닌 내부의 코드를  `synchronized` 블록으로 지정할 수 있다.

```java
public class ServiceUser interface User {
  private static volatile ServiceUser serviceUser = null;
  
  public synchronized static ServiceUser getServiceUser() {
    if (serviceUser == null) {
      synchronized(ServiceUser.class) {
        if (serviceUser == null) {
          serviceUser = new ServiceUser();  
        }
      }
    }
    
    return serviceUser;
  }
}
```

`volatile`을 키워드로 스레드가 인스턴스를 생성할때 CPU 캐시가 아닌 메인메모리에 저장되도록 강제 지정해준다. `null`을 체크하는 `if`내부에 `if`를 한번 더 지정해주는 것을 볼 수 있는데 이는 각 스레드에서 생성한 인스턴스가 아직 메인메모리에 올라가지 않았을 때 발생하는 Race Condition 문제를 해결하기 위함이다. `synchronized` 외부에 `if`를 둔건 필요없는 `synchronized`발생을 방지하기 위함이고 내부의 `if`는 `synchronized`를 실행하는 스레드들의 동시 인스턴스 생성을 방지하기 위함이라고 생각하면 된다. 이를 DCL(Double Checking Looking)이라고 한다.

마지막으로 가장 많이 사용하는 LazyHolder 방식이 있다. 동시성 문제도 해결하고 코드도 더욱 간결해진다.

```java
public class ServiceUser interface User {
  private ServiceUser() {}
  
  private static class InnerInstanceClazz {
    private static final ServiceUser instance = new ServiceUser();
  }
  
  public static ServiceUser getServiceUser() {
    return InnerInstanceClazz.instance;
  }
}
```

`InnerInstanceClazz`라는 이너클래스를 선언하였다. `static` 멤버 클래스더라도 클래스 로더가 초기화를 할 때 초기화되지 않고 `getServiceUser()`메소드를 사용할 때 초기화된다. 동적바인딩(Dynamic Binding, 런타임시에 결정)의 특징을 이용하였기 때문에 Thread-safe하다

## 스프링 싱글톤 Bean과 차이점

### 스프링 빈

1. 아무것도 설정하지 않으면 기본적으로 싱글톤으로 인스턴스가 생성됨
2. 싱글톤 빈은 어플리케이션 구동 시 생성된다
3. 빈을 싱글톤이 아닌 다른방식으로 생성하고 싶으면 @Scope("범위")로 지정해주면됨

| Java 싱글톤                               | 스프링 싱글톤          |
| -------------------------------------- | ---------------- |
| 클래스 로더가 구현                             | 스프링 컨테이너가 구현     |
| scope가 코드 전체                           | scope가 컨테이너 내부   |
| Thread Safety는 개발자가 로직을 어떻게 만드냐에 따라 다름 | Thread Safety 보장 |

> scope: 스프링 빈이 관리되는 범위
>
> - singleton: 스프링 컨테이너의 시작 ~ 종료
> - prototype: 빈의 생성과 의존관계 주입만 관여
> - request: 웹 요청이 들어오고 나갈때까지 유지되는 스코프
> - session: 웹 세션이 생성되고 종료될 때까지 유지되는 스코프
> - application: 웹의 서블릿 컨텍스와 같은 범위로 유지된다.



### 스프링 Bean은 싱글톤인가?

- 싱글톤 역할을 할 수 있지만 싱글톤 개념을 명확히하자면 싱글톤이 아님
- 객체에 대한 인스턴스는 하나만 존재해야 한다
  - 스프링 Bean은 여러 이름으로 존재할 수 있다.
  - Bean의 Scope를 싱글톤이 아닌 다른 것으로 바꾸면 Bean은 싱글톤이 될 수 없다





## 출처

정인상, 채홍석, JAVA 객체지향 디자인패턴

https://chung-develop.tistory.com/56

https://gmlwjd9405.github.io/2018/11/10/spring-beans.html

[싱글턴 패턴(Singleton Pattern). 자바와 스프링의 싱글턴 패턴(Singleton Pattern)과 차이점 \| by Dope \| Webdev TechBlog](https://webdevtechblog.com/%EC%8B%B1%EA%B8%80%ED%84%B4-%ED%8C%A8%ED%84%B4-singleton-pattern-db75ed29c36)