---
title: 3-way handshake
tags: network
---

# Three-Way Handshake

## 이게 뭐야

TCP/IP 네트워크에서 사용되는 three-way handshake는 호스트와 클라이언트 서버의 연결을 생성하는 방법을 말한다.

HTTP, SSH와 같은 데이터가 전송되기 전 통신단에서 TCP 소켓 연결의 매개 변수를 동시에 시작하고 negotiation 할 수 있도록 설계된 3단계 방법이다.

TCP 소켓 연결을 양방향으로 동시에 전송할 수 있다. three-way handshake는 TCP handshake 또는 SYN-SYN-ACK라고 불리고, 실제 데이터 통신이 시작되기 전 클라이언트와 서버 모두 SYN(동기화) ACK(승인) 패킷을 교환해야한다.

## 디테일

![TCP/IP 네트워크에서 3방향 악수의 다이어그램](https://www.techopedia.com/images/uploads/ad900dc1-ad94-4c7b-a3f8-154ad27c35f1.png)

3 WAY HANDSHAKE는 장치 간 데이터를 안정적으로 전송하기 위해 TCP 소켓 연결을 만드는데 사용한다. 예를 들어 사용자가 인터넷을 사용할때 클라이언트 측의 웹 브라우저와 서버 간의 통신을 지원한다. 클라이언트가 서버와의 통신 세션을 요청하는 즉시 3방향 핸드세이크 프로세스는 세단계의 TCP 트래픽을 싲가한다

## 1단계

서버와 클라이언트 간읜 연결이 설정된다.

서버와 클라이언트 간의 연결이 설정되므로 서버에 새 연결을 이을 수 있는 열려있는 포트가 있어야한다. 클라이언트 노드는 ip 네트워크를 통해 SYN(시퀀스 번호 동기화) 데이터 패킷을 동일 하거나 외부 네트워크의 서버로 보낸다.

이 syn 패킷은 클라이언트가 통신에 사용하려는 임의 번호이다. 이 패킷의 목적은 서버거 연결을 위해 열려있는지 묻는 단계이다.

## 2단계

서버가 클라이언트에서 syn 패킷을 받으면 ack(acknowledgement sequence number) 패킷 또는 syn/ack 패킷을 응답한다. 이 패킷은 두가지 시퀀스 넘버를 가지고있다.

하나는 ack인데 이것은 클라이언트로 부터 수산받은 시퀀스 번호보다 하나 더 많은 ack이다. (예를들어 100이 왔다면 101로 설정한다)

다른 하나는 서버로 부터 전송되는 랜덤 시퀀스 번호인 syn이다.

이 시퀀스는 서버가 클라이언트 패킷을 올바르게 승인했음을 나타낸다.

## 3단계

클라이언트 노드는 서버에서 syn/ack를 수신하고 ack 패킷으로 응답한다.

클라이언트가 시퀀스 번호에 하나를 추가하여 서버 패킷을 승인하고 서버에 다시 전송한다.

이 프로세스가 완료되면 연결이 만들어지고 호스트와 서버가 통신할 수 있다.



# Four-Way Handshake

## 이게 무엇

3 way handshake가 tcp 연결을 초기화 할 때 사용했다면 4-way handshake는 세션을 종료하기 위해 수행된다.

![img](https://t1.daumcdn.net/cfile/tistory/2152353F52F1C02835)

## 1단계

클라이언트가 연결을 종료하겠다는 FIN 플래그를 전송한다.

## 2단계

서버는 확인 메시지를 보내고 자신의 통신이 끝날때까지 기다리는데 이 상태가 TIME_WAIT 상태다

## 3단계

서버가 통신이 끝났으면 연결이 종료되었다고 클라이언트에게 FIN 플래그를 전송한다

## 4단계

클라이언트는 확인했다는 메시지를 보낸다.



만일 서버에서 FIN을 전송하기 전에 전송한 패킷이 routing 지연이나 패킷 유실로 인한 재전송 등으로 인해 fin 패킷보다 늦게 도착하는 상황이 발생한다면 데이터가 유실될 것이다. 이러한 현상에 대비하여 client는 서버로 부터 fin을 수신하더라도 일정시간 동안 세션을 남겨놓고 잉여 패킷을 기다리는 과정을 거치는데 이를 TIME_WAIT라고 한다.



## 3 way flooding

3way handshake 과정중 2단계에는 이 연결을 메모리 공간인 밸로그큐(Backlog Queue)에 저장을 하고 클라이언트의 응답 3단계를 기다리게 되고 일정시간 동안 응답이 안오면 연결을 초기화한다. 이를 이용한 공격 방법을 3 way flooding이라고 한다. 클라이언트에서 3단계는 없고 1단계 요청만 무수히 많이 보내어 백로그 큐를 포화상태로 만들어 다른 사용자로부터 연결 요청을 못받게 하는 공격 방법이다.

해결법은, 정해진 시간동안 들어오는 연결 수를 제한 하거나 쿠키를 이용해 전체 연결이 설정되기 전까지 자원의 할당을 연기하는 법이 있다.



## 출처

[What is a Three-Way Handshake? - Definition from Techopedia](https://www.techopedia.com/definition/10339/three-way-handshake)

[[ 네트워크 쉽게 이해하기 22편 \] TCP 3 Way-Handshake & 4 Way-Handshake (tistory.com)](https://mindnet.tistory.com/entry/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%EC%89%BD%EA%B2%8C-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-22%ED%8E%B8-TCP-3-WayHandshake-4-WayHandshake)

[TCP 3 Way-Handshake :: 졸린눈 (tistory.com)](https://sleepyeyes.tistory.com/4)

[[네트워크\] 3-way / 4-way Handshake 란? (tistory.com)](https://bangu4.tistory.com/74)