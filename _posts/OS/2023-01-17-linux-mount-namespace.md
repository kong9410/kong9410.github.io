---
title: 리눅스 마운트 네임스페이스
tags: linux os
layout: post
description: 리눅스의 마운트와 마운트 네임스페이스에 대해서 알아보자
---

## 마운트

![image](https://user-images.githubusercontent.com/37204770/213207584-7f9a8bbf-9c9b-41c5-ae7f-744822b5a81e.png)

마운트(mount)란 붙이는(attach) 것이라고 보면된다. 리눅스의 파일 시스템은 모두 '/'를 기반으로 한 큰 트리구조를 하고 있다. 이러한 파일시스템을 가지는 리눅스에서 새로운 장치가 연결되면 기존 파일시스템의 빈 디렉토리에 연결이되고 마운트된 파일시스템의 최상위 레벨 디렉토리가 기존 파일 시스템의 디렉토리가 된다.

파일 시스템을 마운트 한다는 것은 단순히 linux 디렉토리 특리의 특정 지점에서 특정 파일 시스템에 액세스 할 수 있도록 말하는 것이다. 파일시스템을 마운트할 때 파일시스템이 하드디스크 파티션인지 CD-ROM인지 플로피 혹은 USB인지는 중요하지 않다.

다음은 CD-ROM 장치가 `/dev/cdrom`일때 `/media/cdrom`으로 마운트를 하는 command이다.

```bash
sudo mount /dev/cdrom /media/cdrom
```

이 이후에 CD-ROM에 있는 `/dir/file`은 `/media/cdrom/dir/file`로 접근할 수 있다.

여기서 `/dev/cdrom`은 파일 시스템을 거치지 않고 드라이브에서 직접 데이터를 읽는것이다. 마운트는 데이터가 포함된 장치(`/dev/cdrom`)를 디렉터리(`/media/cdrom`)와 연결할 뿐만 아니라 장치의 데이터가 구성되는 방식을 이해하고 파일 및 디렉토리로 표시하는 파일 시스템 드라이버와도 연결한다.

## 마운트 네임스페이스(mount namespace)

### 네임스페이스(namespace)

- 하나의 시스템에서 수행되지만, 각각 별개의 독립된 공간인 것처럼 격리된 환경을 제공하는 경량 프로세스(쓰레드) 가상화 기술
- 네임스페이스의 목적 중 하나는 컨테이너의 구현을 돕는 것. 컨테이너는 경량 가상화를 위한 툴이고 프로세스 그룹에게 그들이 시스템에서 유일한 프로세스들이라는 환상을 제공한다.
- 리눅스 네임스페이스 구현 시 unshare, setns 시스템 콜을 사용
- unshare: 현재 프로세스를 새로 지정된 네임스페이스에 연결
- setns: 이미 존재하는 네임스페이스에 프로세스 연결

### 마운트 네임스페이스

![image](https://user-images.githubusercontent.com/37204770/213208132-60c8c925-926f-47f2-92a9-1728a16a7b30.png)

- 프로세스와 그 자식 프로세스가 각기 다른 파일시스템 마운트 지점을 제공
- 기본적으로 모든 프로세스는 동일한 기본 네임스페이스를 공유
- clone() 시스템 콜을 통해 프로세스를 생성할때 CLONE_NEWNS 플래그가 전달되면 새로 생성되는 프로세스는 호출한 프로세스가 갖고 있는 마운트 트리의 사본을 가져옴
- 이 시점부터 기본 네임스페이스의 모든 파일 시스템 마운트 및 마운트 해제는 새로운 네임스페이스에서 볼 수 있지만 각 프로세스별 마운트 네임스페이스 내부에서의 변경은 해당 프로세스의 네임스페이스 밖에서는 알 수 없다.
- 시스템이 시작되면 하나의 마운트 네임스페이스가 있고 이것을 **initial namespace**라고 부른다. 새로운 마운트 네임스페이스는 clone 또는 unshare 시스템 콜 그리고 CLONE_NEWNS 플래그를 사용하여 만들어진다.
- 새로운 마운트 네임스페이스가 만들어지면 clone 혹은 unshare를 호출한 호출한 네임스페이스의 마운트 포인트 리스트의 복사본을 받는다. (마운트 포인트 리스트 위치 : `/proc/mounts`)
- clone 또는 unshare 시스템 콜에 따르면 마운트 포인트들은 각각의 네임스페이스에서 독립적으로 추가되거나 제거될 수 있다.
- 마운트 포인트 리스트의 변경사항들은 프로세스가 거주하는 마운트 네임스페이스의 프로세스들에게만 보여진다. 다른 마운트 네임스페이스에서는 변경사항들이 보이지 않는다.

1. unshare를 통해 현재 bash 프로세스를 자체 마운트 네임스페이스로 이동

   ```bash
   $ unshare -m /bin/bash
   ```

2. 부모 프로세스는 자식 프로세스의 마운트 포인트 리스트를 볼 수 없고, 자식은 부모의 마운트 포인트 리스트를 볼 수 있다. readlink를 통해 네임스페이스 inode 번호 확인

   ```bash
   $ readlink /proc/$$/ns/mnt
   mnt:[56789326] # inode 번호
   ```

3. 임시 마운트 지점을 만들어 마운트 시킨다

   ```bash
   $ mount -n -t tmpfs tmpfs /tmp/mount_ns
   ```

4. 새로 만든 네임스페이스에서 마운트 지점을 볼 수 있는지 확인

   ```bash
   $ df -h | grep mount_ns
   tmpfs 7.8G 0 7.8G 0% /tmp/mount_ns
   $ cat /proc/mounts | grep mount_ns
   tmpfs /tmp/mount_ns tmpfs rw,realtime 0 0
   ```

   마운트 지점은 우리가 생성한 네임스페이스의 일부이고 현재의 bash 프로세스가 우리가 생성한 네임스페이스로부터 실행되어서 해당 마운트 지점을 볼 수 있다.

5. 새 터미널을 열고 해당 터미널 세션에서 네임스페이스의 inode ID를 확인하면 다른걸 알 수 있다.

   ```bash
   $ readlink /proc/$$/ns/mnt
   mnt:[589387589]
   ```

6. 마운트한 지점이 새 터미널에서 표시되지 않는다

   ```bash
   $ cat /proc/mounts | grep mount_ns
   $ df -h | grep mount_ns
   ```

#### shared subtrees

- 마운트 네임스페이스의 유용성 문제때문에 존재한다

- 마운트 네임스페이스는 네임스페이스간 강력한 격리를 제공한다.

- 새디스크가 마운트되면 모든 마운트 네임스페이스에서 보이려면 각각 네임스페이스에 따로 디스크 마운트를 해야한다.

- 대게 모든 마운트 네임스페이스에서 디스크를 볼 수 있게 만드는 마운트 명령은 한번만 수행되는 것이 바람직하다.

- 이러한 문제때문에 shared subtress feature가 Linux 2.6.15 버전에 추가되었다. shared subtrees 주요 이점은 네임스페이스간 mount, unmount 이벤트가 자동으로 제어되고 전파될 수 있다.

- 4개의 각기다른 propation type이 존재한다.

  - MS_SHARED: peer group의 멤버인 다른 마운트 포인트들과 mount와 unmount 이벤트를 공유한다.

    ![image](https://user-images.githubusercontent.com/37204770/213208668-be558c48-44e5-46db-8b54-2ff03eb0a211.png)

  - MS_PRIVATE: MS_SHARE와 반대이다. 어떠한 peer에게도 이벤트를 전파하지 않는다.

    ![image](https://user-images.githubusercontent.com/37204770/213208699-6c29aecc-e0b3-4468-bce3-777ec0c7571e.png)

  - MS_SLAVE: shared와 private의 중간이다. 슬레이브 마운트는 멤버가 mount, unmount 이벤트를 슬레이브 마운트로 전파하는 공유 peer 그룹인 마스터가 있다.

    ![image](https://user-images.githubusercontent.com/37204770/213208765-a7d8a2d8-879c-4b63-93d3-92228d8e0b8d.png)

  - MS_UNBINDABLE: 이 마운트 포인트는 바인딩 할 수 없다. private 마운트 포인트 처럼 이 마운트 포인트는 피어로부터 이벤트를 전파하지 않는다.

### peer groups

- mount, unmount 이벤트들을 다른 마운트 포인트에 전파하는 마운트 포인트들의 셋이다.
- peer 그룹은 propagation type이 share인 마운트 포인트가 새 네임스페이스 생성 중에 복제되거나 bind mount의 소스로 사용될 때 새로운 멤버를 얻는다. 두 경우 모두 새로운 마운트 포인트가 기존 마운트 포인트와 동일한 peer 그룹의 멤버로 지정된다.
- 반대로 마지막 멤버 프로세스가 종료되거나 다른 네임 스페이스로 이동하여 마운트 네임스페이스가 종료될 때 peer 그룹이 unmount하면 마운트 포인트가 peer 그룹의 멤버를 그만둔다.

## pivot_root

마운트 네임스페이스는 다른 프로세스가 볼 수 있는 마운트 지점을 제한한다. `chroot`의 경우에는 프로세스가 보는 루트 디렉터리 '/'를 제어한다. 만일 "/A"를 파일시스템의 새 루트 디렉토리로 지정할경우 프로세스는 "/A" 하위 디렉토리로 접근하려 할 것이다.

그러나 chroot 를 사용하면 탈옥 문제를 야기할수도 있다. chroot는 프로세스가 보는 root 디렉토리를 바꾸기는 하지만 실제로는 fake root이기 때문에 isolation이 되지 않기 때문에 호스트의 filesystem, process tree 등에 접근이 가능하다. 또한 root 권한 사용이 남용될 수있고, resource를 무제한으로 사용할 수 있다.

![image](https://user-images.githubusercontent.com/37204770/213216015-39c16f84-7cff-4250-97a9-154a6beb4389.png)

이를 위해 루트 디렉토리를 바꿔야하는 경우 pivot_root라는 것을 사용한다. 이것은 마운트 네임스페이스의 root mount 지점을 바꿔버린다.

![image](https://user-images.githubusercontent.com/37204770/213216049-b2f04d53-7b3d-48d3-8431-f2bb9f61fd93.png)

## 출처

[What does it mean to mount a file system in linux? - Stack Overflow](https://stackoverflow.com/questions/29446836/what-does-it-mean-to-mount-a-file-system-in-linux)

[mount - What is meant by mounting a device in Linux? - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/3192/what-is-meant-by-mounting-a-device-in-linux)

[mount - What is meant by mounting a device in Linux? - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/3192/what-is-meant-by-mounting-a-device-in-linux)

[[Linux\] 마운트 네임스페이스(Mount namespace) 1 (tistory.com)](https://milhouse93.tistory.com/85)

[Mini Container Series Part 1 (hechao.li)](https://hechao.li/2020/06/09/Mini-Container-Series-Part-1-Filesystem-Isolation/)