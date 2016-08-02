**Logstash 정리 Logstash 개요**

Logstash는 실시간 파이프라이닝\(inptu-filter-output\) 능력을 갖춘 데이터 수집 Open source

 다양한 source 로부터의 데이터를 통합하고, 목적지\(destination\) 로 데이터를 정제하여 하여 보낼 수 있음 Elasticsearch 와 Kibana와 함께 확장가능한 데이터 처리를 통해 시너지 효과

 plugin 형태의 다양한 input, filter, output 을 조합 가능\(200가지 이상 가능\) - https:\/\/github.com\/logstash-plugins 모든 타입의 로깅 데이터 핸들링 가능

Logs\/Metric

 웹로그\(apache 등\), 애플리케이션 로그\(log4j...\)

 syslog, window event log, networking.\/firewall 로그 등

 filebeat 가 전달해주는 보안 로그

 ganglia, collectd, NetFlow, JMX, tcp\/udp 기반의 infra\/application 플랫폼 등의 metric 정보

Web

 http request 를 event 로 변환\(http를 통해 event를 수신하는 기능\)

Twitter 같은 소셜 웹서비스 제공\(예: Twit을 작성후 logstash 를 통해 바로 elasticsearch로 저장\)

webhook 으로 이용 가능

 http endpoin 를 polling 하여 event 를 생성\(반복적으로 특정 http endpoint를 호출하고 결과를 가져오는 기능\)

web application 인터페이스를 통해 health, performance, metric 등의 정보를 확인 데이터 저장\/스트림

RDB나 NoSQL로 데이터를 저장 시키술 있음

kafka, rabbitMQ, Amazos SQS, ZeroMq 같은 다양한 메시지 큐로부터의 데이터 스트림을 통합 센서\/IoT

모바일 장비, 스마트홈, 커넥티드카, 헬스케어 센서 등 다양한 산업 장비로부터의 데이터 수집 가능 패턴매칭, geo 맵핑

grok : web, system, networking 등의 이벤트 포맷의 다른 타입을 빨리 인식할 수 있도록 하는 패턴 매칭 플러그인 ip address로부터 지역정보 추출, date 일반화, key-value, cvs 데이터, 익명의 민감한 정보등을 simplifying 함 json, multiline 이벤트 같은 일반적인 구조화된 event를 처리하기 용이함

**Logstash Processing Pipline**

logstash의 파이파라인은 3단계로 구성됨 input filter output

input : event를 생성

 finter : event 수정

 output : event를 다른 곳으로 보냄

 input, output 은 데이터가 파이프라인으로 들어오고 나갈 때 별도의 filter없이도 인코딩\/디코딩 할 수 있도록 codec을 지원

**Inputs**

file syslog redis beats ...

**Filters**

**Output**

filter간의 조함도 가능 유용한 filter

grok : 비정형 텍스트를 파싱하고 구조화 하는 filter 120 가지의 패턴이 built-in

mutate : event field의 변환

 renmae, remove, replace, modify

drop : event를 버림

 clone : event의 복사본 생성

파이프라인의 마지막 단계

 event를 다중 output 으로 보낼수 있지만 모든 output 처리가 끝나야만 해당 event의 실행이 종료됨 elasticsearch

 file

 graphitte : 메트릭을 저장하고 visualization 해주는 tool

 statsd : udp상에 메시지를 수집하는 네트워크 데몬

**Codecs**

input, output 의 일부분으로 동작하는 stream filter Popular codec : json, msgpack, plain

json : json format을 인코딩\/디코딩 하는

 multiline : mutiple-line 텍스트가 합쳐진 경우\(java exception\)

**Fault Tolerance**

Logstash 는 모든 event를 처리하는 동안 메인 메모리에 저장함

 Logstash 는 input을 중단시키기 위한 SIGTERM에 응답하고, 종료하기전 처리를 끝내기 위해 기다리고 있는 event를 기다림 output 또는 filter 의 stuck으로 인해 pipline을 비울 수 없을 때 logstash는 무한 대기

예\) Pipline이 데이터베이스에 output 을 하고자 하는데 Logstash 인스턴스가 데이터베이스에 접근할 수 없을 때 , 인스턴스는 SIGTERM을 받은 후에 무한 대 기상태가 됨

 이러한 경우에 stalled pipline을 종료시키기 위해 --allow-unsafe-shutdown 옵션을 제공

강제 종료이기 때문에 데이터 유실 가능있음, 가능한 안전한 종료를 해야함

**Execution Model**

Logstash pipline은 input, filter, output 의 실행을 조정함

input threads \| pipeline worker threads

Pipline은 현재 버전에서 Logstash Proccess 내에서 filter와 output이 같은 쓰레드에서 동작함 이전에는 쓰레드가 달랐으나 성능향상와 지속성응을 위해 아키텍처를 변경

 새로운 pipline 은 스레드의 수명이 향상되고 리소스 소비가 줄어듬

 현재 Logstash pipline은 마이크로-배치임 \(실시간 방식보다 좀 더 효율적임\)

