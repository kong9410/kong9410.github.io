---
title: 불변 객체 (Immutable Object)와 setter 메소드
tags: java javascript
layout: post
description: 불변 객체가 무엇이고 어떨 때 사용하는 것인가?
---

# 불변 객체

개발을 하다보면 나 자신도 모른채 `setter`를 남발하고 있는 경우가 있다. `setter`를 사용하다 보면 이 set이 무엇을 의미하는지 모를때가 있고 로직 중간에 값이 바뀌어 의도치 않는 결과를 초래할 때가 있다.

이를 해결하기 위한 개념이 불변 객체인데 불변 객체가 무엇이고 어떨 때 사용해야 하는지 얻는 이점이 무엇인지에 대해 정리해보았다.

## 실무에서 겪은 일

작업을 했던 코드는 모든 필드에 `setter` 메소드가 있는 가변 객체가 선언되어 있었다. 메소드 수 뿐만 아니라 뎁스도 꽤 깊기 때문에 코드 파악하는 것 자체도 오래걸리는 코드였다. 

여기서 내가 수정해야 하는 것은 `field3`이 "A"라면 `field4`는 "B"로 세팅하도록 변경을 해야했다. 

`field3`가 세팅이 안되면 로직이 돌지가 않기 때문에 `field3`가 어디서 set이 되는지 메소드 처음부터 찾아봐야 했다. 꽤나 오랫동안 코드를 쳐다보았고 어디서 어떤 값을 set을 하는지 찾아냈다. 그러나 `field3`의 set이 한번만 일어날 것이라는 보장이 없다. 중간에 값이 또 바뀔지도 모르는 불안감에 변경하려는 로직 이전까지의 `field3` set을 계속 추적했다. `field3`의 값 변경을 다 확인하고 `field4`를 변경하는 로직을 작성했다. 그러나 이 이후의 로직에서 `field4`의 값이 변경됨으로써 어떤 사이드 이펙트가 있는지 모르기 때문에 나머지 로직도 파악을 해야만했다. if 하나만을 사용하는 간단한 로직 변경인데 불안감에 휩싸일 수 밖에 없는 코드 수정이 벌어졌었다.

