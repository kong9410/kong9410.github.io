---
title: 도커 실습
tags: docker
layout: post
description: 도커 실습 자료
---

## 이미지 받기

이미지를 내려받기 전 현재 받아져있는 도커 이미지를 확인해본다

```shell
$ docker images
```

이미지를 내려받기 위해서는 `pull` 명령어를 사용하면 된다

```shell
$ docker pull nginx:latest
```

내려받고 이미지를 확인해보면 내려받은 이미지 목록을 볼 수 있다

```shell
$ docker images
```

## 컨테이너 생성하기

`ps` 명령어를 이용해 현재 running 중인 컨테이너를 확인한다

```shell
$ docker ps
```

`create` 명령어를 이용해 이전에 내려받은 `nginx` 이미지 기반 컨테이너를 생성한다. `--name`옵션으로 컨테이너 이름을 줄 수 있다

```shell
$ docker create --name mincheol-nginx nginx:latest
```

생성 후에 `ps` 명령어로 컨테이너 목록을 확인해보자

```shell
$ docker ps
```

아무것도 표시가 되지 않는다. `ps` 만을 입력했을 때는 running 중인 컨테이너만 표시가 된다. `-a` 옵션으로 현재 실행중이지 않은 컨테이너 목록을 볼 수 있다.

```shell
$ docker ps -a
```

`CREATED` 되어 있기만하고 실행중이 아닌 것을 알 수 있다. `start` 명령어를 통해 해당 컨테이너를 실행한다

```shell
$ docker start mincheol-nginx
```

`ps`로 확인해보면 컨테이너 상태가 바뀌었음을 알 수 있다.

```shell
$ docker ps
```

## 컨테이너 삭제하기

컨테이너는 바로 삭제를 할 수는 없다. 먼저 컨테이너를 `stop`으로 중지 시켜야한다

```shell
$ docker stop mincheol-nginx
```

컨테이너 목록을 보면 중지가 된것을 볼 수 있다

```shell
$ docker ps -a
```

중지된 컨테이너를 `rm` 명령어로 지우면 된다

```shell
$ docker rm mincheol-nginx
```

컨테이너가 삭제된 것을 볼 수 있다

```shell
$ docker ps -a
```

## 컨테이너 포트 열기

`create`에서 `start`까지 한번에 동작가능한 `run` 명령어가 있다. `run`을 사용하면서 `-p`옵션으로 컨테이너를 expose할 포트를 지정해줄 수 있다. `<호스트포트>:<컨테이너포트>`로 지정을 해주면 된다.

`--rm` 명령어로 임시 컨테이너를 만들 수 있다. 이 옵션을 주면 해당 컨테이너는 stop이 되면 자동으로 지워진다.

```shell
$ docker run -d -p 80:80 --rm --name mincheol-nginx nginx:latest
d```

생성이 되었는지 확인한다

```shell
$ docker ps
```

해당 포트로 접근할 수 있는지 `curl`을 사용해서 확인해본다

```shell
$ curl http://localhost
```

사용을 다했으면 컨테이너를 stop하여 제거한다

```shell
$ docker stop mincheol-nginx
```

## 컨테이너 바인드 마운트하기

디렉토리 마운트를 하기전에 마운트할 새 디렉토리를 생성한다

```shell
$ mkdir /home1/irteam/mount-test/mincheol
```

새 디렉토리에 `index.html`을 생성한다

```shell
$ echo "<html><body>Hello Umon</body></html>" >> /home1/irteam/mount-test/mincheol/index.html
```

컨테이너를 만들때 `-v`옵션으로 컨테이너 내부의 디렉토리를 마운트 시킬 수 있다. `<호스트 디렉토리 위치>:<컨테이너 디렉토리 위치>`로 마운트할 디렉토리 위치를 지정할 수 있다.

주의할 점은 바인드 마운트는 호스트의 디렉토리가 컨테이너 디렉토리 내용을 덮어쓴다는 점이다.

