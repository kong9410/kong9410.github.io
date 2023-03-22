---
title: 도커 네트워크(docker network)
layout: post
tag: docker
description: 도커 네트워크가 뭔지 알아봅시다
---

![networking - What is the relation between docker0 and eth0? - Stack Overflow](https://www.linuxjournal.com/files/linuxjournal.com/ufiles/imagecache/large-550px-centered/u1002061/11833f1.png)

위는 도커의 네트워크를 도식화 한 그림이다. 여기서 보이는 eth0, docker0 등이 뭔지 알아보자

## Docker0 Interface

docker host를 설치하고 host의 network interface를 살펴보면 docker0라는 virtual interface가 존재하는 것을 확인 할 수 있다.

```shell
docker0   Link encap:Ethernet  HWaddr XX:XX:7a:fe:97:XX  
          inet addr:172.17.42.1  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::5484:7aff:fefe:9799/64 Scope:Link
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:9918 errors:0 dropped:0 overruns:0 frame:0
          TX packets:36261 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:662534 (662.5 KB)  TX bytes:52918471 (52.9 MB)
```

> IP는 172.17.42.1로 설정되고 16bit netmask(255.255.0.0)로 설정된다.
>
> IP는 DHCP를 통해 할당 받는 것은 아니고 docker 내부 로직에 의해 자동 할당 받는 것이다.
>
> docker0는 일반적인 interface가 아니고 virtual ethernet bridge이다

docker0는 container가 통신하기 위한 가상 linux bridge이다. bridge는 L2 통신 기반이고 container 하나가 생성되면 이 bridge에 container interface가 하나씩 binding 되는 형태다. container가 외부로 통신할 때 무조건 docker0 interface를 지나야 한다.

```shell
root@~~:# brctl show docker0

bridge name      bridge id               STP    enabled   interfaces
docker0          8000.d67973326669       no               veth0e580ab
                                                          veth77faee0
```

docker가 설치되면 docker0라는 bridge가 생성되고 container가 running되면 vethXXXXXX라는 이름의 interface가 붙는다. 위는 현재 두개의 container가 running 중임을 알 수 있다.

docker0의 IP는 자동으로 172.17.42.1로 생성되고 subnet mask는 172.17.0.0/16으로 설정된다. subnet 정보는 앞으로 container가 생성될때 마다 할당 받게 될 IP의 range를 결정하게 된다. 즉 172.17.XX.YY 대역에서 IP를 할당받는다.

##  Container Network 구조

container 하나를 생성하면 어떤일이 일어나는지 알아보자

container는 각자 격리된 network 공간을 받는다. 이러한 container가 외부와 통신하는 방법은 다음과 같다

container를 생성하면 container에는 pair interface라고 하는 한 쌍의 interface들이 생성된다.

![How Docker Container Networking Works - Mimic It Using Linux Network  Namespaces - DEV Community](https://res.cloudinary.com/practicaldev/image/fetch/s--tGvJLH6y--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/i/pszchqfvcjpl13dextrc.png)

pair interace는 두개의 interface가 한쌍으로 구성되는데 이 둘은 직접 cable을 연결한 두대의 pc와 같이 서로의 packet을 주고 받는 형태다.

container가 생성되면 이 pair interface의 한 쪽은 container 내부 namespace에 할당되고 eth0라는 이름으로 할당된다. 나머지 하나는 vethXXXX라는 이름으로 docker0 bridge에 binding 되는 형태다.

container가 하나 올라간 상태에서 host의 interace를 확인해보자

```shell
 root@~~# ip link

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 06:7a:4f:1b:ce:ad brd ff:ff:ff:ff:ff:ff
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 9e:89:98:9e:9f:77 brd ff:ff:ff:ff:ff:ff
5: vethfd06149: <BROADCAST,UP,LOWER_UP> mtu 9001 qdisc pfifo_fast master docker0 state UP mode DEFAULT group default qlen 1000
    link/ether 9e:89:98:9e:9f:77 brd ff:ff:ff:ff:ff:ff
```

```shell
root@~~# brctl show docker0
bridge name       bridge id                          STP    enabled     interfaces
docker0              8000.9e89989e9f77      no                       vethfd06149 
```

vethXXX형태의 interface가 docker0 bridge에 binding 되어있는 것을 알 수 있다. eth0는 컨테이너 내부에서만 확인이 가능하다

```shell
 root@~~# docker exec c456623003b1 ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:01  
              inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
              inet6 addr: fe80::42:acff:fe11:1/64 Scope:Link
              UP BROADCAST RUNNING  MTU:9001  Metric:1
              RX packets:8 errors:0 dropped:0 overruns:0 frame:0
              TX packets:9 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:0 
              RX bytes:648 (648.0 B)  TX bytes:738 (738.0 B)
```

보이는 것처럼 container 안에 외부 통신을 위한 eth0 interface가 있음을 알 수 있다. 그리고 172.17.0.0/16 대역에 있는것도 확인할 수 있다.

## Docker의 Network 방식

docker의 default는 bridge 방식이고 만일 다른 방식으로 생성하고 싶으면 `--net`을 이용하여 옵션을 주면 된다

```shell
docker run --net=NETWORK_TYPE
```

### Bridge

도커의 기본 network 방식이다. 

컨테이너를 생성하면 각 컨테이너마다 고유한 network namespace가 생기고 docker0 bridge에 container의 인터페이스들이 하나씩 binding 되는 구조다.

### Host

host로 컨테이너를 생성하면 network namespace를 따로 갖지않고 host와 네트워크 환경을 함께 사용하게 된다.

bridge를 사용하지 않기 때문에 docker0에도 바인딩 되지 않는다.

도커 네트워크 정보를 확인해보면 IP정보가 없다

```shell
$ docker network inspect host
[
  {
    ...
    "Driver": "host",
    ...
    "Containers": {
      "...": {
        ...
        "IPv4Address": "",
        "IPv6Address": ""
      }
    }
  }
]
```

### Container

이 방식은 다른 컨테이너의 network환경을 공유하게 된다.

web02 이라는 컨테이너를 만들고 web03이라는 컨테이너를 web02 네트워크 환경과 공유하게 만들어보자

```shell
docker run --name web02 -d httpd
docker run --name web03 --net=container:e1b4a085348e -d httpd
```

이렇게 만들면 web03은 따로 IP를 갖지 않는다 그리고 web02와 같은 IP와 MAC 주소를 가진다.

```shell
docker exec web02 ip addr show

docker exec web03 ip addr show

# 두명령의 결과를 비교해보면 eth0의 ip가 동일하게 할당된 것을 확인할 수 있다.
```

bridge를 확인해봐도 web03 인터페이스는 찾아볼 수 없다. web03은 따로 네트워크 환경을 할당하지 않았기 때문이다. brctl 명령어를 확인해봐도 확인되는건 web02의 interface뿐이다

### None

`--net=none`으로 컨테이너를 생성하면 격리된 네트워크 영역을 가지지만 인터페이스가 없는 상태로 컨테이너를 생성하게 된다

생성된 인터페이스의 정보를 확인해보면

```shell
docker exec web004 ip addr show
```

통신을 위한 eth0와 같은 인터페이스가 없고, brdige와 연결되어 있지 않으며 이 상태로는 외부와 통신이 불가하다

이 옵션은 인터페이스를 직접 커스터마이징 할 수 있도록 네트워크 환경이 clear한 상태로 만들어진 것으로 추측한다.

## Container Port  노출하기

각 container는 독립된 네트워크 환경을 가지고 mac주소와 private IP도 부여받게 된다. 각 container들은 docker host와 통신하기 위해 linux bridge 방식으로 binding되어 있다. docker host내에 배포된 container 들 사이에 각자 할당받은 private ip를 이용해 자유롭게 통신이 가능하다

web서비스 컨테이너가 있다고 가정했을때 http 통신으로 80포트가 반드시 외부와 통신이 이루어져야 한다면 어떻게 해야할까, 직접적으로 외부와 연결이 되어있지 않다.

container가 외부로 서비스되기 위해 어떤 구조로 동작하는지 알아야한다

container는 기본적으로 외부와 통신이 불가능하다. 외부와 통신을 위해서는 container를 외부로 노출할 port로 지정해야한다.

```shell
docker run -d -p 8080:80 --name web_server01 httpd
```

위 명령을 실행하면 docker host에 8080 포트로 요청이 들어오면 web_server01 컨테이너의 80포트로 포워딩을 하겠다는 의미다.

docker host에서 netstat 명령을 통해 listening 상태를 보면 8080포트가 active 되어있는 것을 확인할 수 있다.

```shell
$ netstat -nlp | grep 8080
tcp6   0    0:8080      :::*     LISTEN    12581/docker-proxy
```

8080포트를 수신하고 있는 프로세스는 docker-proxy라는 프로세스다.

### docker-proxy

이 프로세스의 목적은 docker host로 들어온 요청을 container로 넘기는 역할을 한다. docker-proxy는 kernel이 아닌 userland에서 수행된다. kernel과 상관없이 host가 받은 패킷을 그대로 container의 port로 넘긴다. container를 시작할때 port를 외부로 노출하도록 설정하면 docker host에는 docker-proxy라는 프로세스가 생성된다.

```shell
UID        PID  PPID  C STIME TTY          TIME CMD

root     12581  9576  0 06:31 ?        00:00:00 docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 8080 -container-ip 172.17.0.2 -container-port 80
```

docker-proxy는 container port 노출하도록 설정한 수 만큼 추가로 프로세스가 생성된다.

두개의 container가 port를 노출하고 있다면 docker-proxy는 두개가 생성된다.

container하나가 두개의 port를 노출한다면 docker-proxy는 두개가 생성된다.

그런 실제로 docker host로 들어온 패킷이 container로 전달되는 것은 docker-proxy와 무관하게 docker host의 iptables에 의해 동작된다. 그렇다면 docker-proxy의 존재의미는 무엇일까? 이를 알기위해 iptables를 위한 port포워딩을 살펴보자

### iptables를 이용한 DNAT

docker host의 iptables를 살펴보자

```shell
root@~~# iptables -t nat -L -n

Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
DOCKER     all  --  0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  172.17.0.0/16        0.0.0.0/0
MASQUERADE  tcp  --  172.17.0.2           172.17.0.2           tcp dpt:80

Chain DOCKER (2 references)
target     prot opt source               destination
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8080 to:172.17.0.2:80
```

iptables 내역을 보면, docker host에 들어온 패킷이 PREROUTING chain을 통해 DOCKER chain으로 전달하고, docker chain에서는 DNAT로 8080포트로 들어온 요청을 172.17.0.2 IP를 가진 container의 80 포트로 포워딩 되는것을 알 수 있다.

반대로 container에서 외부로 나갈때는 POSTROUTING을 거쳐 MASQUERADE가 되어 외부로 나간다

여기서 볼수있는건 container 관련 iptables rule 관리는 docker daemon이 자동으로 제어하게 된다. 또 NAT을 위한 ip_forward 설정도 docker daemon에서 제어하게 된다.

### docker-proxy를 사용하는 이유

docker-proxy가 존재하는 이유는 docker host가 iptables의 NAT를 사용하지 못하는 상황에 대한 처리다. 정책상 이유로 docker host의 iptables나 ip_forward를 enable을 하지 못하는 경우에 docker-proxy 프로세스가 패킷을 포워딩 하는 역할을 하게 된다.

이런 역할에도 docker-proxy 필요성에 의구심을 품을 수 있다. docker-proxy의 메모리도 차지하는 편이다. 많은 프로세스가 띄울수로 그만큼 메모리 사용량이 늘어날 것이다.

disable 관련 문의가 많았는지 docker 1.7 부터는 docker-proxy 대신 localhost의 hairpin NAT 방식이 가능하도록 지원한다.

## Docker Link

web서버 컨테이너와 DB 컨테이너가 있고 두 컨테이너를 연동해야할 때 어떻게 사용하는지에 대해 알아본다

### Container 연동시 Link를 사용해야 하는 이유

동일 host상에 배포된 container 사이는 private ip를 이용해 통신이 가능하다. 만약 web 컨테이너가 mysql로 ip기반으로 ping을 보내면 다음과 같이 응답한다

```shell
$ docker exec -t web01 ping 172.17.0.6 # mysql의 ip
PING 172.17.0.6 (172.17.0.6): 56 data bytes
64 bytes from 172.17.0.6: icmp_seq=0 ttl=64 time=0.103 ms
```

하지만 ip기반은 문제점이 있다. container의 ip는 언제든 변할 수도 있기 때문이다. 만일 container가 중지되었다가 생성하면 private ip는 언제든 변할 수 있다. 그때문에 container 사이 연동을 하려면 ip는 권고되지 않는다. link 옵션이 이를 해결해줄 수 있다.

```shell
$ docker run -d --name web02 --link mysql httpd
```

위와같은 명령어로 web02 container를 구동하면 mysql container와 연동이 된다.

```shell
root@~~# docker exec -t web02 ping mysql
PING mysql (172.17.0.6): 56 data bytes
64 bytes from 172.17.0.6: icmp_seq=0 ttl=64 time=0.096 ms
```

위와 같이 ping target이 ip가 아닌 이름으로 가능하다. container 이름을 기반으로 하면 ip가 바뀌어도 문제가 없다.

### link 연동 구조

container가 구동되면 container 환경 일부는 docker daemon에 의해 자동으로 제어된다. 그중하나가 container 내부의 domain name을 관리하는 hosts 파일이다. link 연동이 걸린 web02 container의 hosts 파일을 살펴보면 다음과같다

```shell
$ docker exec -t web02 cat /etc/hosts
172.17.0.6      mysql 17b6c5f037a9
```

위와 같이 컨테이너 내부의 `/etc/hosts`에는 mysql container의 ip주소와 host명이 등록되어 있다. link 옵션이 없다면 존재핮 ㅣ않는다.

만일 mysql container 컨테이너가 재가동된다면 ip는 바뀔것이다. docker daemon이 이때 자동으로 `/etc/hosts`파일을 수정해준다. 이는 어떻게 가능한걸까?

container의 host파일을 살펴보면 `dev/disk`에 의해 mount되어 있다. container 외부의 docker host에서 연동되어 있는 것이다.

### 한계

container사이에 link를 이용해 연동해야 동적 ip에 따른 이슈를 피할 수 있다. 그러나 link 방식은 한계가 있다. 

첫째로 link옵션은 동일 docker host에 존재하는 container 사이에서만 유효하다. 여러 docker host가 있다면 link 사용이 불가하다.

이러한 경우 docker swarm같은 orchestration 도구나 dynamic DNS 를 사용해야한다.

## 출처

[Docker Network 구조(1) - docker0와 container network 구조 (tistory.com)](https://bluese05.tistory.com/15)

[Docker Network 구조(2) - container network 방식 4가지 (tistory.com)](https://bluese05.tistory.com/38)

[Docker Network 구조(3) - container 외부 통신 구조 (tistory.com)](https://bluese05.tistory.com/53)

[Docker Network 구조(4) - container link 구조 (tistory.com)](https://bluese05.tistory.com/54)

