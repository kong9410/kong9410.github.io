## 리눅스 컨테이너

리눅스 커널을 공유하면서 프로세스를 격리된 환경에서 실행하는 기술이다. 하드웨어 가상화를 하는 VM(Virtual Machine)과 달리 커널을 공유하는 방식이기 때문에 실행 속도가 빠르다. 프로세스는 커널을 공유하지만 리눅스 네임스페이스, 컨트롤 그룹, 루트 디렉터리 격리 등의 커널 기능을 활용해 격리되어 실행된다. 이러한 격리기술 덕분에 호스트는 프로세스로 인식하지만 컨테이너 관점에서는 독립적인 환경을 가진 가상 환경으로 보인다.

### 사용해야 하는 이유

컨테이너는 애플리케이션을 환경에 구애 받지 않고 실행하는 기술이다.  가령 서버에 깃랩을 설치해야 하는데 우분투 환경의 경우는 깃랩 설치는 다음과 같은 방식으로 설치해야한다고 가이드 되어있다.

```shell
# ubuntu
sudo apt-get update
sudo apg-get install -y curl openssh-server ca-sertificates
sudo apt-get install -y postfix
curl https://packages.gitlab.com/install/repositories/gitlab/gitblab-ee/script.deb.sh | sudo bash
sudo EXTERNAL_URL="http://gitlab.exmple.com" apt-get install gitlab-ee
```

하지만 만약 다른 서버에도 깃랩을 설치해야하는데 centOs라면 어떨까?

```shell
# centOS
sudo yum install -y curl policycoreutils-python openssh-server
sudo systemctl enable sshd
sudo systemctl start sshd
sudo firewall-cmd --permanent --add-service=http
sudo systemctl reload firewalld
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
sudo EXTERNAL_URL="http://gitlab.example.com" yum install -y gitlab-ee
```

만일 또다른 환경에서 깃랩을 설치하려고 하면 그에 맞는 명령어를 실행하고 실행환경을 갖춰야할 것이다. 그러나 컨테이너를 사용하게 된다면 어떤 환경이든 상관없이 깃랩을 실행할 수 있다. 깃랩이 이미 설치된 이미지를 만들어서 컨테이너로 만들면 되는 것이다. 이미지라는 것은 뒤에서 설명하겠다.

### 그럼에도 사용해야 하는 이유가 잘 와닿지 않음

애플리케이션 실행을 위해서 실행환경을 맞추어주어야 하는데 A서버는 1년전에 구성했고, B서버는 오늘 구성했다고 가정해본다. 운영체제 버전부터 컴파일러, 설치된 패키지 등이 완벽하게 일치하기란 쉽지 않을 것이다. 이러한 차이점으로 장애가 일어났을때 A서버는 왜 되는거고 B서버는 왜 안되는건가?와 같은 일들이 벌어질 수 있는 것이다. 이렇게 서로 모양이 다른 서버들이 존재하는 상황을 눈송이 서버(Snowflakes Server)라고 한다.

때문에 B서버는 멀쩡하고 A서버가 문제가 생겼을 때 A서버를 잘 아는 사람사람이 필요하다. 또는 운영기록을 서버별로 기록을 해서 해결을 해야할 것이다. 서버가 더 많아지면 더 큰문제가 될 것이다.

아파치 보안 권장 업데이트 사항이있어서 서버별로 아파치를 새로 설치하고 설정파일도 복사하여 옮긴다음에 서버 하나하나 정상으로 띄워졌는지 확인을 해야했다. 만일 기존에 설치됐던 아파치 서버의 버전이 다르다던지 설정파일의 설정이 서버별로 조금씩 다르다면 문제가 생길수도 있다. 또 훗날 서버가 추가되었을 때의 운영체제의 기준이 기존에 사용하던 아파치와는 다를수도 있다.

이러한 애플리케이션의 의존성 문제들과 서로간의 버전차이 환경차이 때문에 컨테이너 기술을 사용하는 것이 전략적으로 더 유리할 수 밖에 없는것이다.

### 주요 특징

- 운영체제 수준의 가상화: 운영체제 수준에서 가상화를 한다. 무슨 말이냐 하면 별도의 하드웨어 에뮬레이션 없이 리눅스 커널을 공유한 컨테이너를 실행하기 때문이다.
- 빠른 속도와 효율성: 하드웨어 에뮬레이션이 없기 때문에 컨테이너는 아주 빠르게 실행된다. 프로세스 격리를 위해 아주 약간의 오버헤드가 있지만 일반적인 프로세스를 실행하는 것과 거의 차이가 없다. 또한 하나의 머신에서 프로세스만큼 많이 실행하는 것이 가능하다.
- 높은 이식성(portability): 모든 컨테이너는 호스트 환경이 아닌 독자적인 실행 환경을 가지고 있다. 이 환경은 파일들로 구성되어 있으며 **이미지** 형식으로 공유될 수 있다. 리눅스 커널을 사용하고 같은 컨테이너 런타임을 사용할 경우 컨테이너의 실행 환경을 공유하고 손쉽게 재현할 수 있다.
- 상태를 가지지 않음(stateless): 컨테이너가 실행되는 환경은 독립적이어서 다른 컨테이너에게 영향을 주지 않는다. 도커와 같은 이미지 기반으로 컨테이너를 실행하는 경우 특정 실행 환경을 재사용할 수 있다.

### 컨테이너는 어떻게 독립된 환경을 갖는가?

![img](https://user-images.githubusercontent.com/37204770/202841405-1a410852-36dc-439a-86c8-eb76cb3ced2e.png)

호스트 OS는 기본적으로 root (/) 폴더를 갖는다. 리눅스는 `chroot`를 사용해서 커스텀한 root 디렉토리를 만들 수 있는데 위 그림과 같이 chroot라는 폴더를 root로 만들어서 프로세스의 루트 디렉토리를 변경할 수가 있다. chroot jail로 분리된 프로세스는 chroot 바깥의 디렉토리에는 접근을 할 수 없다. 대신 호스트는 해당 chroot 내부로 접근할 수 있다. chroot에 어플리케이션을 실행할 수 있는 binaries와 libraries를 만들고 해당 실행환경에서 애플리케이션을 실행하게 된다.

### 컨테이너 이미지란

컨테이너는 독립된 파일시스템을 사용한다. 이러한 커스텀한 파일시스템은 컨테이너 이미지로 제공한다. 이미지가 컨테이너 파일시스템을 포함하고 있으므로 애플리케이션 구동에 필요한 모든 디펜던시, 설정, 스크립트, 바이너리 등을 가지고 있어야한다. 이러한 이미지는 환경변수나 실행하기 위한 디폴트 명령어나 다른 메타 데이터등의 설정 정보 또한 포함한다.

## 도커(Docker)

애플리케이션을 신속하게 구축, 테스트 및 배포할 수 있는 소프트웨어 플랫폼이다. Docker는 소프트웨어를 컨테이너라는 표준화된 유닛으로 패키징 하고, 이 컨테이너에는 라이브러리, 시스템 도구, 코드, 런타임 등 소프트웨어를 실행하는 데 필요한 모든 것이 포함되어 있다. 





## 참고

[컨테이너란? 리눅스의 프로세스 격리 기능 | 44BITS](https://www.44bits.io/ko/keyword/linux-container)

[왜 굳이 도커(컨테이너)를 써야 하나요? - 컨테이너를 사용해야 하는 이유 | 44BITS](https://www.44bits.io/ko/post/why-should-i-use-docker-container)

[I’m in Chroot Jail, Get Me Out of Here! – Security Queens](https://securityqueens.co.uk/im-in-chroot-jail-get-me-out-of-here/)