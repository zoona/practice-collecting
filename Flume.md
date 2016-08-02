# 소개

다양한 소스에서 발생한 대량의 로그 데이터를 중앙 데이터 스토어로 효과적으로 수집 집계(aggregating)하거나 이동시킬 수 있는 신뢰할수있는 분산 시스템

스트림 지향의 데이터 플로우를 기반으로 하며 지정된 모든 서버로 부터 로그를 수집한 후 하둡 HDFS와 같은 중앙 저장소에 적재하여 분석하는 시스템을 구축해야 할 때 적합

데이터 소스를 커스터마이징 할 수 있기 때문에 로그 데이터 수집에 제한되지 않고, 소셜미디어 데이터, 이메일 메세지등 다량의 이벤트 데이터를 전송하는데에 사용할 수 있다

System Requirements

- Java Runtime Environment - Java 1.6 or later (Java 1.7 Recommended)

- Memory - Sufficient memory for configurations used by sources, channels or sinks

- Disk Space - Sufficient disk space for configurations used by channels or sinks

- Directory Permissions - Read/Write permissions for directories used by agent

## Architecture

- Data flow model

![](https://flume.apache.org/_images/UserGuide_image00.png)

A Flume event is defined as a unit of data flow having a byte payload and an optional set of string attributes.

A Flume agent is a (JVM) process that hosts the components through which events flow from an external source to the next destination (hop).

A Flume source consumes events delivered to it by an external source like a web server.

The external source sends events to Flume in a format that is recognized by the target Flume source. For example, an Avro Flume source can be used to receive Avro events from Avro clients or other Flume agents in the flow that send events from an Avro sink.

A similar flow can be defined using a Thrift Flume Source to receive events from a Thrift Sink or a Flume Thrift Rpc Client or Thrift clients written in any language generated from the Flume thrift protocol.When a Flume source receives an event, it stores it into one or more channels.

The channel is a passive store that keeps the event until it’s consumed by a Flume sink. The file channel is one example – it is backed by the local filesystem.

The sink removes the event from the channel and puts it into an external repository like HDFS (via Flume HDFS sink) or forwards it to the Flume source of the next Flume agent (next hop) in the flow.

The source and sink within the given agent run asynchronously with the events staged in the channel.


- Complex flows

Flume allows a user to build multi-hop flows where events travel through multiple agents before reaching the final destination.

It also allows fan-in and fan-out flows, contextual routing and backup routes (fail-over) for failed hops.


- Reliability

The events are staged in a channel on each agent. The events are then delivered to the next agent or terminal repository (like HDFS) in the flow. The events are removed from a channel only after they are stored in the channel of next agent or in the terminal repository. This is a how the single-hop message delivery semantics in Flume provide end-to-end reliability of the flow.

Flume uses a transactional approach to guarantee the reliable delivery of the events. The sources and sinks encapsulate in a transaction the storage/retrieval, respectively, of the events placed in or provided by a transaction provided by the channel. This ensures that the set of events are reliably passed from point to point in the flow. In the case of a multi-hop flow, the sink from the previous hop and the source from the next hop both have their transactions running to ensure that the data is safely stored in the channel of the next hop.


- Recoverability

The events are staged in the channel, which manages recovery from failure. Flume supports a durable file channel which is backed by the local file system. There’s also a memory channel which simply stores the events in an in-memory queue, which is faster but any events still left in the memory channel when an agent process dies can’t be recovered.




# Setup

## Setting up an agent

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