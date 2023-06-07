---
title: daemonset
tags: kubernetes
layout: post
description: kubernetes에서의 daemonset
---

# DaemonSet

데몬셋은 모든 노드가 파드를 정확하게 복사해서 사용하게 도와주는 컨테이너 툴이다

쿠버네티스의 데몬셋 용도

- 모든 노드에서 클러스터 스토리지 데몬 실행
- 모든 노드에서 로그 수집 데몬 실행
- 모든 노드에서 노드 모니터링 데몬 실행

데몬셋은 노드가 pod의 replication을 실행하도록 한다. node가 cluster에 추가되면 pod도 추가된다. node가 cluster에서 제거되면 pod는 garbage로 수집된다. daemonset을 삭제하면 pod들이 정리된다.

### 데몬셋 yaml 예제

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

```shell
kuberctl apply -f ./daemonset.yaml
```

#### required fields

- apiVersion
- kind
- metadata
- 데몬셋 오브젝트 이름은 유효한 DNS 서브 도메인 이름이어야 한다.
- spec 섹션 필요

### 데몬셋 업데이트

- 노드 레이블이 변경되면 데몬셋은 새로 일치하는 노드에 파드를 추가하고 일치하지 않는 노드에서 파드를 제거한다.
- 사용자는 데몬셋이 생성하는 파드를 수정할 수 있다
- 파드는 모든 필드가 업데이트 되는 것을 허용하지 않는다
- 사용자는 데몬셋을 삭제할 수 있다
- 파드를 교체해야 한다면 `updateStrategy`에 따라 파드를 교체한다

![An introduction to Kubernetes DaemonSets](https://www.bluematador.com/hs-fs/hubfs/blog/new/An%20Introduction%20to%20Kubernetes%20DaemonSets/DaemonSets.png?width=770&name=DaemonSets.png)