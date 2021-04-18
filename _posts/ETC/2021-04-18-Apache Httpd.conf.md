---
title: Apache Httpd.conf
tags: apache network
---

# Apache Httpd.conf

아파치에 대한 설정이 있는 파일이다. 여러 설정값들이 있지만 자주 사용하는 일부 설정법에 대해서만 기록한다

## ServerRoot

- Apache 서버가 존재하는 디렉토리를 설정
- 다른 지시자의 상대 경로는 ServerRoot 기준으로 지정된다

## Listen

- Apache와 통신하는 포트를 설정
- Listen이 설정되어 있지 않으면 Apache가 실행되지 않음

> 기본포트
> http : 80
> https : 443

- `Listen 80` 혹은 `Listen 443`으로 되어 있는 경우 웹 브라우저로 접근시 `http://example.com`으로의 접근은 `http://example.com:80`과 동일하다. 마찬가지로 `https://example.com`으로 접근시 `https://example.com:443`과 동일하다

## LoadModule

- ServerRoot/modules 디렉토리 안에 있는 모듈을 읽어들이고 사용 가능한 모듈 목록에 추가한다.

> 아파치는 두가지 방식이 있음. 하나는 DSO 모듈 적재. 하나는 Static Object
> DSO: 동적 모듈 적재 방식. 아파치를 먼저 컴파일하고 다른 모듈들을 추가로 설치할 때는 아파치를 재 컴파일하지 않고 한번 설정되어 컴파일된 아파치를 계속 사용하는 것. DSO 방식은 아파치를 단 한번만 컴파일 한다는 것. 필요할 때마다 시스템에 모듈이 적재된다.
> Static Object: 모든 모듈을 아파치 구동과 함께 시스템에 적재된다. 실행은 빠를 수가 있으나 사용하지 않는 모듈을 계속 적재함으로써 시스템 자원을 낭비하게 된다.

## IfModule

- 특정 모듈이 존재하는 경우에만 작동하는 지시자를 설정하기 위해 사용

## ServerAdmin

- 서버에서 오류가 발생했을 때 클라이언트로 전송하는 오류 메세지에 들어갈 이메일 주소를 설정

## ServerName

- 서버가 자신을 식별하기 위해서 사용하는 호스트 이름 및 포트를 설정
- 클라이언트에게 보여주는 호스트 이름을 지정한다.
- 등록된 DNS 이름을 가지고 있지 않다면 IP 주소로 설정
- ServerName을 서버가 확인할 수 업는 IP 주소로 설정하면 Apache 구동 시 경고 메시지를 표시, 시스템에서 사용할 수 있는 모든 호스트 이름을 사용한다.

## DocumentRoot

- Apache가 제공하는 웹 애플리케이션 디렉토리를 설정한다.
- 마지막에 슬래시를 지정해서는 안된다.

## Directory

- 해당 디렉토리 경로 또는 와일드카드, 정규 표현식으로 설정된 디렉토리 또는 파일에 적용되는 옵션을 설정한다.
- Options: 특정 디렉토리에서 설정할 수 있는 서버 기능을 제어한다.
  - All : 모든 Options가 MultiViews에 허용
  - ExecCGI: mod_cgi를 사용하는 CGI 스크립트 실행을 허용
  - FollowSymLinks: 설정한 디렉토리에서 심볼릭 링크 허용
  - Includes: mod_include를 사용하는 SSI(Server Side Includes)를 허용
  - IncludesNOEXEC: SSI를 허용하지만, #exec cmd와 #exec cgi는 사용하지 못한다.
  - Indexes: 클라이언트가 요청한 디렉토리 경로에 DirectoryIndex 지시자에 설정한 파일이 없을 경우, 디렉토리 목록을 화면에 표시한다.
- AllowOverride: .htaccess 파일의 사용 여부를 결정한다. (All | None)
  - 가능하면 사용하지 않길 권장 (성능 하락이 있음)
- Require: 해당 디렉토리의 접근 허용 여부를 설정한다.
  - all denied: 모든 접근을 거부
  - all granted: 모든 접근을 허용
  - ip xxx.xxx.xxx.xxx: 특정 IP의 접근을 허용
  - not ip xxx.xxx.xxx.xxx: 특정 IP의 접근을 거부
  - host example.com: 특정 호스트의 접근을 허용
  - not host example.com: 특정 호스트의 접근을 거부

## DirectoryIndex

