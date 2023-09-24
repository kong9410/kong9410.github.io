---
title: 도커 기본 커맨드 정리
tags: docker
layout: post
description: 도커에서 사용하는 기본 커맨드 정리
---

### 도커 버전 확인

```shell
docker -v
```

### 도커 이미지 받기 (nginx)

```shell
docker pull nginx:latest #이미지명:버전
```

### 도커 이미지 확인

```shell
docker images
```

### 도커 컨테이너 생성

```shell
docker create -p 80:80 --name mynginx nginx:latest
```

`--name` 은 컨테이너 이름

`80:80`은 호스트의 포트와 컨테이너의 포트를 매핑

### 도커 컨테이너 시작

```shell
docker start mynginx
```

### 컨테이너가 실행됐는지 확인

```shell
docker ps
```

#### url로 확인

```shell
curl http://127.0.0.1
```

### 컨테이너 중지

```shell
docker stop mynginx
```

### 중지된 컨테이너 확인

```shell
docker ps -a
```

`-a`: 실행 및 중지된 컨테이너까지 출력

### 컨테이너 삭제

```shell
docker rm mynginx
```

### 도커 이미지 삭제

```shell
docker rmi nginx:latest
```

### RUN

```shell
docker run -d -p 80:80 --name mynginx2 nginx:latest
```

`-d`: 백그라운드로 실행

컨테이너 생성 및 실행

