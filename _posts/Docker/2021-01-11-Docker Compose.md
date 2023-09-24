---
title: 도커 컴포즈(Docker Compose)
tags: docker
layout: post
description: 여러 컨테이너를 한번에 생성해주는 도커 컴포즈에 대해 정리
---

# 도커 컴포즈

여러 개의 컨테이너가 하나의 애플리케이션으로 동작한다고 할 때 이를 테스트하려면 각 컨테이너를 하나씩 생성해야 한다. 예를 들어 mysql 컨테이너를 사용하는 웹 서버를 만든다고 하자

```shell
docker run --name mysql -d tester/composetest:mysql mysql

docker run -d -p 80:80 --link mysql:db --name web tester/composetest:web apachectl -DFOREGROUND
```

이처럼 여러 컨테이너를 사용하려면 run을 여러번 사용해서 만들어야한다. 이러한 여러개 컨테이너를 하나로 묶어서 관리할 수 있다. 도커 컴포즈는 컨테이너를 이용한 서비스의 개발과 CI를 위해 여러 개의 컨테이너를 하나의 프로젝트로서 다룰 수 있는 작업 환경을 제공한다.

도커 컴포즈는 여러 개의 컨테이너 옵션과 환경을 정의한 파일을 읽어 컨테이너를 순차적으로 생성한다. 도커 컴포즈 설정 파일은 run 명령어 옵션과 각 컨테이너의 의존성, 네트워크, 볼륨 등을 같이 정의할 수 있다. 설정 파일에 정의된 서비스 컨테이너의 수를 유동적으로 조절할 수 있다.

## 도커 컴포즈 설치

```shell
curl -L https://github.com/docker/compose/releases/download/1.11.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose # 심볼릭 링크 지정

docker-compose -v # 설치된 컴포즈 버전 확인
```

## 도커 컴포즈 사용법

도커 컴포즈는 YAML파일을 읽어 컨테이너를 생성하기 때문에 YAML 파일을 먼저 작성해야 한다.

```yaml
# tab은 인식을 못하기 때문에 공백 2개로 구분해야함

version: '3.0'
services:
  web:
    image: ubuntu:latest
	ports:
	  - "80:80"
	links:
      - mysql:db
  mysql:
	image: mysql:latest
	command: mysqld
```

위 명령어는 다음과 같은 도커 명령어와 같다

```shell
docker run -d --name mysql mysql:latest mysqld

docker run -d -p 80:80 \
--link mysql:db --name web \
ubuntu:latest
```



[1] version : YAML 파일 포맷의 버전을 나타낸다. 컴포즈 버전은 도커 엔진 버전에 의존성이 있으므로 가능하다면 최신 버전을 사용할 것을 권장한다.

[2] services : 생성될 컨테이너들을 묶어 놓은 단위이다.

[3] web, mysql : 생성될 서비스의 이름이다. 이 항목 아래에 컨테이너가 생성될 옵션들을 정할 수 있다.



생성된 컨테이너는 `docker ps` 혹은 `docker-compose ps`로 확인할 수 있다.



### 프로젝트, 서비스, 컨테이너

도커 컴포즈는 컨테이너를 프로젝트 및 서비스 단위로 구분하므로 컨테이너 이름은 일반적으로 다음과 같은 방식으로 이루어진다

```
[프로젝트 이름]_[서비스 이름]_[서비스 내에서 컨테이너 번호]
```

위에서 정의한대로 말하면 프로젝트 이름은 따로 정의하지 않았기 때문에 컴포즈 파일이 있는 디렉토리명으로 정의된다. 여기서는 dockercompose라는 디렉토리에서 사용했기 때문에 프로젝트명은 dockercompose, 서비스의 이름은 각 mysql, web이다. 그 이후에 컨테이너 번호를 붙여서 생성된다. `docker-compose scale` 명령어로 컨테이너를 늘려 생성할 수 있다.

```shell
docker-compose scale mysql=2
```

이렇게 지정한다면 dockercompose-mysql-2까지 생성할 수 있다.

> `docker-compose up` 명령어 뒤에 서비스 이름을 입력해 yml에 정의된 특정 서비스 컨테이너만 생성할 수 있다. 
>
> ```shell
> docker-compose up -d mysql
> ```
>
> `docker-compose run` 명령어도 사용할 수 있으며 Interactive 셸을 사용할 수 있다
>
> ```shell
> docker-compose run web /bin/bash
> ```



프로젝트를 삭제하려면 `docker-compose down`으로 삭제할 수 있다. 이 명령어를사용하면 서비스 컨테이너 모두 정지가 된 뒤에 삭제가 된다. `-p` 옵션을 사용하면 프로젝트 이름을 가진 프로젝트를 삭제할 수 있다. 생성할때는 디렉토리명이 아닌 `-p`에 정의된 프로젝트 명으로 생성된다.

`-f` 옵션을 사용하면 도커 컴포즈 파일의 위치를 지정해서 사용할 수 있다

```shell
docker-compose \
-f /home/tester/my_compose_file.yml \
up -d
```



## 컴포즈 활용

