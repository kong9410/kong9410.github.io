---
title: 객체 지향 원리
tags: java cs
---

# 추상화

- 무엇?

  어떤 집단의 공통된 특징을 추상적으로 파악해 인식할 수 있는 대상으로 삼는 행위이다. 객체지향 프로그래밍에선 관심이 있는 기능이나 클래스를 추출하는 것을 말한다.

- 왜?

  가령 신용카드 관리 프로그램을 만든다 할 때 신용카드는 삼성카드 롯데카드 신한카드 농협카드 등 여러 종류의 카드가 있다. 전부 카드라는 점과 기능은 동일하고 이름과 카드번호 CVC 멤버쉽 혜택 등의 구체적인 정보만 다르다. 사용자가 카드 정보를 입력할 때 카드의 종류를 알기 위해서 신한카드 클래스, 농협카드 클래스를 따로 만들게 되면 카드수가 많아질때 생성되는 클래스 또한 무수히 많아지게 될 것이다. 이럴때 사용하려는 카드의 주요 관심사는 카드번호, 이름, CVC 번호 그리고 카드의 공통된 기능정도일것이다 이를 추상화 하면 카드라는 하나의 객체로 추상화 하여 담을 수 있을 것이다.

```java
void main() {
    카드 신한카드 = new 카드("철수", "9999-8888-7777-6666");
    카드 농협카드 = new 카드("길동", "1111-2222-3333-4444");
    
    신한카드.환불();
    농협카드.결제();
}
```

# 캡슐화

- 무엇?

  캡슐화는 내부 요소를 은닉하여 필요한 부분만 공개하는 설계 원리다.

- 왜?

  결합도란 기능을 실행하는데 있어서 다른 클래스나 모듈들이 얼마나 의존적인지를 나타내고, 응집도는 클래스나 모듈안의 요소들이 얼마나 밀접하게 관련되어 있는가를 나타낸다. 캡슐화는 응집도를 높이고 결합도를 낮추는 설계 원리다. 필요한 정보를 뺀 나머지 정보들은 은닉하여 결합도를 낮출수있다. 예를 들어 스택과 유사한 자료구조 클래스를 만들었다고 가정했을때 스택 저장소 역할을 하는 ArrayList에 직접 접근하게 되면 손쉽게 데이터에 접근을 할 수 있게한다. 그러나 만일 스택 클래스의 기능이 바뀌어 `pop`이 맨 뒤의 요소를 제거하는 것이 아닌 맨 앞의 원소를 제거하는 것으로 바뀐다면 ArrayList를 직접 참조하는 부분의 코드도 바뀌게 될 것이다. 만일 ArrayList를 은닉하고 pop의 메소드만 공유한다면 사용하는쪽에서는 pop이 앞에서 빼든 뒤에서 빼든 상관없이 pop의 역할만 한다고 보면 될것이다.

```java
// 결합도 예
String getService(Content content) {
    ServiceParameter serviceParameter = makeServiceParameter(Content content);
	return get(serviceParameter);
}

ServiceParameter makeServiceParameter(Content content) {
    return ServiceParameter.builder()
        .id(content.getId())
        .body(content.getBody())
        .writer(content.getWriter())
        .requestDate(content.getRequestDate())
        .build();
}
```

`makeServiceParameter` 메소드는 `Content` 클래스에 의존하고 있으며 `Content`의 필드가 변경될 시에 같이 변경이 되어야한다. 마찬가지로 `get` 메소드도 파라미터로 들어가는 `ServiceParameter`의 변경이 일어나면 수정이 필요할 수도 있다.

```java
// 응집도 예
void post(Content content) {
    updateContent(content);
    insertContentHistory(content);
    alarm(content.getWriter());
}
```

각 모듈들이 얼마나 독립적인지를 볼 수 있는데 위와 같은 경우는 컨텐츠를 `update`하고 이력을 `insert`하고 `alarm`으로 어딘가로 알람을 보내주게 된다. 각 메소드들이 서로의 역할만 하고 다른 모듈에 영향을 주지않기 때문에 각 메소드의 응집력은 높다고 볼 수 있다.

# 일반화 관계

## 일반화

일반화 관계는 객체지향 프로그래밍 관점에서 상속 관계라 한다. 사과, 배, 바나나 등은 과일의 종류이므로 과일을 특수화한 개념이다. 각 요소들을 따로따로 보지 않고 과일이라는 하나의 공통 사물을 다룬다면 과일 전체를 다룰 수 있다. 또한 일반화는 캡슐화의 일종으로 바나나인지 배인지는 알필요가 없게 객체를 은닉할 수 있다. 추상화와 비슷한 개념인데 추상화는 관심사에서 클래스를 추출했다면 일반화는 공통점에서 클래스를 추출하는 관점의 차이가 있다.

## 일반화 관계와 위임

일반화 관계는 'is a kind of 관계'가 성립되어야 한다. 예를 들어 "Stack is a kind of ArrayList"는 Stack이 ArrayList를 대체할 수 있는가의 문제인데 대부분의 프로그램에서 Stack은 ArrayList를 대신할 수가 없기 때문에 성립되지 않는다. 그러나 ArrayList의 기능을 일부 재사용하고 싶다고 하면 위임을 사용하면 된다. 이는 자신이 직접 기능을 실행하는 것이 아닌 다른 클래스의 객체가 기능을 실행하도록 **위임**하는 것이다.

