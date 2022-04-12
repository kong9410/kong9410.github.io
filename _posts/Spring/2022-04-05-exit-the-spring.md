---
title: 스프링 애플리케이션을 안전하게 종료시키는 방법(Graceful Shutdown)
tags: spring java
layout: post
description: 스프링 애플리케이션을 안전하게 종료시키는 방법과 데몬 스레드에 대해
---

## 프로세스를 안전하게 종료해야 하는 이유

사용중인 프로세스를 리소스 정리나 Task 정리 없이 종료를 하게되면 진행중인 사항을 모두 잃게되거나 생각치 못한 버그가 발생할 수 있다. 블루-그린 배포(blue-green deployment) 환경을 가진 경우에는 기존에 실행중인 애플리케이션 종료 없이 배포를 할 수 있지만 롤링 배포(rolling deployment) 환경에서는 실행중인 애플리케이션을 순서대로 종료를 해야하기 때문에 프로세스를 신중하게 종료해야한다. 가령 사용자가 쇼핑 웹사이트에서 결제를 진행중에 애플리케이션이 강제 종료된다면 결제는 됐는데 주문이 안되는 경우가 발생할 수 있기 때문이다.

## 프로세스 종료

리눅스 환경에서 프로세스를 종료하는 방법은 `kill` 명령어이다. 조금 더 정확히 말하자면 프로세스에 시그널을 보내는 명령어이다. `kill` 명령어는 `kill {options} <pid>`와 같은 방법으로 사용할 수 있다. 시그널 종류는 다음과 같은 것들이 있다.

