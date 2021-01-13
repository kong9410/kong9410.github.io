---
layout: post
title: 도커 파일
tags: Docker
---

# 도커 파일(Dockerfile)

FROM : 생성할 이미지의 베이스가 될 이미지, 반드시 한 번 이상 입력해야 한다.

MAINTAINER: 개발자의 정보를 나타낸다. 도커 1.13.0 ㅓ벚ㄴ 이후로 사용하지 않는다. 대신 다음과 같이 쓸수 있다. LABEL maintainer "kim <kim@kim.com>"

LABEL: 이미지에 메타데이터를 추가한다. 

RUN: 이미지를 만들기 위해 컨테이너 내부에서 명령어를 수행한다.  명령어에 ["/bin/bash", "echo hello >> test2.html"]과 같이 입력하면 /bin/bash 셀을 이용해 "echo hello >> test2.html"을 수행하게 된다. 이처럼 배열 형태로 사용할 경우 다음과같이 수행된다. `RUN ["실행가능파일", "명령어1", "명령어2", ...]`

ADD: 파일을 이미지에 추가한다. 추가하는 파일은 Dockerfile이 위치한 디렉터리인 컨텍스트에서 가져온다.

WORKDIR: 명령어를 실행할 디렉터리를 나타낸다. cd 명령어와 같은 기능을 나타낸다.

EXPOSE: Dockerfile의 빌드로 생성된 이미지에서 노출할 포트를 설정한다. 그러나 반드시 이 포트가 호스트의 포트와 바인딩 되는것은 아니고, 단지 컨테이너의 80번 포트를 사용할 것임을 나타낸다.

CMD: 컨테이너가 시작될 때마다 실행할 명령어를 설정하며, dockerfile에서 한번만 사용할 수가 있다. apachectl -DFOREGROUND 명령어를 넣으면 컨테이너를 생성할 때 자동으로 아파치 웹 서버가 실행된다.

이미지 생성은 `docker build -t 이름:태그 ./` 식으로 사용하면 된다. -t 옵션은 생성될 이미지의 이름을 설정한다. 끝에는 dockerfile이 저장된 경로를 입력한다.

build 명령어는 도커파일에 기록된 대로 컨테이너를 실행한 뒤에 이미지를 만들어 낸다. 그러나 이미지로 만드는 과정이 하나의 컨테이너에서 일어나는 것은 아니다. add, run 등의 명령어가 실행될 때마다 새로운 컨테이너가 하나씩 생성되고 이를 이미지로 커밋한다. 도커파일의 명령어 한줄 한줄이 이미지 레이어로 저장된다는 의미이다.



## 캐시 빌드

도커파일로 빌드를 할 때 이전에 빌드했던 도커파일에 같은 내용이 있다면 새로 빌드하지 않고 이전에 빌드한 이미지 레이어를 활용해 이미지를 생성한다. 그러나 가끔 git clone과 같은 명령어를 사용하게 되면 캐싱기능이 필요하지 않을 때가 있다. 명령어는 전과 동일하지만 내용물이 다를 수가 있기 때문이다. 캐시를 사용하지 않으려면 --no-cache 옵션을 추가해야한다.

`docker build --no-cache -t mybuild:0.0`

또는 캐시로 사용할 이미지를 직접 지정할 수도 있다. 도커 허브의 nginx 공식 저장소에서 nginx:latest 이미지를 빌드하는 dockerfile에 일부 내용을 추가해 사용한다면 nginx:latest 이미지 캐시로 사용할 수 있다.

`docker build --cache-from nginx -t my_extend_nginx:0.0 .`



## 멀티 스테이지

애플리케이션을 빌드할 때 의존성 패키지들과 라이브러리가 필요로 하다. Dockerfile에서 Go 로 작성된 소스코드를 빌드하기 위해서는 Go 관련 빌드 툴과 라이브러리가 미리 설치되어 있어야한다. 단순히 FROM에 이미지를 정의하여 실행하면 소스코드를 빌드할 때 사용된 패키지 및 라이브러리가 불필요하게 이미지의 크기를 차지하고 있는 것을 확인할 수 있다. 이를 줄이기 위해 멀티 스테이지를 사용한다. 하나의 Dockerfile 안에 여러개의 FROM 이미지를 정의해 빌드 완료시 최종 이미지 크기를 줄이는 역할을 한다. 다음과 같이 사용할 수 있다

```dockerfile
FROM golang
ADD main.go /root
WORKDIR /root
RUN go build -o /root/mainApp /root/main.go

FROM alpine:latest
WORKDIR /root
COPY --from=0 /root/mainApp .
CMD ["./mainApp"]
```

처음 빌드에는 main.go 파일을 /root/mainApp으로 빌드했다.

