## System Tools

### Consumer offset Checker

Consumer Group, Topic, Partition, Offset, logSize등을 보여줌


```
bin/kafka-run-class.sh kafka.tools.ConsumerOffsetChecker
```

### Dump Log Segment

로그 파일로부터 직접 메세지를 프린트 하거나 단지 인덱스가 정확한지 확인

```
bin/kafka-run-class.sh kafka.tools.DumpLogSegments
```

### Export Zookeeper Offsets

주키퍼에 저장된 브로커 파티션의 offset을 가져와 파일에 기록하는 유틸리지

```
bin/kafka-run-class.sh kafka.tools.ExportZkOffsets
```

### Get Offset Shell

Topic Offset을 출력
```
bin/kafka-run-class.sh kafka.tools.GetOffsetShell
```

### Import Zookeeper Offsets

Export Zookeepr Offsets의 반대로 offset정보를 주키퍼로 업

```
bin/kafka-run-class.sh kafka.tools.ImportZkOffsets
```

### Replay Log Producer

Topic으로 부터 메시지를 받아 다른 Topic으로 입력


```
bin/kafka-run-class.sh kafka.tools.ReplayLogProducer
```


### Simple Consumer Shell

console로 수집된 메시지를 출력하는 Consumer

```
bin/kafka-run-class.sh kafka.tools.SimpleConsumerShell
```


### Update Offsets In Zookeeper

모든 브로커 파티션의 offset을 earliset난 latest로 설정해주는 유틸리티

```
bin/kafka-run-class.sh kafka.tools.UpdateOffsetsInZK
```

### Verify Consumer Rebalance

모든 파티션에 owner가 할당하도록 하는 툴

각 파티션은 `/brokers/topics/[topic]/[broker-id]`
owner는 `/consumers/[consumer_group]/owners/[topic]/[broker_id-partition_id]`에 등록 되어 있음을 의미


```
bin/kafka-run-class.sh kafka.tools.VerifyConsumerRebalance
```
