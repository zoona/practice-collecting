# Kafka Scripts

## 1. Kafka 설치

### 다운로드
```shell
$ wget http://apache.tt.co.kr/kafka/0.10.0.1/kafka_2.10-0.10.0.1.tgz
```

### 압축해제
```shell
$ tar -xvzf kafka_2.10-0.10.0.1.tgz
$ cd kafka_2.10-0.10.0.1
```

### zookeeper 실행
```shell
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```

### kafka server 실행
```shell
$ bin/kafka-server-start.sh config/server.properties
```

## 2. Topic

### create
```shell
$ bin/kafka-topics.sh \
  --create \
  --zookeeper localhost:2181 \
  --replication-factor 1 \
  --partitions 1 \
  --topic test
```

### list
```shell
$ bin/kafka-topics.sh \
  --list \
  --zookeeper localhost:2181
```

### describe
```shell
$ bin/kafka-topics.sh \
  --describe \
  --topic test \
  --zookeeper localhost:2181
```

## 3. Consumer

### consume
```shell
$ bin/kafka-console-consumer.sh \
  --zookeeper localhost:2181 \
  --topic test \
  --from-beginning
```

## 4. Producer

### send message
```shell
$ bin/kafka-console-producer.sh \
  --broker-list localhost:9092 \
  --topic test
```

## 5. Multi Broker

### config 수정
```shell
$ cp config/server.properties config/server-1.properties
$ cp config/server.properties config/server-2.properties
```

```properties
config/server-1.properties:
    broker.id=1
    port=9093
    log.dir=/tmp/kafka-logs-1
    zookeeper.connect=localhost:2181

config/server-2.properties:
    broker.id=2
    port=9094
    log.dir=/tmp/kafka-logs-2
    zookeeper.connect=localhost:2181
```

### start node

```shell
$ bin/kafka-server-start.sh config/server-1.properties
$ bin/kafka-server-start.sh config/server-2.properties
```

### create topic

```shell
$ bin/kafka-topics.sh \
  --create \
  --zookeeper localhost:2181 \
  --replication-factor 3 \
  --partitions 1 \
  --topic topic-r3p0
```

### describe topic

```shell
$ bin/kafka-topics.sh \
  --describe \
  --zookeeper localhost:2181 \
  --topic topic-r3p0
```

## 5. fault-tolerance

### kill process

```shell
$ ps -ef | grep server-1

$ kill `process id`
```

### describe topic

```shell
$ bin/kafka-topics.sh \
  --describe \
  --zookeeper localhost:2181 \
  --topic my-replicated-topic
```

### consume

```shell
$ bin/kafka-console-consumer.sh \
  --zookeeper localhost:2181 \
  --topic my-replicated-topic \
  --from-beginning
```

## 6. Monitoring

### Kafka Tools

ConsumerOffsetChecker

```shell
$ bin/kafka-run-class.sh \
  kafka.tools.ConsumerOffsetChecker \
  --broker-info localhost:9092 \
  --zookeeper localhost:2181 \
  --group `group name`
```

GetOffsetShell

```shell
$ bin/kafka-run-class.sh \
  kafka.tools.GetOffsetShell \
  --broker-list localhost:9092 \
  --topic test \
  --time -2
```

SimpleConsumerShell

```shell
$ bin/kafka-run-class.sh \
  kafka.tools.SimpleConsumerShell \
  --broker-list localhost:9092 \
  --topic test \
  --partition 0
```

### Kafka Monitor

```
https://github.com/linkedin/kafka-monitor
```

### Kafka Manager

```
https://github.com/yahoo/kafka-manager
```

### KafkaOffsetMonitor

[https://github.com/quantifind/KafkaOffsetMonitor](https://github.com/quantifind/KafkaOffsetMonitor)

### 다운로드

```shell
$ wget https://github.com/quantifind/KafkaOffsetMonitor/releases/download/v0.2.1/KafkaOffsetMonitor-assembly-0.2.1.jar
```

### 실행

```shell
$ java -cp KafkaOffsetMonitor-assembly-0.2.1.jar \
  com.quantifind.kafka.offsetapp.OffsetGetterWeb \
  --zk localhost:2181 \
  --port 9991 \
  --refresh 10.seconds \
  --retain 6.hours
```