두번째 빌드에서는 copy 명령어는 첫번째 from에서 사용된 이미지의 최종 상태에 존재하는 /root/mainApp 파일을 두 번째 이미지인 alpine:latest에 복사한다.

--from=0는 from에 빌드된 이미지의 최종 상태를 의미한다. 즉, FROM 이미지에서 빌드한 /root/mainApp 파일을 두 번째 FROM에 명시된 이미지인 alpine:latest 이미지에 복사하는 것이다.



## 추가 명령어들

자주 쓰는 옵션들

-ENV: Dockerfile에서 사용될 환경변수를 지정한다. ${ENV_NAME} 과 같은 형태로 사용할 수 있다. 값도 같이 지정을 할수가 있는데 ${ENV_NAME:-value}, ${ENV_NAME:+value}로 사용할 수 있다. 첫번째는 env_name 변수값이 설정되어 있지 않는다면 value를 사용하고 두번째는 env_name의 값이 설정되어있으면 value를 값으로 사용하고 그렇지 않으면 빈문자열을 사용한다.

-VOLUME: 빌드된 이미지로 컨테이너를 생성했을 때 호스트와 공유할 내부의 디렉터리를 설정한다. `VOLUME ["/home/dir", "/home/dir2"]`처럼 json 배열 혹은 `VOLUME /home/dir /home/dir2` 처럼 사용할수가 있다.

-ARG: 빌드시에 dockerfile 내에서 변수로 사용할 값을 추가로 입력받아 사용할수 있다. default값을 지정할 수 있으며 ENV처럼 ${arg_name} 형식으로 사용한다. 빌드시에 `--build-arg <키>=<값>` 옵션을 추가해 지정할 수 있다.

-USER: 컨테이너 내에서 사용될 사용자 계정의 이름이나 UID를 설정하면 그 아래의 명령어는 해당 사용자 권한으로 실행된다.

-ONBUILD: 빌드된 이미지를 기반으로 하는 다른이미지가 DOCKERFILE로 생성될 때 실행할 명령어를 추가한다. onbuild가 설정된 이미지가 생성되면 다른 도커파일에서 이 생성된 이미지를 이용해 이미지를 빌드하게 되면 onbuild 에 정의한 명령어가 실행된다.

-STOPSIGNAL: 컨테이너가 정지될 때 사용될 시스템 콜의 종류를 지정한다.

-HEALTHCHECK: 이미지로부터 생성된 컨테이너에서 동작하는 애플리케이션의 상태를 체크하도록 설정한다. `HEALTHCHECK --interval=1m --timeout=3s --retries=3 CMD curl -f http://localhost || exit 1` 이 명령어는 3번 이상 타임아웃이 발생하면 해당 컨테이너는 unhealthy 상태가 된다.

-SHELL: 사용하려는 셀을 지정하려고 할때 사용한다.

-COPY: 로컬 디렉토리에서 읽어 들인 컨텍스트로부터 이미지에 파일을 복사하는 역할을 한다. 로컬의 파일만 추가 가능하다.

-ADD: COPY와 동일하지만 외부 URL 및 TAR 파일에서도 추가가 가능하다. 그러나 권장되지는 않는다.

-CMD: CMD는 컨테이너가 시작될 때 실행할 명령어를 설정

-ENTRYPOINT: 컨테이너가 시작될 때 수행할 명령을 지정한다. CMD와 달리 커맨드를 인자로 받아 사용할 수 있다. 컨테이너 생성할 때 ENTRYPOINT를 지정하면 CMD는 단지 ENTRYPOINT에 대한 인자의 기능을 한다. 컨테이너가 시작될 때 마다 스크립트 파일이 실행되도록 설정하고, 스크립트 파일을 ENTRYPOINT에 설정하려면 파일의 이름을 ENTRYPOINT 인자로 입력한다.

`docker run -i -t --name entrypoint_sh --entrypoint="/test.sh" ubuntu:14.04 /bin/bash`

단 실행할 스크립트 파일은 컨테이너 내부에 존재해야 한다. 이를 위해 사용하기 좋은게 COPY다

1. 어떤 설정 및 실행이 필요한지에 대해 스크립트 정리
2. COPY로 스크립트 이미지로 복사
3. ENTRYPOINT를 이 스크립트로 설정
4. 이미지를 빌드해 사용
5. 스크립트에서 필요한 인자는 docker run 명령어에서 cmd로 entrypoint의 스크립트로 전달



## 도커파일 빌드시 주의사항

좋은 습관

하나의 명령어를 역슬래시로 나누어서 가독성을 높일 수 있도록 작성하거나, .dockerignore파일을 작성해 불필요한 파일을 빌드 컨텍스트에 포함하지 않는 것이 있다. 또는 빌드 캐시를 이용해 기존에 사용했던 이미지 레이어를 재사용하는 방법도 도커파일을 활용하는 방법 중 하나이다.