```shell
$ docker run -d --rm -p 80:80 -v /home1/irteam/mount-test/mincheol:/ust/share/nginx/html --name mincheol-nginx nginx:latest
```

아까와같이 `curl`로 호출하면 작성했던 `index.html` 내용이 적용된 것을 볼 수 있다.

```shell
$ curl http://localhost
```

## 볼륨 만들기

볼륨은 도커가 직접 관리하기 때문에 디렉토리를 별도로 생성할 필요는 없다. 다음 명령어로 생성할 수 있다

```shell
$ docker volume create nginx-volume
```

`inspect` 명령어로 만들어진 볼륨의 상세 정보를 볼 수 있다

```shell
$ docker volume inspect nginx-volume
```

insect정보에서 mountpoint 정보를 확인해보자

```shell
$ ls /var/lib/docker/volumes/nginx-volumes/_data
```

아무런 정보가 없는 것을 확인할 수 있다. 다음은 볼륨을 할당한 컨테이너를 생성해보자. 바인드 마운트와 마찬가지로 `<볼륨이름>:<컨테이너디렉토리>`로 지정하면된다. 바인드 마운트와 다른점이라면 호스트의 디렉토리가 컨테이너의 내용을 덮어씌우는 것이 아닌 공유가 된다.

```shell
$ docker run -d -v nginx-volume:/usr/share/nginx/html -p 80:80 --rm --name mincheol-nginx nginx:latest
```

컨테이너 실행후 마운트포인트를 다시 조회해보자

```shell
$ ls /var/lib/docker/volumes/nginx-volumes/_data
```

컨테이너의 디렉토리와 동기화가 된 것을 볼 수 있다. 볼륨은 바인드 마운트와는 달리 nginx의 기본 index.html이 그대로 표시된다.

```shell
$ curl http://localhost
```

양쪽이 동기화 되는 구조이기 때문에 볼륨내의 index.html을 변경하면 nginx의 index.html도 변경이 된다

```shell
$ vi html.html
<html><body>Hello World</body></html>
```

```shell
$ curl http://localhost
```

## 네트워크 만들기

다음 명령어로 현재 도커 네트워크 목록을 확인할 수 있다

```shell
$ docker network ls
```

`network create`를 통해 새로운 네트워크를 만들 수 있다

```shell
$ docker network create --drvier bridge mincheol-bridge
```

생성된 네트워크는 `inspect` 명령어로 자세한 정보를 볼 수 있다

```shell
$ docker inspect network mincheol-bridge
```

생성된 네트워크는 컨테이너를 생성할때 할당할 수 있다

```shell
$ docker run -d -it --net mincheol-bridge --rm --name mincheol-network-ubuntu ubuntu:latest
```

컨테이너 생성 이후에 네트워크 연결을 끊고싶으면 `disconnect`를 사용한다

```shell
$ docker network disconnect mincheol-bridge mincheol-network-ubuntu
```

다시 연결하려면 `connect`를 사용한다

```shell
$ docker network connect mincheol-bridge mincheol-network-ubuntu
```

## 도커 이미지 만들기

이미지는 레이어가 겹겹이 쌓여있는 하나의 레이어 뭉치다. ubuntu 이미지로 컨테이너를 만든다는 것은 ubuntu 이미지를 위한 레이어가 있는상태에서 writeable한 레이어를 올려놓고 사용하는 것과 같다.

ubuntu 컨테이너를 먼저 생성한다.

```shell
$ docker run -d -it --name mincheol-ubuntu ubuntu:latest
```

생성된 컨테이너를 변경시키기 위해서 해당 컨테이너 bash를 실행한다

```shell
$ docker exec -it mincheol-ubuntu /bin/bash
```

`-i -t`를 이용해 쉘을 통한 상호 입출력이 가능하게 했다. 컨테이너 내부에서 bash shell을 사용할 수 있게 되었다. 여기서 아무런 파일하나를 생성한다

```shell
$ echo "anything" >> anything_file
```

그리고 `CTRL` + `p` + `q`를 사용해서 컨테이너를 빠져나온다.

