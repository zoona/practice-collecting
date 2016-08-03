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

Thrift 프로토콜도 마찬가지

When a Flume source receives an event, it stores it into one or more channels.

Flume Source 가 이벤트를 수신하면 한개 이상의 채널에 저장한다.

The channel is a passive store that keeps the event until it’s consumed by a Flume sink.

Channel은 Flume Sink에 의해 소비될 때 까지 이벤트를 저장한다.

Sink는 Channel에서 이벤트를 가져와 HDFS와 같은 외부 저장소나 flow의 다음 Flume Source로 보낸다.

Agent의 Source와 Sink는 Channel에 저장 된 이벤트들과 비동기로 실행 된다.

### Complex flows

Flume으로 이벤트들이 종단에 도착하기 전에 여러개의 Agent들을 통해 전송되는 Multi-hop Flow를 구성할수 있다.

Fan-in과 Fan-out, Contextual routing과 fail-over를 위한 백업 전송을 지원

### Reliability

이벤트는 각 Agent의 channel에 임시저장됨.

이벤트는 flow의 다음 agent나 HDFS와 같은 종단 저장소로 전달됨.

이벤트는 다음 agent의 channel이나 종단 저장소에 저장된 후에만 channel에서 삭제 됨.

이것이 Flume에서 단일 hop 메세지 전달 의미구조가 flow의 end-to-end 신뢰성을 제공하는 방식임.

Flume은 이벤트 전달의 신뢰를 보장하기 위해 트랜젝션 접근 방식을 사용함.

The Sources and Sinks encapsulate the storage/retrieval of the Events in a Transaction provided by the Channel.

source와 sink는 각각 이벤트의 쓰기/읽기가 channel이 제공하는 트랜젝션 안에 있음.

각각 source와 sink는 배치되거나 channel에 의해 제공된 트랜젝션에 의해 제공된 이벤트들의 저장/검색 트랜젝션이나 캡슐화한다.

이것은 flow에서 이벤트셋이 포인트에서 포인트로 신뢰할 수 있게 지나가게 해줌.

다중 hop flow의 경우, 이전 hop의 sink와 다음 hop의 source는 둘다 data가 안전하게 다음 hop의 channel에 저장하도록 하는 트랜젝션을 갖고 있음.

### Recoverability

The events are staged in the channel, which manages recovery from failure.

이벤트는 실패로부터 복구를 관리하는 channel에 임시 저장됨.

Flume supports a durable file channel which is backed by the local file system.

Flume은 로컬 파일 시스템에 위치하는 내구성의 파일 channel을 지원함

메모리 큐에 이벤트를 저장하지만 속도가 빠르지만 agent가 죽어 복구가 불가능할 때에도 여전히 메모리에 남겨지는 메모리 channel도 지원함

# Setup

## Agent 설정

Agent에 대한 설정을 로컬 파일에서 관리

각 Source, Sink, Channel에 대한 설정과 이들이 어떻게 엮여있는지를 설정

한개 이상의 Agent 설정을 한 설정 파일에서 관리

Java Properties 형식을 따름

### Configuring individual components

각 Component(Source, Sink, Channel)는 종류와, 인스턴스를 정의하는 name, type, property들을 갖음

예를들어 Avro Source는 데이터 수신을 위한 hostname과 port number가 필요함

메모리 채널은 최대 큐 사이즈("capacity")를 지정할 수 있고, HDFS Sink는 파일시스템 URI와 파일을 생성할 경로, 파일 로테이션 주기("hdfs.rollInterval") 등이 필요함

All such attributes of a component needs to be set in the properties file of the hosting Flume agent.

이러한 모든 속성들은 Flume Agent의 속성 파일에 정의되어 있어야 함

### Wiring the pieces together

Agent는 어떤 Component가 로딩되어 어떻게 연결되어 flow를 구성해야 되는지를 알아야 한다.

각 source, sink, channel들의 이름을 열거하고 각 Sink와 Source간에 Channel의 연결을 지정

예를들어 avroWeb이란 이름의 Avro Source로부터 hdfs-cluster1이란 이름의 HDFS Sink로 file-channel이란 이름의 file channel을 통해 이벤트를 흘린다면


설정 파일은 이런 component들의 이름과 avroWeb Source와 hdfs-cluster1 sink 모두의 공유 channel로써의 file channel을 포함할 것임

### Starting an agent

