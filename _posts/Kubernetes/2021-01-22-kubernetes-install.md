---
layout: post
title: 쿠버네티스 설치
tags: kubernetes linux
---

# 쿠버네티스 설치

## 설치환경

1. 메모리: 2GB, CPU: 2코어
2. 운영체제: 우분투 18.04 LTS
3. 1개 이상의 마스터 노드 서버
4. 1개 이상의 워커 노드 서버

## 설치과정

### 도커 설치

#### 마스터, 워커 노드 모두 설정

**모든 과정은 sudo 권한으로 이루어짐**

1. 스왑메모리 비활성화 후 리부트
   - `swapoff -a`
   - `sed -i '2s/^/#/' /etc/fstab`
   - `reboot`

2. ```shell
   # 패키지 관리 도구 업데이트
   apt update
   apt-get update
   ```

3. ```shell
     apt-get install apt-transport-https ca-certificates curl software-properties-common -y
     ```
   ```
4. ```shell
   # gpg key 내려받기
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
   ```
5. ```shell
   # 패키지 관리도구에 도커 다운로드 링크 추가
   add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   ```

6. ```shell
   # 패키지 관리 도구 업데이트
   apt-get update
   ```

7. ```shell
   # kubernetes에 맞도록 도커 18.06.2~3 버전 설치 (최신버전은 안될 수 있음)
   apt-get install docker-ce=18.06.2~ce~3-0~ubuntu -y
   ```

8. ```shell
   # 도커 명령어가 실행되는지 확인
   docker ps
   ```

9. kubernetes에서 권장하는 도커 데몬드라이버는 systemd 이므로 도커 데몬의 드라이버를 교체한다.

   ```shell
   cat > /etc/docker/daemon.json <<EOF
   {
     "exec-opts": ["native.cgroupdriver=systemd"],
     "log-driver": "json-file",
     "log-opts": {
       "max-size": "100m"
     },
     "storage-driver": "overlay2"
   }
   EOF
   ```

   ```shell
   mkdir -p /etc/systemd/system/docker.service.d
   systemctl daemon-reload
   systemctl restart docker
   ```

### 쿠버네티스 설치

#### 마스터, 워커 노드 둘 다 설치

1. ```shell 
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    ```

2. ```shell
   cat <<EOF | tee /etc/apt/sources.list.d/kubernetes.list deb https://apt.kubernetes.io/ kubernetes-xenial main
   EOF
   ```

3. ```shell
   apt-get update
   ```

4. ```shell
   apt-get install -y kubelet kubeadm kubectl
   ```

5. ```shell
   # 패키지가 자동으로 설치, 업그레이드, 제거되지 않도록 함
   apt-mark hold kubelet kubeadm kubectl
   ```

6. ```shell
   # 설치 완료 확인
   kubeadm version
   kubelet -version
   kubectl version
   ```

## 마스터 노드 세팅

1. 호스트 네트워크의 ip를 확인

   ```shell
   ifconfig
   ```

   마스터 노드의 IPv4 주소를 확인 후 복사

2. 마스터 노드 생성 및 실행

   ```shell
   # 예시로 마스터노드 IP는 192.168.99.102 라고하고
   # 대역폭은 192.168.0.0/16이라고 가정
   kubeadm init --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.168.99.102
   ```

3. 명령어 실행하면 다음과 같은 긴 메시지가 나올텐데 `kubeadm join` 에 해당하는 메시지를 **복사해서 저장해둔다(사용할 수 있는 유효기간이 있으니 주의)**

   ```shell
   kubeadm join 192.168.99.102:6443 --token fnbiji.5wob1hu12wdtnmyr --discovery-token-ca-cert-hash sha256:701d4da5cbf67347595e0653b31a7f6625a130de72ad8881a108093afd06188b
   ```

4. root 계정이 아닌 다른 사용자 계정에서 kubectl 커맨드를 사용하도록 하는 명령어다

   ```shell
   mkdir -p $HOME/.kube
   cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   chown $(id -u):$(id -g) $HOME/.kube/config
   ```

5. 네트워크 설치는 calico로 한다.

   ```shell
   kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
   ```

6. 마스터 노드가 잘 세팅되어 있는지 확인한다.

   ```shell
   kubectl get nodes

   kubectl get pod --namespace=kube-system -o wide
   ```

## 워커 노드 세팅

워커 노드에서 마스터 노드를 세팅할 때 복사해두었던 `kubeadm join` 명령어를 사용한다.

```shell
kubeadm join 192.168.99.102:6443 --token fnbiji.5wob1hu12wdtnmyr --discovery-token-ca-cert-hash sha256:701d4da5cbf67347595e0653b31a7f6625a130de72ad8881a108093afd06188b
```

잘 추가되었는지 확인하려면 마스터 노드에서 `kubectl get nodes` 명령어를 사용한다

```shell
kubectl get nodes
```



### 출처

[쿠버네티스(kubernetes) 설치 및 환경 구성하기. How to configure a Kubernetes cluster \| by ShinChul Bang \| FINDA 기술블로그 \| Medium](https://medium.com/finda-tech/overview-8d169b2a54ff)