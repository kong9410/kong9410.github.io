---
title: 카프카의 내부 동작 원리와 구현 - 레플리케이션
layout: post
tags: kafka
description: 카프카 레플리케이션의 내부 동작
---



## 개요

카프카는 장애 대하여 빠르게 대응함과 동시에 안정성을 확보하기 위해 사용된다. 카프카 레플리케이션 동작을 위해 카프카 토픽 생성시 replication factor라는 옵션을 설정해야한다.

```shell
/usr/local/kafka/bin/kafka-topics.sh --bootstrap-server <카프카서버주소>:9092 --create --topic test01 --partitions 1 --replication-factor 3
```

```shell
/usr/local/kafka/bin/kafka-topics.sh --bootstrap-server <카프카서버주소>:9092 --topic test01 --describe
Topic: test01 PartitionCount: 1 ReplicationFactor: 3 Configs: segment.bytes=1073741824
# 토픽의 파티션 수인 1과 레플리케이션 팩터 수인 3이 표시되어 있다.
Topic: test01 Partition: 0 Leader: 1 Replicas: 1,2,3 Isr: 1,2,3
# 파티션0에 대한 상세 내용. 리더는 브로커1을 나타내고 레플리케이션들은 브로커 1,2,3에 있음을 나타내고 동기화되고 있는 레플리케이션(ISR을 말함)은 1,2,3을 이라는 의미이다.
```

콘솔을 이용해서 test message1이라는 메시지를 test01 토픽으로 전송해보자

```shell
/usr/local/kafka/bin/kafka-console-producer.sh --bootstrap-server <카프카서버주소>:9092 --topic test01
> test message1
```

메시지 전송 이후 해당 메시지가 세그먼트 파일에 저장되었는지 확인해본다.

```shell
/usr/local/kafka/bin/kafka-dump-log.sh --print-data-log --files /data/kafka-logs/test01-0/00000000000000000000.log
Dumping /data/kafka-logs/test01-0/00000000000000000000.log
Starting offset: 0 # 시작 오프셋위치
baseOffset: 0 lastOffset: 0 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 0 CreateTime: 1601008070323 size: 81 magic: 2 compresscodec: NONE crc: 3417270022 isvalid: true # count:1로 메시지 카운트가 1임을 알수있다.
| offset: 0 CreateTime: 1601008070323 keysize: -1 valuesize: 13 sequence: -1 headerKeys: []payload: test message1 # 프로듀서를 통해 메시지가 test message1임을 알 수 있다
```

마찬가지로 브로커 2,3에 접속해서 확인해보면 모든 브로커가 동일한 메시지를 가지고 있음을 확인할 수 있다. 즉 한개의 메시지를 총 3대의 브로커들이 모두 갖고있는 것이다. 즉 장애가 나더라도 마지막 한개의 브로커가 장애나지 않는 이상 클라이언트 요청을 안전하게 처리할 수 있다.

## 리더와 팔로워

리더는 레플리케이션 중에 선정되며 모든 읽기와 쓰기는 리더를 통해서만 가능하다. 다시 말해 프로듀서는 리더에게만 메시지를 전송하고 컨슈머는 리더에게서만 메시지를 가져온다.