- 클라이언트가 파일이 아닌 디렉토리 경로를 요청했을 경우 제공할 파일 목록을 설정한다.
- example.com/abc를 요청했을 때 abc 디렉토리의 index.html을 찾아서 반환

## AccessFileName

- 특정 디렉토리의 접근 제어를 할 파일 이름을 정의한다.
- 해당 디렉토리의 AllowOverride에서 None으로 설정되어 있지 않아야 한다.

## Files

- 해당 파일 경로, 와일드카드 또는 정규 표현식으로 설정된 파일에 대한 옵션을 설정한다.

## ErrorLog

- Apache 에러 로그가 생성될 경로를 지정한다.

## LogLevel

- ErrorLog의 로그 수준을 설정한다
  - emerg: 서버를 동작할 수 없는 긴급상황이 발생했을 경우
  - alert: 반드시 조치해야 할 상황이 발생했을 경우
  - cirt: 매우 치명적인 상황이 발생했을 경우
  - error: 에러가 발생했을 경우
  - warn: 경고 수주의 상황이 발생했을 경우
  - notice: 오류가 아닌 정상적인 상황이지만 중요하여 사용자에게 알려야 할 경우
  - info: 일반적인 서버 운용 로그
  - debug: 매우 자세한 디버그 수준의 로그

## LogFormat

- 커스텀 로그에 사용되는 로그 형식을 설정할 수 있다.
- 설정한 로그 포맷에 이름을 설정하여 간단하게 저장하고 사용할 수 있다.

## CustomLog

- Access 로그 파일이 저장되는 위치와 포맷을 설정한다
- LogFormat에서 지정한 포맷을 이름으로 불러와서 사용할 수 있다

## Alias

- DocumentRoot 하위에 존재하지 않는 디렉토리나 파일에 접근해야하는 경우에 사용한다.
- 특정 디렉터리를 Alias 한다.
- Alias /my-apache/ "/usr/local/apache/" 일 경우 /usr/local/apache 디렉토리를 my-apache라는 이름으로 alias 하는 것이다.
- 앞의 /는 DocumentRoot를 의미

## AddType

- Apache에서 사용할 media-type과 확장자를 매핑하여 추가한다.

## AddEncoding

- Apache에서 사용할 인코딩과 확장자를 매핑하여 추가한다.

## ErrorDocument

- 에러가 발생했을 때 서버가 클라이언트에게 반환할 것을 설정한다.
- 서버는 클라이언트에게 1. 에러 메시지 내역 2. 간단한 텍스트 3. 내부 에러 페이지 4. 외부 에러 페이지를 반환할 수 있다.
- 각 HTTP 상태 코드 별로 반환할 수 있다.

## Include

- httpd.conf가 아닌 다른 설정 파일을 포함하여 적용한다.

## KeepAlive

- 클라이언트와 서버의 연결을 계속 유지할 것인가를 설정한다.
- 기본값은 `On`이고 사용하지 않으려면 `Off`로 설정한다
- http 응답이 빠르고, CPU 사용을 적게 가져감
- 클라이언트와 연결을 유지하기 때문에 지속적으로 메모리를 점유하게됨

## MaxKeepAliveRequests

- 단일 연결이 처리 할 최대 요청 수(50~75가 적당)

## KeepAliveTimeout

- 서버가 연결된 클라이언트의 새 요청을 기다려야하는 시간
- 기본값은 15

## MaxClient

- Apache에서 시작하는 최대 자식 프로세스 수이다. KeepAlive를 활성화하면 사용량이 많은 시간에 더 많은 수의 하위 프로세스가 활성화된다. 그로인해 MaxClients 값을 늘려야 할 수도 있다.

## MaxRequestsPerChild

- 자식 프로세스가 kill되고 재생성 되기 전에 처리 할 요청 수
- request가 child process의 임계치에 다다랐을 때 child process를 죽이고 재생성하게 하는 설정이다.
- 0이면 child process가 죽지않고 계속 request를 처리하게 된다.

## ExtendedStatus

- server-status로 Apache의 상태를 모니터링 할때 상태정보 기능을 제공할 것인지 설저앟는 지시어. 기본값은 Off이다

## ServerSignature

- Apache 버전정보를 웹 브러우저에 노출 할 것인지에 대해 정의한다.
- On또는 Off

## ServerTokens

