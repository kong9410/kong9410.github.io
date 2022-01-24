---
layout: post
title: 흐름/혼잡/오류 제어
tags: network
---

## 흐름 제어

- 수신측과 송신측의 데이터 처리 속도 차이를 해결하기 위한 기법
- 송신측의 패킷 전송량을 제어한다.

### 흐름 제어 방법

#### 정지-대기(Stop and Wait)

- 매번 전송한 패킷에 대해 응답을 받아야만 그 다음 패킷을 전송할 수 있다.
- 구조가 간단한 대신, 하나를 주고 하나를 받기 때문에 비효율적이다.

#### 슬라이딩 윈도우(Sliding Window)

![slidingwindow](https://user-images.githubusercontent.com/37204770/142751271-7ec93a0d-14b2-45b3-bec1-4e343962eb8f.png)

- 수신측에서 설정한 윈도우 크기만큼 송신측에서 확인 응답 없이 세그먼트를 전송할 수 있게 하여 데이터 흐름을 동적으로 조절하는 기법이다.

- 송신 버퍼 범위는 수신 측 여유 버퍼 공간을 반영하여 동적으로 바뀐다.

- 윈도우는 전송, 수신 스테이션 양쪽에서 만들어진 버퍼의 크기다.

- 윈도우 크기 = 최근 ACK로 응답한 프레임 수 - 이전 ACK 프레임을 보낸 프레임 수

- ACK 프레임을 수신하지 않더라도 여러개의 프레임을 연속적으로 전송할 수 있다.

- 송신 호스트는 정보 프레임을 순서 번호에 따라 순차적으로 전송한다.

- 수신 호스트가 응답하는 순서 번호는 다음에 수신하길 원하는 번호다.

- 송신 호스트는 송신한 정보 프레임을 송신 윈도우를 유지해야한다. 송신 윈도우는 전송이 되었지만 긍정 응답이 회신되지 않은 프레임을 보관한다.

- 수신 호스트는 수신한 정보 프레임을 내부 버퍼인 수신 윈도우를 유지해야한다. 수신 호스트가 관리하는 수신 윈도우는 프로토콜 동작 방식에 따라 크기가 달라진다. 동작 방식은 선택적 재전송, 고백N 방식이 있다.

- 프레임: 긍정 응답 프레임을 받지 않고도 연속으로 전송할 수 있는 프레임 개수이다.

- 연속형 전송: ACK 프레임을 받지 않고 여러 프레임을 연속으로 전송할 수 있는 방식이다.

- 고백N방식: 오류 복구 과정에서 오류가 발생한 프레임을 포함해 이후에 전송되는 모든 정보 프레임을 재전송하는 방식이다.

  ![gobackn](https://user-images.githubusercontent.com/37204770/142751757-366bf1e5-1d19-4fb0-8f62-627df511281f.png)

- 선택적 재전송 방식: 올바르게 수신한 정보 프레임을 버리지 않고 오류만 재전송하는 방법이다. 12번 순서에서 오류가 발생했다면 11번까지의 긍정 응답과 12번의 부정 응답을 송신 호스트에 보내고, 12번에 대한 프레임을 송신에서 재전송한다. 이후 마지막 올바른 응답부터 ack한다

  ![선택적재전송](https://user-images.githubusercontent.com/37204770/142751758-af71434a-a107-4ea0-a247-3faaee06d0a7.png)

-  피기배킹: 양방향 전송을 할때 사용된다. 응답 프레임과 정보 프레임을 동시에 전송하는 경우 사용하는 방식이다. 정보 프레임과 응답 프레임을 각각 보내주는 것이아닌 동시에 전송할 수 있도록 프레임 구조를 변형시킨 것이다.

  ![피기배킹](https://user-images.githubusercontent.com/37204770/142751760-934f3aec-444d-4198-a94f-2f28e261221e.png)

## 혼잡 제어

- 송신 측의 데이터 전달과 네트워크의 데이터처리 속도 차이를 해결하기 위한 기법
- 네트워크 상 라우터에게 데이터가 몰릴 경우 라우터는 자신에게 온 데이터를 모두 처리할 수 없다.
- 이러한 네트워크의 혼잡을 피하기 위해 송신측에서는 보내는 데이터의 전송 속도를 강제로 줄이게 된다.

### AIMD(Additive Increase / Multiplicative Decrease)

- 패킷을 하나 보내고 문제없이 도착하면 window 크기를 1씩 증가 시켜 전송한다
- 패킷 전송을 실패하면 패킷 전송속도를 절반으로 줄인다.
- 네트워크가 혼잡해지고 나서야 대역폭을 줄이는 방식이다.

### 슬로우 스타트

![slow start](https://user-images.githubusercontent.com/37204770/142752858-46369a23-fa11-4298-b1cb-f43a079a53f7.jpg)

- AIMD와 마찬가지로 패킷을 하나씩 보내는데 도착하면 각각의  ack 패킷마다 window size를 1씩 늘린다. 한 주기가 지나면 window size가 2배가 된다.
- 혼잡 현상이 발생하면 window size를 1로 떨어뜨리게 된다.
- 전송 되어지는 데이터의 크기가 임계값에 도달하면 혼잡 회피 단계로 넘어간다.

### 혼잡 회피(Congestion Avoidance)

- 전송한 데이터에 대한 ACK를 받으면 윈도우 크기를 1씩 증가시킨다.
- 전송하는 데이터의 증가를 왕복시간 동안에 하나만 증가시킨다.
- 타임 아웃이 발생하면 윈도우 크기를 1로 줄이고 임계 값을 패킷 손실이 발생했을 때의 윈도우 크기의 반으로 줄인다.

### 빠른 회복(Fast Recovery)

- 혼잡이 발생했을 때 윈도우 크기를 1로 줄이지 않고 반으로 줄이고 선형 증가시키는 방법이다.
- Fast Recovery를 적용하면 혼잡 상황을 한 번 겪고 나서는 순수한 AIMD 방식으로 동작한다.

### 빠른 재전송(Fast Retransmission)

- 3개의 연속된 중복 ACK를 수신하는 경우 패킷 손실로 간주하여 타임아웃이 발생 하기전 해당 패킷을 재전송하고 window size를 반으로 줄인다.

### TCP Reno

N개의 중복 ACK 발생 시 ssthresh(slow start threshold) 값을 congestion window 사이즈의 반으로 줄여 빠른 복구를 수행하여 선형적 증가를 하게되며, TCP time out에 이르면 slow start를 시작한다.

### TCP Tahoe

N개의 중복 ACK 발생 시 바로 Slow Start를 시작한다.

## 오류 제어

- 오류 검출과 재전송을 포함한다.
- ARQ(Automatic Repeat Request) 기법을 사용한다.

### Stop and Wait ARQ

- 전송측은 수신측에서 보내준 ACK를 받을 때까지 프레임 복사본을 유지한다.
- 식별을 위해 데이터 프레임과 ACK 프레임은 각 0, 1번호를 부여한다.
- 수신 측이 데이터를 받지 못했을 경우, NAK를 송신측에게 보내고 NAK를 받은 송신측은 데이터를 재전송한다.
- ACK나 데이터가 분실되었을 경우 일정 시간을 두고 타입아웃이 되면 송신측은 데이터를 재전송한다.

### Go-Back-n ARQ(GBn ARQ)

- 전송된 프레임이 손상되거나 분실될 경우, 확인된 마지막 프레임 이후로 모두 재전송하는 기법
- 연속적인 프레임 전송 기법으로 전송 스테이션은 전송된 모든 프레임의 복사본을 가지고 있어야하며, ACK와 NAK 모두 구분을 해야한다.
- ACK: 다음 프레임 전송
- NAK: 손상된 프레임 자체 번호를 반환

#### NAK 프레임을 받을 경우

- 만일 수신측에서 데이터 오류 프레임이 잘못되었다는 것을 발견하면 NAK를 전송측에 보낸다.
- NAK을 받은 전송측은 해당 프레임을 재전송한다.
- GBn ARQ는 NAK(n)을 받아 데이터를 재전송하게 되면 n 데이터 이후의 데이터도 모두 재전송한다.

#### 전송 데이터 프레임의 분실

- 수신측에서 1을 받고 다음으로 3을 받으면 3을 폐기하고 NAK 2를 전송측에 보낸다.
- NAK 2를 받은 전송측은 위와 같이 NAK(n)으로 보내었던 대상 데이터를 모두 보낸다.

#### 지정된 타임아웃 내의 ACK 프레임 분실

- 전송측에서 타이머의 타임 아웃 동안 ACK를 받지 못했을 경우 마지막 ACK부터 재전송한다.
- 해당 ACK 이후의 프레임은 모두 폐기한다.

### Selective-Reject ARP

- SR ARQ는 손상된 분실된 프레임만 재전송한다.
- 별도의 데이터 재정렬을 수행해야한다.
- 별도의 버퍼를 필요로 한다.



## 출처

\[[네트워크\] 흐름/혼잡/오류 제어 기법 - VictoryWoo (woovictory.github.io)](https://woovictory.github.io/2018/12/28/Network-Erro-Flow-Control/)
