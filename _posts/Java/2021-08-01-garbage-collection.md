---
title: 가비지 컬렉션(Garbage Collection, GC)
tags: java
---

## 가비지 컬렉션이란

가비지 컬렉션은 가비지(Garbage)는 JVM의 Heap과 Method Area에서 사용되지 않는 Object를 의미한다. 가비지 컬렉션은 이러한 가비지가 차지하고 있는 메모리 영역을 풀어주어서 메모리를 확보하는 역할을 한다.

### 문제점

프로그래머가 직접 Memory를 핸들링 할 필요가 없어지게 되었지만, 명시적인 메모리 해제보다 느리고 GC 순간 발생하는 Suspend Time(Stop-The-World)으로 인해 여러 문제가 생긴다. 그 때문에 보통 GC 튜닝을 하기도 한다.

> Stop-The-World: GC 실행을 위해서 JVM이 애플리케이션 실행을 멈춘다.

### Root Set

가비지는 사용되지 않는 Object를 말하는데, Object의 사용 여부는 Root Set과의 관계로 판단한다. Root Set에서 Reference 관계가 있다면 Reachable Object라고 판단해 사용하고 있는 Object로 간주한다.

다음은 Root Set의 Reachable Object 판단 방법이다.

- Local Variable Section, Operand Stack에 Object의 Reference 정보가 있다면 Reachable Object이다.
- Method Area에 로딩된 클래스 중 constant pool에 있는 Reference 정보를 토대로 스레드에서 직접 참조하진 않지만 constant pool을 통해 간접 link를 하고 있는 Object는 Reachable Object이다.
- Memory에 남아있고 Native Method Area로 넘겨진 Object의 Reference가 JNI(Java Native Interface) 형태로 참조관계가 있는 Object는 Reachable Object이다.