### YAML 파일 작성

도커 컴포즈를 사용하려면 컨테이너 설정을 저장해 놓은 YAML 파일이 필요하다. 기존에 사용하던 run 명령어를 YAML 파일로 변환하는 것이 도커 컴포즈 사용법의 대부분이다.

YAML은 크게 버전 정의, 서비스 정의, 볼륨 정의, 네트워크 정의 4가지로 구성된다.



#### 버전 정의

앞서 설명했듯이 YAML의 파일 포맷 버전을 말한다. 최신 버전 사용을 권장한다

```yaml
version: '3.0'
```



#### 서비스 정의

도커 컴포즈로 생성할 컨테이너 옵션을 정의한다. 서비스의 이름은 servies 하위 항목으로 정의하고, 컨테이너의 옵션은 서비스 이름의 하위 항목에 정의한다.

```yaml
services:
  my_container_1:
    image: ...
  my_container_2:
    image: ...
```

다음은 컨테이너 설정에 사용하는 주요 옵션들이다.

- image: 서비스 컨테이너를 생성할 때 쓰일 이미지의 이름을 설정한다. 이미지가 없다면 도커 허브에서 pull한다.

  ```shell
  services:
    my_container_1:
      image: ubuntu:latest
  ```

- links: `docker run`의 `--link`와 같으며 다른 서비스에 서비스명만으로 접근할 수 있도록 설정한다. [SERVICE:ALIAS] 형식을 사용하면 서비스에 별칭으로도 접근할 수 있다.

  ```shell
  services:
    web:
      links:
        - db
        - db:database
        - redis
  ```

- environment: `docker run` 명령어의 `--env`, `-e` 옵션과 동일하다. 딕셔너리나 배열 형태로 사용할 수 있다.

  ```shell
  services:
    web:
      environment:
        - MYSQL_ROOT_PASSWORD=mypassword
        - MYSQL_DATABASE_NAME=mydb
  ```

- command: 컨테이너가 실행될 때 수행할 명령어를 설정한다, `docker run` 명령어 마지막에 붙는 커맨드와 같다. Dockerfile의 RUN과 같은 배열 형태로도 사용할 수 있다.

  ```shell
  services:
    web:
      image: composetest:web
      command: apachectl -DFOREGROUND # 혹은 [apachectl, -DFOREGROUND] 처럼 사용가능
  ```

- depends_on: 특정 컨테이너에 대한 의존 관계를 나타낸다. 이 항목에 명시된 컨테이너 먼저 생성되고 실행이 된다. links도 depends_on과 같이 생성 순서와 실행 순서를 결정하지면 depends_on은 서비스 이름만으로만 접근할 수 있다는것이 다르다.

  ```shell
  services:
    web:
      image: composetest:Web
      depends_on
        - mysql
    mysql:
      image: composetest:mysql
  ```

  특정 서비스의 컨테이너만 생성하되 의존성이 없는 컨테이너를 생성하려면 `--no-deps` 옵션을 사용한다.

  ```shell
  docker-compose up --no-deps web
  ```

  > links, depends_on 모두 애플리케이션이 준비가 된 상태인지에 대해서는 확인하지 않는다. 예를들어 db와 web이 정해진 순서대로 생성될지라도, db가 초기화 중이면 웹 서버 컨테이너가 정상적으로 작동하지 않는다. 이를 해결하는 방법으로 컨테이너에 entrypoint를 지정하는 방법이 있다.
  >
  > ```shell
  > services:
  >   web:
  >   ...
  >     entrypoint: ./sync_script.sh mysql:3306
  > ```
  >
  > entrypoint에 지정된 sync_script.sh는 다른 컨테이너의 애플릴케이션이 준비됐는지 확인하는 명령어를 가진다.

- ports: `docker run`의 `-p`와 같으며 서비스의 컨테이너를 개방할 포트를 설정한다. 그러나 `80:80` 과같이 호스트의 특정 포트를 서비스 컨테이너에 연결하면 `scale` 명령어로 서비스의 컨테이너 수를 늘릴수는 없다.

  ```yaml
  services:
    web:
      image: web:web
      ports:
        - "8080"
        - "8081-8085"
        - "80:80"
  ```

- build: build 항목에 정의된 도커파일에서 이미지를 빌드해 서비스의 컨테이너를 생성하도록 설정한다. 도커파일에 사용될 컨텍스트나 도커파일의 이름, 도커파일에서 사용될 인자 값을 설정할 수 있다. image 항목을 설정하지 않으면 이미지의 이름은 [프로젝트이름]:[서비스의이름]이 된다.

  ```yaml
  services:
    web:
      build: ./composetest
      image: composetest:web
  ```

  ```yaml
  services:
    web:
      build: ./composetest
      context: ./composetest
      dockerfile: myDockerfile
      args:
        HOST_NAME: web
        HOST_CONFIG: self_config
  ```

  > build 항목을 YAML파일에 정의해 프로젝트 생성을 한 뒤에 도커파일을 변경하고 다시 프로젝트를 생성해도 이미지를 새로 빌드하지 않는다. `docker-compose up -d`에 `--build` 옵션을 추가하거나 `docker-compose build [yml 파일에서 빌드할 서비스 이름]`을 사용해 도커파일이 변경돼도 컨테이너를 생성할 때마다 빌드하도록 설정할 수 있다.

