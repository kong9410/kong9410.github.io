---
title: 쿠버네티스 서비스
tags: kubernetes
layout: post
description: 쿠버네티스 서비스가 뭔지에 대해 알아보자
---

쿠버네티스에서 서비스는 쿠버네티스에서는 외부에서 쿠버네티스 클러스터에 접속하는 방법을 말한다. 서비스를 '소비를 위한 도움을 제공한다'는 관점으로 바라본다며 쿠버네티스가 외부에서 쿠버네티스 클러스터에 접속하기 위한 '서비스'를 제공한다 볼 수 있다.

## 노드포트

외부에서 클러스터 내부에 접속하는 가장 쉬운 방법은 노드포트(NodePort) 서비스를 이용하는 것이다. 노드포트 서비스를 설정하면 모든 워커 노드의 특정 포트를 열고 여기로 오는 모든 요청을 노드포트 서비스로 전달한다. 그리고 노드포트 서비스는 해당 요청을 처리할 수 있는 파드로 요청을 전달한다.

![NodePort](https://hyoublog.com/wp-content/uploads/2020/05/k8s-NodePort-1024x576.png)

노드 포트 서비스는 다음처럼 만들 수 있다

```yaml
apiVersion: v1
kind: Service
metadata:
  name: np-svc
spec:
  selector:
    app: np-pods
  ports:
  # 사용할 프로토콜과 포트들을 지정
    - name: http
      protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30000
# 서비스 타입 지정
type: NodePort
```

기존 파드 구조에서 kind가 Service로 바뀌고, spec에 컨테이너에 대한 정보가 없다.  그리고 접속에 필요한 네트워크 관련 정보와 서비스의 type을 NodePort로 지정했다.

위의 yaml을 다음처럼 실행시킨다

```shell
$ kubectl create -f ./nodeport.yaml
```

서비스는 get services로 확인한다

```shell
$ kubectl get services
```

노드 포트의 번호가 30000번으로 지정된다. CLUSTER-IP는 쿠버네티스 클러스터의 내부에서 사용하는 IP로 자동으로 지정된다

```shell
$ kubectl get nodes -o wide
```

위 명령어로 워커 노드 ip를 확인한다.

노드 ip를 확인했으면 30000번 포트를 통해 브라우저로 접근되는지 확인하면 된다

### 부하분산 테스트

디플로이먼트로 생성된 파드 한개에 접속하고 있는 와중에 파드가 3개로 증가하면 접속이 어떻게 바뀔까?

1. 호스트에서 명령 창을 띄우고 다음 명령어를 실행한다.

```powershell
> $i=0; while($true)
{
  % { $i++; write-host -NoNewLine "$i $_" }
  (Invoke-RestMethod "http:://192.168.1.101:30000")-replace '\n\, " "
}
```

이 명령을 실행하면 다음과 같이 현재 접속한 호스트 이름을 순서대로 출력한다.

```
np-pods-85ddc87668-jvcqq
np-pods-85ddc87668-jvcqq
np-pods-85ddc87668-jvcqq
np-pods-85ddc87668-jvcqq
np-pods-85ddc87668-jvcqq
np-pods-85ddc87668-jvcqq
```

파워셀로 코드를 실행시키고 나면 마스터 노드에서 scale을 실행해 파드를 3개로 증가시킨다

```shell
$ kubectl scale deployment np-pods --replicas=3
```

배포된 pod를 확인한다

```shell
$ kubectl get pods
```

파워셀 명령을 확인하면 파드3개가 돌아가면서 표시된다