![What all signals you can send using kill](https://www.howtoforge.com/images/usage_of_pfsense_to_block_dos_attack_/kill-l-option.png?ezimgfmt=rs:500x165/rscb5/ng:webp/ngcb5)

이중에 OS에서 강제적으로 종료시킬 수 있는 시그널은 `9(SIGKILL)` 시그널이다. `kill -9 [pid]`입력하면 현재 실행중인 프로세스가 애플리케이션 종료 절차를 진행하지 않고 강제적으로 종료가 된다.

그러나 이러한 `SIGKILL` 시그널을 보내는 것을 권장하지 않는다. 프로세스가 실행도중 강제로 종료가된다면 진행중인 사항을 잃거나 비지니스 로직이 전부 수행되지 않은 상태에서 롤백도 되지 못한 채로 끝날 수 있기 때문이다.

`15` 시그널은 프로세스가 종료절차를 수행한 뒤에 프로세스를 종료하게된다. 물론 프로세스가 해당 시그널을 받았을 때 처리하는 시그널 핸들러가 없다면 여전히 진행사항을 잃게 된다. 프로세스의 종료까지 신경 쓴 애플리케이션이라면 `9` 시그널보단 `15`를 사용하기를 권장한다.

참고로 아무런 시그널을 주지 않고 `kill <pid>`를 사용했을 때의 기본값은 `15(SIGTERM)`이다.

> SIGTERM vs SIGINT
>
> SIGINT는 유저가 직접 프로그램 종료를 지시했다는 점이다. 결과적으로는 프로그램이 종료되기 때문에 SIGTERM과 다를바 없다. 물론 프로그램마다 해당 시그널을 받았을 때 결과를 다르게 처리할 수 있도록 커스텀 할 수 있다.

## JVM 종료

JVM이 종료되는 두 가지 경우가 있는데 하나는 예정된 절차대로 종료되는 경우이고 또 하나는 예기치 못하게 임의로 종료되는 경우이다. 절차에 맞게 종료되는 경우는 다음과 같다.

1. non-daemon thread들이 모두 종료 됨
2. `System.exit()` 메소드가 호출 됨
3. `SIGINT` 시그널을 받음
4. `Ctrl + C`입력을 받음

예기치 못하게 종료되는 경우는 다음과 같다.

1. `Runtime.getRuntime().halt()`

2. `SIGKILL` 시그널을 받음

3. 호스트 OS가 죽었을 때

   ​

> **System.exit() vs Runtime.getRuntime().halt()**
>
> 1. System.exit()
>
>    JVM을 멈추는 메소드로 shutdown sequence가 수행된다.  shutdown sequence는 등록되어 있는 shutdown hook을 실행하고 완료될 때까지 기다린다. 모든 종료자(finalizer)가 실행되고 마지막으로 JVM을 종료하게 된다.
>
> 2. Runtime.getRuntime().halt()
>
>    실행 중인 JVM을 강제 종료하는데 사용할 수 있는 `halt` 메소드는 `exit`메소드와는 달리 JVM shutdown sequence를 실행하지 않는다. 따라서 `halt` 메소드는 shutdown hook이나 finalizer가 실행되지 않는다.

> **데몬스레드(Daemon Thread)**
>
> 자바는 user thread와 daemon thread 두 종류의 스레드를 사용한다. 
> **user thread**는 높은 우선순위의 스레드로 JVM은 user thread가 완료되기 전까지는 JVM을 종료하지 않는다. 
> **daemon thread**는 낮은 우선순위의 스레드로 user thread의 보조 역할을 하는 스레드이다. daemon thread는 오로지 user thread가 실행되는 동안에 필요한 스레드이고 JVM이 종료될 때 daemon thread의 완료여부는 확인하지 않는다. 때문에 모든 user thread가 실행을 완료하면 daemon thread가 무한 루프를 돌고있어도 JVM이 종료되게 된다. daemon thread는 주로 Garbage Collection, Resource Cleanup과 같은 백그라운드 테스크를 수행한다.
>
> ```java
> public class MyThread extends Thread {
>     @Override
>     public void run() {
>         if (Thread.currentThread().isDaemon()) {
>             while (true) {
>                 System.out.println("daemon running");
>             }
>         } else {
>             for (int i = 0; i < 3; i++) {
>                 System.out.println("user running");
>             }
>         }
>     }
> }
> ```
>
> ```java
> public class ThreadMain {
>     public static void main(String[] args) {
>         Thread userThread = new MyThread();
>         Thread daemonThread = new MyThread();
>         daemonThread.setDaemon(true);
>
>         daemonThread.start();
>
>         userThread.start();
>     }
> }
> ```
>
> ```shell
> user running
> user running
> user running
> daemon running
> daemon running
> <중략>
> daemon running
> daemon running
> Process finished with exit code 0
> ```
>
> Daemon 스레드 지정은 `setDaemon`메소드를 사용해서 지정을 할 수 있다. `daemonThread`의 `run`은 무한루프를 돌고 있지만 메인 스레드인 `userThread`가 끝나자 JVM이 종료된 것을 확인할 수 있다

### Shutdown Hook

JVM을 사용하면 시스템 종료를 완료하기 전에 사전에 등록된 메소드들을 실행할 수 있다. 이러한 기능은 리소스 같은 것을 해제하거나 진행중인 Task를 완료하는 방법으로 사용된다. 이러한 종료 함수를 shutdown hook이라고 한다. shutdown hook는 기본적으로 초기화되어 있지만 unstarted한 스레드다. JVM의 종료 프로세스가 실행되면 등록되어 있던 hook들이 실행되고 모든 hook이 실행이 끝나면 JVM이 종료가 된다.

#### Hook 추가 방법

`Runtime.getRuntime().addShutdownHook({Thread})`에 hook Thread를 등록하면 된다.

```java
Thread printingHook = new Thread(() -> System.out.println("shutdown")));
Runtime.getRuntime().addShutdownHook(printingHook);

System.exit(0);
```

```shell
shutdown...

Process finished with exit code 0
```

다만 ShutdownHook은 어디까지나 JVM이 정상 종료되었을 때 수행이된다. 앞서 말한 강제적인 종료 방법을 사용한다면 ShutdownHook은 실행되지 않는다.

## 스프링 애플리케이션 종료

웹 애플리케이션이라면 애플리케이션이 종료될 때 더이상의 요청을 받지 않아야하며 이미 받은 요청이 존재한다면 해당 요청들을 모두 처리한 후에 종료해야한다. 그렇지 않으면 사용자는 에러 응답을 받는 등의 문제가 발생할 수 있다.

배치 애플리케이션이라면 처리중이던 Task를 모두 처리하고 종료하거나 처리된 지점까지의 Save Point를 만들어 다시 실행시켰을 때 Save Point 지점부터 다시 처리해야 한다.

Kafka를 사용한다면 어느 offset까지 처리하였는지 commit을 하거나 별도 저장해야 한다.



### SpringBoot 2.3 이상 버전

(원문 : [JVM의 종료와 Graceful Shutdown (tistory.com)](https://effectivesquid.tistory.com/entry/JVM%EC%9D%98-%EC%A2%85%EB%A3%8C%EC%99%80-Graceful-Shutdown))

**Spring Boot 2.3** 이상부터는 웹 서버를 종료할 때 graceful shutdown을 할지 즉시 종료할지 정하는 property가 존재한다. `application.properties`에 `server.shutdown`옵션으로 설정할 수 있고 기본값은 `server.shutdown=immediate`이다. graceful하게 종료하려면 `server.shutdown=graceful`으로 설정하면 된다. graceful shutdown은 application 종료 절차가 수행되면 더이상 요청을 받지 않도록 처리한다. 다만 graceful 종료의 경우 진행 중인 요청을 처리하는데 deadlock이 발생하여 종료되지 못하는 경우도 있기 때문에 timeout을 설정해주는 것도 고려해야한다. 이는 `spring.lifecycle.timeout-per-shutdown-phase=1m`과 같이 명시하여 필요한 시간을 설정할 수 있다.

 graceful shutdown을 설정한다면 Tomcat기준으로는 `org.springframework.boot.web.embedded.tomcat` 패키지에 있는 `GracefulShutdown` 클래스에 `shutDownGracefully` 메소드가 있다. `TomcatWebServer` 클래스에서 호출하며 `TomcatWebServer` 클래스는 `WebServerGracefulShutdownLifecycle` 클래스 의존성을 가지고 있다. `WebServerGracefulShutdownLifecycle`은 `ServletWebServerApplicationContext` 클래스에서 singleton bean으로 등록되고 Spring Context가 종료될 때 shutdownHook으로 등록한 Thread가 호출하는 `AbstractApplicationContext`클래스의 `doClose()`가 호출되어 정리되는 `bean` 중 하나이다.  

***즉 Spring Context가 JVM이 종료될 때 호출하는 shutdownHook으로 등록한 Task들 중 `GracefulShutDown` 클래스의 `shutDownGracefuly` 메소드도 포함된다.***



#### Graceful Shutdown 적용 예제

```java
@RestController
@RequestMapping("/health")
public class TestController {
  @GetMapping
  public String slowRequest() throws InterruptedException {
    System.out.println("start");
    Thread.sleep(10000);
    System.out.println("end");
    
    return "OK";
  }
}
```

SpringBoot 2.3이상 버전으로 프로젝트를 생성했다. 애플리케이션을 실행하고 요청을 주었을 때 sleep으로 10초를 멈추고 그동안 애플리케이션을 종료하여 end가 찍히고 "OK"응답이 오는지 확인하였다.

##### Graceful 미적용

```shell
# 요청 중 end가 뜨기전에 애플리케이션 중지
start
Disconnected from the target VM, address: '127.0.0.1:5489', transport: 'socket'
2022-04-12 02:03:38.753  INFO 21360 --- [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2022-04-12 02:03:38.755  INFO 21360 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2022-04-12 02:03:38.764  INFO 21360 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.

Process finished with exit code 130
```

```powershell
PS C:\Users\user> curl 127.0.0.1:8080/health
curl : 원격 서버에 연결할 수 없습니다.
위치 줄:1 문자:1
+ curl 127.0.0.1:8080/health
```

end 메시지를 확인할 수 없고 "OK" 문자열도 반환되지 않는다.

##### Graceful 적용

```yaml
# application.yml
server:
  shutdown: graceful
```

```shell
start
Disconnected from the target VM, address: '127.0.0.1:5763', transport: 'socket'
2022-04-12 02:11:33.949  INFO 32564 --- [ionShutdownHook] o.s.b.w.e.tomcat.GracefulShutdown        : Commencing graceful shutdown. Waiting for active requests to complete
end
2022-04-12 02:11:36.711  INFO 32564 --- [tomcat-shutdown] o.s.b.w.e.tomcat.GracefulShutdown        : Graceful shutdown complete
2022-04-12 02:11:36.728  INFO 32564 --- [ionShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
2022-04-12 02:11:36.730  INFO 32564 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
2022-04-12 02:11:36.737  INFO 32564 --- [ionShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.

Process finished with exit code 130
```

```powershell
PS C:\Users\user> curl 127.0.0.1:8080/health

StatusCode        : 200
Content           : OK
```

`GracefulShowdown`클래스가 호출된 것을 확인할 수 있고 end문자열이 찍힐때 까지 애플리케이션은 종료되지 않는다. "OK" 클라이언트에서 반환된 것을 확인할 수 있다.

또한 애플리케이션이 종료되고 있을 때는 이미 진행중인 요청에 대해서는 처리를 하지만 새로운 요청을 받지 않는다.

```powershell
# 종료 중에 요청
PS C:\Users\user> curl 127.0.0.1:8080/health
curl : 원격 서버에 연결할 수 없습니다.
위치 줄:1 문자:1
+ curl 127.0.0.1:8080/health
```



### SpringBoot 2.3 미만 버전

`kill -SIGTERM` 시그널이 왔을 때 처리하도록 구현을 하면된다. Tomcat을 사용한다면 `ApplicationListener<ContextClosedEvent>`를 상속 받아 직접 Tomcat Shutdown을 구현하면 된다.

구현 예시 블로그 : [Spring Boot 기존 요청 스레드 처리하고 안전하게 종료하기 (tistory.com)](https://granger.tistory.com/60)



## 참고

[Linux kill Command Tutorial for Beginners (5 Examples) (howtoforge.com)](https://www.howtoforge.com/linux-kill-command/)

[Runtime.getRuntime().halt() vs System.exit() in Java | Baeldung](https://www.baeldung.com/java-runtime-halt-vs-system-exit)

[Adding Shutdown Hooks for JVM Applications | Baeldung](https://www.baeldung.com/jvm-shutdown-hooks)

[Daemon Threads in Java | Baeldung](https://www.baeldung.com/java-daemon-thread)

[JVM의 종료와 Graceful Shutdown (tistory.com)](https://effectivesquid.tistory.com/entry/JVM%EC%9D%98-%EC%A2%85%EB%A3%8C%EC%99%80-Graceful-Shutdown)

[JVM 의 실행, 종료, 클래스 로더, 아키텍쳐 (tistory.com)](https://kok202.tistory.com/315)

[Graceful Shutdown과 SIGINT/SIGTERM/SIGKILL (tistory.com)](https://2kindsofcs.tistory.com/53)

[Spring Boot 기존 요청 스레드 처리하고 안전하게 종료하기 (tistory.com)](https://granger.tistory.com/60)