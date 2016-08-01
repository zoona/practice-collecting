Flume 1.6.0 User Guide# Introduction
## Overview

다양한 소스에서 발생한 대량의 로그 데이터를 중앙 데이터 스토어로 효과적으로 수집 집계(aggregating)하거나 이동시킬 수 있는 신뢰할수있는 분산 시스템
스트림 지향의 데이터 플로우를 기반으로 하며 지정된 모든 서버로 부터 로그를 수집한 후 하둡 HDFS와 같은 중앙 저장소에 적재하여 분석하는 시스템을 구축해야 할 때 적합
데이터 소스를 커스터마이징 할 수 있기 때문에 로그 데이터 수집에 제한되지 않고, 소셜미디어 데이터, 이메일 메세지등 다량의 이벤트 데이터를 전송하는데에 사용할 수 있다

## System Requirements
## Architecture
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