---
title: 리눅스에서 도커 설치하기
tags: linux docker
---

# 설치 전 유의사항

- 최신 버전 커널을 사용하고 있는지 확인한다.

  호스트 운영체제가 최소한 3.10 버전 이상이어야 도커 컨테이너를 정상적으로 사용할 수 있다. 터미널에서 `uname -r` 명령어를 입력해 커널의 버전이 이를 만족하는지 확인해야 한다.

- 지원 기간 내에 있는 배포판인지 확인한다.

  오래된 리눅스 배포판은 업데이트 등의 지원을 받지 못할 수 있다. 현재 사용 중인 리눅스 배포판의 지원 종료 여부는 각 리눅스 운영체제의 공식 웹사이트에서 확인할 수 있다.

- 64비트 리눅스인지 확인한다.

  도커는 64비트에 최적화돼 있다. 

- sudo 명령어를 통해 설치하거나 root 권한을 소유한 계정에서 설치를 진행해야한다.

## 우분투 설치

```shell
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

add-apt-repository "deb [arch-amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

apt-get update

apt-get install docker-ce
```

## CentOS, RHEL7 도커 설치

#### 방법1

```shell
yum install -y yum-utils
yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce
systemctl start docker
```

#### 방법2

도커에서 제공하는 설치 스크립트로 손쉽게 설치하는 방법도 있다. 다음은 현재 사용 가능한 최신 버전의 도커 엔진을 설치한다. 그러나 외부 스크립트를 실행하는 것은 권장되는 방법이 아니라서 가능하면 **방법1**을 사용하길 권장한다.

```shell
wget -q0- get.docker.com | sh
```



## 설치 완료 확인 법

도커 커맨드를 이용해 도커 명령어가 실행된다면 설치가 성공적으로 완료된 것이다.

```shell
# 설치된 도커 버전을 확인한다
docker -v

# 도커 정보를 출력한다
docker info
```