---
title: config map
tags: kubernetes
layout: post
description: 쿠버네티스 컨피그맵
---

# 컨피그 맵

- 키-값으로 데이터를 저장하는데 사용하는 API 오브젝트
- 파드는 볼륨에서 환경 변수, 커맨드라인 또는 파일로 컨피그맵을 사용할 수 있다
- 컨피그맵은 컨테이너 이미지에서 환경별로 분리하여 애플리케이션에서 설정정보를 분리 시킬 수 있다
- 보안과 암호화는 지원하지 않기 때문에 기밀 데이터는 시크릿 또는 서드파티 도구를 사용해야한다

## 사용 이유

개발자 컴퓨터(로컬)과 클라우드(리얼)에서 사용할 수 있는 애플리케이션을 개발한다고 가정한다. database 호스트는 로컬에서의 데이터베이스를 사용하고 리얼은 리얼 데이터베이스를 사용한다. 이를 컨피그맵으로 설정해 분리시킬 수 있다

컨피그맵 데이터는 1mib를 초과할 수 없다. 이보다 큰 데이터는 볼륨 마운트나 별도의 데이터베이스 혹은 파일 서비스를 사용해야 한다

## 컨피그맵 오브젝트

컨피그맵은 다른 오브젝트가 사용할 구성요소를 저장할 수 있는 API 오브젝트다. 다른 오브젝트는 spec을 사용하지만 컨피그맵은 data 와 binaryData가 있다. data는 UTF-8 문자열을 포함하고 binaryData는 바이너리 데이터를 base64로 인코딩된 문자열로 포함하도록 설계되어있다.

## 컨피그맵과 POD

컨피그맵을 참조하는 pod spec을 작성하고 컨피그맵의 데이터를 기반으로 해당 pod의 컨테이너를 구성할 수 있다. pod와 configmap은 동일 네임스페이스에 있어야한다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-configmap
data:
  member_name: "testName"
  member_age: 30
  message: "my test message"
  
  application.properties: |
    member.types=naver
    member.retension-time-seconds=10
```

위는 config map의 예시다. 원하는 설정값의 key value를 만들었다

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-sample
spec:
  containers:
  - image: test-image:1.0
    name: app-sample
    env:
    - name: MESSAGE
      valueFrom:
        configMapKeyRef:
          name: demo-configmap
          key: message
  dnsPolicy: ClusterFirst
  restartPolicy: Always
```

위는 pod 설정 yaml이고 configMapKeyRef를 통해 env를 설정하게 했다