Agent는 Flume 배포의 bin 디렉티로에 있는 flume-ng라는 shell script로 시작된다.

agent 이름과 config 디렉토리, config파일을 커맨드라인에 입력해야한다.


```bash
$ bin/flume-ng agent -n $agent_name -c conf -f conf/flume-conf.properties.template
```

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

이 설정은 a1이라는 agent를 정의함

a1은 44444 포트로 데이터를 listen하는 source, 메모리에 이벤트 데이터를 버퍼링하는 channel, 콘솔로 이벤트 데이터를 로깅하는 sink를 갖고 있다.

설정 파일은 다양한 component들을 이름짓고, 그것들의 종류와 속성값들을 설정한다.

이 설정파일은 몇몇 agent들을 이름지어 정의할 수 있는데, Flume 프로세스가 기동될 때 어떤 agent를 실행할 지 flag를 넘겨받게 된다.

이 설정 파일은 아래와 같이 실행 시킬 수 있다.

```
$ bin/flume-ng agent --conf conf --conf-file example.conf --name a1 -Dflume.root.logger=INFO,console
```

실제 배포시에는 `--conf=<conf-dir>` 옵션을 추가하는데 `flume-env.sh'를 포함하고, log4j 속성 파일이 포함된 디렉토리를 지정한다.

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
Flume이 실행 된 터미널에서는 아래와 같이 로그가 찍힌다.

```
12/06/19 15:32:19 INFO source.NetcatSource: Source starting
12/06/19 15:32:19 INFO source.NetcatSource: Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/127.0.0.1:44444]
12/06/19 15:32:34 INFO sink.LoggerSink: Event: { headers:{} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21 0D Hello world!. }
```

### Zookeeper based Configuration

Flume은 실험적으로 Zookeeper를 통한 Agent 설정을 지원함.

설정 가능한 prefix하에 설정파일이 Zookeeper에 업로드 되어야 함.

설정은 Zookeeper 노드 데이터에 저장 됨.

아래는 Agent a1과 a2를 위한 Zookeeper의 노드 트리의 모습임

```
- /flume
 |- /a1 [Agent config file]
 |- /a2 [Agent config file]