각 input { } 문은 자신의 쓰레드 위에서 돌어감

 input은 일반적인 자바 SynchronouseQueue에 이벤트를 기록함

 이 큐는 free worker에게 들어온 event를 전송하고, 모든 worker 가 바쁘면 event의 유입을 쓰롤링 함

 각 pipline 워커 쓰레드는 이 큐의 event 배치를 취하고, worker당 버퍼를 생성하고, 설정된 filter를 통해 event 배치를 새성하고, output 을 통해 필터링 된 이벤트를 실행함

 배치의 크기와 pipline worker 쓰레드의 갯수는 설정 가능

--pipline-workers

 filter\/output 처리를 위한 스레드의 수

 event 가 늘거나 cpu가 바쁘지 않은면 늘리는 것을 고려

--pipline-batch-size

 filter\/output 을 실행하기 전에 개별 woker 쓰레드가 수집할 수 있는 event의 최대 수 garbage collection 때문에 성능 문제를 야기할 수 있으므로 최적의 범위를 넘으면 안됨 output plugin 은 각각의 배치를 논리적인 단위로 처리함

ES의 경우 bulk request의 사이즈를 조정하기 위해 이 값을 조정 --pipline-batch-delay

설정할 일이 거이 없음

 logstash의 latency를 조정 \(현재 pipline worker 쓰레드가 event를 수신하고 새로운 메시지를 기다리는 시간\)

 pipline 배치 지연은 최대 수 millisecond임

 이 latency 가 지나고 나서 filter, output을 수행

 logstash 가 event를 수신하고 filter 에서 event를 처리하는 사이에 기다리는 시간은 pipline\_batch\_delay, pipeline\_batch\_size 설정

**Notes on Pipiline Configuration and Performance**

inflight event 의 총 수는 pipline\_workers와 pipline-batch-size 값에 의해 결정

 이 값을 inflight count로써 참고

 pippline\_worker와 pipline\_batch\_size 파라미터 값을 조정함으로써 inflight count 값을 유지할 수 있음 pipline은 간헐적으로 불규칙한 요청에서 수신되는 큰 event를 처리하기 위해 충분한 메모리가 요구됨

LS\_HEAP\_SIZE를 조정

 Logstash 기본 세팅은 대부분의 사용자에게 안정적이고 빠른 성능을 제공함

 성능을 올리기 위해 pipline worker나 batch size를 증가하는 것은 다음 가이드를 따라야 함

성능 감소보다 확실히 증가하는지 측정

 갑작스러운 이벤트 크기의 증가에 대응하기 위해 충분한 메모리를 남겨둠

 worker의 수는 cpu코어수보다 높게 설정 \(output이 종종 i\/o 대기중인 idle time을 사용하기 때문\) visualvm 같은 그리픽컬한 툴을 통해 쓰레드가 사용하는 리소스를 체크

**Configuring Logstash**

Logstash 를 설정하기 위해서는 사용하고자 하는 plugin과 이에 대한 설정을을 명시하는 설정 파일을 생성해야 함

**Structure of a Config File**

Logstash 설정 파일은 event 처리 파이프라인에 추가하고자 하는 plugin을 각 타입별로 섹션을 나눔

\# This is a comment. You should use comments to describe \# parts of your configuration. input { ... }

 filter { ... }

 output { ... }

각 섹션은 하나 이상의 plugin에 대한 설정 옵션을 포함

 다중 필터를 명시하면, 설정 파일안에서 작성 순서에 따라 순서대로 적용

**Plugin Configuration**

plugin 설정은 plugin 이름과 설정 블럭을 포함한다.

input { file {

path =&gt; "\/var\/log\/messages" type =&gt; "syslog" }

file {

 path =&gt; "\/var\/log\/apache\/access.log" type =&gt; "apache"

} }

**Value Types**

Plugin은 특정 type을 설정하기 위한 값이 요구될 수 있음\(boolean or hash\)

**Array**

path =&gt; \[ "\/var\/log\/messages", "\/var\/log\/\*.log" \] path =&gt; "\/data\/mysql\/mysql.log"

**Boolean**

ssl\_enable =&gt; true **Bytes**

SI \(k M G T P E Z Y\)

 Binary \(Ki Mi Gi Ti Pi Ei Zi Yi\)

my\_bytes =&gt; "1113" \# 1113

 bytes my\_bytes=&gt;"10MiB"\#10485760bytes my\_bytes=&gt;"100kib"\#102400bytes my\_bytes=&gt;"180 mb"\#180000000bytes

**Codec**

codec =&gt; "json" **Hash**

key, value 쌍의 collection

match =&gt; { "field1"=&gt;"value1"

```
      "field2"=>"value2"

```

위 설정엣는 file inpu 두 가지 설정이 되어 있음 \(path, type\) plugin type에 따라 다양한 설정을 할 수 있음

 Input Plugins, Output Plugins, Filter Plugins, Codec Plugins

데이터를 나타내는 Logstash Codec 의 이름 input, output 둘 다 쓰임

 input 으로 들어오기 전 데이터를 디코딩 함 ouput 하기 전 데이터를 인코딩 함

...}

**Number**

port =&gt; 33 **Password**

my\_password =&gt; "password" **Path**

my\_path =&gt; "\/tmp\/logstash" **String**

name=&gt;"Hello world" name=&gt;'It\'s a beautiful day'