![replication](https://user-images.githubusercontent.com/37204770/173222368-448e0b84-c306-4038-9d99-ce9e1be23670.png)

리더를 제외한 나머지 레플리케이션들은 리더를 바라보고 있는 팔로워(Follwer)라고 하며 리더에 문제가 있을 경우를 대비해 언제든지 새로운 리더가 될 준비를 해야한다. 따라서 컨슈머가 토픽의 메시지를 꺼내 가는 것과 비슷한 동작으로 지속적으로 파티션의 리더가 새로운 메시지를 받았는지 확인하고, 새로운 메시지가 있다면 해당 메시지를 리더로부터 복제한다.

### 복제 유지와 커밋

리더와 팔로워는 ISR(InSyncReplica)라는 논리적 그룹으로 묶여있다. 이렇게 그룹을 나누는 이유는 해당 그룹 안에 속한 팔로워들만이 새로운 리더의 자격을 가질 수 있기 때문이다.

ISR내의 모든 팔로워들은 리더의 데이터를 따라가게 되고 리더는 ISR 내 모든 팔로워가 메시지를 받을 때까지 기다린다. 하지만 네트워크 오류, 브로커 장애 등으로 인해 리더로부터 레플리케이션을 하지 못하는 경우도 발생할 수 있다.  만약 이러한 팔로워에게 리더를 넘겨준다면 데이터 정합성이나 메시지 손실 등의 문제가 발생할 수 있다. 따라서 파티션의 리더는 팔로워들이 뒤처지지 않고 레플리케이션 동작을 잘하고 있는지 감시한다. 따라서 리더에 뒤쳐지지 않는 팔로워들만 ISR 그룹에 속하게 된다.

리더는 읽고 쓰는 동작은 물론 팔로워가 레플리케이션 동작을 수행하고 있는지도 판단한다. 만일 팔로워가 특정 주기의 시간만큼 복제 요청이 들어오지 않는다면 리더는 해당 팔로워가 레플리케이션 동작에 문제가 있다고 생각해 ISR 그룹에서 추방한다.

ISR 내에서 모든 팔로워 복제가 완료되면 리더는 내부적으로 커밋되었다는 표시를 하게 된다. 마지막 커밋 오프셋위치는 **하이워터마크(high water mark)**라고 한다. 즉 커밋되었다는 것은 레플리케이션 팩터 수의 모든 레플리케이션이 전부 메시지를 저장했음을 의미한다. 즉 커밋되었다는 것은 레플리케이션 팩터 수의 모든 레플리케이션이 전부 메시지를 저장했음을 의미한다. 이렇게 커밋된 메시지만 컨슈머가 읽어갈 수 있다. 카프카에서 커밋되지 않은 메시지를 컨슈머가 읽을 수 없게 하는 이유는 바로 메시지의 일관성을 유지하기 위해서다.

만일 커밋되지 않은 메시지를 컨슈머가 소비할 수 있게되는 경우 컨슈머가 컨슘 후 리더 파티션이 장애가 발생한다면 레플리케이션 중에 새로운 리더를 선출하게 되는데 새롭게 선출된 리더에는 리더에 있는 메시지가 존재하지 않게된다. 이러한 데이터 불일치가 발생될 수 있기 때문에 커밋은 중요하다.

모든 브로커는 재시작할 때 커밋된 메시지를 유지하기 위해 로컬 디스크의 replication-offset-checkpoint라는 파일에 마지막 커밋 오프셋 위치를 저장한다. replication-offset-checkpoint는 브로커 설정파일에서 설정한 로그 디렉토리 경로에 있다. 로그 디렉토리는 /data/kafka-logs로 설정되어 있으므로 해당 디렉토리 하위에 위치한다.

```shell
cat /data/kafka-logs/replication-offset-checkpoint
test01 0 1 # 토픽이름 파티션번호 커밋된오프셋번호
```

오프셋이 증가하는지 확인하기 위해 test message2를 전송한 뒤 cat 명령어를 다시 실행한다.

```shell
/usr/local/kafka/bin/kafka-console-producer.sh --bootstrap-server <카프카서버주소>:9092 --topic test01
> test message2

cat /data/kafka-logs/replication-offset-checkpoint
test01 0 2 # 오프셋 번호가 2로 증가함을 확인할 수 있다.
```

다른 브로커들에게도 동일한 명령어를 이용해 확이낳면 모두 같은 오프셋 번호임을 알 수 있다. 만일 특정 토픽 또는 파티션에 복제가 되지않거나 문제가 있다고 판단되는 경우, replication-offset-checkpoint 파일의 내용을 확인하고 레플리케이션되고 있는 다른 브로커들과 비교하면 어떤 브로커, 토픽, 파티션에 문제가 있는지 확인할 수 있다.



