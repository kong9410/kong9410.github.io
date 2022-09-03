---
title: String Constant Pool
tags: java
layout: post
description: JVM의 String Constant Pool에 대해 알아본다
---

흔히들 String을 효율적으로 사용하려면 `new` 객체 생성방법이 아닌 따옴표로 사용하라고 한다. 그 이유는 무엇일까?

```java
String a = "AAA";
String b = "AAA";
assertEquals(a, b); // true

String c = new String("CCC");
String d = new String("CCC");
assertNotEquals(c, d); // true
```

위와 같이 따옴표로 동일한 문자열을 사용한 String 변수와 `new` 생성자를 통한 String 변수의 비교 결과값이 다르다. 이유는 서로 참조하고있는 reference가 다르기 때문이다.

![stringconstantpool](https://user-images.githubusercontent.com/37204770/188258848-0ac536f4-6091-4842-b752-5b2a116fc41b.png)

위의 그림과 같이보면 따옴표로 생성한 s1과 s2 변수는 Java Heap의 String Constant Pool이라는 곳에 저장된다. String Constant Pool에 해당 문자열이 있는지 확인하고 없으면 새로 생성하여 저장하고 해당 String의 reference를 갖게되고, 있으면 기존의 String의 reference를 갖게된다.

반면 `new` 연산자로 생성한 String은 Java Heap 영역에 새로운 String 객체로 생성된다. s1과 s3는 서로 다른 reference를 갖고 있기 때문에 서로 같지 않다는 결론이 나온다.