**Comments**

\#thisis a comment

 input{ \#comments can appear at the end of a line,too

\#... }

**Event Dependent Configuration**

logstash agent는 piplince 을 inputs filters outputs 단계로 처리함

 input은 event를 생성하고, filter는 이를 수정하고, output은 다른곳으로 보냄

 모든 event들은 속성을 가짐 \(apache log의 경우는 status code, request path, HTTP verb, clientIP 등\) Logstash에서는 이런 속성들을 "filed" 라고 부름

 logstash에서 일부 설정 옵션들은 function을 위한 필수 field를 요구함

 input 은 이벤트를 생성하기 때문에 input block 내에 필드가 없음

**Field Refrences**

field를 이름으로 참조할 때 유용

 방법 : \[fieldname\]

 가장 상위의 field를 참조할 때는 \[ \] 생략 가능

 nested field를 참조가기 위해서는 full path를 명시해야 함 : \[top-level field\] \[nested field\]

 아래 예제는 5개의 top-level field\(agent, ip, request, response, ua\) 와 3개의 nested fields\(status, bytes, os\)

{

 "agent":"Mozilla\/5.0 \(compatible; MSIE 9.0\)", "ip":"192.168.24.44", "request":"\/index.html"

 "response" : {

```
           "status":200,

```

```
           "bytes":52353
      },

```

```
      "ua":{
           "os":"Windows 7"

```

} }

위 예제에서 "os" field 를 참조가 위해서는 \[ua\]\[os\]라고 명시, "request"를 참조가 위해서는 request만 쓰면됨

**sprintf format**

field 참조 포맷은 sprint format이라고 부르는 것을 사용함

 다른 문자열에서 field의 값을 참조하는 것을 가능하게 함

 아래 예제는 statsd output 은 status code에 따른 apache log 카운트를 세는 iincrement 설정

output { statsd {

```
           increment=>"apache.%{[response][status]}"
      }

```

}

output { file{

```
           path=>"/var/log/%{type}.%{+yyyy.MM.dd.HH}"
      }

```

}

**Conditional**

특정 조건에서만 filter나 output 하고자 하는 경우 프로그래밍 랭귀지의 조건문과 같음

if EXPRESSION { ...

}

 else ifEXPRESSION{

... } else{ ...

}

filter { if\[action\]=="login" {

```
           mutate {remove=>"secret"}
      }

```

}

output {

 \#Sendproduction errors to pagerduty if\[loglevel\]=="ERROR"and\[deployment\]=="production" {

```
           pagerduty {...}
      }

```

}

filter { if\[foo\]in\[foobar\]{

```
           mutate{add_tag=>"field in field"}
      }

```

if \[foo\]in"foo" { mutate{add\_tag=&gt;"field in string"}

}

 if "hello"in\[greeting\]{

```
           mutate{add_tag=>"string in field"}
      }

```

if \[foo\]in\["hello","world","foo"\]{ mutate{add\_tag=&gt;"field in list"}

}

 if \[missing\]in\[alsomissing\]{

```
           mutate{add_tag=>"shouldnotexist"}
      }

```

@timestamp field안에 timestamp를 컨버팅 할 수 있음

 괄호 안에 field 명을 명시하는 게 아니라 +FORMAT 형태로 timeformat을 사용할 수 있음

EXPRESSION 은 길고 복잡할 수 있고, 다른 EXPRESSION을 포함할 수도 있고 그룹핑도 됨 예제1\) login 인 경우 secret 필드를 삭제하는 mutate filter

예제2\) 다중 EXPRESSION

예제3\) in 조건절

if !\("foo"in\["hello","world"\]\){ mutate{add\_tag=&gt;"shouldexist"}

} }

예제4\) not in 조건절

output {

 if"\_grokparsefailure"not in\[tags\]{

```
           elasticsearch{...}
      }

```

}

**The @metadata field**

special field

 @metadata 의 내용은 output시 포함되지 않음

 조건절 또는 sprint formatting 을 통한 event field의 확장과 구축에 사용

 예제1\) STDIN input이데, 타이핑 하는것은 전부 event안에 "message" field가 됨. filter 블럭 안에 mutate event는 @metadata 안에 중첩된 일부의 새로운 field를 추가

input { stdin{}

} filter {

} output {

if \[@metadata\]\[test\]=="Hello" { stdout { codec =&gt; rubydebug }

} }

위 예제의 결과는

$ bin\/logstash-f..\/test.conf Logstashstartup completed asdf {

"message" =&gt;"asdf",

 "@version" =&gt; "1",

 "@timestamp" =&gt; "2015-03-18T23:09:29.595Z", "host" =&gt; "example.com",

 "show"=&gt;"This data will be in the output"

}

stdout { codec =&gt; rubydebug { metadata =&gt; true } } 변경 후, 결과

mutate { add\_field =&gt; {"show"=&gt;"This data will be in the output"} }

 mutate { add\_field =&gt; {"\[@metadata\]\[test\]"=&gt;"Hello"} }

 mutate { add\_field =&gt; {"\[@metadata\]\[no\_show\]"=&gt;"This data will not be in the output"} }

test field에 대한 조건절에 대해서는 정상적으로 처리 했지만, output에는 @metadata 필드들이 출력되지 않음 rubydebug codec에 @metadata field가 나나타도록 설정 하면

$ bin\/logstash -f ..\/test.conf Logstash startup completed asdf

 {

"message" =&gt; "asdf", "@version" =&gt; "1",

"@timestamp" =&gt; "2015-03-18T23:10:19.859Z", "host" =&gt; "example.com",

 "show" =&gt; "This data will be in the output",

"@metadata" =&gt; { "test" =&gt; "Hello",

"no\_show" =&gt; "This data will not be in the output" }

}

결과에 @metadata와 sub-field들이 노출됨

 \(rubydebug 코덱만이 @metadata를 노출할 수 있음\)

 @metadata는 임시적으로 field가 필요하지만 output에 포함되는 걸 원하지 않을 때 사용하면 됨

 가장 많이 사용 하는 경우 중 하나는 "date" filter 에 임시적으로 timstamp를 붙힐 때임

 예제 2\) 아래 예제는 간단하지만 apache와 nginx 웹 서버에서 일반적으로 사용하는 timestamp임

 과거에는 @timestamp field를 덮쓰기 위해 사용하는 timestamp field 를 직접 삭제해야만 했으나 @metadata field를 사용하면 그럴 필요 없음

input { stdin { } }

filter {

 grok { match =&gt; \[ "message", "%{HTTPDATE:\[@metadata\]\[timestamp\]}" \] } date { match =&gt; \[ "\[@metadata\]\[timestamp\]", "dd\/MMM\/yyyy:HH:mm:ss Z" \] }

}

output {

 stdout { codec =&gt; rubydebug }

}

grok filter에서 \[@medata\]\[timestamp\] field에 추출된 날짜를 설정함

$ bin\/logstash -f ..\/test.conf Logstash startup completed 02\/Mar\/2014:15:36:43 +0100 {

"message" =&gt; "02\/Mar\/2014:15:36:43 +0100", "@version" =&gt; "1",

"@timestamp" =&gt; "2014-03-02T14:36:43.000Z", "host" =&gt; "example.com"

}

date filter에서 timestamp 필드를 컨버전 한 후 timestamp field를 삭제할 필요가 없음

 예제 3\) CouchDB Changes input plugin 에서의 사용

 이 plugin은 자동으로 @metatdata field에 CouchDB document field를 캡쳐함

 Elasticsearch에 index하기 위해 event를 보낼 때 Elastcisearch output plugin 은 action\(delete, update, insert 등\) 과 document\_id를 명시하는 것을 가능하게 함

output { elasticsearch {

action =&gt; "%{\[@metadata\]\[action\]}" document\_id =&gt; "%{\[@metadata\]\[\_id\]}" hosts =&gt; \["example.com"\]

 index =&gt; "index\_name"

protocol =&gt; "http" }

}

**Logstash Configuration Examples**

아래 예제들은 event를 filtering 하고 apach log과 syslog 메시지를 처리하기 위해 logstash 를 어떻게 설정하고, filter 또는 output 에 의해 처리된 event들을 제어 하기 위해 조건절을 어떻게 사용하는 보기 위함

**Configuration Filters**

Filter는 데이터를 입맞에 맞게 자를수 있도록 유연성을 제공하는 in-line processing 메커니즘 아래 예제는 grok 과 date filter 설정임

input { stdin { } }

filter { grok {

match =&gt; { "message" =&gt; "%{COMBINEDAPACHELOG}" } }

date {

 match =&gt; \[ "timestamp" , "dd\/MMM\/yyyy:HH:mm:ss Z" \]

} }

output {

 elasticsearch { hosts =&gt; \["localhost:9200"\] } stdout { codec =&gt; rubydebug }

}

input message

127.0.0.1--\[11\/Dec\/2013:00:01:45-0800\]"GET \/xampp\/status.php HTTP\/1.1"2003891"http:\/\/cadenza\/xampp\/navi. php" "Mozilla\/5.0 \(Macintosh; Intel Mac OS X 10.9; rv:25.0\) Gecko\/20100101 Firefox\/25.0"

ouput message

{

"message" =&gt; "127.0.0.1 - - \[11\/Dec\/2013:00:01:45 -0800\] \"GET \/xampp\/status.php HTTP\/1.1\" 200 3891 \"http:\/\/cadenza\/xampp\/navi.php\" \"Mozi 5.0 \(Macintosh; Intel Mac OS X 10.9; rv:25.0\) Gecko\/20100101 Firefox\/25.0\"",

"@timestamp" =&gt; "2013-12-11T08:01:45.000Z", "@version" =&gt; "1",

"host" =&gt; "cadenza", "clientip" =&gt; "127.0.0.1",

"ident" =&gt; "-", "auth" =&gt; "-",

"timestamp" =&gt; "11\/Dec\/2013:00:01:45 -0800", "verb" =&gt; "GET",

"request" =&gt; "\/xampp\/status.php", "httpversion" =&gt; "1.1",

"response" =&gt; "200", "bytes" =&gt; "3891",

"referrer" =&gt; "\"http:\/\/cadenza\/xampp\/navi.php\"",

 "agent" =&gt; "\"Mozilla\/5.0 \(Macintosh; Intel Mac OS X 10.9; rv:25.0\) Gecko\/20100101 Firefox\/25.0\""

}

lla\/

logstash가 grok filter를 이용하여 log line을 Apache "combined log" 포맷에 맞춰 parsing 했음

 그리고 정보들을 여러 단위로 쪼갬

 log data 를 질의하고 분석하는데 매우 유용함

 logstash에는 상당수의 gork 패턴이 있음\(https:\/\/github.com\/logstash-plugins\/logstash-patterns-core\/blob\/master\/patterns\/grok-patterns\) 위 예제에서 date filter 도 사용됨

date filter는 timestamp field를 파싱하여 event timestamp로 사용함\(logstash가 수집한 시간과 상관없이\) @timestamp field 값이 2013-11-11로 변경되었음

 유실된 로그를 메울 때 유용

**Processing Apache Logs**

localhsot의 파일을 input으로 읽고 event를 처리하기 위해 조건절을 사용한 예제

input { file {

path =&gt; "\/tmp\/access\_log"

start\_position =&gt; "beginning" }

}

filter {

 if \[path\] =~ "access" {

mutate { replace =&gt; { "type" =&gt; "apache\_access" } } grok {

match =&gt; { "message" =&gt; "%{COMBINEDAPACHELOG}" } }

} date {

match =&gt; \[ "timestamp" , "dd\/MMM\/yyyy:HH:mm:ss Z" \] }

}

output { elasticsearch {

hosts =&gt; \["localhost:9200"\] }

stdout { codec =&gt; rubydebug } }

\/tmp\/access\_log

실행

bin\/logstash-f logstash-apache.conf

수행하고 나면 elasticsearch에 데이터가 들어감

 logstash가 특정 input 파일을 열어서 잃고 각 event들을 처리함

 추가로 이 파일에 로그를 남기면 캡쳐되고 event를 처리하고 elasticsearch에 저장함 추가적으로 type field는 apache\_asccess 로 저장됨

 input 설정에 \*를 써서 매칭 조건을 줄 수 있음

위 설정대로 변경 후, logstash 를 재시작 하면 access\_log는 field별로 쪼개져 있지만 error\_log는 그렇지 않음 grok filter 가 표준 apach log 포맷으로 파싱하여 자동으로 쪼개기 때문

logstash는 이미 읽은 파일은 다시 읽지 않음. 최종 위치를 저장하고 있다가 새로운 라인만 읽어들임

**Using Conditionals**

filter 또는 output에서 처리되는 event를 제어하기 위해 조건절을 사용 어떤 파일이냐에 따라 각 event를 분류할 수 있음

NE nd

71.141.244.242 - kurt \[18\/May\/2011:01:48:10 -0700\] "GET \/admin HTTP\/1.1" 301 566 "-" "Mozilla\/5.0 \(Windows; U; Windows NT 5.1; en-

 US; rv:1.9.2.3\) Gecko\/20100401 Firefox\/3.6.3"

 134.39.72.245 - - \[18\/May\/2011:12:40:18 -0700\] "GET \/favicon.ico HTTP\/1.1" 200 1189 "-

 " "Mozilla\/4.0 \(compatible; MSIE 8.0; Windows NT 5.1; Trident\/4.0; .NET CLR 2.0.50727; .NET CLR 3.0.4506.2152; .NET CLR 3.5.30729; InfoPath.2; . T4.0C; .NET4.0E\)"

