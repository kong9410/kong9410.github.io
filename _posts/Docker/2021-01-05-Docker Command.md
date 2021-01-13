---
layout: post
title: 도커 명령어
tags: Docker
---

# 도커 명령어

## 컨테이너 생성

1. `docker run -i -t [이미지이름]`

   도커 이미지가 로컬에 존재하지 않는다면 docker 중앙저장소에서 찾아와 내려받고 실행된다.

   -i옵션은 컨테이너와 호스트간의 입출력 상호작용

   -t옵션은 shell을 사용하기 위함

2. `docker create -i -t --name [컨테이너이름] [이미지이름]`

   이미지로 지정한 이름으로 컨테이너를 생성한다.

   run과 달리 컨테이너 내부로 바로 들어가지 않는다

   run과달리 start와 attach 명령이 없다.
   
3. `docker run -i -t -p [호스트아이피 또는 아이피+호스트포트]:[컨테이너포트] [이미지이름]`

   -p를 여러번써서 여러개의 포트를 외부에 개방할 수 있다.

### run 옵션

- -i : 컨테이너와 호스트간 입출력 상호작용

- -t : shell을 사용

- -d : detached 모드로 컨테이너를 실행, detached 모드는 컨테이너를 백그라운드에서 동작하는 애플리케이션으로 실행하도록 설정, 입출력이 없는 상태로 컨테이너 실행. 포그라운드로 실행돼 사용자의 입력을 받지 않는다.

- -p : 포트 설정, [호스트 아이피]:[컨테이너포트]

- -e : 컨테이너 내부의 환경변수 설정

  ```shell
  docker run -d -e MYSQL_ROOT_PASSWORD=mypassword --name mysql mysql:5.7
  ```

- -link : 내부 IP를 알 필요 없이 컨테이너에 alias로 접근하도록 설정한다. 도커 엔진으로 부여받은 IP는 컨테이너가 실행될때마다 바뀌므로 IP로 접근하는 것은 쉽지 않다. 다만 link로 연결된 컨테이너가 중지되어 있으면 link로 연결된 컨테이너를 실행하면 오류가 발생한다. 이 기능은 deprecated되어 있어 도커 브릿지 네트워크 사용을 권장한다.

- -v : [호스트 디렉터리]:[컨테이너 디렉터리] 호스트 디렉터리와 컨테이너 디렉터리를 공유시키는 옵션이다. 이미 있는 디렉터리를 지정할 경우 덮어씌워져 안에 있는 데이터가 지워질수가 있다. 혹은 [볼륨이름]:[컨테이너 디렉터리]로 이미 만들어져있는 볼륨으로 디렉터리를 공유시킬 수 있다.

- --mount: v와 동일한 옵션이나 표기법이 다름, type=?, source=?, target=?

- -volumes-from : -v 옵션을 적용한 컨테이너의 볼륨 디렉터리를 공유할 수 있다

- --net : 네트워크를 설정할 수 있다. host로 설정하면 호스트와 동일한 네트워크 환경을 사용할 수 있다. none을 설정하면 외부와 연결이 되지 않는다. 컨테이너 이름을 입력시 네트워크 환경을 공유하게 된다. 공유되는 속성은 내부ip, 맥 주소 등이다. 컨테이너 지정시 [container:다른컨테이너id] 처럼 입력을한다.

- --net-alias : 특정 호스트 이름으로 여러개의 컨테이너에 접근할 수 있다.

- --log-opt : 로그 옵션 설정

- --memory : 메모리 사용량 제한 (최소4m)

- --memory-swap : 메모리스왑

- --cpu-shares : 컨테이너가 cpu를 얼마나 쓸수있는지 설정, 컨테이너에 cpu 한개씩 할당하는것이 아닌 cpu를 어느 비중만큼 나눠 쓸것인지 명시하는 옵션

- --cpuset-cpus : 특정 cpu만 사용하도록 설정

- --cpu-period : 컨테이너의 CFS(Completely Fair Scheduler) 주기를 변경할 수 있다. (100000=100ms)