```

설정 파일이 업르도 되면, 아래 옵션과 함께 실행 함

```
$ bin/flume-ng agent –conf conf -z zkhost:2181,zkhost1:2181 -p /flume –name a1 -Dflume.root.logger=INFO,console
```


| Argument Name | Default |	Description |
| ---- | ---- | ---- |
| z	| –	| Zookeeper connection string. Comma separated list of hostname:port |
| p	| /flume | Base Path in Zookeeper to store Agent configurations |


### Installing third-party plugins

Flume은 plugin 기반의 구조를 가지고 있음.

Flume은 많은 source, channel, sink, serializer 등등을 포함하지만, Flume과 별도로 많은 implementation들이 존재한다.

커스텀 Flume Components들을 flume-env.sh에 있는 FLUME_CLASSPATH에 지정한 경로에 jar를 추가해 포함시킬 수 있지만, 특정 형식의 패키지를 자동으로 불러오는 plugins.d라는 특별 디렉토리를 제공함

이는 프러그인 패키지 이슈 관리를 쉽게 해줄 뿐만 아니라, 특히 라이브러리 종속성 충돌과 같은 몇몇 클래스의 트러블슈팅이나 디버깅을 간편하게 해준다.

#### The plugins.d directory

plugins.d 디렉토리는 $FLUME_HOME/plugins.d에 위치함.

시작시에 flume-ng 스크립트는 아래 형식을 따르는 프러그인을 plugins.d 디렉토리에서 찾고, java 시작시에 적절한 경로에 포함시킴.

#### Directory layout for plugins

plugins.d 안에 있는 각 플러그인은 3가지의 하위 디렉토리를 갖을 수 있음.

1. lib - 플러그인의 jar
2. libext - 플러그인의 종속 jar
3. native - 필요한 네이티브 라이브러리 (.so)

plugins.d 디렉토리의 2개 플러그인에 대한 예제

```
plugins.d/
plugins.d/custom-source-1/
plugins.d/custom-source-1/lib/my-source.jar
plugins.d/custom-source-1/libext/spring-core-2.5.6.jar
plugins.d/custom-source-2/
plugins.d/custom-source-2/lib/custom.jar
plugins.d/custom-source-2/native/gettext.so
```

## 데이터 유입

Flume은 외부 소스로부터 데이터를 유입시키기 위한 몇가지 메커니즘을 지원한다.

### RPC

Flume에 포함되어 있는 Avro 클라이언트는 avro RPC 메커니즘을 사용해서 파일을 Flume Avro source로 보낼 수 있다.

```
$ bin/flume-ng avro-client -H localhost -p 41414 -F /usr/logs/log.10
```
위 커맨드는 /usr/logs/log.10를 저 포트로 리스닝하고 있는 Flume source로 보낸다.

### Executing commands

주어진 커맨드를 실행하고, carriage return이나 line feed, 혹은 둘다가 붙는 텍스트와 같은 싱글라인 텍스트를 수집하는 exec source가 있음.

Note: Flume은 source로 tail을 지원하지 않음. 그 file 스트림에 대한 exec source안에 tail 커맨드를 래핑할 수 있음

### Network streams

Flume은 아래와 같은 보편적인 로그 스트림들로부터 데이터를 읽기 위한 매커니즘을 지원함
  - Avro
  - Thrift
  - Syslog
  - Netcat

## Setting multi-agent flow

![](https://flume.apache.org/_images/UserGuide_image03.png)

다중 Agent나 hop을 통하는 데이터를 흘리기 위해, 이전 Agent의 sink와 현재 홉의 source는
sink가 source의 hostname과 port를 바라보는 avro 형태일 필요가 있음.

## 병합(Consolidation)

로그 수집의 아주 일반적인 시나리오는 많은 수의 로그 생성 클라이언트가 스토리지 하위시스템에 있는 몇개의 수집 Agent들로 데이터를 보내는 것임.

예를들어, 100개의 웹서버로부터 수집되는 로그들이 HDFS클러스터에 기록하는 10개의 Agent들로 보내짐.

![](https://flume.apache.org/_images/UserGuide_image02.png)

첫 단계의 Agent들을 avro sink로 설정하고, 모두 한개의 avro source를 바라보게 하는 방식으로 Flume에서 수행할 수 있음.
(thrift source/sink/client를 사용할 수도 있음)

두번째 단계 Agent의 이 source는 수신 된 이벤트들을 마지막 단계로의 sink에 의해 수집되는 한개의 channel로 병합함.


## Multiplexing the flow

Flume은 한개 이상의 목적지로로 보내지는 이벤트 flow의 multiplexing을 지원함.

이벤트를 한개 이상의 채널로 복제하거나 선택적으로 전송할 수 있는 flow multiplexer를 정의해서 수행함.

![](https://flume.apache.org/_images/UserGuide_image01.png)

위 예제는 3개의 다른 channel로 flow를 펼치는 "foo" agent로 부터의 소스임.

이 fan out은 복제되거나 multiplexing될 수 있음.

복제할 경우, 각 이벤트는 3개의 channel 모두로 전송됨.

multiplexing일 경우, 이벤트의 속성이 설정된 값과 매칭 될 때 가능한 채널로 전송 됨.

예를들어, "txnType"이란 이벤트 속성이 "customer"로 설정된 경우, channel1과 channel3으로 전송되어야하고
"vendor"라고 설정된 경우, channel2 아니면 channel3로 전송되어야 함.

이 맵핑은 agent의 설정파일에 설정될 수 있음.

# Configuration

앞선 설명과 같이 Flume Agent의 설정은 계층을 갖는 속성 설정의 Java Property 파일 형식과 닮은 파일로 부터 읽어 들여짐.

## Defining the flow

한개의 Agent안의 흐름을 정의하려면, source들과 sink들을 channel로 연결해야함.

agent에 사용할 source와 sink, channel을 리스팅하고, source와 sink가 channel을 바라보게 함.

source 인스턴스는 다수의 channel을 지정할 수 있지만, sink 인스턴스는 한개의 channel만 지정할 수 있음

```
# list the sources, sinks and channels for the agent
<Agent>.sources = <Source>
<Agent>.sinks = <Sink>
<Agent>.channels = <Channel1> <Channel2>

# set channel for source
<Agent>.sources.<Source>.channels = <Channel1> <Channel2> ...