방금까지 우분투 컨테이너에 새로운 파일을 만드는 작업을 했다. 우분투레이어에서 새로운 레이어가 추가되었다. 이 레이어까지 적용시켜 새로운 이미지를 만들어보자

```shell
$ docker commit -m "make anything" mincheol-ubuntu mincheol-ubuntu:1.0
```

git의 commit과 같이 변경사항에 대한 메시지를 적어주고 이미지 이름 및 이미지 버전을 작성해주고 실행을 시키면 새로운 이미지가 만들어진다

```shell
$ docker images
```

만들어진 이미지를 확인할 수 있다

만들어진 이미지 기반으로 새로운 컨테이너를 만들어보자

```shell
$ docker run -d -it --name mincheol-custom mincheol-ubuntu:1.0
```

컨테이너가 생성되고나면 컨테이너 내부에 anything_file이 존재하는지 확인한다

```shell
$ docker exec -it mincheol-custom /bin/bash
```

```shell
$ ls
```

내부에 anything_file이 있는 것을 확인할 수 있다

## 도커 파일 만들기

도커 파일은 도커 이미지를 생성하기 위한 스크립트 파일이다.

우선은 도커파일을 만들고 실행할 폴더를 만든다

```shell
$ docker mkdir mincheol-dockerfile && cd mincheol-dockerfile/

그리고 도커파일을 작성한다. 도커파일 빌드옵션을 특별히 주지않는한 디폴트 파일명은 `Dockerfile`이다

```shell
$ vi Dockerfile
```

도커 파일은 다음과 같이 작성할 수 있다

```dockerfile
FROM ubuntu:18.04

RUN apt-get update && apt-get install -y apache2

EXPOSE 80

CMD ["apachectl", "-D", "FOREGROUND"]
```

위의 도커파일은 다음과 같은 뜻을 가진다

- `FROM`은 베이스 기반 이미지를 ubuntu:18.04으로 컨테이너를 생성한다
- `RUN`은 레이어를 생성하고 실행된다. 컨테이너 생성후 `apt-get update`로 패키지를 업데이트하고 `apt-get install -y apache2`로 아파치를 설치한다. RUN 커맨드 하나당 하나의 레이어가 생성되고 캐시된다.
- `EXPOSE 80` 옵션으로 해당 이미지에서 열어줄 포트를 지정한다.
- `CMD`로 컨테이너를 생성할 때마다 실행되는 옵션을 설정한다. 위는 컨테이너 생성 이후에 아파치 서버가 항상 실행중인 상태로 돌아가게 만들어준다. apachectl을 foreground 상태로 돌아가도록 한다

도커파일을 작성했으니 Dockerfile을 실행시켜보자

```shell
$ docker build -t mincheol-apache .
```

이렇게 실행하면 `mincheol-apache`라는 이미지가 하나 생기게된다

`docker images`를 통해 생성된 이미지를 확인해보자

```shell
$ docker images
```

### 웹 애플리케이션 빌드 및 배포해보기

```dockerfile
FROM centos:7

RUN yum update -y && \
    yum -y install java-11-openjdk-devel && \
    yum -y install git && \
    yum clean all

WORKDIR /app

RUN git clone https://github.com/kong9410/spring-example.git

WORKDIR /app/spring-example

RUN ./mvnw clean package && \
    rm -rf ./src

EXPOSE 8080