- --cpu-quota : 컨테이너의 CFS(Completely Fair Scheduler) 주기를 변경할 수 있다.

- --cpus : period와 quota와 동일하지만 좀더 직관적으로 cpu개수를 지정한다. (0.5 = 50%사용)

- --device-write-bps : 쓰는 작업 초당 제한 설정 [디바이스 이름]:[값]

  ```shell
  docker run -it --device-write-bps /dev/xvda:1mb ubuntu:14.04
  ```

- --device-read-bps : 읽는 작업 초당 제한 설정

- --device-write-iops : 쓰는 작업, 상대적인 값 설정

- --restart: 컨테이너가 종료되었을 때 재시작에 대한 정책 설정 옵션 : [always, on-failure, unless-stopped]



## 이미지 내려받기

1. `docker pull [이미지이름]`

   도커 이미지를 내려받는다.

## 도커 엔진에 존재하는 이미지의 목록 확인

1. `docker images`

## 컨테이너 나가기

1. `ctrl+d`
2. `exit` 입력

위 두개의 방법은 컨테이너가 중지된다.

1. `ctrl + p, q`

컨테이너를 중지하지 않고 shell에서 벗어난다

## 컨테이너 실행

1. `docker start [컨테이너이름]`

## 컨테이너 내부로 이동

1. `docker attach [컨테이너이름]`
2. `docker exec -i -t [컨테이너이름] /bin/bash`: -d옵션으로 생성한 컨테이너는 실행중인 프로그램의 로그만 보게된다. 따라서 컨테이너 내부에서 명령어를 실행한뒤에 그 결과값을 받아야한다. 이 명령어는 -i -t를 이용해 쉘을 통한 상호 입출력이 가능한 형태로 사용했고, 결과값만 보려면 -i -t를 없애면 된다.

## 컨테이너 목록 확인

1. `docker ps` : 실행중인 컨테이너만 출력
2. `docker ps -a` : 정지된 컨테이너도 포함하여 출력
3. `docker ps -q` : 컨테이너의 ID만 출력

| container id                                                 | image              | command                     | create                       | status                                            | ports                                               | names                                 |
| ------------------------------------------------------------ | ------------------ | --------------------------- | ---------------------------- | ------------------------------------------------- | --------------------------------------------------- | ------------------------------------- |
| 컨테이너에게 할당된 고유ID<br />ps명령어로는 일부밖에 안나옴 | 사용한 이미지 이름 | 컨테이너가 시작될 때 명령어 | 컨테이너 생성 뒤에 흐른 시간 | 컨테이너 상태<br />Up: 실행중<br />Exited: 중지됨 | 컨테이너가 개방한 포트, 호스트에 연결된 포트를 나열 | 컨테이너의 고유 이름, 중복될 수 없다. |

## 컨테이너ID 확인

1. `docker inspect [컨테이너이름] | grep Id`

## 컨테이너 이름 변경

1. `docker rename [컨테이너이름] [변경할이름]`

## 컨테이너 삭제

1. `docker rm [컨테이너이름]`
2. `docker rm -f [컨테이너이름]` : 강제 삭제
3. `docker rm $(docker ps -a -q)` : 변수 설정으로 삭제

## 컨테이너 정지

1. `docker stop [컨테이너이름]`
2. `docker stop $(docker ps -a -q)` : 변수로 중단

## 모든 컨테이너 삭제

1. `docker container prune`

## 컨테이너 정보

1. `docker container inspect [컨테이너이름]`

## 도커 볼륨

### 생성

1. `docker volume create --name [볼륨이름]`

### 목록 확인

1. `docker volume ls`

### 정보 출력

1. `docker inspect --type volume`

### 사용하지 않는 볼륨 삭제

1. `docker volume prune`

## 도커 네트워크

### 네트워크 확인

1. `docker network ls`
2. `docker inspect network [네트워크이름]`

### 브릿지 네트워크

#### 네트워크 생성