# set channel for sink
<Agent>.sinks.<Sink>.channel = <Channel1>
```

예를들어, agent_foo라는 이름의 agent가 외부 avro 클라이언트로부터 데이터를 읽고, memory channel을 통해 HDFS로 보낼 경우
설정 파일은 다음과 같음

```
# list the sources, sinks and channels for the agent
agent_foo.sources = avro-appserver-src-1
agent_foo.sinks = hdfs-sink-1
agent_foo.channels = mem-channel-1

# set channel for source
agent_foo.sources.avro-appserver-src-1.channels = mem-channel-1

# set channel for sink
agent_foo.sinks.hdfs-sink-1.channel = mem-channel-1
```

이는 mem-channel-1이란 메모리 channel을 통해 avro-AppSrv-source로부터 hdfs-Cluster1-sink로의 이벤트 flow를 생성함.

Agent가 weblog.config을 설정파일로 사용해 실행되면, 이대로 flow를 초기화 함.

## Configuring individual components

flow를 정의한 후 source, sink, channel 각각의 속성들을 정의해야 함.

이는 component 타입과 각 component의 속성값을 설정한것과 같은 계층의 namespace 안에서 이뤄짐.

```
# properties for sources
<Agent>.sources.<Source>.<someProperty> = <someValue>

# properties for channels
<Agent>.channel.<Channel>.<someProperty> = <someValue>

# properties for sinks
<Agent>.sources.<Sink>.<someProperty> = <someValue>
```

"type" 속성은 어떤 종류의 오브젝트가 필요한지를 알려주기 위해 각 Flume component들에 설정되어야 함.

각 source, sink와 channel 타입들은 각각의 기능을 수행하기 위한 고유의 속성들을 갖고 있음.

필요에 따라 속성들이 설정되어야 함.

이전 예제에서의 memory channel mem-channel-1을 통해 avro-AppSrv-source에서 hdfs-Cluster1-sink로 흐르는 flow에 대한
각 component들의 설정

```
agent_foo.sources = avro-AppSrv-source
agent_foo.sinks = hdfs-Cluster1-sink
agent_foo.channels = mem-channel-1

# set channel for sources, sinks

# properties of avro-AppSrv-source
agent_foo.sources.avro-AppSrv-source.type = avro
agent_foo.sources.avro-AppSrv-source.bind = localhost
agent_foo.sources.avro-AppSrv-source.port = 10000

# properties of mem-channel-1
agent_foo.channels.mem-channel-1.type = memory
agent_foo.channels.mem-channel-1.capacity = 1000
agent_foo.channels.mem-channel-1.transactionCapacity = 100

# properties of hdfs-Cluster1-sink
agent_foo.sinks.hdfs-Cluster1-sink.type = hdfs
agent_foo.sinks.hdfs-Cluster1-sink.hdfs.path = hdfs://namenode/flume/webdata

#...
```

## Adding multiple flows in an agent

단일 Flume Agent는 몇몇 독립적인 flow를 포함 할 수 있음.

config에 여러개의 source, sink, channel을 나열할 수 있음.

이 component들은 다중 flow의 형태로 연결 될 수 있음

```
# list the sources, sinks and channels for the agent
<Agent>.sources = <Source1> <Source2>
<Agent>.sinks = <Sink1> <Sink2>
<Agent>.channels = <Channel1> <Channel2>
```

그러면 2개의 다른 flow를 설정하기 위해 source들과 sink들을 channel(sink를 위한)에 대응하는 channel들(source를 위한)로 연결할 수 있음

예를들어, 한 agent 안에서 두개의 flow를 설정할 필요가 있고,
한 flow는 외부 avro 클라이언트로부터 외부 HDFS로,
그리고 다른 flow는 tail의 출력으로부터 avro sink로 간다면 설정은 다음과 같다.

```
# list the sources, sinks and channels in the agent
agent_foo.sources = avro-AppSrv-source1 exec-tail-source2
agent_foo.sinks = hdfs-Cluster1-sink1 avro-forward-sink2
agent_foo.channels = mem-channel-1 file-channel-2

# flow #1 configuration
agent_foo.sources.avro-AppSrv-source1.channels = mem-channel-1
agent_foo.sinks.hdfs-Cluster1-sink1.channel = mem-channel-1

