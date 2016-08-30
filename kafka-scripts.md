# Kafka Scripts

## 1. Kafka 설치

### 다운로드
```bash
wget http://apache.tt.co.kr/kafka/0.10.0.1/kafka_2.10-0.10.0.1.tgz
```

### 압축해제
```bash
tar -xvzf kafka_2.10-0.10.0.1.tgz
cd kafka_2.10-0.10.0.1
```

### zookeeper 실행

```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
```

### kafka server 실행
```bash
bin/kafka-server-start.sh config/server.properties
```

## 2. Topic

### create

```bash
bin/kafka-topics.sh \
  --create \
  --zookeeper localhost:2181 \
  --replication-factor 1 \
  --partitions 1 \
  --topic test
```

### list

```bash
bin/kafka-topics.sh \
  --list \
  --zookeeper localhost:2181
```

### describe

```bash
bin/kafka-topics.sh \
  --describe \
  --topic test \
  --zookeeper localhost:2181
```

## 3. Consumer

### consume

```bash
bin/kafka-console-consumer.sh \
  --zookeeper localhost:2181 \
  --topic test \
  --from-beginning
```

## 4. Producer

### send message

```bash
bin/kafka-console-producer.sh \
  --broker-list localhost:9092 \
  --topic test
```

## 5. Multi Broker

### config 수정

```bash
cp config/server.properties config/server-1.properties
cp config/server.properties config/server-2.properties
```

```
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

```
bin/kafka-server-start.sh config/server-1.properties
bin/kafka-server-start.sh config/server-2.properties
```

### create topic

```bash
bin/kafka-topics.sh \
  --create \
  --zookeeper localhost:2181 \
  --replication-factor 3 \
  --partitions 1 \
  --topic my-replicated-topic
```

### describe topic

```bash
bin/kafka-topics.sh \
  --describe \
  --zookeeper localhost:2181 \
  --topic my-replicated-topic
```

## 5. fault-tolerance

### kill process

```bash
ps -ef | grep server-1

kill `process id`
```

### describe topic

```bash
bin/kafka-topics.sh \
  --describe \
  --zookeeper localhost:2181 \
  --topic my-replicated-topic
```

### consume

```bash
bin/kafka-console-consumer.sh \
  --zookeeper localhost:2181 \
  --topic my-replicated-topic \
  --from-beginning
```

## 6. Monitoring

### Kafka Tools

ConsumerOffsetChecker

```bash
bin/kafka-run-class.sh \
  kafka.tools.ConsumerOffsetChecker \
  --broker-info localhost:9092 \
  --zookeeper localhost:2181 \
  --group `group name`
```

GetOffsetShell

```bash
bin/kafka-run-class.sh \
  kafka.tools.GetOffsetShell \
  --broker-list localhost:9092 \
  --topic test \
  --time -2
```

SimpleConsumerShell

```bash
bin/kafka-run-class.sh \
  kafka.tools.SimpleConsumerShell \
  --broker-list localhost:9092 \
  --topic test \
  --partition 0
```

### Kafka Monitor

```
git clone https://github.com/linkedin/kafka-monitor.git
cd kafka-monitor
```

```
./gradlew jar
```

```
./bin/kafka-monitor-start.sh config/kafka-monitor.properties
```

```
./bin/end-to-end-test.sh --topic test --broker-list localhost:9092 --zookeeper localhost:2181
```

```
vi config/kafka-monitor.properties
```

```
"jetty-service": {
    "class.name": "com.linkedin.kmf.services.JettyService",
    "jetty.port": 8989
  },
```


### Kafka Manager

https://github.com/yahoo/kafka-manager

```
git clone https://github.com/yahoo/kafka-manager.git
cd kafka-manager
```

```
vi conf/application.conf
kafka-manager.zkhosts="localhost:2181"
```

```
./sbt clean dist
```

```
$ bin/kafka-manager -Dconfig.file=/path/to/application.conf -Dhttp.port=8990
```

### quantifind/KafkaOffsetMonitor

[https://github.com/quantifind/KafkaOffsetMonitor](https://github.com/quantifind/KafkaOffsetMonitor)

### 다운로드

```bash
wget https://github.com/quantifind/KafkaOffsetMonitor/releases/download/v0.2.1/KafkaOffsetMonitor-assembly-0.2.1.jar
```

### 실행

```bash
java -cp KafkaOffsetMonitor-assembly-0.2.1.jar \
  com.quantifind.kafka.offsetapp.OffsetGetterWeb \
  --zk localhost:2181 \
  --port 9991 \
  --refresh 10.seconds \
  --retain 6.hours
```