```java
public class MyStack<String> extends ArrayList<String> {
    public void push(String element) {
        add(element);
    }
    
    public String pop() {
        return remove(size() - 1);
    }
}
```

위와 같은 상속관계를 아래와 같이 다른 클래스가 동작하도록 위임한다.

```java
public class MyStack<String> {
    private ArrayList<String> arList = new ArrayList<String>();

    public void push(String element) {
        arList.add(element);
    }
    
    public String pop() {
        return arList.remove(arList.size() - 1);
    }
    
    public boolean isEmpty() {
        return arList.isEmpty();
    }
    
    public int size() {
        return arList.size();
    }
}
```

## 집합론 관점에서 본 일반화 관계

집합론적 관계에서는 다음과 같은 관계가 성립되어야 한다.

(부모 클래스A, 자식 클래스 A1 A2 A3)

- A1 = A1 ∪ A2 ∪ A3
- A1 ∩ A2 ∩ A3 = ø

일반화 관계에서는 {disjoint, complete}라는 제약조건을 사용한다. disjoint는 자식 클래스 객체가 동시에 두 클래스에 속할 수 없다는 의미다. complete는 자식 클래스의 객체에 해당하는 부모 클래스의 객체와 부모 클래스의 객체에 해당하는 자식 클래스의 객체가 하나만 존재한다는 의미다.

집합론적 관점에서는 일반화 관계를 조금 더 단순하게 만들 수 있다. Vip회원이 상품을 사고, 일반회원이 상품을 산다고 했을 때 VIP와 일반 회원 모두 회원 모두 물건을 살 수 있으므로 공통적으로 갖는 것은 회원이라는 부모 클래스와 상품 클래스와 연관관계를 맺어서 간결하게 만들 수 있다.

만일 member를 vip와 ordinary 뿐만 아니라 지역 주민, 비지역 주민으로도 분류할 수 있다면 한 인스턴스트는 vip임과 동시에 local 클래스에도 속하게 된다. 이 경우는 한 인스턴스가 동시에 여러 클래스에 속할 수 있는 다중 분류라고 한다. 만일 기존 회원이면서 동시에 지역 주민임을 구분하고자 할때는 다음과 같이 클래스로 만들 수 있다.

# 다형성

- 무엇?

  서로 다른 클래스의 객체가 같은 메시지를 받았을 때 각자의 방식으로 동작하는 능력이다. 

- 왜?

  예를 들어 개, 고양이 클래스가 있는데 talk()를 했다면 개는 "멍멍", 고양이는 "야옹"이 될 것이다. 같은 연산을 수행하지만 자식 클래스에 따라서 결과가 다른 것이다. 이를 각자 구현하는 것과 자식 클래스에서 구현하는 것에는 차이가 있다.

  ```java
  public class Animal {
      public abstract void talk();
  }
  
  public class Dog extends Animal {
      @Override
      public void talk() {
          System.out.println("멍멍");
      }
  }
  
  public class Cat extends Animal {
      @Override
      public void talk() {
          System.out.println("야옹");
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Pet[] p = {new Dog(), new Cat()};
          
          for(int i = 0; i < 2; i++) {
              System.out.println(p.talk());
          }
      }
  }
  ```

  ```java
  public class Dog {
      public void talk() {
          System.out.println("멍멍");
      }
  }
  
  public class Cat {
      public void talk() {
          System.out.println("야옹");
      }
  }
  
  public class Main {
      public static void main(String[] args) {
          Dog dog = new Dog();
          Cat cat = new Cat();
          
          dog.talk();
          cat.talk();
      }
  }
  ```

  다형성을 사용한 경우엔 Pet의 구현체에 어떤 동물이 들어가있던 talk()를 호출하면 그에 맞는 연산이 수행된다. 다형성을 사용하지 않은 경우에는 각 동물에 대한 클래스를 구현하고 각각의 talk() 메소드를 호출한다. 예시를 든 코드는 Dog와 Cat 두개 밖에 없어서 크게 차이가 느껴지지 않지만 외부에서 객체정보가 들어오는 경우 각 객체에 맞는 연산 로직을 처리하는 메소드를 생성해야 할 것이다. 또한 자식 클래스가 늘어날 경우 그에 맞는 객체를 생성하고 talk()를 호출하는 코드를 넣어야 할 것이다. 다형성은 이처럼 코드를 간결하게 하는 것은 물론 변화에도 유연하게 대처할 수 있다.

# 피터 코드의 상속 규칙

상속의 오용을 막기 위해 상속의 사용을 엄격하게 제한하는 규칙이다. 다음 5가지 규칙이 있고 하나라도 만족하지 않는다면 상속을 사용해서는 안된다.

- 자식 클래스와 부모 클래스 사이는 '역할 수행'관계가 아니어야 한다.
- 한 클래스의 인스턴스는 다른 서브 클래스의 객체로 변환할 필요가 절대 없어야한다.
- 자식 클래스가 부모 클래스의 책임을 무시하거나 재정의하지 않고 확장만 수행해야 한다.
- 자식 클래스가 단지 일부 기능을 재사용할 목적으로 유틸리티 역할을 수행하는 클래스를 상속하지 않아야 한다.
- 자식 클래스가 역할, 트랜잭션, 디바이스 등을 특수화 해야한다.