# flow #2 configuration
agent_foo.sources.exec-tail-source2.channels = file-channel-2
agent_foo.sinks.avro-forward-sink2.channel = file-channel-2
```

## Configuring a multi agent flow

다 단계의 flow를 설정하기 위해, 다음 hop의 avro/thrift source를 가리키는 첫번째 hop의 avro/thrift sink가 필요함.

이 결과는 다음 Flume Agent로 이벤트를 전달하는 첫 Flume Agent가 될 것이다.

예를들어, avro 클라이언트를 사용해 주기적으로 파일들을 로컬 Flume Agent로 보내고 있다면(이벤트 당 1 파일),
이 로컬 Agent는 그것을 스토리지에 마운트된 다른 agent로 전달 할 수 있다.

Weblog Agent 설정

```
# list sources, sinks and channels in the agent
agent_foo.sources = avro-AppSrv-source
agent_foo.sinks = avro-forward-sink
agent_foo.channels = file-channel

# define the flow
agent_foo.sources.avro-AppSrv-source.channels = file-channel
agent_foo.sinks.avro-forward-sink.channel = file-channel

# avro sink properties
agent_foo.sources.avro-forward-sink.type = avro
agent_foo.sources.avro-forward-sink.hostname = 10.1.1.100
agent_foo.sources.avro-forward-sink.port = 10000

# configure other pieces
#...
```

HDFS Agent 설정

```
# list sources, sinks and channels in the agent
agent_foo.sources = avro-collection-source
agent_foo.sinks = hdfs-sink
agent_foo.channels = mem-channel

# define the flow
agent_foo.sources.avro-collection-source.channels = mem-channel
agent_foo.sinks.hdfs-sink.channel = mem-channel

# avro sink properties
agent_foo.sources.avro-collection-source.type = avro
agent_foo.sources.avro-collection-source.bind = 10.1.1.100
agent_foo.sources.avro-collection-source.port = 10000

# configure other pieces
#...
```

weblog Agent의 avro-forward-sink를 hdfs Agent의 avro-collection-source로 연결 함.

결국 외부 appserver 소스로부터 오는 이벤트가 HDFS에 저장되게 됨

## Fan out flow

지난 섹션에서 논의한대로, Flume은 flow의 한 source로 부터 여러 channel로의 fanning out을 지원함.

replicating, multiplexing 두가지 모델이 있음.

replicating flow에서는 이벤트가 모든 설정된 channel들로 전송됨.

multiplexing에서는 이벤트가 제한 된 channel들로만 보내짐.

flow를 fan out 하려면, source를 위한 channel들의 목록과 fan out 정책을 정의해야 함.

이는 replicating이나 multiplexing될 수 있는 channel "selector"를 추가해서 할 수 있음.

multiplexing의 경우 selection 룰을 더 정의 해야 함.

selector를 지정하지 않는다면, replicating이 기본값임.

```
# List the sources, sinks and channels for the agent
<Agent>.sources = <Source1>
<Agent>.sinks = <Sink1> <Sink2>
<Agent>.channels = <Channel1> <Channel2>

# set list of channels for source (separated by space)
<Agent>.sources.<Source1>.channels = <Channel1> <Channel2>

# set channel for sinks
<Agent>.sinks.<Sink1>.channel = <Channel1>
<Agent>.sinks.<Sink2>.channel = <Channel2>

<Agent>.sources.<Source1>.selector.type = replicating
```

multiplexing select는 flow를 분기하는 속성들을 더 갖고 있음.

이벤트 속성과 channel셋간의 맵핑 정의를 해야 함.

selector는 이벤트의 각 설정된 속성을 체크 함.

만약 특정 값과 매칭되면, 이벤트가 값기 맵핑되는 모든 channel들로 보내짐.

매칭되는 것이 없다면, 이벤트는 default로 설정 된 channel로 보내짐

```
# Mapping for multiplexing selector
<Agent>.sources.<Source1>.selector.type = multiplexing
<Agent>.sources.<Source1>.selector.header = <someHeader>
<Agent>.sources.<Source1>.selector.mapping.<Value1> = <Channel1>
<Agent>.sources.<Source1>.selector.mapping.<Value2> = <Channel1> <Channel2>
<Agent>.sources.<Source1>.selector.mapping.<Value3> = <Channel2>
#...

<Agent>.sources.<Source1>.selector.default = <Channel2>
```

각 값들에 channel들을 오버랩핑 할 수 있음

다음은 두 방향으로 multiplexing되는 단일 flow의 예제임.

agent_foo란 이름의 agent는 한개의 avro source와 두 sink로 연결 된 두개의 channel로 되어 있음

```
# list the sources, sinks and channels in the agent
agent_foo.sources = avro-AppSrv-source1
agent_foo.sinks = hdfs-Cluster1-sink1 avro-forward-sink2
agent_foo.channels = mem-channel-1 file-channel-2