98.83.179.51 - - \[18\/May\/2011:19:35:08 -0700\] "GET \/css\/main.css HTTP\/1.1" 200 1837 "http:\/\/www.safesand.com\/information.htm" "Mozilla\/5.0 \(Wi ows NT 6.0; WOW64; rv:2.0.1\) Gecko\/20100101 Firefox\/4.0.1"

input { file {

path =&gt; "\/tmp\/\*\_log" ...

input { file {

path =&gt; "\/tmp\/\*\_log" }

}

filter {

 if \[path\] =~ "access" {

mutate { replace =&gt; { type =&gt; "apache\_access" } } grok {

match =&gt; { "message" =&gt; "%{COMBINEDAPACHELOG}" } }

date {

 match =&gt; \[ "timestamp" , "dd\/MMM\/yyyy:HH:mm:ss Z" \]

}

 } else if \[path\] =~ "error" {

mutate { replace =&gt; { type =&gt; "apache\_error" } } } else {

mutate { replace =&gt; { type =&gt; "random\_logs" } } }

}

output {

 elasticsearch { hosts =&gt; \["localhost:9200"\] } stdout { codec =&gt; rubydebug }

}

위 예제에서 type field로 event를 분류함 \(실제 error와 random 파일은 파싱하지 않음\) 마찬가지로, 특정 output에 대해 이벤트를 정하기 위해 조건절을 사용할 수 있음

아파치 event에서 status code가 5xx 인 경우 nagios에 alert 4xx 인경우 elasticsearch 저장

 모든 status code는 statsd에 저장

5xx stasu code 는 nagios 에 alert하기 위해서는 먼저 type field를 확인

 만약 apache\_access 이면 5xx error 코드를 포함하는지 체크하고 nagio에 전달 4xx error 코드이면 elasticsearch에 전달

 최종적으로 error 코드와 상관없이 모두 statsd 에 전달

output {

 if \[type\] == "apache" {

if \[status\] =~ \/^5\d\d\/ { nagios { ... }

} else if \[status\] =~ \/^4\d\d\/ { elasticsearch { ... }

}

statsd { increment =&gt; "apache.%{status}" } }

}

**Processing Syslog Messages**

가장 흔한 use case 중 하나

 Syslog는 클라이언트 머신에서 로컬파일에 또는 중앙 로그 서버로\(rsyslog\) 남기는 UNIX 네트워크 로그 표준임

input { tcp {

port =&gt; 5000

type =&gt; syslog }

udp {

 port =&gt; 5000 type =&gt; syslog

} }

filter {

 if \[type\] == "syslog" {

grok {

 match =&gt; { "message" =&gt; "%{SYSLOGTIMESTAMP:syslog\_timestamp} %{SYSLOGHOST:syslog\_hostname} %{DATA:syslog\_program}\(?:\\[%

{POSINT:syslog\_pid}\\]\)?: %{GREEDYDATA:syslog\_message}" } add\_field =&gt; \[ "received\_at", "%{@timestamp}" \]

 add\_field =&gt; \[ "received\_from", "%{host}" \]

} date {

match =&gt; \[ "syslog\_timestamp", "MMM d HH:mm:ss", "MMM dd HH:mm:ss" \]

} }

}

output {

 elasticsearch { hosts =&gt; \["localhost:9200"\] } stdout { codec =&gt; rubydebug }

}

일반적으로 클라이언트 머신은 Logstash의 5000 번 포트에 접속하고 메시지를 보냄 이 예제에서는 텔넷으로 접속하여 직접 작성하여 로그를 보냄

 샘플 데이터 \(Syslog\)

Logstash output 결과

Dec 23 12:11:43 louis postfix\/smtpd\[31499\]: connect from unknown\[95.75.93.154\]

 Dec 23 14:42:56 louis named\[16000\]: client 199.48.164.7\#64817: query \(cache\) 'amsterdamboothuren.com\/MX\/IN' denied

 Dec 23 14:30:01 louis CRON\[619\]: \(www-data\) CMD \(php \/usr\/share\/cacti\/site\/poller.php &gt;\/dev\/null 2&gt;\/var\/log\/cacti\/poller-error.log\) Dec 22 18:28:06 louis rsyslogd: \[origin software="rsyslogd" swVersion="4.2.0" x-pid="2253" x-

 info="http:\/\/www.rsyslog.com"\] rsyslogd was HUPed, type 'lightweight'.

{

```
                 "message" => "Dec 23 14:30:01 louis CRON[619]: (www-
data) CMD (php /usr/share/cacti/site/poller.php >/dev/null 2>/var/log/cacti/poller-error.log)",

```

```
              "@timestamp" => "2013-12-23T22:30:01.000Z",
                "@version" => "1",

```

```
                    "type" => "syslog",

```

```
                    "host" => "0:0:0:0:0:0:0:1:52617",
        "syslog_timestamp" => "Dec 23 14:30:01",

```

```
         "syslog_hostname" => "louis",
          "syslog_program" => "CRON",

```

```
              "syslog_pid" => "619",
          "syslog_message" => "(www-

```

```
data) CMD (php /usr/share/cacti/site/poller.php >/dev/null 2>/var/log/cacti/poller-error.log)",
             "received_at" => "2013-12-23 22:49:22 UTC",

```

```
           "received_from" => "0:0:0:0:0:0:0:1:52617",
    "syslog_severity_code" => 5,

```

```
    "syslog_facility_code" => 1,
         "syslog_facility" => "user-level",
         "syslog_severity" => "notice"

```

}

**Managing Multiline Events**

multiple line 텍스트의 event를 생성하는 use case가 있음

 multiline event를 정확히 제어하기 위해서는 logtsash가 어느 줄이 단일 event를 위한 것인지 알 필요가 있음 multiline event 처리는 북잡하며 적절한 event 정렬에 의존함

 정렬된 log 처리를 보장하는 가장 좋은 방법은 가능한한 pipline에서 일찍 처리하도록 구현하는 것임 Logstash pipline에서 바람직한 도구는 multiline codec임

간단한 rule set으로 단일 input의 라인들을 병합 multiline plugin 을 설정하는데 중요한 측면

"pattern" 옵션은 regular expression 을 명시함

 특정 regular expression에 매칭되는 라인들은 이전 라인의 연속이거나 새로운 multiline event의 시작으로 고려됨 위 설정과 함께 "grok"의 regular expression template을 사용할 수 있음

"what" 옵션은 두 가지 값을 가짐 previous, next

previous는 "pattern" 옵션에 매칭되는 라인은 이전 라인들의 부분으로 간주함

 next는 "pattern" 옵션에 매칭되는 라인은 다음에 오는 라인들의 부분으로 간주함

 "negate" 옵션은 multiline codec이 "pattern" 옵션에 명시된 regular expression과 매칭되지 않는 라인들에 적용

**Example of Multiline Plugin Configuration**

단일 event 에서의 Java stack trace

 단일 event에서 C 스타일의 연속적인 라인 time-stamped event 로부터의 multiplie line

**Java Stack Traces**

Java stack trace는 multiplie 라인으로 이루어짐 최초 라인 이후 라인들은 각각 공백으로 시작함

Logstach에서 단일 event안에 이러한 라인들을 하나로 묶기 위해 multiline codec을 사용

위 설정은 공백으로 시작하는 라인은 이전 라인과 병합함

**Line Continuations**

일부 프로그래밍 언어에서 라인 끝에 "\" 문자는 아래 문장에서 계속 이어짐을 나타냄

printf\("%10.10ld\t%10.10ld\t%s\%f", w, x, y, z \); multiline code을 사용하여 하나로 묶기

위 설정은 "\" 문자 아래 오는 라인을 하나로 병합함

**Timestamps**

Elasticsearch 같은 서비스들의 activity log는 timestamp로 시작함

Exception in thread "main" java.lang.NullPointerException

 at com.example.myproject.Book.getTitle\(Book.java:16\)

 at com.example.myproject.Author.getBookTitles\(Author.java:25\)

```
        at com.example.myproject.Bootstrap.main(Bootstrap.java:14)

```

input { stdin {

codec =&gt; multiline { pattern =&gt; "^\s" what =&gt; "previous"

} }

}

input { stdin {

codec =&gt; multiline { pattern =&gt; "\\$" what =&gt; "next"

} }

}

\[2015-08-24 11:49:14,389\]\[INFO \]\[env \] \[Letha\] using \[1\] data paths, mounts \[\[\/ \(\/dev\/disk1\)\]\], net usable\_space \[34.5gb\], net total\_space \[118.9gb\], types \[hfs\]

input { file {

path =&gt; "\/var\/log\/someapp.log" codec =&gt; multiline {

pattern =&gt; "^%{TIMESTAMP\_ISO8601} " negate =&gt; true

 what =&gt; previous

} }

}

위 설정에서 "negtate" 옵션은 timestamp로 시작하지 않는 라인은 이전 라인에 속하는 것으로 간주함

**Deploy and Scaling Logstash**

주어진 규모에 따라 바람직한 아키텍처는 변함

 복잡성의 순서에 따라 Logstash 아키텍처의 범위를 논할 것 최소 설치에서 시작해서 시스템에 요소들을 붙혀 나갈것

**The Minimal Installation**

최소 Logstash 설치는 하나의 logstash 인스턴스와 하나의 elasticsearch 인스턴스를 구성

 서로 직접 연결됨

 Logstash는 input plugin으로 데이터를 수집하고 Elasticsearch output plugin 통해 데이터를 Elasticsearch에 인덱싱함 Logstash는 구동시 설정 파일을 기반으로 구성된 고정된 pipline을 가지고 있음

 input plugin 은 반드시 명시 하여야 하며, output은 기본값이 stdout이고, filter는 optional임

**Using Filters**

로그데이터는 일반적으로 비정형이며, 관련없는 이절적인 정보를 포함하기도 하고, 로그내용에서 파생된 관련 정보를 놓치기도 함 field안에 로그를 파싱하고, 불필요한 정보를 삭제하고, 기존 field 에서 추가적인 정보를 이끌어 내기 위해 filter plugin을 사용

예\) ip주소에서 지역 정보를 얻어 로그에 추가하고, "grok" filter를 통해 텍스트를 파싱하고 구조화 함 filter plugin을 추가하는 것은 성능에 상당한 영향을 줌