1. `docker network create --driver bridge [브릿지이름]`
2. `docker run -i -t --name mynetwork_container --net mybridge ubuntu:14.04`

#### 네트워크 끊기

1. `docker network disconnect [브릿지이름] [컨테이너이름]`

#### 네트워크 연결

1. `docker network connect [브릿지이름] [컨테이너이름]`

### MacVLAN

#### 네트워크 생성

```shell
docker network create -d macvlan --subnet=[ip] --ip-range=[ip] --gateway=[ip] -o [options...]
```

## 로깅

### 로그확인

1. `docker logs [컨테이너이름]`: -i -t를 이용하여 생성한 컨테이너에서 셀에서 입출력한 내용을 확인할 수 있다

#### logs 옵션

- `--tail [숫자]`: 끝에 몇줄까지 출력할 것인지
- `--since [날짜]`: 특정 시간 이후 로그를 확인
- `-t` 타임스탬프 표시
- `-f` 로그를 스트림으로 확인



## 도커 이미지 생성

docker commit [OPTIONS] [컨테이너] [레포지토리:태그]

예: `docker commit -a "kong" -m "first commit" myubuntu myubuntu:1.0`

## 도커 이미지 목록

`docker images`

`docker images --filter "label=[검색어]"` 특정 라벨을 갖는 이미지를 찾을 수 있다.

## 도커 이미지 삭제

`docker rmi 이미지이름`

사용중인 컨테이너가 있다면 에러가 발생함

-f 옵션으로 강제로 삭제할 수는 있지만 실제 이미지가 삭제되는 것이 아닌 이미지 이름이 목록에서 사라지는 것이므로 컨테이너 종료후 삭제 권장

그러나 하위 이미지가 있다면 실제로 이미지를 삭제하더라도 이미지 파일을 실제로 삭제하지 않고 레이어에 부여된 이름만 삭제한다.

## 도커 이미지 추출

도커 이미지를 별도로 저장하거나 필요에 따라 이미지를 단일 바이너리 파일로 저장해야 할 때가 있다. docker save 명령어로 이름, 태그 등의 메타데이터를 하나의 파일로 추출할 수 있다.

`docker save -o 추출파일명 이미지이름`

load이미지로 다시 로드할 수 있다. 이전의 이미지와 완전 동일한 이미지가 도커 엔진에 생성됨

`docker load -i 이미지이름`

비슷한 명령어로 import, export가 있다. 그러나 export 명령어는 컨테이너 및 이미지에 대한 설정 정보를 저장하지 않는다.

`docker export -o rootFS.tar mycontainer`

`docker import rootFS.tar myimage:0.0`

 ## 이미지 배포

이미지 이름의 접두어는 이미지가 저장되는 저장소 이름으로 설정해야한다. 보통 사용자의 이름이다.

tag 명령어로 이미지의 이름을 추가할 수 있다.

`docker tag 기존이름 새이름`

이름을 변경해도 기존 이름이 사라지는것은 아니고 한 이미지를 가르키면서 새로운 이름이 추가된다.



`docker push 레포지토리이름/이미지이름:버전`



## 도커 사설 레지스트리 생성

`docker run -d --name 레지스트리이름 -p 포트:포트 registry:버전`

레지스트리 컨테이너를 생성

```shell
curl localhost:5000/v2
```

생성된 컨테이너의 레지스트리 api를 사용가능

레지스트리 컨테이너에 이미지를 올리려면 이미지의 접두어를 레지스트리 컨테이너가 존재하는 호스트의 ip와 레지스트리 컨테이너의 5000번 포트와 연결된 호스트의 포트로 설정해야한다.

다음 명령어로 이름을 변경한다.

`docker tag 이미지이름 도커호스트아이피:포트번호/이미지이름:버전`

그리고 push를 한다

`docker push 호스트아이피:포트번호/이미지이름:버전`

http, https 사용설정이 안되어있으면 push가 안되므로 local에서 push할때 daemon.json에 insecurity-registries를 설정해야 한다.

## 포트 확인

`docker port 컨테이너 이름`