- extends: 다른 YAML 파일이나 현재 YAML 파일에서 서비스 속성을 상속받게 설정한다. 다음 예시를 든다.

  ```yaml
  # docker-compose.yml

  version: '3.0'
    services:
      web:
        extends:
          file: extend_compose.yml
          service: extend_web
  ```

  ```yaml
  # extend_compose.yml

  version: '3.0'
    services:
      extend_web:
        image: ubuntu:latest
        ports:
          - "80:80"
  ```

  docker-compose.yml의 web서비스는 extend_compose.yml의 extend_web 서비스 옵션을 그대로 갖게된다. 즉, web 서비스의 컨테이너는 ubuntu:latest 이미지의 80:80 포트로 설정된다. file항목을 설정하지 않으면 현재 YAML 파일에서 extends할 서비스를 찾는다.

  ```yaml
  version: '3.0'
    services:
      web:
        extends:
          service: extend-web
      extend_web:
        image: ubuntu:latest
        ports:
          - "80:80"
  ```

  이 경우엔 같은 yaml 내에서 extend-web을 찾게 된다. depends_on이나 link같은 의존성을 띄고 있으면 extends 받을 수 없다.

#### 네트워크 정의

- driver: 도커 컴포즈는 생성된 컨테이너를 위해 기본적으로 브리지 타입의 네트워크를 생성한다. driver 설정으로 브리지 네트워크가 아닌 다른 네트워크를 사용하도록 설정할 수 있다. 드라이버에 필요한 옵션은 driver_ops로 전달할 수 있다.

  ```yaml
  version: '3.0'
  services:
    myservice:
      image: nginx
      network:
        - mynetwork
  networks:
    mynetwork:
      driver: overlay
      driver_ops:
        subnet: "255.255.255.0"
        IPAdress: "10.0.0.2"
  ```

  위 예제는 mynetwork라는 overlay 타입의 네트워크를 생성하고, myservice 서비스의 네트워크를 mynetwork로 사용할 수 있도록 설정한다.

- ipam: IPAM(IP Address Manager)를 위해 사용할 수 있는 옵션으로서 subnet, ip 범위 등을 설정할 수 있다. driver는 ipam을 지원하는 드라이버여야 한다.

  ```yaml
  services:
    ...
  networks:
    ipam:
      driver: mydriver
      config:
        subnet: 172.20.0.0/16
        ip_range: 172.20.5.0/24
        gateway: 172.20.5.1
  ```

- external: YAML 파일을 통해 프로젝트를 생성할 때마다 네트워크를 생성하는 것이 아닌, 기존의 네트워크를 사용하도록 설정. 사용하려는 외부 네트워크 이름을 하위 항목으로 하고 external값을 true로 한다.

  ```yaml
  services:
    web:
      image: tester/composetest:web
      networks:
        - tester_network
  networks:
    tester_network:
      external: true
  ```

#### 볼륨 정의

- driver: 볼륨을 생성할 때 사용될 드라이버를 설정한다. 기본값은 local로 설정되며 driver_ops로 인자를 설정할 수 있다.

  ```yaml
  ...
  services:
  ...
  volumes:
    driver: flocker
      driver_opts:
        opt : "1"
        opt2 : 2
  ```

- external: 네트워크 external과 마찬가지로 기존에 사용하던 볼륨을 사용할 수 있다. external을 true로 해주어야한다.

  ```yaml
  services:
    web:
      image: tester/composetest:web
      volumes:
        - myvolume: /var/www/html
  volumes:
    myvolume:
      external: true
  ```

#### YAML 파일 검증

yaml 파일의 오타나 포맷이 적절한지 검사하려면 `docker-compose` 명령어를 사용한다. 기본적으로 현재 디렉토리의 docker-compose.yml 검색하도록 하지만, `-f (파일 경로)`옵션을 사용해 검사 경로를 설정할 수 있다.

### 도커 컴포즈 네트워크

YAML파일에 네트워크를 설정하지 않으면 프로젝트별 브리지 네트워크를 생성한다. 네트워크 이름은 {프로젝트 이름}_default로 생성되고 `docker-compose up`로 생성되고 `docker-compose down` 명령어로 삭제된다. `up` 뿐만 아니라 `scale` 로 생성되는 컨테이너도 전부 이 브릿지 네트워크를 사용한다. 서비스 내 컨테이너는 `--net-alias`가 서비스의 이름을 갖도록 자동으로 설정되므로 네트워크에 속한 컨테이너는 서비스 이름으로 서비스 내의 컨테이너에 접근할 수 있다.

web서비스와 mysql 서비스가 각기 존재할 때 web서비스의 컨테이너가 mysql 호스트 이름으로 접근하면 mysql 서비스의 컨테이너중 하나의 ip로 변환되고 mysql 컨테이너가 여러개일 경우 라운드 로빈으로 연결을 분산한다.