filter plugin이 수행하는 계산의 양에 따라, 처리되는 로그의 양에 따라

 "grok" 필터의 regular expression 계산은 특히 resource-intensive\(자원활용에 집중?\)

 계산 자원의 요구 증가를 처리하기 위한 한가지 방법은 멀티 코어 장비에서 병렬 처리 하는 것임

-w 옵션을 통해 Logstash 필터링 작업을 위한 쓰레드 갯수를 설정할 수 있음 ex\) bin\/logstash -w 8

**Using Filebeat**

Filebeat 는 서버의 파일로부터 로그를 수집하고 이를 처리하기 위한 다른 머신에게 로그를 전달하기 위한 가볍고 자원친화적인 도구\(Go 언어로 개발\) Filebeat 는 Logstash 인스턴스와 통신하기 위해 "Beats" 프로토콜 사용

 Logstash 는 "Beats input plugin" 으로 Beats 데이터를 수신할 수 있음

 FileBeat 는 원천 데이터 머신의 계산 자원을 사용하기 때문에, Logstash 인스턴스 상에서 "Beats input plugin"은 최소한의 자원만 요구함

이러한 아키텍처는 자원 제약이 많은 곳에서 유용

**Scaling to a Larger Elasticsearch Cluster**

일반적으로 Logstash는 단일 ES node 보다는 Cluster와 통신함

 기본적으로 Logstash는 cluster에 데이터를 보내기 위해 HTTP 프로토콜 이용 Elasticsearch HTTP REST API 를 통해 ES cluster 에 데이터를 인덱싱

