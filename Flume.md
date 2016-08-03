# 소개

다양한 소스에서 발생한 대량의 로그 데이터를 중앙 데이터 스토어로 효과적으로 수집 집계\(aggregating\)하거나 이동시킬 수 있는 신뢰할수있는 분산 시스템

스트림 지향의 데이터 플로우를 기반으로 하며 지정된 모든 서버로 부터 로그를 수집한 후 하둡 HDFS와 같은 중앙 저장소에 적재하여 분석하는 시스템을 구축해야 할 때 적합

데이터 소스를 커스터마이징 할 수 있기 때문에 로그 데이터 수집에 제한되지 않고, 소셜미디어 데이터, 이메일 메세지등 다량의 이벤트 데이터를 전송하는데에 사용할 수 있다

시스템 요구사항

* Java Runtime Environment - Java 1.6 or later \(Java 1.7 Recommended\)

* Memory - Sufficient memory for configurations used by sources, channels or sinks

* Disk Space - Sufficient disk space for configurations used by channels or sinks

* Directory Permissions - Read\/Write permissions for directories used by agent


## 구조

![](https://flume.apache.org/_images/UserGuide_image00.png)

### Data flow model

Flume Event는 **byte 단위의 데이터**와 **속성\(optional\)**들을 포함한 데이터 흐름의 단위

Flume Agent는 Event들이 외부에서 다음 목적지로 흐르는 Component들을 호스트하는 JVM 프로세스

Flume source는 웹서버와 같은 외부 source에 의해 전달 된 이벤트들을 수집함

외부 source는 대상 Flume Source가 인식 할 수 있는 형식으로 이벤트를 전송함

예를들어 Avro 클라이언트나 Avro Sink로부터 온 이벤트를 보내는 Flow의 Flume Agent로부터 Avro 이벤트를 수신하는데에 Avro Flume Source가 쓰일 수 있다.

A similar flow can be defined using a Thrift Flume Source to receive events from a Thrift Sink or a Flume Thrift Rpc Client or Thrift clients written in any language generated from the Flume thrift protocol.

Thrift 프로토콜도 마찬가지

When a Flume source receives an event, it stores it into one or more channels.

Flume Source 가 이벤트를 수신하면 한개 이상의 채널에 저장한다.

The channel is a passive store that keeps the event until it’s consumed by a Flume sink.

Channel은 Flume Sink에 의해 소비될 때 까지 이벤트를 저장한다.

The file channel is one example – it is backed by the local filesystem.

The sink removes the event from the channel and puts it into an external repository like HDFS \(via Flume HDFS sink\) or forwards it to the Flume source of the next Flume agent \(next hop\) in the flow. 

Sink는 Channel에서 이벤트를 가져와 HDFS와 같은 외부 저장소나 flow의 다음 Flume Source로 보낸다.

The source and sink within the given agent run asynchronously with the events staged in the channel.

Agent의 Source와 Sink는 Channel에 저장 된 이벤트들과 비동기로 실행 된다.

### Complex flows

Flume으로 이벤트들이 종단에 도착하기 전에 여러개의 Agent들을 통해 전송되는 Multi-hop Flow를 구성할수 있다.

Fan-in과 Fan-out, Contextual routing과 fail-over를 위한 백업 전송을 지원

### Reliability

The events are staged in a channel on each agent.

The events are then delivered to the next agent or terminal repository \(like HDFS\) in the flow.

The events are removed from a channel only after they are stored in the channel of next agent or in the terminal repository.

This is a how the single-hop message delivery semantics in Flume provide end-to-end reliability of the flow.

Flume uses a transactional approach to guarantee the reliable delivery of the events.

The sources and sinks encapsulate in a transaction the storage\/retrieval, respectively, of the events placed in or provided by a transaction provided by the channel. 

This ensures that the set of events are reliably passed from point to point in the flow. 

In the case of a multi-hop flow, the sink from the previous hop and the source from the next hop both have their transactions running to ensure that the data is safely stored in the channel of the next hop.

### Recoverability

The events are staged in the channel, which manages recovery from failure. 

Flume supports a durable file channel which is backed by the local file system. 

There’s also a memory channel which simply stores the events in an in-memory queue, which is faster but any events still left in the memory channel when an agent process dies can’t be recovered.


# Setup

## Agent 설정

Agent에 대한 설정을 로컬 파일에서 관리

각 Source, Sink, Channel에 대한 설정과 이들이 어떻게 엮여있는지를 설정

한개 이상의 Agent 설정을 한 설정 파일에서 관리

Java Properties 형식을 따름

### Configuring individual components

Each component (source, sink or channel) in the flow has a name, type, and set of properties that are specific to the type and instantiation. 

각 Component(Source, Sink, Channel)는 종류와, 인스턴스를 정의하는 name, type, property들을 갖음

For example, an Avro source needs a hostname (or IP address) and a port number to receive data from. 

예를들어 Avro Source는 데이터 수신을 위한 hostname과 port number가 필요함

A memory channel can have max queue size (“capacity”), and an HDFS sink needs to know the file system URI, path to create files, frequency of file rotation (“hdfs.rollInterval”) etc.

메모리 채널은 최대 큐 사이즈를 지정할 수 있고, HDFS Sink는 파일시스템 URI와 파일을 생성할 경로, 파일 로테이션 주기 등이 필요함

All such attributes of a component needs to be set in the properties file of the hosting Flume agent.

이러한 모든 속성들은 Flume Agent의 속성 파일에 정의되어 있어야 함

### Wiring the pieces together

The agent needs to know what individual components to load and how they are connected in order to constitute the flow. 

Agent는 어떤 Component가 로딩되어 어떻게 연결되어 flow를 구성해야 되는지를 알아야 한다.

This is done by listing the names of each of the sources, sinks and channels in the agent, and then specifying the connecting channel for each sink and source.

각 source, sink, channel들의 이름을 열거하고 각 Sink와 Source간에 Channel의 연결을 지정

 For example, an agent flows events from an Avro source called avroWeb to HDFS sink hdfs-cluster1 via a file channel called file-channel. 

예를들어 avroWeb이란 이름의 Avro Source로부터 hdfs-cluster1이란 이름의 HDFS Sink로 file-channel이란 이름의 file channel을 통해 이벤트를 흘린다면

The configuration file will contain names of these components and file-channel as a shared channel for both avroWeb source and hdfs-cluster1 sink.

설정 파일은 이런 component들의 이름과 avroWeb Source와 hdfs-cluster1 sink 모두의 공유 channel로써의 file channel을 포함할 것임

### Starting an agent

An agent is started using a shell script called flume-ng which is located in the bin directory of the Flume distribution.

Agent는 Flume 배포의 bin 디렉티로에 있는 flume-ng라는 shell script로 시작된다.

You need to specify the agent name, the config directory, and the config file on the command line:

agent 이름과 config 디렉토리, config파일을 커맨드라인에 입력해야한다.


```bash
$ bin/flume-ng agent -n $agent_name -c conf -f conf/flume-conf.properties.template
```

Now the agent will start running source and sinks configured in the given properties file.

Agent는 이제 설정 파일에 설정한 대로 source와 sink들을 실행 시킨다.


### A simple example

```
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444
# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

This configuration defines a single agent named a1. 

이 설정은 a1이라는 agent를 정의함

a1 has a source that listens for data on port 44444, a channel that buffers event data in memory, and a sink that logs event data to the console. 

a1은 44444 포트로 데이터를 listen하는 source, 메모리에 이벤트 데이터를 버퍼링하는 channel, 콘솔로 이벤트 데이터를 로깅하는 sink를 갖고 있다.

The configuration file names the various components, then describes their types and configuration parameters. 

설정 파일은 다양한 component들을 이름짓고, 그것들의 종류와 속성값들을 설정한다.

A given configuration file might define several named agents; when a given Flume process is launched a flag is passed telling it which named agent to manifest.

이 설정파일은 몇몇 agent들을 이름지어 정의할 수 있는데, Flume 프로세스가 기동될 때 어떤 agent를 실행할 지 flag를 넘겨받게 된다.

이 설정 파일은 아래와 같이 실행 시킬 수 있다.

```
$ bin/flume-ng agent --conf conf --conf-file example.conf --name a1 -Dflume.root.logger=INFO,console
```

Note that in a full deployment we would typically include one more option: --conf=

The <conf-dir> directory would include a shell script flume-env.sh and potentially a log4j properties file. 

실제 배포시에는 `--conf=<conf-dir>` 옵션을 추가하는데 `flume-env.sh'를 포함하고, log4j 속성 파일이 포함된 디렉토리를 지정한다.

In this example, we pass a Java option to force Flume to log to the console and we go without a custom environment script.

예제에서는 콘솔에 로깅하고, 커스텀 환경 스크립트 없이 실행하도록 Java 옵션을 넘겼다.

다른 터미널에서 텔넷 포트 44444로 접속 할 수 있고, Flume에 이벤트를 보낼 수 있다.

```
$ telnet localhost 44444

Trying 127.0.0.1...

Connected to localhost.localdomain (127.0.0.1).

Escape character is '^]'.

Hello world! <ENTER>

OK
```

### Zookeeper based Configuration

### Installing third-party plugins

### The plugins.d directory

### Directory layout for plugins


## Data ingestion

## Setting multi-agent flow

## Consolidation

## Multiplexing the flow

# Configuration

## Defining the flow

## Configuring individual components

## Adding multiple flows in an agent

## Configuring a multi agent flow

## Fan out flow

## Flume Sources

### Flume Sinks

### Flume Channels

## Flume Channel Selectors

## Flume Sink Processors

## Event Serializers

### Flume Interceptors

### Flume Properties

# Log4J Appender

# Load Balancing Log4J Appender

# Security

# Monitoring

## JMX Reporting

## Ganglia Reporting

## JSON Reporting

## Custom Reporting

## Reporting metrics from custom components

# Tools

## File Channel Integrity Tool

## Event Validator Tool

# Topology Design Considerations

## Is Flume a good fit for your problem?

## Flow reliability in Flume

## Flume topology design

## Sizing a Flume deployment

# Troubleshooting

## Handling agent failures

## Compatibility

### Tracing

### More Sample Configs

# Component Summary

# Alias Conventions