- httpd.conf 내 정보를 클라이언트에게 얼마나 노출시킬 것인지를 결정한다.
- 옵션
  - Prod: 웹 서버의 이름만 알려준다 (Apache)
  - Major: 웹 서버의 이름과 Major 버전번호만 알려준다. (Apache2)
  - Minor: 웹 서버의 이름과 Minor 버전까지 알려준다. (Apache2.4)
  - Min: 웹 서버 이름과 Minimum 버전까지 알려준다. (Apache2.4.6)
  - OS: 웹 서버의 이름과 버전, OS 정보를 알려준다. (Apache2.4.6 (Unix), 기본값)
  - Full: 최대한의 정보를 모두 알려준다. (Apache2.4.6 (Unix) Resin/4.x.x)

## 모듈

많은 모듈이 있으니 일부만 기록

## MPM 모듈

httpd-mpm.conf를 Include하여 사용

### mpm_prefork

- StartServers: 아파치 서버의 자식 프로세스 개수
- MinSpareServers: 최소한의 유지 개수
- MaxSpareServers: 최대한의 유지 개수
- MaxClients: 초기 시작시 실행가능한 최대 아파치 자식 프로세스 개수
- MaxRequestsPerChild: 클라이언트 요청 개수 제한

### mpm_worker

- StartServers: 시작시에 생성되는 서버 프로세스 개수
- ServerLimit: 구성 가능한 자식 프로세스 제한 수
- MaxClients: 동시에 처리될 최대 커넥션 수
- MinSpareThreads: 최소 스레드 수
- MaxSpareThreads: 최대 스레드 수
- ThreadsPerChild: 자식 프로세스가 지속적으로 가질 수 있는 스레드 수
- MaxRequestsPerChild: 자식 프로세스가 서비스할 수 있는 최대 요청 개수

### mpm_event

- worker와 동일하다

> Apache MPM
>
> 1. prefork
>    - 아파치의 기본 설정임
>    - 자식 프로세스 하나에 스레드 하나를 연결하는 방식
>    - 스레드간 메모리를 공유하지 않음
>    - 프로세스의 메모리도 할당해야 하기 때문에 메모리를 많이 사용함
> 2. worker
>    - 프로세스 하나에 여러개의 스레드를 연결
>    - 스레드간 메모리를 공유
>    - 통신량 많은 서버에 적합함
> 3. event
>    - 아파치 2.4 버전부터 사용
>    - worker 기반
>    - keepalive 시 클라이언트의 요청을 기다리고 있는 자식 프로세스, 스레드 전체가 keep 하게 되는 문제를 해결하기 위해 사용

### mod_jk.so

- 톰캣과 아파치를 연동하기 위한 모듈
- AJP13 프로토콜을 사용하여 통신
- 서블릿을 필요로 하는 요청은 톰캣에서 접속하여 처리한다

#### 추가 설정

```yaml
<IfModule jk_module>
   JkWorkersFile conf/workers.properties # jk 모듈 실행을 위해 선언된 속성 값
   JkLogLevel info # 로깅 레벨 정의
   JkLogFile /etc/apache2/logs/mod_jk.log # 로그 파일 위치
   JkMount /* worker1 # /* url로 접근한 경우 톰캣으로 재전송 즉, url에 요청에 서블릿 관련 처리가 필요하면 workers.properties 파일에 명시된 worker에게로 넘긴다라는 의미다
</IfModule>
```

### workers.properties

JkMount에서 사용한 worker1을 worker.properties 파일에 정의해주어야 한다.

```properties
worker.list=worker1

worker.worker1.type=ajp13
worker.worker1.host=localhost
worker.worker1.port=8009
worker.worker1.lbfactor=1
```

### mod_proxy.so mod_proxy_http.so

아래와 같이 모듈을 불러서 사용

LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so

가상 호스트를 사용하기 때문에 httpd-vhost.conf를 Include해야함

```conf
<VirtualHost 127.0.0.1:80>
   DocumentRoot "/home/web/public.html" # 웹 소스 파일이 있는 디렉토리 절대 경로
   ServerName example.com # 호스트를 제외한 도메인 주소
   ProxyRequests Off # on일 경우 Forward Proxy로 동작, Off일 경우 Reverse Proxy
   ProxyPass / http://127.0.0.1:8080/ # /로 들어온 요청을 127.0.0.1:8080으로 변환
   ProxyPassReverse / http://127.0.0.1:8080/ # 클라이언트 우회시 리버스 프록시 된 웹사이트로 접속하는 것을 방지하기 위해 헤더 정보에서 127.0.0.1:8080을 /에 해당하는 주소로 변경하라는 의미
</VirtualHost>
```