이 API들은 JSON 으로 index된 데이터를 나타냄

Shield plugin 을 통해 SSL 지원 가능

 HTTP 프로토콜을 사용하면, Cluster내의 지정된 호스트 집합에 indexing 요청을 자동으로 로드밸런싱 하도록 output plugin은 설정할 수 있음 다중 ES node들을 지정하는 것은 ative 한 ES node 에 트래픽을 라우팅 함으로써 ES cluster에 대해 고가용성을 제공함

 ES Java API를 통해 Transport Protocol을 사용하여 바이너리 형태로 데이터를 직렬화 할 수 있음

Transport Protocol 은 request의 endpoint를 알아챌 수 있고, ES cluster 내에 임이의 클라이언트나 데이터노드를 선택할 수 있음 HTTP 또는 Transport 프로토콜을 사용하여 Logstash 인스턴스를 ES Cluster로부터 분리함

 반대로 node 프로토콜은 Logstash 인스턴스가 실행중인 머신을 ES 인스터스가 실행중인 ES Cluster 와 결합함\(?\)

이 노드에서 cluster의 여분 노드로 인덱싱을 복제하고자 하는 데이터\(?\)

 머신은 cluster의 일부이기 때문에, cluster topology는 작은 수의 계속적인 접속이 이루어지는 경우에 노드 프로토콜이 잘 맞음\(?\) logstash와 외부 애플리케이션 사이의 커넥션을 핸들링 하기 위해 3rd-party 하드웨어나 소프트웨어 로드 밸런서를 사용해도 됨

