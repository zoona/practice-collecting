# 개요

다양한 소스에서 발생한 대량의 로그 데이터를 중앙 데이터 스토어로 효과적으로 수집 집계\(aggregating\)하거나 이동시킬 수 있는 신뢰할수있는 분산 시스템

스트림 지향의 데이터 플로우를 기반으로 하며 지정된 모든 서버로 부터 로그를 수집한 후 하둡 HDFS와 같은 중앙 저장소에 적재하여 분석하는 시스템을 구축해야 할 때 적합

데이터 소스를 커스터마이징 할 수 있기 때문에 로그 데이터 수집에 제한되지 않고, 소셜미디어 데이터, 이메일 메세지등 다량의 이벤트 데이터를 전송하는데에 사용할 수 있다

```python
import com.zoona.practice

print "Hello"
```

# Goals

* Reliability - 장애가 발생시 로그의 유실없이 전송할 수 있는 기능
* Scalability - Agent의 추가 및 제거가 용이
* Manageability - 간결한 구조로 관리가 용이
* Extensibility - 새로운 기능을 쉽게 추가할 수 있음

# Core Concepts

* Event : Flume이 source로부터 destination까지 전송하는 데이터 단위 \(byte payload\)
* Flow : Event의 흐름
* Client : Agent로 데이터를 전달하는 interface의 구현
* Agent: Event를 수집, 저장, 전달하는 독립된 프로세스
* Source : Event를 수집해서 Channel에 저장
* Sink : Channel로 부터 가져와서 목적지, 혹은 다른 Agent로 전달
* Channel : Source에서 Sink로 보내는 Event의 임시 저장소

![](https://camo.githubusercontent.com/fc7c341952a08eeeaecac978559a79eda840b1a7/68747470733a2f2f666c756d652e6170616368652e6f72672f5f696d616765732f5573657247756964655f696d61676530302e706e67)

# Architecture

## Data flow Model

## Complex Flows

## Reliability

## Recoverability