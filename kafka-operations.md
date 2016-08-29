# Kafka Operations

## Graceful shutdown

- 카프카는 브로커가 실패하거나 내려갔을때 자동으로 감지해 파티션의 새 리더를 선출
- 이는 서버 오류, 유지보수나 설정변경으로 내렸을 때 모두 발생
- 후자일 경우에 카프카는 단순히 죽이는것 보다는 좀더 graceful 한 서버 정지를 지원

장점

- 재시작 시 로그 복구가 필요하지 않도록 모든 로그를 디스크에 동기화 한다. (예를들어 로그 tail에 있는 모든 메세지의 체크섬의 확인)
- 로그 복구는 시간이 걸리기 때문에 재시작의 속도를 높일 수 있다.
- 셧다운 이전에 리더 서버의 파티션들을 다른 레플리카로 migration한다.
- 이는 리더십 전송을 빠르게 하고, 각 파티션이 사용불가능한 시간을 몇 밀리세컨으로 최소화한다.

`controlled.shutdown.enable=true`

## Balancing leadership

- 브로커가 정지하거나 충돌했을 때 프로커 파티션의 리더십은 다른 레플리카로 위임된다.
- 이는 기본적으로 브로커가 재시작 되었을 면 오직 파티션들의 follower만이 될 수 있고 읽기/쓰기 동작으로 사용될 수 없다는 것을 의미한다.

- 이 불균형을 피하기 위해 카프카에는 우선적인 레플리카 개념이 있다.
- 파티션의 레플리카가 1,5,9일 때 1번 노드가 리더 우선권이 있다.
- 왜냐하면 레플리카 리스트에 앞에 있기 때문이다. 다음 커맨드를 실행해 복구된 레플리카에게 리더십을 복구할 수 있다.

```
bin/kafka-preferred-replica-election.sh \
    —zookeeper zk_host:port/chroot`
```

- 이 커맨드를 실행하는게 귀찮기 때문에 다음 configuration을 설정해서 자동화할 수 있다.

`auto.leader.rebalance.enable=true`

## Mirroring data between clusters

- 카프카는 카프카 클러스터간의 데이터 미러링을 위한 툴을 갖고 있다.
- 한개 이상의 source 클러스터로부터 데이터를 읽어 destination cluster로 쓰는 툴이다.

- 미러링의 일반적인 사용예는 다른 데이터 센터로 복제하는 것이다.
source, destination 클러스터는 완전히 독립적이고, 다른 파티션 갯수를 갖고, 오프셋이 같지 않을 것이다.
- 그래서 클러스터 미러링은 fault-tolerance 매커니점으로서의 진정한 목적은 아니다.

- 하지만, 미러링 프로세스는 파티션 메시지 키를 유지하고 사용해서 키 기반에서 순서가 유지는 된다.

## Expanding your cluster

- 카프카 서버에 서버를 추가하는것은 쉽다. 새 서버에서 카프카 유니크 브로커아이디를 할당해 기동하면 된다.
- 하지만 이 새로운 서버는 자동으로 데이터 파티션으로 할당되지 않기 때문에, 파티션들이 새 서버로 이동되지 않는 한, Topic이 새로 생길 때까지 어떤 일도 하지 않는다.
- 그래서 보통 클러스터에 서버를 추가하면 기존 데이터를 이 새 서버로 이동시키려 할 것이다.
- 새 서버를 마이그레이션 하는 파티션의 follower로 추가하고, 모든 기존 데이터를 복제하도록 한다.
- 파티션의 내용이 새 서버에 모두 복제 되고 in-sync 레플리카에 연결 되면, 기존 레플리카중 하나가 파티션의 데이터를 삭제한다.

- —generate: 이 모드는 토픽과 브로커 리스트가 주어지면 툴이 특정 토픽의 모든 파티션들을 새 브로커들로 이동 시킬 재할당 후보를 생성한다. 이 옵션은 단지 주어진 토픽과 브로커 리스트로 파티션 재할당 플랜을 생성하는 편리한 방법을 제공한다.

- —execute: 이 모드는 제공된 재할당 플랜을 기반으로 파티션 재할당을 시작한다. 관리자가 작성한 재배치 플랜이 될수도 있고, -generate 옵션 사용으로 제공될수 있다.

- —verify: 이 모드는, 마지막으로 execute 되고 있는 재할당 상태를 확인한다.
상태는 성공적인 완료, 실패, 수행중일 수 있다.

topic, broker 리스트

```
`“topics”: [{“topic”: “foo1”](),
“topic”: “foo2”],
 “version”:1
}
```

툴 실행
```
bin/kafka-reassign-partitions.sh —zookeeper localhost:2181 —topics-to-move-json-file topics-to-move.json —broker-list “5,6” —generate
```

결과
```
Current partition replica assignment

{“version”:1,
 “partitions”:[{“topic”:”foo1”,”partition”:2,”replicas”:[1,2]()},
   “topic”:”foo1”,”partition”:0,”replicas”:[3,4](),
   “topic”:”foo2”,”partition”:2,”replicas”:[1,2](),
   “topic”:”foo2”,”partition”:0,”replicas”:[3,4](),
   “topic”:”foo1”,”partition”:1,”replicas”:[2,3](),
   “topic”:”foo2”,”partition”:1,”replicas”:[2,3]()]
}

Proposed partition reassignment configuration

{“version”:1,
 “partitions”:[{“topic”:”foo1”,”partition”:2,”replicas”:[5,6]()},
   “topic”:”foo1”,”partition”:0,”replicas”:[5,6](),
   “topic”:”foo2”,”partition”:2,”replicas”:[5,6](),
   “topic”:”foo2”,”partition”:0,”replicas”:[5,6](),
   “topic”:”foo1”,”partition”:1,”replicas”:[5,6](),
   “topic”:”foo2”,”partition”:1,”replicas”:[5,6]()]
}
```

## Decommissioning brokers

- 아직 파티션 재할당 툴은 디커미셔닝 브로커를 위한 재할당 플랜을 자동으로 생성할 수 없다.
- 관리자가 브로커의 호스팅 되는 모든 파티션 레플리카를 디커미션할 브로커로 옮겨한다.
- 브로커 디커미셔닝을 지원하는 툴을 추가할 계획

## Increasing replication factor

- 재할당 json에 추가할 레플리카를 지정하고, -execute 옵션을 사용해 실행

topic, broker partition 리스트
```
{“version”:1,
 “partitions”:[{“topic”:”foo”,”partition”:0,”replicas”:[5,6,7]()}]}
```

툴실행
```
bin/kafka-reassign-partitions.sh —zookeeper localhost:2181 —reassignment-json-file increase-replication-factor.json —execute
```

결과
```
Current partition replica assignment

{"version":1,
 "partitions":[{"topic":"foo","partition":0,"replicas":\[5]()}]}

Save this to use as the --reassignment-json-file option during rollback
Successfully started reassignment of partitions
{"version":1,
 "partitions":[{"topic":"foo","partition":0,"replicas":\[5,6,7]()}]}
```