# set channels for source
agent_foo.sources.avro-AppSrv-source1.channels = mem-channel-1 file-channel-2

# set channel for sinks
agent_foo.sinks.hdfs-Cluster1-sink1.channel = mem-channel-1
agent_foo.sinks.avro-forward-sink2.channel = file-channel-2

# channel selector configuration
agent_foo.sources.avro-AppSrv-source1.selector.type = multiplexing
agent_foo.sources.avro-AppSrv-source1.selector.header = State
agent_foo.sources.avro-AppSrv-source1.selector.mapping.CA = mem-channel-1
agent_foo.sources.avro-AppSrv-source1.selector.mapping.AZ = file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.mapping.NY = mem-channel-1 file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.default = mem-channel-1
```

selector는 "State"라는 헤더를 체크함.

만약 값이 "CA"라면 mem-channel-1로, "AZ"라면 file-channel-2로, "NY"라면 양쪽 모두로 보내짐

"State" 헤더가 설정되지 않거나 이 3개와 모두 매칭되지 않는다면 'default'로 설정 된 mem-channel-1으로 보내짐.

selector는 optional channel을 지원함.

헤더의 optional channel을 지정하려면, 다음과 같이 'opional' 파라미터가 사용 됨

```
# channel selector configuration
agent_foo.sources.avro-AppSrv-source1.selector.type = multiplexing
agent_foo.sources.avro-AppSrv-source1.selector.header = State
agent_foo.sources.avro-AppSrv-source1.selector.mapping.CA = mem-channel-1
agent_foo.sources.avro-AppSrv-source1.selector.mapping.AZ = file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.mapping.NY = mem-channel-1 file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.optional.CA = mem-channel-1 file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.mapping.AZ = file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.default = mem-channel-1
```

selector는 설정된 channel들에 먼저 쓰려하고, 이 channel들 중 하나라도 이벤트 수집에 실패하면, 트랜젝션을 실패시킴.

트랜젝션은 모든 channel들에 재시도 됨.

모든 설정된 channel들이 이벤트를 수집하면, selector는 optional channel들에 쓰려고 함

optional channel들 중 이벤트 수신이 실패하면 단지 무시하고 재시도 하지 않음.


만약 특정 헤더에 optional channel과 설정된 channel이 오버랩핑된다면 channel은 설정 channel로 적용되고,

channel에서의 실패는 전체 설정 channel의 재시도를 야기시킴.

예를들어 위의 예에서, "CA" 헤더에 required와 optional 모두 마킹되어 있어도, mem-channel-1은 설정 channel로 적용됨.

그리고, 이 channel의 쓰기 실패는 이 selector에 설정된 모든 channel들의 재시도를 야기시킴.


헤더에 어떤 필요 channel도 설정되지 않았다면, 이벤트는 default channel들로 쓰여지고, 헤더의 optional channel들로 쓰려고 할 것임.

설정된 channel이 없다면, optional channel을 설정하는 것은 여전히 default channel로 쓰여지게 함.

default channel이 없고, 설정된 channel도 없다면, selector는 이벤트를 optional channel에 쓰려 할 것임.

이 경우 모든 실패는 무시 됨.


## Flume Sources

### Avro Source

Avro port로 대기하고, 외부 Avro 클라이언트로부터 이벤트를 수신함.

내장된 Avro Sink나 다른 Flume agent와 결합하면, 계층 컬렉션 토폴로지를 만들 수 있음.

### Thrift Source

Thrift port로 대기하고, 외부 Thrift 클라이언트로부터 이벤트를 수신함.

내장된 Thrift Sink나 다른 Flume agent와 결합하면, 계층 컬렉션 토폴로지를 만들 수 있음.

Thrift source는 kerberos 인증을 활성화 해서 보안 모드에서 시작하도록 설정될 수 있음.

### Exec Source

Exec source runs a given Unix command on start-up and expects that process to continuously produce data on standard out (stderr is simply discarded, unless property logStdErr is set to true). If the process exits for any reason, the source also exits and will produce no further data. This means configurations such as cat [named pipe] or tail -F [file] are going to produce the desired results where as date will probably not - the former two commands produce streams of data where as the latter produces a single event and exits.

### JMS Source

JMS Source reads messages from a JMS destination such as a queue or topic. Being a JMS application it should work with any JMS provider but has only been tested with ActiveMQ. The JMS source provides configurable batch size, message selector, user/pass, and message to flume event converter. Note that the vendor provided JMS jars should be included in the Flume classpath using plugins.d directory (preferred), –classpath on command line, or via FLUME_CLASSPATH variable in flume-env.sh.

  - Converter

  The JMS source allows pluggable converters, though it’s likely the default converter will work for most purposes. The default converter is able to convert Bytes, Text, and Object messages to FlumeEvents. In all cases, the properties in the message are added as headers to the FlumeEvent.

    - BytesMessage:
      Bytes of message are copied to body of the FlumeEvent. Cannot convert more than 2GB of data per message.
    - TextMessage:
      Text of message is converted to a byte array and copied to the body of the FlumeEvent. The default converter uses UTF-8 by default but this is configurable.
    - ObjectMessage:
      Object is written out to a ByteArrayOutputStream wrapped in an ObjectOutputStream and the resulting array is copied to the body of the FlumeEvent.

### Spooling Directory Source

This source lets you ingest data by placing files to be ingested into a “spooling” directory on disk. This source will watch the specified directory for new files, and will parse events out of new files as they appear. The event parsing logic is pluggable. After a given file has been fully read into the channel, it is renamed to indicate completion (or optionally deleted).

Unlike the Exec source, this source is reliable and will not miss data, even if Flume is restarted or killed. In exchange for this reliability, only immutable, uniquely-named files must be dropped into the spooling directory. Flume tries to detect these problem conditions and will fail loudly if they are violated:

If a file is written to after being placed into the spooling directory, Flume will print an error to its log file and stop processing.
If a file name is reused at a later time, Flume will print an error to its log file and stop processing.
To avoid the above issues, it may be useful to add a unique identifier (such as a timestamp) to log file names when they are moved into the spooling directory.

Despite the reliability guarantees of this source, there are still cases in which events may be duplicated if certain downstream failures occur. This is consistent with the guarantees offered by other Flume components.

  - Event Deserializers
    - line
      This deserializer generates one event per line of text input.

    - AVRO
      This deserializer is able to read an Avro container file, and it generates one event per Avro record in the file. Each event is annotated with a header that indicates the schema used. The body of the event is the binary Avro record data, not including the schema or the rest of the container file elements.

      Note that if the spool directory source must retry putting one of these events onto a channel (for example, because the channel is full), then it will reset and retry from the most recent Avro container file sync point. To reduce potential event duplication in such a failure scenario, write sync markers more frequently in your Avro input files.

    - BlobDeserializer
      This deserializer reads a Binary Large Object (BLOB) per event, typically one BLOB per file. For example a PDF or JPG file. Note that this approach is not suitable for very large objects because the entire BLOB is buffered in RAM.

### Twitter 1% firehose Source (experimental)

Experimental source that connects via Streaming API to the 1% sample twitter firehose, continously downloads tweets, converts them to Avro format and sends Avro events to a downstream Flume sink. Requires the consumer and access tokens and secrets of a Twitter developer account. Required properties are in bold.

### Kafka Source

Kafka Source is an Apache Kafka consumer that reads messages from a Kafka topic. If you have multiple Kafka sources running, you can configure them with the same Consumer Group so each will read a unique set of partitions for the topic.

### NetCat Source

A netcat-like source that listens on a given port and turns each line of text into an event. Acts like nc -k -l [host] [port]. In other words, it opens a specified port and listens for data. The expectation is that the supplied data is newline separated text. Each line of text is turned into a Flume event and sent via the connected channel.

### Sequence Generator Source

A simple sequence generator that continuously generates events with a counter that starts from 0 and increments by 1. Useful mainly for testing.

### Syslog Sources

Reads syslog data and generate Flume events. The UDP source treats an entire message as a single event. The TCP sources create a new event for each string of characters separated by a newline (‘n’).

  - Syslog TCP Source
    The original, tried-and-true syslog TCP source.

  - Multiport Syslog TCP Source
    This is a newer, faster, multi-port capable version of the Syslog TCP source. Note that the ports configuration setting has replaced port. Multi-port capability means that it can listen on many ports at once in an efficient manner. This source uses the Apache Mina library to do that. Provides support for RFC-3164 and many common RFC-5424 formatted messages. Also provides the capability to configure the character set used on a per-port basis.

  - Syslog UDP Source

### HTTP Source

A source which accepts Flume Events by HTTP POST and GET. GET should be used for experimentation only. HTTP requests are converted into flume events by a pluggable “handler” which must implement the HTTPSourceHandler interface. This handler takes a HttpServletRequest and returns a list of flume events. All events handled from one Http request are committed to the channel in one transaction, thus allowing for increased efficiency on channels like the file channel. If the handler throws an exception, this source will return a HTTP status of 400. If the channel is full, or the source is unable to append events to the channel, the source will return a HTTP 503 - Temporarily unavailable status.

All events sent in one post request are considered to be one batch and inserted into the channel in one transaction.

  - JSONHandler
    A handler is provided out of the box which can handle events represented in JSON format, and supports UTF-8, UTF-16 and UTF-32 character sets. The handler accepts an array of events (even if there is only one event, the event has to be sent in an array) and converts them to a Flume event based on the encoding specified in the request. If no encoding is specified, UTF-8 is assumed. The JSON handler supports UTF-8, UTF-16 and UTF-32. Events are represented as follows.

  - BlobHandler
    By default HTTPSource splits JSON input into Flume events. As an alternative, BlobHandler is a handler for HTTPSource that returns an event that contains the request parameters as well as the Binary Large Object (BLOB) uploaded with this request. For example a PDF or JPG file. Note that this approach is not suitable for very large objects because it buffers up the entire BLOB in RAM.

### Stress Source

StressSource is an internal load-generating source implementation which is very useful for stress tests. It allows User to configure the size of Event payload, with empty headers. User can configure total number of events to be sent as well maximum number of Successful Event to be delivered.

### Legacy Sources

The legacy sources allow a Flume 1.x agent to receive events from Flume 0.9.4 agents. It accepts events in the Flume 0.9.4 format, converts them to the Flume 1.0 format, and stores them in the connected channel. The 0.9.4 event properties like timestamp, pri, host, nanos, etc get converted to 1.x event header attributes. The legacy source supports both Avro and Thrift RPC connections. To use this bridge between two Flume versions, you need to start a Flume 1.x agent with the avroLegacy or thriftLegacy source. The 0.9.4 agent should have the agent Sink pointing to the host/port of the 1.x agent.

  - Avro Legacy Source
  - Thrift Legacy Source

### Custom Source

A custom source is your own implementation of the Source interface. A custom source’s class and its dependencies must be included in the agent’s classpath when starting the Flume agent. The type of the custom source is its FQCN.

### Scribe Source

Scribe is another type of ingest system. To adopt existing Scribe ingest system, Flume should use ScribeSource based on Thrift with compatible transfering protocol. For deployment of Scribe please follow the guide from Facebook. Required properties are in bold.

## Flume Sinks

### HDFS Sink
### Hive Sink
### Logger Sink
### Avro Sink
### Thrift Sink
### IRC Sink
### File Roll Sink
### Null Sink
### HBaseSinks
  - HBaseSink
  - AsyncHBaseSink

### MorphlineSolrSink
### ElasticSearchSink
### Kite Dataset Sink
### Kafka Sink
### Custom Sink

## Flume Channels

### Memory Channel
### JDBC Channel
### Kafka Channel
### File Channel
### Spillable Memory Channel
### Pseudo Transaction Channel
### Custom Channel

## Flume Channel Selectors

### Replicating Channel Selector (default)
### Multiplexing Channel Selector
### Custom Channel Selector

## Flume Sink Processors

### Default Sink Processor
### Failover Sink Processor
### Load balancing Sink Processor
### Custom Sink Processor

## Event Serializers

### Body Text Serializer
### Avro Event Serializer

## Flume Interceptors

### Timestamp Interceptor
### Host Interceptor
### Static Interceptor
### UUID Interceptor
### Morphline Interceptor
### Search and Replace Interceptor
### Regex Filtering Interceptor
### Regex Extractor Interceptor
### Example 1:
### Example 2:

## Flume Properties

### Property: flume.called.from.service

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

### HDFS
### AVRO
### Additional version requirements

## Tracing

## More Sample Configs

# Component Summary

# Alias Conventions
