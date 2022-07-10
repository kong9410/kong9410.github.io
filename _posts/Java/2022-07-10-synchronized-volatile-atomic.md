---
title: 자바 동시성 문제(synchronized, volatile, atomic)
tags: java
layout: post
description: 자바 동시성 키워드 synchronized, volatile, atomic에 대해 알아보자
---

# 자바 동시성 문제

![동시성문제](https://user-images.githubusercontent.com/37204770/178132990-3c87e25d-ce57-43b9-a45e-90dd8302c81d.png)

CPU는 데이터를 처리하기 위해 RAM의 일부분을 CPU Cache Memory로 가져와 사용한다. RAM에 저장하기 위해 CPU Cache Memory에 저장하고 RAM에 쓰는 일을 한다. 캐시에 데이터를 쓴다고 바로 RAM에 쓰기 작업을 진행하는 것은 아니다. 반대로 읽어들일 때 RAM의 데이터가 바뀌었다고 CPU 캐시 데이터가 바뀌었다고 보장할 수 없다. CPU Cache1과 CPU Cache2의 데이터가 다를 수도 있는 일이 발생하는 것이다.

동시성 프로그래밍은 CPU와 RAM 중간에 위치하는 CPU Cache Memory 와 병렬성이라는 특징때문에 다수의 스레드가 공유 자원에 접근할 때 두가지 문제가 발생 가능하다.

- 가시성 문제
- 원자성(동시 접근) 문제

#### 가시성 문제

CPU - Cache - Memory 관계에서 Cache와 Memory의 데이터가 일치하지 않는 현상을 말한다.

#### 원자성 문제

여러 스레드가 동시에 공유자원에 접근하여 값을 변경하여 발생하는 문제

한 줄의 프로그램 문장이 컴파일러에 의해 기계어로 변경되면서 이를 기계가 순차적으로 처리하기 위한 여러개의 Machine Instruction이 만들어져 실행되기 때문에 일어나는 현상

멀티 스레드 환경에서는 한스레드가 각 기계 명령을 수행하는 동안 다른 스레드가 개입하여 공유 변수에 대해 접근하여 같은 기계 명령어를 수행할 수 있으므로 값이 꼬이게 된다. (race condition)

## synchronized

자바 멀티 스레드환경에서 공유 자원 동기화 해결을 위한 키워드이다. synchronized는 4가지의 lock을 이용해 동기화를 수행한다.

#### synchronized method

synchronized method는 instance 단위로 lock을 걸지만 synchronized 키워드가 붙은 method에 대해서만 lock을 공유한다. 한 thread가 synchronized가 걸린 method를 call하면 다른 thread가 이 instance내의 synchronized method에 접근할 수 없다. 단, synchronized가 안붙은 일반 method에 대해서는 접근이 가능하다.

```java
public class Method {
    public static void main(String[] args) {
        Method method = new Method();
        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1시작");
            method.syncMethod1("스레드1");
            System.out.println("스레드1종료");
        });
        
        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2시작");
            method.syncMethod2("스레드2");
            System.out.println("스레드2종료");
        });
        
        Thread thread3 = new Thread(() -> {
            System.out.println("스레드3시작");
            method.method3("스레드3");
            System.out.println("스레드3종료");
        });
        
        thread1.start();
        thread2.start();
        thread3.start();
    }
    
    private synchronized void syncMethod1(String msg) {
        System.out.println(msg + "메소드 실행중");
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    private synchronized void syncMethod2(String msg) {
        System.out.println(msg + "메소드 실행중");
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    private void syncMethod3(String msg) {
        System.out.println(msg + "메소드 실행중");
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```shell
스레드1 시작
스레드2 시작
스레드3 시작
스레드1 메소드 실행중
스레드3 메소드 실행중
스레드1 종료
스레드3 종료
스레드2 메소드 실행중
스레드2 종료
```

#### static synchronized method

static synchronized method는 인스턴스 단위의 lock이 아니라 class 단위로 lock이 공유된다. 인스턴스 단위 lock이랑은 공유가 되지 않기 때문에 주의해야한다.

```java
public class StaticMethodSync {
    public static void main(String[] args) {
        StaticMethodSync staticMethodSync = new StaticMethodSync();

        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작");
            StaticMethodSync.syncMethod1("스레드1");
            System.out.println("스레드1 종료");
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작");
            StaticMethodSync.syncMethod2("스레드2");
            System.out.println("스레드2 종료");
        });

        Thread thread3 = new Thread(() -> {
            System.out.println("스레드3 시작");
            staticMethodSync.method3("스레드3");
            System.out.println("스레드3 종료");
        });

        thread1.start();
        thread2.start();
        thread3.start();
    }

    public static synchronized void syncMethod1(String msg) {
        System.out.println(msg + " 실행중");
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static synchronized void syncMethod2(String msg) {
        System.out.println(msg + " 실행중");
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public synchronized void method3(String msg) {
        System.out.println(msg + " 실행중");
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```shell
스레드1 시작
스레드2 시작
스레드3 시작
스레드1 실행중
스레드3 실행중
스레드1 종료
스레드3 종료
스레드2 실행중
스레드2 종료
```

#### synchronized block

synchronized block은 block단위로 lock을 걸며 두가지 방식이 있다.

##### synchronized(this)

모든 synchronized block에 lock이 걸리게 된다. 여러 스레드에서 서로다른 synchronized block을 호출해도 this를 사용해 자기 자신에게 걸었으므로 lock이 걸린다.

```java
import java.util.concurrent.TimeUnit;

public class BlockSync {
    public static void main(String[] args) {
        BlockSync blockSync = new BlockSync();

        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작");
            blockSync.method1("스레드1");
            System.out.println("스레드1 종료");
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작");
            blockSync.method2("스레드2");
            System.out.println("스레드2 종료");
        });

        thread1.start();
        thread2.start();
    }

    public void method1(String msg) {
        synchronized (this) {
            System.out.println(msg + "실행");
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public void method2(String msg) {
        synchronized (this) {
            System.out.println(msg + "실행");
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```shell
스레드1 시작
스레드2 시작
스레드1실행
스레드1 종료
스레드2실행
스레드2 종료
```

##### synchronized(object)

synchronized(this)는 모든 block에 lock을 걸기 때문에 상황에 따라서는 비효율적일 수도 있다. synchornized(object)는 block마다 다른 lock이 걸리게 하는 방법이다.

```java
public class BlockObjectSync {
    public static void main(String[] args) {
        Object object1 = new Object();
        Object object2 = new Object();

        BlockObjectSync blockObjectSync = new BlockObjectSync();

        Thread thread1 = new Thread(() -> {
            System.out.println("스레드1 시작");
            blockObjectSync.sync1("스레드1", object1);
            System.out.println("스레드1 종료");
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("스레드2 시작");
            blockObjectSync.sync2("스레드2", object2);
            System.out.println("스레드2 종료");
        });

        thread1.start();
        thread2.start();
    }

    private void sync1(String msg, Object object) {
        synchronized (object) {
            System.out.println(msg + "실행");
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void sync2(String msg, Object object) {
        synchronized (object) {
            System.out.println(msg + "실행");
            try {
                TimeUnit.SECONDS.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```shell
스레드1 시작
스레드2 시작
스레드1실행
스레드2실행
스레드1 종료
스레드2 종료
```

#### static synchronized block

static method안에 synchronized block을 사용할 수 있다.  static 키워드 특성상 this를 사용할 수 없다. static synchronized method 방식과 차이는 lock 객체를 지정하고 block으로 범위를 한정 시킬 수 있다는 점이다.

## volatile

멀티쓰레드 환경에서 가시성에 대해서 문제가 되는 경우가 있는데 예를들어 Thread1에서 공유 자원이 Thread2에서는 cpu cache가 아직 RAM에서 읽어오지 않아서 이미 바뀐 데이터를 가지고 놀게되는 경우가 있다. 이 경우를 해결하기위해 volatile을 사용할 수 있다. volatile은 cpu cache를 거치지 않고 직접 ram을 읽고 쓰는 과정을 거칠 수 있게하는 키워드이다.

```java
public class NonVolatileKeyword {
    private static volatile boolean check = false;

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            int i = 0;
            while (!check) {
                i++;
            }
        });
        thread.start();

        Thread.sleep(1000);
        check = true;
    }
}
```

위 코드는 얼핏 보면 스레드 시작하고 1초 뒤에 종료될 것 같지만 실제로는 그렇지 않고 무한루프를 돌고있다. 이처럼 스레드별 cache가 동기화 되지 않는다면 문제가 발생할 수 있다

```java
public class VolatileKeyword {
    private static volatile boolean check = false;

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            int i = 0;
            while (!check) {
                i++;
            }
        });
        thread.start();

        Thread.sleep(1000);
        check = true;
    }
}
```

volatile 키워드 하나 붙임으로써 RAM을 직접 참조할 수 있게되어 무한루프를 돌지 않게 된다.

## atomic

syncrhonized의 문제는 blocking하여 다른 스레드의 접근을 막는 방법으로 동기화를 한다. 그러나 이것은 성능 문제가 있다. 특정 스레드가 lock을 걸어버리면 다른 스레드들은 접근하지 못하고 lock이 풀릴 때까지 블로킹 상태가 된다.  또한 blocking 상태에서 준비 혹은실행 상태로 바꾸기 위해 시스템 자원을 사용해야 한다. 이는 성능저하로 이어진다.

이러한 문제 때문에 non-blocking하면서 원자성을 보장하기 위한 방법이 atomic이다. atomic의 핵심 동작 원리는 CAS(Compare And Sweep) 알고리즘이다.

### CAS(Compare And Sweep)

![CAS](https://user-images.githubusercontent.com/37204770/178136896-ca98ba24-8bd0-4306-81e3-b994724b1a09.png)

CAS알고리즘은 다음과 같은 흐름이다.

1. 인자로 기존 값(Compared Value)와 변경 값(Exchanged Value)를 받는다.
2. 기존 값(Compared Value)과 메모리가 가지고 있는 값(Destination Value)과 같다면 변경 값(Exchanged Value)를 반영하고 true를 반환한다. 
3. 기존 값(Compared Value)과 메모리가 가지고 있는 값(Destination Value)과 다르다면 변경 값(Exchanged Value)를 반영하지 않고 false를 반환한다.

#### Atomic 예시

여러 Atomic 클래스가 있지만 예시로는 AtomicInteger 클래스를 사용한다.

```java
public class AtomicTest {
    private static int count = 0;

    public static void main(String[] args) throws InterruptedException {
        AtomicInteger atomicInteger = new AtomicInteger(0);

        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 50000; i++) {
                count++;
                atomicInteger.incrementAndGet();
            }
        });

        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 50000; i++) {
                count++;
                atomicInteger.incrementAndGet();
            }
        });

        thread1.start();
        thread2.start();

        Thread.sleep(5000);

        System.out.println(count);
        System.out.println(atomicInteger.get());
    }
}
```

```shell
97821 # count
100000 # atomicInteger
```

두개의 스레드가 동시에 하나의 변수에 접근하면서 증가 연산을 하고 있다. 하나는 count integer 변수이고 하나는 atomicInteger변수이다. count는 예상한 값인 100000이 아니고 atomicInteger의 경우만 100000이 되었다. 이처럼 atomic 변수는 원자성을 지킬 수 있게 해준다.

## 싱글톤 패턴

학교에서 배우는 싱글톤 패턴은 대개 다음과 같은 방식이다.

```java
public class SingleTon {
    private static SingleTon singleTon;
    
    private SingleTon() {}
    
    public static SingleTon getInstance() {
        if (singleTon == null) {
            singleTon = new SingleTon();
        }
        return singleTon;
    }
}
```

생성자를 감추며 `getInstance`를 호출할 때 인스턴스를 생성하는 방식이다. 단일 스레드 환경에서는 문제가 없지만 멀티스레드 환경에서는 문제가 된다. 스레드1이 null체크 블록을 넘어 새 인스턴스를 생성하고있고 아직 singleTon 변수에 할당되지 않았을 때 스레드2가 null체크 블록을 통과해버려 객체 생성을 두번하는 경우가 생길 수 있다. 이를 해결하기 위해 위에서 이야기한 `synchronized`와 `volatile` 키워드를 사용할 수 있다.

```java
public class SingleTon {
    private volatile static SingleTon singleTon;
    
    private SingleTon() {}
    
    public static SingleTon getInstance() {
        if (singleTon == null) {
            synchronized(SingleTon.class) {
                if (singleTon == null) {
                    singleTon = new SingleTon();
                }
            }
        }
        
        return singleTon;
    }
}
```

특이한점은 singleTon null체크를 두번하고 있는데 여러가지 이유가 있다. 우선 synchronized method를 사용 하지 않은 이유는 인스턴스를 생성하는 것이 아닌 단순히 사용하는 용도에서도 synchronized를 걸어버리면 다른 해당 객체의 getInstance를 사용할 때마다 접근하려하는 다른 스레드가 전부 blocking이 걸려버린다. 이는 성능 저하로 이어지므로 메소드에 걸 수가 없고, 같은 이유로 첫번째 null 체크를 synchronized block으로 감쌀 수 없다. 첫번째 null체크 이후 내부 null체크시 synchronized block으로 감싼걸 볼 수 있는데 이는 위에서 설명한 것처럼 null 체크를 여러 스레드가 동시에 통과할 수 있기 때문에 synchronized block으로 감싸 다른 스레드의 접근을 막고 내부에서 null 체크를 한번 더 하는 것이다. volatile 변수를 사용한 이유도 cpu cache를 거치지 않고 바로 ram에 읽고 쓰기 위함이다.

그러나 이러한 방법에도 문제가 생길수가 있는데 스레드1가 인스턴스 생성을 완료하기 전에 메모리 공간에 할당한다. 이 때 스레드2가 이미 인스턴스가 생성된 것으로 보고 singleTon 인스턴스를 사용하려 하지만 아직 생성 완료가 되지는 않았기 때문에 오작동할 가능성이 있다.

그래서 다음과 같이 싱글턴 객체를 생성한다.

```java
public class SingleTon {
    private SingleTon() {}
    
    public static SingleTon getInstance() {
        return Holder.INSTANCE;
    }
    
    private static class Holder {
        public static final SingleTon INSTANCE = new SingleTon();
    }
}
```

싱글턴 객체의 생성을 JVM의 원자적 특성을 이용해 생성하는 방식이다. 최초 SingleTon 클래스가 로드될 때 SingleTon이 가지고 있는 변수가 없으므로 아무것도 생성되지 않지만 getInstance() 메소드를 호출할때 Holder 클래스가 로드되어 INSTANCE 변수에 객체가 할당된다. 클래스가 로드되는 순간은 JVM 영역이기 때문에 synchronized나 volatile를 사용할 필요가 없고 동기화를 보장하면서 성능도 좋은 방식이다.

## 출처

[[Java\] synchronized 키워드란? (tistory.com)](https://steady-coding.tistory.com/556)

[[Java\] volatile 키워드란? (tistory.com)](https://steady-coding.tistory.com/555)

[[Java\] Java의 동시성 이슈 (tistory.com)](https://steady-coding.tistory.com/554)

[[Java\] atomic과 CAS 알고리즘 (tistory.com)](https://steady-coding.tistory.com/568)