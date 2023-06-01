---
title: RAC cache fusion
tags: oracle
layout: post
description: oracle rac cache fusion
---

DB Buffer cache

- 유저가 요청한 SQL문을 수행하기 위해 필요한 데이터 블록을 디스크로부터 메모리로 올리는 영역이다.
- Cache Fusion이 서로 다른 Instance의 DB Buffer Cache에서 Block을 이동시키는 부분에서 발생한다.

Cache Fusion

- OPS의 Block Transformation의 문제를 극복하기 위해 나타난 아키텍쳐다

> Block Transformation
>
> 특정 인스턴스에 존재하는 블럭을 다른 인스턴스에서 액세스 하기 위해 블럭을 공유 stroage에 write하고 그 block을 호출한 instance의 db buffer cache로 캐싱하여 data block을 다른 node로 이동시키는 방식이다

각각의 Instance에서 많은 작업이 수행되고 각각의 instance 사이에서 이동하는 block이 많아지면 성능 저하가 발생한다

cache fusion 수행 방식

![img](https://blog.kakaocdn.net/dn/mWDgy/btqGbPkYCgH/TRBLS1a3od4WDqTkih6eWk/img.png)

- instance 2에 접속한 a프로세스는 instance 2에 존재하는 1번 block을 액세스하기위해 호출한다
- instance 1에 존재하는 1번 block은 instance 2의 호출에 응답하기 위해 instance 2의 SGA의 DB Buffer Cache로 이동해야한다. 이동을 위해 공유 Storage에 존재하는 DB를 이용하지 않고 Interconnect를 이요ㅣㅇ한다
- Interconnect를 이용하여 Instance 2로 캐싱된 1번 block을 A 프로세스가 액세스 할 수 있게 된다.



block이 특정 node의 db buffer cache에 로드되어있다면 disk io가 발생하지 않는다는 특징이 있다