![immutableObject](https://user-images.githubusercontent.com/37204770/157276376-1a7e0373-a33d-4941-93af-dc674346c3e0.png)

이처럼 가변 객체(객체 그 자체 혹은 내부 필드값이 변경될 수 있는 객체)를 사용하는 경우에는 모든 메소드 및 클래스를 확인을 해야하기 때문에 유지보수도 오래걸리고 코드를 한눈에 파악하기 어렵다.

### 그렇게해서 나온 해결책 중 하나

간단하게 해결하는 방법으로 코딩 정책을 설정하는 것이었다. 여러 메소드에 나누어져있던 `setter`들을 한곳에 모았다. 그리고 객체는 여기서 생성한다는 나름대로의 정책을 만들었다. 확실히 보기도 깔끔해지고 추적하기도 편하게 되었다. 그러나 이러한 정책은 `setter`가 존재하는 한 불완전해진다. 다른 누군가가 이 정책을 무시하고 메소드 중간에 객체의 값을 바꾸는 코드를 넣게 된다면 다음에 코드를 읽는 사람은 객체 내부의 값이 언제 바뀔지 모르는 불안감에 휩싸일 수 밖에 없다.

![불변2](https://user-images.githubusercontent.com/37204770/157683572-df3f8b2b-6a27-40f7-bc90-ccfe469e1828.png)

결국에는 이러한 것을 해결하기 위해 값이 절대 변하지 않는 불변 객체가 필요한 것이다. 값이 변하지 않는다는 것을 개발자가 알고 있다면 처음 생성된 곳을 보는 것외에는 값이 불변한다는 확신을 갖게 된다면 개발이 더 쉬워질 것이다.

## 불변 객체란?

불변 객체란 무엇인가? 객체가 생성된 이후 내부 상태가 변화되지 않는 객체를 말한다. 다르게 말하자면 조회와 복사만 가능한 객체라고 보면 된다. 이러한 객체는 read-only 메소드만을 제공한다. 혹은 내부 정보를 제공하더라도 방어적 복사(defensive-copy)를 통해 제공한다.

> 방어적 복사(defensive-copy)란?
>
> 일반적으로 새로 생성할 객체에 원본 객체를 `=` 연산자로 할당을 할시에 새 객체는 *얕은 복사*를 하여 생성하게 된다. 새 객체는 원본 객체와 동일한 메모리 주소를 참조하는 형태이기 때문에 새 객체에서 변화가 일어날 시에 원본 객체도 같이 변화가 일어난다.
>
> 때문에 방어적 복사는 얕은 복사가 아닌 *깊은 복사* 방식으로 메모리 참조가 아닌 새로운 주소에 할당하는 방식을 사용한다. 또한 필드를 조회하는 메소드도 원본을 주는 것이 아닌 새로운 인스턴스를 생성해 전달을 한다.
>
> ```java
> // 방어적 복사가 안될 때 #1
> public static main(String[] args) {
>   MyArgument myArgument = new MyArgument("1"); // MyArgument 필드값을 세팅
>   MyClass myClass = new MyClass(myArgument);
>   
>   // myArgument의 필드값 세팅
>   // myArgument를 변경함으로써 myClass의 내부도 수정이 된다.
>   myArgument.setMyField("2");
> }
>
> // 방어적 복사가 안될 때 #2
> public static main(String[] args) {
>   List<String> myList = new ArrayList();
>   MyClass myClass = new MyClass(myList);
>   
>   // myArgument의 필드를 get 참조해서 수정을한다.
>   myClass.getMyList().add("2");
> }
> ```
>
> ```java
> // 방어적 복사가 적용될 때 #1
> public class MyClass {
>   private MyArgument myArgument;
>   
>   public MyClass(MyArgument myArgument) {
>     // 복사본을 새로 만들어 방어적 복사
>     this.myArgument = new MyArgument(myArgument.getMyField());
>   }
> }
>
> // 방어적 복사가 적용될 때 #2
> public class MyClass {
>   private List<String> myList;
>   
>   // constructor...
>   
>   public List<String> getMyList() {
>     return new ArrayList(myList);
>   }
> }
> ```
> 그러나 현재 내 업무에서는 단순히 `lombok` 어노테이션만으로 `getter` 메소드들을 자동 생성하는 경우가 대부분이기 때문에 방어적 복사를 사용하는 것은 사실상 불가능하다. (모든 필드에 방어적 복사를 적용시킨다면 개발시간이 너무 많이 늘어날 것이다.)

## 불변 객체를 사용해서 얻는 이점

### Thread-Safe 하다

멀티 쓰레드 환경에서는 공유 자원의 수정이 일어난다면 동기화의 문제가 발생할 수가 있다. 불변 객체는 자원의 수정이 일어나지 않으므로 동기화 문제를 신경쓰지 않아도 된다.

### Failure Atomic 메소드를 만들 수 있다

가변 객체를 통해 작업하던 도중 문제가 발생한다면 객체가 불안정해진 상태에 빠질 수 있다. 이런 상태로 로직을 수행한다면 또다른 에러를 발생시킬 수 있다.

### Side Effect 최소화

`setter`등을 통해 값을 중간에 바꾸는 일이 일어난다면 앞의 결과와 뒤의 결과가 다르게 되는 일이 발생될 수가 있다. 이는 다른 로직의 오류를 초래할 수 있다. 불변 객체는 값이 고정이기 때문에 내부값이 변경되는 것을 고려하지 않아도 된다.

### 예측 가능하다

가변 객체가 여러 곳에서 값을 세팅을 해준다면 해당 객체를 사용하는 모든 곳에서 값이 바뀌는 곳을 확인을 해야하기 때문에 유지보수도 느리고 값을 예측하기가 쉽지 않다.

### Garbage Collection 성능 향상

불변 객체는 `final` 키워드로 생성할 수 있고 이 객체를 가지는 객체도 존재한다. 이 가장 바깥쪽의 컨테이너 객체는 `final` 객체가 먼저 생성되고 생성될 수 있기 때문에 내부 객체를 모두 포함해서 가장 어린 객체가 된다. 이럴경우 가비지 컬렉터가 컨테이너 하위의 불변 객체(`final` 객체)들은 Skip 할 수 있게 한다. 해당 컨테이너 객체가 살아있으면 하위 불변 객체는 처음 그대로 할당되고 있기 때문이다.

## 불변 객체를 만드는 법

### final 키워드

`String`을 이용한 간단한 예다.

```java
final String name = "kim";
name = "lee"; // error!
```

`name`변수를 `final`로 선언함으로서 이후에 값이 바뀌지 않게 불변성을 확보할 수가 있다. 개발자는 코드의 길이가 100줄이든 1만줄이든 메소드가 몇개가 있던 `name`이라는 변수의 값인 "kim"은 변하지 않는다는 확신을 가질 수 있다.

하지만 `final`이라고 객체 내부까지 바꾸지 못하는 것은 아니다.

```java
final List<String> list = new ArrayList();
list.add("a");
```

위와 같이 `list`는 객체 자체는 다른 값으로 교체할 수 없는 불변이 되었지만 내부에는 변화가 생길 수 있다. 개발자는 이것이 의도한 것인지 아닌 것인지 명확히 해야한다. 만일 내부 상태를 바꾸고 싶지 않다면 위에서 설명한 방어적 복사(deffensive-copy)를 사용해야한다.

열심히 방어적-복사를 이용한 객체를 만들어 두었는데 상속받은 클래스가 메소드가 오버라이딩이 될 수가 있다. `final` 클래스를 선언하면 클래스의 상속 자체를 막아버릴 수 있다.

```java
public final class ParentObject {}

public class ChildObject extends ParentObject {} // compile error
```

## 객체를 생성하는 방법

객체를 생성하는 패턴은 3가지가 있다. *점층적 생성자 패턴* , *자바빈 패턴*, *빌더 패턴* 이 있다.

그 중에 필드를 final로 가질 수 있는 것은 점층적 생성자 패턴과 빌더 패턴이 있다.

### 점층적 생성자 패턴

생성자를 통해 생성하는 방법이다. 필요한 만큼 생성자를 생성해야하고 필드가 추가 될 때마다 코드를 수정을 해야한다는 단점이 있다.

```java
public class Student {
  private final String firstName;
  private final String middleName;
  private final String lastName;
  private final int age;
  private final String gender;
  
  public Student(String firstName, String middleName, String lastName) {
    this(firstName, middleName, lastName, 0, "unknown");
  }
  
  public Student(String firstName, String middleName, String lastName, int age, String gender) {
    this.firstName = firstName;
    this.middleName = middleName;
    this.lastName = lastName;
    this.age = age;
    this.gender = gender;
  }
}
```

### 자바빈 패턴

여지껏 이야기한 `setter`를 통해서 생성하는 방법이다. 일관성을 유지하기가 힘들다. 또한 인스턴스 생성과 동시에 초기화가 되는 것이 아니기 때문에 필드를 `final`로 선언할 수 없다.

```java
public class Student {
  private String name;
  private int age;
  
  public void setName(String name) {
    this.name = name;
  }
  
  public void setAge(int age) {
    this.age = age;
  }
}
```

### 빌더 패턴

생성자 패턴의 안정성 + 자바빈 패턴의 가독성을 합친 패턴이다.

```java
public class Student {
  private final String name;
  private final int age;
  
  public Student(String name, int age) {
    this.name = name;
    this.age = age;
  }
  
  public static StudentBuilder builder() {
    return new StudentBuilder();
  }
  
  public static class StudentBuilder {
    private String name;
    private int age;
    
    public StudentBuilder name(String name) {
      this.name = name;
    }
    
    public StudentBuilder age(int age) {
      this.age = age;
    }
    
    public Student build() {
      return new Student(this.name, this.age, /*...*/);
    }
  }
}

public class Main {
  public static void main(String[] args) {
    Student student = Student.builder()
      .name("홍길동")
      .age(12)
      .build();
  }
}
```

안정적으로 `final` 필드의 값을 초기화 하는 방법이기에 불변객체를 만든다하면 `Builder` 패턴을 사용하는 방법이 괜찮다.

#### Lombok + Builder 사용시 주의사항

클래스 상단에 `@Builder` 어노테이션을 적용시 기본생성자는 사라지고 클래스의 모든 필드를 매개변수로 갖는 생성자 하나만 갖게된다. 단순히 이러한 객체를 조회 용도로만 사용한다면 문제가 없을 것이다. 하지만 MyBatis나 JPA를 통해서 객체 값을 초기화하게 되는 경우는 문제가 된다. MyBatis나 JPA는 DB 조회 결과를 바인딩하기 위해 기본생성자를 필요로하기 때문이다. 

때문에 `@Builder`를 사용했으면 기본 생성자도 같이 만들어 주어야하는데 여기서 또 `@NoArgsArguments`만 사용하면 `@Builder`에 필요한 생성자가 존재하지 않게되기 때문에 `@AllArgsArguements`도 같이 사용해주어야 한다.

```java
@Builder
@NoArgsArguments
@AllArgsArguments
public class Student {
  /*...*/
}
```

>mybatis는 기본생성자만 있으면 값을 초기화 세팅할 수 있다.
>
>myBatis는 `getter`, `setter` 없이도 `private` 필드에 값을 세팅할 수가 있는데 `Reflector` 클래스를 이용해 resultType의 클래스에 대한 필드정보와 `getter`/`setter` 매핑 정보를 만들어두고 이를 통해 바인딩시 해당 메소드를 만들어 필드값을 바인딩 해준다.

## setter를 지양해야 하는 이유

`setter`를 지양해야 한다는 말은 많이 들어봤지만 어떠한 이유 때문일까? 위에서 설명한 것과 합쳐서 다음과 같은 이유가 있다고 볼 수 있다.

1. 객체의 일관성이 없다
   - 객체 내부의 값이 언제든지 수정될 수 있기 때문에 의도치 않은 변경이 생길 수 밖에 없다.
   - 의도치 않은 변경은 개발자가 프로그램을 예측할 수 없게 만들어 버린다.
2. 의도를 알 수 없다.
   - 객체를 처음 생성할 때 set을 모아두면 개발자들은 "아 객체를 생성하고 값을 초기화 하는구나"라고 의도를 어느정도 파악할 수는 있다.
   - 그러나 어떤 메소드에서 뚝하고 `setField1("a")`가 생겨났을 경우에는 이 `set`을 왜 사용하는지 그 의도를 파악하기 힘들다.

## 그럼에도 불구하고 가변객체(setter 등)를 사용해야 할 때

앞서 가변객체는 나쁜것 마냥 설명했지만 무조건 불변 객체가 옳다는 이야기는 아니다. 필요에 따라 가변객체를 사용해야할 경우가 있다.

### Get Controller의 파라미터를 받을 때

`GET`요청을 받을 때 파라미터 클래스에 `setter`가 없다면 오류가 발생한다. 이유는 GET요청의 파라미터는 JSON과 같은 형식이 아닌 Query Parameter이기 때문인데 Spring은 여기서 `WebDataBinder`라는 것을 사용한다. 여기서 기본값으로 값을 할당하는 방식이 *자바빈* 방식이다. 즉 `setter`를 통해서 값을 받기 때문에 GET요청의 파라미터 클래스는 `setter` 선언이 필요하다.

### JPA의 Dirty Checking 사용

JPA의 Entity 클래스의 경우에는 최초 조회하여 가져온 값이 이후에 변경되면 `update` 쿼리가 실행된다. 그 때문에 필드의 값을 변경하는 메소드가 필요하다. 물론 이때 `setter`를 선언하는 것이 아닌 적절한 이름으로 메소드를 설정하면 메소드의 의도를 명확하게 할 수 있다.

```java
@Entity
@DynamicUpdate
public class Student {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  private int grade;
  private int schoolClass;
  
  public void promotionStudent(int grade, int schoolClass) {
    this.grade = grade;
    this.schoolClass = schoolClass;
  }
}
```

### 다음 로직에서 이전의 상태를 알아야 할 때

예를 들어 Spring Batch의 JobExecutionContext가 있다. Spring Batch에서는 step의 처리 결과가 다음 step에 전달하는 것이 불가능한데 JobExecutionContext에 저장해서 다음 step에서도 참조가 가능하게 할 수 있다.

마찬가지로 만일 로직을 작성하는데 있어서 이전의 상태를 알아야 하는 경우가 있다면 중간에 필드값을 변경하고 다음 로직으로 넘겨줄 수 있도록 만들어야 할 것이다.

### 그 외...

그 밖에 다른 여러가지 이유로 가변객체를 생성해야하는 경우가 있다. 다만 이럴경우에는 무조건 적인 `setter` 설정보다는 값을 세팅하는 의도가 담겨있는 의미있는 메소드를 생성하는 방향으로 생각해야 한다.

## 결론

1. 불변객체란 객체 생성 이후에 값이 변하지 않는 객체를 말한다.
2. 불변객체는 멀티스레드 환경에서 비교적 안전하다.
3. 불변객체는 쉽게 예측이 가능하여 개발 비용이 단축된다.
4. 불변객체는 GC에 이점이 있다.
5. `setter`는 가급적 지양하자.
6. `setter`의 대안으로 `builder`를 사용하자.
7. 데이터의 일관성을 위해 가급적이면 불변객체를 이용하자
8. 가변 객체를 사용시에 무조건적인 `setter` 사용보다는 의도가 명확한 메소드를 선언해 사용하자

## 출처

[Immutable Objects in Java | Baeldung](https://www.baeldung.com/java-immutable-object)

[[Java\] 불변 객체(Immutable Object) 및 final을 사용해야 하는 이유 - MangKyu's Diary (tistory.com)](https://mangkyu.tistory.com/131)

[[Java\] Garbage Collection(가비지 컬렉션)의 성능을 높이는 코딩 방법 - MangKyu's Diary (tistory.com)](https://mangkyu.tistory.com/120)

[자바 빌더 패턴 (Java Builder Pattern) 장단점 (tistory.com)](https://gofnrk.tistory.com/60)

[MyBatis가 setter/getter를 찾는 방법 | Jasper Ra66it (jasper-rabbit.github.io)](https://jasper-rabbit.github.io/posts/mybatis-refector/)

[더티 체킹 (Dirty Checking)이란? (tistory.com)](https://jojoldu.tistory.com/415)