logstash 는 ES 의 master node\(cluster 관리를 수행해야 하므로\) 에 적접 접속하면 안됨. ES client나 data node에 접속 \(ES 의 안정석을 위해\)

**Managing Throughput Spikes with Messae Queueing**

데이터를 소화할수 있는 ES cluster의 능력을 초과하는 데이터가 Logstash pipline으로 오는 경우, 버퍼로 메시지 큐를 사용할 수 있음 기본적으로, logstash는 데이터 인덱싱 처리율이 데이터 유입률보다 떨어지면 들어오는 event를 쓰롤링함

이러한 쓰롤링은 데이터 원천에서 event 가 버퍼에 쌓이는 것을 야기할 수 있기 때문에 메시징 큐를 이용하여 backpressure\(역압\)을 방지하는 것은 시스템을

관리하는데 중요한 부분임

 logstash 에 메시지 큐를 추가하는 것은 데이터 유실을 방지하기 위한 단계임

 메시지 큐로부터 데이터를 수집하는 logastash 인스턴스가 실패할 때, 그 데이터는 메시지 큐에서 active logstash 인스턴스로 다시 보내짐

3rd-party 메시지 큐들이 있음 Redis, Kafka, RabbitMQ 등

Logstash는 일부 3rd-party 메시지 큐들에 대해 input, output plugin을 제공함 Logstash가 메시지 큐를 가지도록 설정되었을 때, Logstash는 기능적으로 2단계가 존재함

