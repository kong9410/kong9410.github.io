---
title: statefulset
tags: kubernetes
layout: post
description: kubernetes statefulset
---

# 스테이트풀셋

## 기존 레플리카셋 

레플리카셋이 파드 복제본을 생성할 때 파드의 이름과 IP만 다를 뿐 모두 똑같은 파드를 생성한다. 만약 파드가 PVC를 참조한다면 역시 똑같은 PVC를 연결하게 된다. 해당 PVC는 특정한 하나의 PV에만 연결되어 있다. 기존 컨트롤러에서 각 파드가 분리된 저장소 볼륨을 사용해야 한다면 스테이트풀셋을 사용할 수 있다.

## 스테이트풀셋

- 애플리케이션의 스테이트풀을 관리하는데 사용하는 워크로드 API 오브젝트다
- deployment의 파드 scale을 관리하면서 파드들의 순서를 보장한다
- deployment와 유사하게 동일한 컨테이너 스펙을 기반으로 둔 파드들을 관리한다.
- deployment와 다르게 각 파드의 독자성을 유지한다
- 각 파드들은 동일한 스펙으로 생성되어도 교체는 불가능하다
- 스케줄링 간에도 지속적으로 유지되는 식별자를 가진다
- volumeClaimTemplates를 사용해 PVC를 자동으로 생성할 수 있다
- 각 파드가 순서대로 생성되기 때문에 고정된 이름, 볼륨, 설정을 가질 수 있다

![img](https://blog.kakaocdn.net/dn/cRnKUb/btrcON16KvO/oaTksklRCQi7IDiSd8hCUK/img.png)

- stateful 애플리케이션은 primary 메인 db가 있고 secondary로 primary가 죽으면 대체할 db가 존재하고 이를 감시하는 arbiter가 있다
- 아비터가 죽으면 아비터 역할을 살려줘야 한다
- 각각 역할마다 볼륨을 사용하기 때문에 기존에 사용하던 볼륨에 접근해야 해당 역할을 이어갈 수 있다

## 사용처

다음과 같은것에 유용하다

- 유니크한 네트워크 식별자
- 퍼시스턴트 스토리지
- graceful 배포와 스케일링
- 자동 롤링 업데이트

위와 같은 식별자 및 순차적인 배포, 삭제 또는 스케일링이 필요하지 않으면 디플로이먼트나 레플리카셋이 더 적절하다고 볼 수 있다.

## 제한사항

- 파드에 지정된 스토리지는 persistent volume provisioner를 기반으로 하는 storage class를 요청해서 프로비전해야한다.
- statefulset을 삭제 또는 스케일 다운을 하더라도 스테이트풀셋과 연관된 볼륨이 삭제되지 않는다.
- 현재 파드의 네트워크 식별자를 책임지고 있는 헤드리스 서비스가 필요하다
- 롤링 업데이트와 기본 파드 관리 정책을 함께 사용하면 파손상태로 빠질 수 있다.

### 헤드리스 서비스

스테이트풀셋은 헤드리스 서비스가 필요하다

- 일반적인 서비스는 label selector가 일치하는 랜덤한 포드로 트래픽을 전달한다
- 스테이트풀셋은 포드가 고유하게 식별되야 한다
- 헤드리스 서비스는 서비스의 이름으로 포드의 접근 위치를 알아내기 때문에 서비스의 이름과 포드의 이름을 통해 포드에 직접 접근한다

![img](https://blog.kakaocdn.net/dn/cJlbav/btq75DJQdWf/666aBYXfWJJQt1To07MfO0/img.png)

## 예제

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: myapp-sts
spec:
  selector:
    matchLabels:
      app: myapp-sts
  serviceName: myapp-svc-headless
  replicas: 2
  template:
    metadata:
      labels:
        app: myapp-sts
    spec:
      containers:
      - name: myapp
        image: ghcr.io/c1t1d0s7/go-myweb
        ports:
        - containerPort: 8080
```

스테이트 풀셋 yaml 예제이다 kind가 StatefulSet인거를 제외하면 다른 ReplicaSet과 유사하다

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc-headless
  labels:
    app: myapp-svc-headless
spec:
  ports:
  - name: http
    port: 80
  clusterIP: None
  selector:
    app: myapp-sts
```

스테이트풀셋에서 사용할 헤드리스 서비스다

clusterIP를 None으로 설정해 헤드리스 서비스를 구성한다. 적용은 kubectl create나 apply로 실행하면 된다

이후 `curl 파드이름.서비스이름으로 :포트`로 요청을 보내면 해당 파드에서만 응답이 오는 것을 확인할 수 있다