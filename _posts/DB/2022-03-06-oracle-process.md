---
title: 오라클의 여러 프로세스
tags: oracle database
layout: post
description: 오라클은 여러 프로세스로 실행이 되고 그중 서버 프로세스와 백그라운드 프로세스가 있다.
---

![l9791185890302](https://user-images.githubusercontent.com/37204770/151164431-f00d8ce3-9fe8-4269-ade2-eb43cbc84240.jpg)



> 원문 : 『그림으로 공부하는 오라클 구조 개정판』 , 스기타 아츠시 외 4명 지음, 제이펍

## 오라클의 역할 이미지

오라클을 '창고 업자' 라고 가정하면 오라클은 고객의 짐을 맡아주고 창고에 보관하고 요청에 따라 짐을 돌려준다. DBMS의 기본 동작은 데이터를 맡아서 보관하고, 요청에 따라 데이터를 반환한다.

DBMS에 다음과 같은 의문이 생길 수 있고 답은 다음과 같다.

- 시스템에서 DB는 어떻게 사용되고 있는가?

  - DB는 데이터를 보관하는 장소로 사용되고 있으므로, 애플리케이션이 데이터를 사용하고 싶을 때 사용한다.
  - 애플리케이션 자체적으로 변수와 같은 데이터를 저장하기는 하지만 메모리에 임시로 저장되는 것이다. 그러므로 일반적으로 애플리케이션이나 프로세스 간에 데이터를 공유할 수 없다.

- SQL*Plus라는 것도 사용하던데 이를 애플리케이션이 사용해도 괜찮은가?

  - SQL*Plus는 오라클 클라이언트 도구로 오라클 데이터베이스 사용자가 상호작용을 하기 위한 사용자 인터페이스이다.

  - SQL*Plus는 대부분 데이터베이스의 관리를 위해서 사용된다.

  - 실제 시스템에서는 JDBC 같은 것을 통해 SQL문을 수행한다.

    ```java
    // 자바에서 사용 예
    public class Main {
        public static void main(String[] args) {
            Connection conn = DriverManager.getConnection(url, user, pwd);
            Statement st = conn.createStatement();
            ResultSet rs = st.executeQuery("SELECT no, name FROM test WHERE no = 1");
        }
    }
    ```

    

## 데이터베이스의 데이터는 모두의 것

### 실제 모습에 가까운 오라클 이미지

![oracle image](https://user-images.githubusercontent.com/37204770/156913832-9bc523d9-a3ca-4337-a7b8-0e7c452f37ff.png)

고객인 프로그램은 여러 개 존재할 수 있고, 받는 입장인 오라클의 프로세스도 여러 개 존재한다.

데이터베이스에서는 여러 프로세스나 사용자가 하나의 데이터베이스에 접근하기 때문에 데이터가 동시에 변경되는 경우가 있기 때문에 다른 사용자나 애플리케이션을 신경쓸 수 밖에 없다. 데이터는 기본적으로 여러 사용자나 프로그램이 데이터베이스의 데이터를 공유하고 있다 라고 생각하는 것이 중요하다.

> 프로세스는 실행 중인 상태에 있는 프로그램을 의미한다. 실행 중인 상태이기 때문에 메모리나 자원을 가지고 있다. materialization(실체화) 했다고 할 수 있다.
>
> 스레드는 프로세스 내에 존재하는 실행 단위를 말한다. 하나의 프로세스 안에서 병렬로 작업을 처리하고 싶을 때 사용한다.
>
> 프로세스와 스레드의 다른 점은 부하의 크기와 메모리를 공유하는지 여부이다. 프로세스는 각자 독립적으로 수행되어 자원을 독자적으로 사용하므로 부하가 크고, 메모리를 공유하지 않는다. 스레드는 한 프로세스 안에서 수행되므로 부하가 적지만 스레드끼리 메모리를 공유한다.

### 엑셀과 DBMS의 차이

액셀은 하나의 PC에서 작동하고 단일 사용자가 사용하는 것이다. 그에 비해 DBMS는 다수의 사용자나 애플리케이션이 데이터를 공유한다는 전제로 만들어져 있어서 많은 사용자가 동시에 검색하거나 변경할 수 있다. 다수의 사용자가 동시에 데이터를 처리하는 경우에는 사용자의 실수로 데이터가 손상되지 않도록 처리하는 Lock이라는 장치를 가지고 있다. Lock이 걸린 데이터를 조작하려고 하면 Lock이 풀릴 때까지 기다린다와 같은 방식이다.

![dblock](https://user-images.githubusercontent.com/37204770/156914968-7ed23d74-510a-44d4-9df1-7867b32cbdfd.png)

▲Lock의 종류에 따른 데이터베이스 변경 작업

## 오라클이 여러 개의 프로세스로 구성된 이유

오라클이 여러 프로세스로 이루어진 이유 중 하나는 다중 처리를 위함이다. SQL 처리는 길게는 몇 시간 이상이 걸리는 경우도 있다. 현재 사용자가 작업을 하는 동안 다른 사용자는 아무 작업도 하지 못하고 마냥 기다릴 수는 없다. 디스크는 메모리 액세스에 비해서 속도가 느리기 때문에 느린 I/O가 반복되고 있는 동안 CPU와 같은 자원이 방치되어 있는 것은 아깝기 때문에 I/O가 반복되는 그 순간에는 다른 SQL을 처리하는 것이 효율적이다.

이러한 이유 때문에 여러 개를 동시에 처리하기 위해 여러 개의 프로세스를 사용한다. 오라클은 여러 개의 프로세스를 실행하는 방식으로 병렬 처리를 한다.

단, 오라클은 같은 프로세스가 여러 개 작동하는 것이 아니다. 다른 역할을 가진 여러 가지 프로세스가 존재하고 있다.

> 자원이 아까운 것이란?
>
> 컷 오버(cut-over, 개발 종료 후 신규 시스템 운영 시작) 직후나 성능 테스트 단계에서 복잡한 업무가 다양하게 유입되기 시작하면 성능이 매우 좋지 않은 상황에 직면하게 될 것 이다. 프로세스를 늘려 SQL문을 동시에 처리한다고 해도 그것을 처리할 CPU는 한정되어 있기 때문이다. 그러므로 이 분야 종사하는 사람이라면 자원을 효율적으로 사용해야 한다.

## 서버 프로세스와 백그라운드 프로세스

오라클은 다음 두 가지 프로세스로 구성되어 있다.

- 서버 프로세스 : SQL문을 처리하는 프로세스
- 백그라운드 프로세스 : 서버 프로세스를 지원하는 프로세스

### 백그라운드 프로세스의 일

백그라운드 프로세스와 역할은 다음과 같다.

- ora_dbwX_XXXXXX : DBWR(DataBase Writer)이라고 부른다. 데이터를 디스크에 기록한다.
- ora_lgwr_XXXXXX : LGWR(LoG WRiter)이라고 부른다. 로그(데이터 변경 이력)를 디스크에 기록한다.
- ora_pmon_XXXXXX : PMON(Process MONitor)이라고 부른다. 프로세스를 감시하고, 프로세스에 장애(비정상 종료)를 발견했을 때 정리하는 역할을 한다.
- ora_arcX_XXXXXX : ARCH(ARCHiver)라고 부른다. 로그 데이터를 아카이브함 (장기 보관하기 위해 별도의 파일로 보관)
  - XXXXXX는 DB를 식별하는 SID가 들어간다.
  - X에는 0과 1 같은 프로세스의 수를 표현하는 숫자가 들어간다.

![oracle processes](https://user-images.githubusercontent.com/37204770/156915780-2a0783da-7daa-4ce6-bc14-f9290ffd00f3.png)

오라클 클라이언트는 서버 프로세스와 통신한다. 서버 프로세스는 고객 담당으로 볼 수 있고 고객 담당을 지원하는 스태프가 백그라운드 프로세스라고 볼 수 있다.

## 각 프로세스가 수행하는 처리

### SQL 문의 처리에 필요한 작업

1. SQL문 수신
2. SQL문 분석
3. 데이터 읽기
4. 결과 회신(공유 서버 구성에서는 1과 4는 서버 프로세스와는 별도의 프로세스가 수행한다.)

오라클은 SQL문을 수신하지 않으면 작업을 처리하지 않고, 파싱(parsing)이라고 불리는 절차를 거치지 않으면 어떤 테이블에 접근해야 하는지 조차 알 수 없다. 데이터를 읽어오지 않으면 데이터를 처리할 수 없으며 처리하더라도 결과를 클라이언트로 전달하지 않으면 종료 할 수 없다. 데이터 기록과 같은 것은 SQL문의 결과를 회신하는 것과는 상관이 없다. 그 때문에 DBWR과 같은 것이 데이터를 디스크에 기록하는 일을 한다. 서버 프로세스가 디스크에서 데이터를 읽어오기는 하지만 기록은 하지 않는 이유가 이런 부분에서 기인한 것이다.

다시 한번 프로세스의 역할을 정리하자면 SQL의 결과를 회신하는데 필요한 것은 서버 프로세스가 수행하고, 그 이외의 것은 백그라운드 프로세스가 수행한다. (단 예외 사항도 있긴하다.)

> 포어그라운드 프로세스
>
> 서버 프로세스를 말한다. 백그라운드와 반대의 의미를 가진 서버 프로세스는 이렇게 불리기도 한다.

## 요약

- 데이터베이스는 모두 공유해서 사용한다.
- 애플리케이션 같은 클라이언트가 여러 개 존재하고 여러개의 SQL문이 오라클에 전달된다.
- 오라클 위에서 여러 개의 SQL문이 동시에 동작하고 있다.
- 서버 프로세스는 SQL문의 결과를 가능하면 빠르게 회신하기 위해 일을 한다
- 서버 프로세스를 도와주는 백그라운드 프로세스가 존재한다.

![server and background processes](https://user-images.githubusercontent.com/37204770/156916607-29089ee3-717f-422e-a6aa-ab8c8d250f59.png)

▲ 백그라운드 프로세스는 이들 이외에도 존재한다. 위는 기본적인 동작만을 말하고 있다.



#### 튜닝할 때는 어떤 프로세스를 봐야하는가?

실제로 작업을 처리하는 서버 프로세스이다. 백그라운드 프로세스는 서버 프로세스를 방해하지 않는 한 볼 필요는 없다.

#### OS에서 확인해보니 서버 프로세스가 디스크에서 데이터를 대량으로 읽어온다. 이것을 문제가 있다고 말할 수 있는가?

문제가 있다고 말할 수 없다. 서버 프로세스는 디스크에서 데이터를 읽어 오는 일을 하므로 읽어온다는 것 자체를 이상이라고 할 수는 없다. 나머지는 SQL문을 보았을 때 데이터를 대량으로 읽는 것이 타당한지 아닌지 조사해야 한다.

> 오라클 RAC란?
>
> RAC란 Real Application Clusters의 약자로서 오라클 데이터베이스의 클러스터 기술을 말한다. 여러 개의 서버상에 가동된 인스턴스를 하나의 데이터베이스 처럼 사용한다. 여러 개의 서버로 구성되어 있지만 데이터의 일관성을 위해 스토리지는 함께 사용한다.
>
> ![GUID-208BC74A-B44F-4A45-B469-B88740DBE763-default](https://user-images.githubusercontent.com/37204770/156916920-0549dc74-b7e8-4bc5-a293-bd5c4b89f6ce.png)
>
> 일반적인 HA(High Availability. 고가용성) 구성과 비교해서 RAC 구성이 가진 특징은 각 서버가 액티브(Active)/스탠바이(Standby)가 아닌 액티브/액티브 구성이므로 서버의 CPU나 메모리를 100% 활용할 수 있다. 한대가 장애가 발생해도 나머지 인스턴스로 작업을 처리할 수 있기 때문에 지속적인 운영이 가능하다.
>
> 스토리지를 함께 사용하기 때문에 변경 정보는 디스크에 기록될 때까지 다른 서버에서 확인할 수 없는 것이 아닌가? 라고 생각할 수도 있지만 RAC는 메모리에 캐시되어 있는 블록을 서버 간에 공유하므로 디스크의 I/O를 기다릴 필요없이 데이터의 일관성을 유지할 수 있는 구조를 가지고 있다.