![img](https://1.bp.blogspot.com/-2VNE1qOlKVI/WwowNwmg6hI/AAAAAAAAV3Y/6mfAJhLWKog9KrxtyKCaJZzoUYyaSpu0wCLcBGAs/s1600/reachable.jpg)

#### Reachable but not Live Object

```java
public class Main {
    public static void main(String[] args) {
        Leak lk = new Leak();
        for (int i = 0; i < 900000000; i++) {
            lk.add(i);
            lk.remove(i);
        }
    }
}

class Leak {
    List<Object> arrayList = new ArrayList();

    public void add(int a) {
        arrayList.add("Memory Leak" + a);
    }

    public void remove(int a) {
        Object object = arrayList.get(a);
        object = null;
    }
}
```

결과

```
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
at java.base/java.lang.StringConcatHelper.newString(StringConcatHelper.java:348)
	at java.base/java.lang.invoke.DirectMethodHandle$Holder.invokeStatic(DirectMethodHandle$Holder)
	at java.base/java.lang.invoke.LambdaForm$MH/0x0000000100099440.invoke(LambdaForm$MH)
	at java.base/java.lang.invoke.Invokers$Holder.linkToTargetMethod(Invokers$Holder)
	at jvm.Leak.add(Main.java:20)
	at jvm.Main.main(Main.java:10)
```

`remove` 메소드는 list의 원소 오브젝트에 `null`을 줌으로써 객체가 사라지는 것처럼 보이지만 실제로는 Reference가 null로 바뀐 것일 뿐 실제로 `ArrayList`에 들어가 있는 객체가 사라지는 것은 아니다. 이런 경우에 OutOfMemoryError가 발생한 것이고 이렇게 사용되지 않는 것들을 Reachable but not Live 객체라고 한다. 이러한 객체가 많아지면 Heap에 **Memory Leak**이 발생했다고 한다.

### GC 목적

보통 메모리의 압박이 있을 때 수행된다. GC는 새로운 Object의 할당을 위해 한정된 Heap 공간을 재활용하려는 목적으로 수행된다. 할당이 해제되면 그 공간을 다시 재활용하여 Object를 할당하기 때문에 나간자리가 듬성듬성 해진다. 이 경우에 남은 공간의 크기가 Object 하나의 크기보다 큰 객체를 할당한 경우 재활용의 의미가 사라진다. 이를 Heap의 단편화(Fragmentation)이라고 한다. 이러한 현상을 방지하기 위해 Garbage Collector는 Compaction과 같은 다양한 알고리즘을 사용한다.

## GC 알고리즘

![Mark and Sweep Garbage Collection](https://plumbr.io/app/uploads/2015/05/Java-GC-mark-and-sweep.png)

위 그림은 root로부터 시작해서 이어지는 전체 개체 그래프를 나타낸다. 연결이 끊겨 더이상 사용할 수 없는 오브젝트들은 GC가 제거하게 된다. 연결 그래프가 계속 바뀔수 있기 때문에 GC동안에는 애플리케이션이 정지된다. 이를 Stop-The-World라고 한다. 이 일시정지 기간은 활성 개체 수에 따라 달라진다. 힙의 크기는 직접적인 영향이 없다. GC는 루트로부터 이어져 살아있는 객체에 Mark를 한다. 살아있지 않은 객체를 제거하는 방법은 다음과 같은 방법이 있다.

### Sweep

![Java GC sweep](https://plumbr.io/app/uploads/2015/06/GC-sweep.png)

이 방법은 mark되지 않은 객체들을 모두 메모리를 사용할 수 있는 공간으로 바꾸는 방법이다. 가장 간단하지만 메모리 파편화가 일어나 할당을 할 수 있는 만큼의 공간이 안나오면 Free Space가 존재해도 할당에 실패할 수가 있다.

### Compact

![Java GC mark sweep compacting](https://plumbr.io/app/uploads/2015/06/GC-mark-sweep-compact.png)

Sweep이후 파편화된 메모리 영역을 시작지점으로 모아주는 것이다. Suspend Time이 늘어나는 단점이 있다. 항상 공간에 여유가 있으며 파편화 문제도 발생하지 않는다.

### Copy

![Java GC Mark and Copy Collector](https://plumbr.io/app/uploads/2015/06/GC-mark-and-copy-in-Java.png)

객체의 메모리 영역을 한곳으로 모아준다는 것이 compact와 유사하지만 메모리의 다른 영역에 모아진다는 것이다. Live Object들을 수용할 수 있을만큼의 메모리영역이 충분히 커야한다는 단점이 있다.

## Hotspot JVM의 Garbage Collection

Hotspot JVM은 Generational Collection 방식을 사용한다. Heap을 Object의 Generation 별로 Young Area, Old Area를 구분하고  Young Area는 Eden Area와 Survivor Area로 구분해서 사용한다.

GC 메커니즘은 두가지 가설을 가지고 있다.

- Object는 생성된 후 금방 Garbage가 된다.
  - 이는 새로 할당되는 Object가 모이는 곳은 단편화가 발생할 확률이 높다.
  - Sweep 작업(Mark되지 않은 Object를 제거)을 수행하면 단편화가 발생하게 된다.
  - 그 때문에 Object 할당만을 위한 전용공간인 Eden Area를 만들게 되었고, GC에서 살아남은 Object들을 피신시키는 Survivor Area를 따로 구성했다.
- Older Object가 Young Object를 참조할 일은 드물다.



Garbage를 추적하는 부분은 Tracing 알고리즘을 사용한다.

Root Set에서 Reference 관계를 추적하고 Live Object는 Marking한다.

Marking 작업도 Young Generation에 국한 되는데 Marking 작업은 Memory Suspend 상황에서 수행되기 때문이다.

전체 Heap에 대한 Marking 작업은 Suspend Time을 길게 가져간다. 만일 이런 상황에 Older Object가 Young Object를 참조하는 상황이 있다고 가정하면 존재 여부 체크를 위해 Old Generation을 모두 찾아 다닐 수 있고 Suspend Time도 길어진다. 때문에 Hotspot JVM은 Card Table(이벤트 프로그램)이라는 장치를 마련했다.



#### Card Table

Old Generation의 Memory를 대표하는 별도 Memory 구조이다. 만일 Old에서 Young의 오브젝트를 참조하면 Old 오브젝트의 시작주소를 Dirty로 표시하고 (Flag같은) 해당 내용을 Card Table에 기록한다. Old 오브젝트가 Young 오브젝트를 참조할때 모든 Old 오브젝트의 참조를 확인하지않고 Card Table을 확인하여 참조하는지 확인한다. 이후 해당 Reference가 해제되면 표시한 Dirty Card도 사라지게 하여 Reference 관계를 쉽게 파악할 수 있게 한다. Card는 Old Area의 Memory 512Bytes에서 1Byte 공간을 차지한다.

#### TLAB(Thread-Local Allocation Buffers)

GC가 발생하거나 객체가 각 영역에서 다른 영역으로 이동할때 발생하는 병목 현상에 대응하기 위해 스레드 로컬 할당 버퍼를 사용한다. Heap 공간에 스레드 마다 할당 주소 범위를 부여하여 그 범위 내 동기화 작업 없이 빠른 할당을 하고, TLAB이 부족하거나 최초 할당될 때 동기화이슈가 발생하지만 Object Allocation 횟수에 비하면 걸리는 시간은 상당히 줄어든다.

### GC 대상 및 범위

GC 대상은 Young, Old Generation, Permanent Area이다. Young Generation에서 발생하는 GC를 Minor GC라고 한다. Old Generation에서 발생하는 GC를 Full GC(Major GC)라고 한다. Permanent는 너무 많은 Class가 로드되어 Free Space가 없어졌을 경우 GC가 발생할 수 있다.

### GC 주요 옵션

> HotSpot JVM - Client : 바이트 코드로부터 최대한 많은 정보를 뽑아내어 실제 동작하는 코드블럭에 대한 최적화를 수행한다.
>
> HotSpot JVM - Server: 죽은 코드 삭제, loop 변수 끌어올리기, 공통 부분식 제거, 상수 지연, 전역 코드 이동등 최적화를 한다.

| 옵션                              | 상세설명                                                     |
| --------------------------------- | ------------------------------------------------------------ |
| -Client                           | Client Hotspot VM으로 구동한다.                              |
| -Server                           | Server Hostpot VM으로 구동한다.                              |
| -Xms\<Size\><br />-Xmx\<Size\>    | Young Generation의 최소, 최대 크기                           |
| -XX: PermSize                     | Permanent Area의 초기 크기                                   |
| -Xss\<Size\>                      | Stack 사이즈                                                 |
| -XX:MaxPermSize                   | Permanent Area의 최대 크기 (디폴트=64mb)                     |
| -XX:TargetSurvivorRadio=\<value\> | Survivor Area가 지정된 값%만큼 차면 Minor GC가 발생한다. (디폴트=50) |
| -XX: MinHeapFreeRadio=\<percent\> | 전체 Heap 대비 Free Space가 지정수치 이상이면 -Xmx까지 확장(디폴트=40) |
| -XX: MaxHeapFreeRatio=\<percent\> | 전체 Heap 대비 Free Space가 지정수치 이상이면 -Xms까지 축소(디폴트=70) |

다음은 Java8 이후 변경된 옵션

| 옵션                 | 상세                       |
| -------------------- | -------------------------- |
| -XX:MaxMetaspaceSize | -XX:MaxPermSize에서 변경됨 |
| -XX:MetaspaceSize    | -XX:PermSize에서 변경됨    |

> Server Class 기준 Default값 동작 방식
>
> - Default Old 영역의 크기는 Young 크기의 두배
> - Survivor 영역의 크기는 Young 영역의 1/10
> - Permanent 영역의 Max값은 64MB
> - 최초 Young 영역의 크기는 약 2M, NewRatio 값에 의해 최대값이 조정됨
> - 전체 Heap에서 Free Size가 40% 이하면 Heap Size가 증가, 70% 이하면 Heap Size가 감소
> - JVM의 최대, 최소값 설정을 안하면 기본 64MB를 최대 Memory로 지정

> Hotspot JVM 옵션 종류
>
> - Standard Option: 옵션 앞에 '-'를 붙여 표기
> - Non-Standard Option: JVM 및 버전마다 약간 다르게 표기
>
> - '-X'를 붙이는건 Macro 한 측면의 제어를 위해 사용
> - '-XX:'를 붙이는건 Micro 측면의 제어를 위해 사용
> - Boolean의 경우 On을 표기하면 '-XX:+\<옵션\>'처럼 '+'를 붙여 사용
> - Off의 경우 '-XX:-' 와 같이 '-'를 사용, default로 설정된 옵션을 끌때 주로 사용
> - '-XX:\<옵션\>=\<numeric\>'은 용량과 같은 단위를 포함하는 경우가 많음
> - '-XX:\<옵션\>=\<string\>'의 형식을 사용 이는 특정 파일 또는 Path 지정에 많이 사용

### Garbage Collector 종류

#### Serial Collector

- Option=-XX:+UseSerialGC
- Young Generation Collection 알고리즘 = Serial
- Old Generation Collection 알고리즘 = Serial Mark-Sweep-Compact
- client class의 기본 collector
- 한 개의 thread가 serial로 수행됨
- Single CPU를 사용하고 1개의 Thread로 GC를 수행함

#### Parallel Collector

- -XX+UseParellelGC
- Young Generation Collection 알고리즘 = Parallel Scavenge, 멀티 스레드가 동시에 GC를 수행
- Old Generation Collection 알고리즘 = Serial Mark-Sweep-Compact = 싱글 스레드 방식
- 모든 자원을 투입하여 GC를 빨리 끝내는 전략
- 대용량 Heap 적합
- Survivor Area로 오브젝트를 복사하는 과정을 여러 스레드가 동시에 수행해 Suspend Time이 발생하지만 그만큼 Suspend Time을 단축시킬 수 있음
- PLAB을 사용하여 같은 메모리 공간을 두 스레드가 접근했을 때 Corruption이 일어나는것을 방지한다

> PLAB(Parallel Allocation Buffer)
>
> GC Thread 작동시 스레드마다 Old Generation의 일정 부분을 할당하고 다 사용하면 다시 Buffer를 재할당한다. Old Area에 발생하는 단편화는 스레드가 자신의 버퍼를 할당 받고 사용하지 않거나 어쩔 수 없이 발생하는 버퍼 내 자투리 공간 때문이다.
>
> TLAB은 Young Area의 빠른 할당을 위한 것이고 PLAB은 Promotion 과정 중 동기화 문제를 회피하기 위한 것이다.

#### CMS Collector

- -XX:+UseConcMarkSweepGC
- Young Parallel
- Old Concurrent Mark-Sweep
- Suspend Time을 분산시켜 체감 Pause Time을 줄임
- 힙 메모리 영역이 클때 적합
- 동작방식
  - Initial Mark Phase: 싱글 스레드만 사용, 애플리케이션이 중지되고 애플리케이션에서 직접 Reference 되는 Live Object만 구별
  - Concurrent Mark Phase: 싱글 스레드만 사용, 애플리케이션이 중지되지 않고  Initial Mark Phase에서 선별된 Live Object가 Reference하고 있는 Object를 추적해 Live 여부 구별
  - Remark Phase: 멀티 스레드가 사용되며 애플리케이션 중지. 이미 마킹된 오브젝트 다시 추적, Live 여부 확정, 모든 리소스 투입
  - Concurrent Sweep Phase: 싱글 스레드만 사용 애플리케이션은 중지되지 않고 최종 Live가 아닌 Object를 제외한 Dead Object를 제거. 단 Sweep 작업만 하고 Compaction 작업은 수행하지 않는다.

> Free List
>
> 승격(Promotion) 할당 할때 Young Area에서 승격된 Object와 크기가 비슷한 Old Area의 Free Space를 Free List에서 탐색한다. GC 수행중에 Free List에서 적절한 Chunk 크기를 찾아 Allocation 해야해서 시간도 오래 걸린다.

CMS Collector의 단점은 애플리케이션이 중지하지 않으면서 GC가 이루어지는 작업이 수행되는 단계가 있기 때문에 Reachable Object가 Dead Object가 될 수 있다. Initial Mark Phase에서 GC 수행 대상 여부로 판단되지 않은 객체가 Old Area로 Promotion되었고 이 객체가 곧바로 Dead Object가 되더라도 GC로 수거되지 않는다. 때문에 이러한 객체는 다음 GC때 사라지게 된다. 이러한 객체는 Float Garbage라고 한다. 이는 잠재적으로 Old Area를 확장시키게 되는 요인이다. 또한 Young Area가 Suspend Time을 갖고 CMS의 Remark phase를 연달아 수행하면 Suspend Time보다 길어진다. 때문에 CMS Collection에서 Schedule을 고려한다.

#### Incremental Mode of CMS Collector

CMS Collector는 Incremental Mode를 지원한다. 이는 정교한 스케줄링으로 Concurrent phase를 작은 시간 단위로 쪼개 MinorGC와 겹치지 않게한다. Duty Cycle을 두어 한개의 CPU를 점유하는 시간 제한을 둬서 GC의 일량을 조절한다.

#### Parallel Compaction Collector

- -XX:+UseParallelOldGC
- Parallel Scavenge
- Parallel Mark-Sweep-Compact

Parallel Collector에서 Old Area에 새로운 알고리즘 추가된 것으로 멀티 cpu에 유리하다.young 영역의 GC는 Parallel Collector와 동일하지만 Old 영역은 3단계를 거친다.

- Mark Phase : 살아있는 객체를 식별하여 표시해 놓는 단계
- Summary Phase: 이전에 GC를 수행하여 compaction된 영역에 살아있는 객체의 위치를 조사
- Compact Phase: compaction을 수행하는 단계

Mark 단계에서는 Old Area의 Region이라는 단위로 균등하게 나누고 Live Object를 체크한다. Live Object의 사이즈, 위치 정보 등은 Region에 갱신된다.

Sumary 단계에서는 싱글 스레드만 GC를 수행하고 나머지는 애플리케이션을 수행한다. Mark 단계 결과를 대상으로 작업하는데 Region 단위이고 GC 스레드가 Region 통계 정보로 각 Region의 Destiny를 평가한다. Destiny는 각 Region별 Reachable Object의 밀도를 나타낸다. 이를 바탕으로 Dense prefix를 설정하게 된다. Dense Prefix는 Compaction이 일어날 때 해당 Prefix보다 Memory 번지가 작은 곳에 있는 Region은 이후 GC에서 제외시킨다. 그래서 기존 compaction에서는 한쪽으로 reachable object를 몰아줬던 것과 달리 destination region을 두어 Region마다 배정된 Thread가 Garbage Object를 Sweep한다. compaction의 범위를 줄여 GC 소요시간을 줄이는 것이다.

#### G1 Collector

CMS Collector에 비해 Pause Time을 개선하였다. 물리적 Generation 구분을 없애고 전체 힙을 1mb단위 region으로 재편한다. Region 별로 순차적인 작업이 진행되고 Remember Set을 이용한다. Gatbage로 대부분 이루어진 Region은 Old Region에서 다른 Old Region으로 compaction이 이루어진다. remember set은 region 외부에서 들어오는 참조 정보를 가진다. marking 작업시 trace 일량을 줄여준다.

G1 Collector는 4단계, 세부적으로 6단계로 이루어진다.

1. Young GC

   멀티 스레드로 작업한다. Live Object는 Survivor Region 혹은 Old Region으로 이동한다. 새로운 Object가 할당되는 Young Region은 Survivor Region 근처에 있는 Region이 된다.

2. Concurrent Mark - Marking

   싱글 스레드, 이전 단계 때 변경된 정보를 바탕으로 Initial Mark를 빠르게 수행한다.

   snapshot-at-the-beginning(SATB) Marking 알고리즘을 사용한다. 이는 GC 시작당시 Reference 기준으로 Live Object의 Reference를 추적하는 방식이다.

3. Current Mark - Remarking

   멀티 스레드로 동시작업, 각 Region마다 Reachable Object의 Destiny 계산, 이 후 Garbage Region은 다음 단계로 안가고 해지된다.  

4. Old Region Reclaim - Remarking

   Concurrent 작업, 멀티스레드 사용, Live Object의 비율이 낮은 몇개의 Region을 골라낸다.

5. Old Region Reclaim - Evacuation Pause

   Remark 단계에서 골라낸 OldRegion은 Young Region과 같은 식으로 비운다. Remark된 Young Region과 Old Region들까지 같이 Collection을 한다. 결과적으로 Heap에 몇개의 Garbage Object만 존재하는  Live Object의 Destiny가 비교적 높은 Region들만 남게된다.

6. Compaction Phase

   Concurrent 작업을 수행한다. 주 목적은 Free Space를 병합해 단편화를 방지한다.

## 참고

https://plumbr.io/handbook/garbage-collection-algorithms#sweep