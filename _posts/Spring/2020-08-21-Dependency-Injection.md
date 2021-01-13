---
title: 의존관계 주입
tags: Spring
---

# 의존관계 주입(Dependency Injection, DI)

### 강한 결합

객체 내부에서 다른 객체를 생성하는 것은 강한 결합도를 가지는 구조, A에서 B를 생성하고 B를 C로 바꿀 경우 A도 같이 바꾸어야 하는 경우

### 느슨한 결합

객체를 주입 받는다는 것은 외부에서 생성된 객체를 인터페이스를 통해 넘겨받는 것. 이러면 결합도를 낮출 수 있고, 런타임시에 의존관계가 결정된다.



## 주입 방법

### 수정자를 통한 주입 (Setter)

```java
public class Controller {
    private Service service;
    
    public void setService(Service serivce) {
        this.service = service;
    }
    
    public void callService() {
        service.doSomething();
    }
}
```

```java
public interface Service {
    void doSomething();
}
```

```java
public class ServiceImpl implements Service {
    @Override
    public void doSomething() {
        System.out.println("ServiceImpl is doing something");
    }
}
```

- `Controller` 클래스는 `callService()` 메소드는 `Service`타입 객체에 의존
- `Service` 인터페이스를 구현하면 어떤 타입의 객체라도 `Controller`에서 사용할 수 있다. `Controller`는 구현체의 내부 동작을 알지 못하고 알필요도 없다.
- 낮은 결합도를 가지고 구현하였지만, Service의 구현체를 주입해주지 않아도 Controller 객체는 생성 가능하다. 때문에 NullPointerException이 발생하거나 주입이 필요한 객체가 주입이 되지 않아도 얼마든지 객체를 생성할 수 있다는 문제가 생긴다



### 생성자를 통한 주입(Constructor)

`setter`를 없애고 생성자를 이용해 주입한다.

```java
public class Controller {
    private final Service service;
    
    public Controller(Service service) {
        this.service = service;
    }
    
    public void callService() {
        service.doSomething();
    }
}
```

- null을 주입하지 않는 한 NullPointerException이 발생하지 않는다.
- 의존관계를 주입하지 않는이상 Controller 객체를 생성할 수 없다.
- final을 사용할 수 있다.



## 스프링에서 의존관계 주입방법 3가지

### Field Injection

```java
@Controller
public class MyController {
    @Autowired
    private MyService myService;
    
    public void callSomething() {
        myService.callSomething();
    }
}
```

### Setter based Injection

```java
@Controller
public class MyController {
    private MyService myService;
    
    @Autowired
    public void setMyService(MyService myService) {
        this.myService = myService;
    }
    
    public void callSomething() {
        myService.callSomething();
    }
}
```

### Constructor based Injection

```java
@Controller
public class MyController {
    private final MyService myService;
    
    @Autowired
    public MyController(myService) {
        this.myService = myService;
    }
    
    public void callSomething() {
        myService.callSomething();
    }
}
```

생성자 주입의 장점

- `NullPointerException`을 방지할 수 있다.
- 주입받을 필드를 `final`로 선언 가능하다.
- 단위 테스트 작성하기가 좋아진다.



### 생성자 주입을 이용한 순환참조 방지

기존 `setter`주입 방식을 선택하여 작성한다면 Service1에서 Service2에 의존, Service2에서 Service1에 의존한다면 서로를 끊임없이 호출하다가 결국 `StackOverFlow`가 발생하게된다.

`constructor`주입 방식을 선택한다면 다음과 같은 코드가 실행된다.

```java
new Service1(new Service2(new Service1(new ...)))
```

이런 긴 순환은 스프링 애플리케이션에서 구동되지 않게 한다.



#### 원문

https://yaboong.github.io/spring/2019/08/29/why-field-injection-is-bad/
