---
title: 파드 생명 주기
tags: kubernetes
layout: post
description: pod life cycle
---

![Kubernetes lifecycle of pod](https://images.velog.io/images/idnnbi/post/018b7022-c037-4b83-a2c8-f769786f1332/image.png)

1. kubectl 을 통해 API 서버에 파드 생성 요청
2. api서버에 전달된 내용이 있으면 api 서버는 etcd에 전달된 내용을 모두 기록해 클러스터의 상태를 최신으로 유지한다.
3. api 서버에 파드생성이 요청된 것을 컨트롤러 매니저가 인지하면 컨트롤러 매니저는 파드를 생성하고 이 상태를 api 서버에 전달한다. 어떤 워커 노드에 파드를 적용할지는 결정되지 않은 상태로 파드만 생성된다.
4. api 서버에 파드가 생성됐다는 정보를 스케줄러가 인지한다. 스케줄러는 생성된 파드를 어떤 워커 노드에 적용할지 조건을 고려해 결정하고 해당 워커 노드에 파드를 띄우도록 요청한다.
5. api 서버에 전달된 정보대로 지정한 워커 노드에 파드가 속해 있는지 스케줄러가 kubelet으로 확인한다
6. kubelet에서 컨테이너 런타임으로 파드 생성을 요청한다
7. 파드가 생성된다
8. 파드가 사용 가능한 상태가 된다.



쿠버네티스는 작업을 순서대로 진행하는 워크플로(workflow) 구조가 아니라 선언적인(declarative) 시스템 구조를 가지고있다. 각 요소가 추구하는 상태(desired status)를 선언하면 현재 상태(current status)와 맞는지 점검하고 그것에 맞추려고 노력하는 구조로 돼 있다는 뜻이다.

