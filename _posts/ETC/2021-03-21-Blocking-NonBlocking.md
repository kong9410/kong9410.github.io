---
title: 블로킹(Blocking)과 논블로킹(NonBlocking) 그리고 동기(Synchronous)와 비동기(Asynchronous)
tags: network
---

처음 블로킹(Blocking)과 논블로킹(NonBlocking)이라는 것을 들었을 때 동기(Synchronous), 비동기(Asynchronous)와 무슨 차이가 있는지를 잘 이해하지 못하였다. 알고보니 내가 알고있던 동기, 비동기가 블로킹과 논블로킹의 개념이 약간 섞여있던 것이다. 좀 더 자세히 둘을 분해해보자.

## 블로킹, 논블로킹

### 블로킹(Blocking)

함수 A가 함수 B를 호출했을 때 함수 B를 처리할 동안 함수 B가 제어권을 가지고 있어 작업이 모두 끝날 때 까지 함수 A가 대기상태가 된다.

쉽게 말해 처리해야 할 작업이 있으면 그것을 처리할동안 가만히 대기를 하고있는 것을 말한다.

> 직원A: 이 업무 좀 처리해주세요
> 직원B: 네~ (업무를 처리한다)
> 직원A: (직원B의 업무가 끝날때까지 다른 업무를 하지 않고 기다린다)
> 직원B: 끝났습니다.
> 직원A: 고마워요. (다시 자신의 업무를 한다)

### 논블로킹(NonBlocking)

함수 A가 함수 B를 호출할 때 함수 A는 제어권을 넘기지 않고 자신의 할일을 할 수 있다.

처리해야할 작업이 있다면 일을 시켜놓고 자신은 자신의 일을 하는 것이다.

> 직원A: 이 업무 좀 처리해주세요
> 직원B: 네~ (업무를 처리한다)
> 직원A: (자리로 돌아가 자신의 업무를 한다)

## 동기(Synchronous), 비동기(Asynchronous)

### 동기(Synchronous)

함수 A가 함수 B를 호출하고 함수 B의 결과값을 함수 A가 기다린다.

말 그대로 요청을 해놓고 요청한 결과를 계속 기다리는 것이다. 호출한 쪽에서 결과를 처리하게 된다.
각 작업의 시작 시간이 다른 한 작업의 끝나는 시간과 서로 일치해야한다.

> 직원A: 이 업무 좀 처리해주세요
> 직원B: 네~ (업무를 처리한다)
> 직원A: (업무 결과를 기다린다)
> 직원B: 업무 처리했습니다
> 직원A: 고마워요~ (업무 결과를 받는다)

### 비동기(Asynchronous)

함수 A가 함수 B를 호출하고 함수 B의 결과값을 기다리지 않는다. 함수 B의 결과에 대한 처리는 Callback으로 한다.

처리해야 할 작업을 시켜놓고 그 결과에 대한 것을 따로 신경쓰지 않는 것이다. 결과는 콜백으로만 따로 처리한다.
서로의 작업 시작 시간 끝나는 시간은 일치하지 않아도 상관이 없다.

> 직원A: 이 업무 좀 처리해주세요
> 직원B: 네~ (업무를 처리한다)
> 직원A: (이제 직원B가 업무를 처리하는지 신경쓰지 않는다.)

## 조합

예시가 적절했는지 모르겠지만 이렇게 차이가 있으므로 둘을 분리해서 개념을 이해하는 것이 좋다. 이런 블로킹 개념과 동기 개념들은 조합을 해서 사용한다.

### 동기 블로킹(Sync-Blocking)



![img](https://blog.kakaocdn.net/dn/d7u7Ip/btq09i03oUD/A8bUn5lKD6QtWmLKMp4wKk/img.png)

- A가 B를 호출한다.
- B가 제어권을 얻고 B가 작업을 처리한다.
- A는 B가 작업을 처리하는 동안 아무것도 하지않고 결과를 기다린다.
- B의 작업이 끝남과 동시에 A가 제어권을 가져간다.
- A는 B의 결과를 받는다.

두 개의 작업이 시작과 동시에 끝나는 시간이 일치한다.(Sync) 한쪽에서 작업이 시작되면 한쪽에서는 작업을 멈추고 기다린다.(Blocking)

애플리케이션이 DB 쿼리를 날리고 쿼리 결과를 받는다면 이것은 동기 블로킹이 된다.

### 비동기 논블로킹(Async-Blocking)

![img](https://blog.kakaocdn.net/dn/zWTiR/btq06itZhm5/qFtTSniHmw9tLbMPxYI2a1/img.png)

- A가 B를 호출한다.
- A는 제어권을 유지한채로 작업을 이어나간다.
- B는 작업을 처리한다.
- B의 작업이 끝나고 결과를 Callback한다.

서로의 시작과 끝시간을 맞출 필요도 없고 결과를 기다릴 필요도 없다.(Async) 각각 자신의 일을 할 수 있기 때문에(NonBlocking) 효율적인 구조다.

API 같은 것이 예로 될수가 있다.

### 동기 논블로킹(Sync-NonBlocking)

![img](https://blog.kakaocdn.net/dn/b0kJXe/btq08weqsZE/9NMUJcY9ZjlyAS5D5KmsmK/img.png)

- A가 B를 호출한다.
- A는 제어권을 유지한다.
- A는 다른 작업을 처리하면서 B의 완료 여부를 지속적으로 확인한다.
- B의 작업이 완료되면 A는 데이터 회신을 하고 나머지 작업을 진행한다.

두 개 이상의 작업이 시작 시간이나 끝나는 시간이 같아야 하며(Sync) 다른 작업을 기다리지 않는다.(NonBlocking) 결국에는 제어권을 잃지 않아도 다른 작업이 끝날때까지는 현재 작업을 마칠 수가 없다.

### 비동기 블로킹(Async-Blocking)

![img](https://blog.kakaocdn.net/dn/bQSO5v/btq08unnY3X/M2KhAsknEdgrKmSN8o0mRK/img.png)

- A가 B를 호출한다.
- B는 제어권을 가져간다.
- B가 작업을 처리하고 callback 반환을 한다.
- A가 다시 제어권을 가져간다.



다른 작업의 작업 마침여부를 기다리지는 않지만 결국엔 제어권이 작업중인 쪽에 넘어가 있기 때문에 동기-블로킹과 비슷하게 동작한다.

비동기 블로킹의 경우는 효율적이지 않아 많이 사용되는 방법은 아니며, 결국엔 A가 제어권을 잃기 때문에 비동기여도 다른 작업을 할 수 없다. 이러한 구조는 Node.js와 MySQL 조합이 있다.



### 여담

Async를 여태까지 "어싱크"라고 읽었는데, "에이싱크"가 영미권에서 사용하는 정확한 발음이라고 한다.



## 참고

[Async & Sync, Blocking & Non-Blocking (velog.io)](https://velog.io/@hotdari90/Async-Sync-Blocking-Non-Blocking)

[Sync vs Async / Blocking vs Non-Blocking - Try (has3ong.github.io)](https://has3ong.github.io/syncasync-nonblock/)

[동기/비동기와 블로킹/논블로킹 (tistory.com)](https://deveric.tistory.com/99)