Shipping 인스턴스 : 데이터를 수집하고 메시지 큐에 저장함

 Indexing 인스턴스 : 데이터를 메시지 큐로부터 탐색하고, 설정된 filter를 적용하고 ES index에 필터링 된 데이터를 쓰는 역할

**Multiple Connections for Logstash High Availabity**

Logstash를 개별 인스트스의 실패로부터 좀 더 탄력적으로 하기 위해 원천 데이터 머신과 Logstash clutser 사이에 로드 밸런서를 둘 수 있음

 로드 밸런서는 개별 인스턴스가 이용가능하지 않더라도 데이터 수집과 처리의 연속성을 보장하기 위해 Logstash 인스턴스로의 개별적인 커넥션을 제어함

위 아케틱처는 특정 type 을 담당하고 있는 logstash가 이용불가능 경우 해당 타입의 input 처리가 불가함 좀 더 견고한 input 처리를 위해 각 logstash 인스턴스는 multiple input으로 설정해야 함

위 아키텍처는 Logstash 의 업무량을 input을 기준으로 병렬화 하였음

 입력이 늘어나면 Logstash 인스턴스를 horizontally 하게 확장하면 됨

 Pipline을 병렬로 분리하여 Single 포인트에서의 장애를 제거함으로 써 신뢰성이 증가함

**Scalling Logstash**

성숭한 Logstash 배치는 대게 다음 pipline을 따름

 input 계층은 원천으로부터 데이터를 수집하고 적절한 input plugin 으로 Logstash 인스턴스를 구성한다. 메시지 큐는 failover 보호를 제공하기 위해 수집된 데이터의 보관목적으로 버퍼를 제공한다.

 filter 계층은 메시지 큐로부터 수집한 데이터의 파싱과 처리를 수행한다

 inexing 계칭은 처리된 데이터를 ES로 이동시킨다

위 레이어들은 어디라도 컴퓨팅 자원을 추가함으로써 확장시킬 수 있음

 컴포넌트들의 성능 상태를 주기적으로 살피고 필요한만큼 자원을 추가

 Logstash에서 incoming evetn에 대해 반복적으로 쓰롤링이 발생하면 메시지 큐에 스토리지를 늘리는 것을 고려 Logstash indexing 인스턴스를 추가함으로써 ES 클러스트의 데이터 처리율을 증가시키는 것도 대안

