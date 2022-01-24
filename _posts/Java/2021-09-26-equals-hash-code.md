---
layout: post
title: Object클래스의 equals와 hashCode
tags: java
---

```java
class Student {
  private int id;
  private String name;
  
  public void setId(int id) {
    this.id = id;
  }
  
  public void getId() {
    return id;
  }
  
  public void setName(String name) {
    this.name = name;
  }
  
  public String getName() {
    return name;
  }
}
```

```java
void main() {
  Student student1 = new Student();
  Student student2 = new Student();
  
  student1.setId(1);
  student1.setName("낙타");
  student2.setId(1);
  student2.setName("낙타");
  
  System.out.println(student1.equals(student2)); // false
}
```

위와 같이 같은 학생클래스인데 이름을 둘다 낙타로 생성하여 `equals`로 비교를 했다. 결과는 `false`가 되었다. 이는 내부의 값을 비교하는 것이 아니라 단순 오브젝트 비교(`this == obj`)를 하기 때문에 객체의 주소값이 동일한지, 동일 객체인지를 확인한다.

`String` 클래스의 경우는 `equals`가 재정의 되어있어서 문자열 값을 비교한다.

`hashCode`는 객체를 식별하는 정수값을 말한다. 오브젝트의 `hashCode()` 메소드는 객체의 메모리번지를 이용해 해시코드를 만들어 리턴하기 때문에 객체마다 값이 다르다. 객체 값 동등성 비교시 `hashCode()`를 오버라이딩 할 필요가 있다. `HashSet`, `HashMap`, `HashTable`은 다음 방법으로 두 객체가 동등한지 본다.

- `hashCode()` 해시코드 값이 같은지를 본다. 해시 코드값이 다르면 다른 객체로 판단한다.
- `equals()` 로 한번더 비교한다. 해시코드가 다르면 동치성 비교조차 하지 않는다.

동일한 객체는 동일한 메모리 주소를 가지고있고, 동일한 객체는 동일한 해시코드를 가져야 한다. 때문에 `equals()`를 오버라이드 한다면 `hashCode()` 메소드도 오버라이드 되어야 한다. 

Object 클래스의 `hashCode()`는 해당 메모리 주소값을 반환한다. 해시코드를 동일하게 하기 위해 `hashCode()`를 오버라이드해 사용해야한다.

```java
@Override
public int hashCode() {
  final int PRIME = 31;
  return getId() * PRIME;
}
```

이렇게 사용한다면 id가 같은 두개의 `Student` 클래스를 비교할시에 동일 해시값이기 때문에 HashTable에서 `equals`를 판단할 수 있다