CMD ["java", "-jar", "/app/spring-example/target/spring-example.jar"]
```
위는 다음과 같은 과정을 통해 이미지가 생성이 된다

- centos:7 이미지를 기반으로 생성한다
- jdk11과 git 의존성을 추가한다
- /app 디렉토리로 이동한다
- git clone 을 통해 spring-example을 받는다
- 프로젝트 폴더 내부로 들어간다
- mvnw를 통해 jar로 빌드하고 필요없는 소스파일은 지운다
- expose 를 통해 컨테이너가 노출할 포트를 지정한다
- cmd 로 컨테이너가 실행되면 수행할 명령어를 지정하여 웹 애플리케이션이 실행되도록 한다

도커 파일을 작성했으면 dockerfile을 통해 빌드해보자

```shell
$ docker build -t mincheol-springweb .
```

빌드가 완료되면 이미지가 만들어졌는지 확인해보자

```shell
$ docker images
```

다음 명령어를 통해 만들어진 이미지로 웹 애플리케이션 배포를 해보자

```shell
$ docker run -d -it -p 8083:8080 --name mincheol-springweb mincheol-springweb:latest
```

정상적으로 컨테이너가 작동하는지 확인하고 웹화면을 확인해보자

```shell
$ docker ps
```
```shell
$ curl "http://localhost:8083/test?name=hello"
```

지금은 빌드서버 구분없이 이미지 내부에서 빌드 및 배포를 하도록 만들었지만 jenkins 같은 빌드서버를 따로 두고 jar 파일만 copy 해서 배포를 하게 할 수도 있다


## 도커 컴포즈

도커 컴포즈는 여러개의 컨테이너로부터 이루어진 서비스를 구축, 실행하는 순서를 자동으로 하여, 관리를 간단히하는 기능이다.

`docker-compose.yml`을 작성학 각각 독립된 컨테이너의 실행 정의를 실시한다

```dockerfile
# 빌드를 위한 Base 이미지 지정. 이미지에 builder 별칭 지정 
FROM maven:eclipse-temurin AS builder

WORKDIR /usr/src/
# /usr/src 하위에 guestbook-demo 프로젝트 clone 받음
RUN git clone -b dev https://yung3k7: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@oss.navercorp.com/yung3k7/guestbook-demo.git
WORKDIR /usr/src/guestbook-demo
# /usr/src/guestbook-demo 내에서 메이븐 빌드 수행
RUN mvn clean package

# 배포를 위한 베이스 이미지 지정
FROM openjdk:17-jdk-alpine3.13
WORKDIR /app
# builder 이미지의 /usr/src/guestbook-demo/target/guestbook-demo-0.0.1-SNAPSHOT.jar 파일을 배포 이미지의 ./app.jar 로 가져온다.
COPY --from=builder /usr/src/guestbook-demo/target/guestbook-demo-0.0.1-SNAPSHOT.jar ./app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
```

```yaml
version: "3"

services:
  mysql:
    image: mysql:8.0 # Base 이미지
    container_name: mysql # 컨테이너 이름 설정
    ports: # 호스트와 포트 매핑
      - 3306:3306
    environment: # 환경변수 정의
      MYSQL_ROOT_PASSWORD: test123 # MySQL 컨테이너의 root 비밀번호 지정
    command: # dockerfile 의 CMD 구문과 같다. mysql:8.0 이미지는 docker-entrypoint.sh 스크립트 파일 실행이 Entrypoint 이므로, command 에 지정해주는 값들은 docker-entrypoint.sh 스크립트 파일의 옵션이 된다.
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    volumes: # 도커 볼륨 설정. <호스트 디렉토리>:<컨테이너 디렉토리> 와 같이 호스트 디렉토리와 컨테이너 디렉토리를 매핑.
      - /home/irteam/practice/jys/mysql/data:/var/lib/mysql # mysql 컨테이너의 /var/lib/mysql 디렉토리 하위에 실제 데이터들이 저장되는데, 해당 데이터들을 호스트에도 저장한다. 컨테이너를 사용하면서 영속성을 보장할 수 있다.
      - /home/irteam/practice/jys/mysql/init.sql:/docker-entrypoint-initdb.d/init.sql # 컨테이너가 처음 생성될 경우 실행될 init.sql 파일을 volume을 통해 컨테이너에 전달하여 실행할 수 있도록 한다.

  guestbook:
    container_name: guestbook
    build: . # 현재 디렉토리의 Dockerfile 로 이미지 빌드.
    ports:
      - "8080:8080"
    depends_on:
      - mysql # mysql 컨테이너를 먼저 실행할 것을 보장한다.
```


http://10.168.144.145:8080/guestbook/main