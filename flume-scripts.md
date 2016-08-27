# Flume Scripts

Basic Flume agent configuration

```properties

# list the sources, sinks and channels for the agent
<Agent>.sources = <Source>
<Agent>.sinks = <Sink>
<Agent>.channels = <Channel1> <Channel2>

# set channel for source
<Agent>.sources.<Source>.channels = <Channel1> <Channel2> ...

# set channel for sink
<Agent>.sinks.<Sink>.channel = <Channel1>

# properties for sources
<Agent>.sources.<Source>.<someProperty> = <someValue>

# properties for channels
<Agent>.channel.<Channel>.<someProperty> = <someValue>

# properties for sinks
<Agent>.sources.<Sink>.<someProperty> = <someValue>

```

## Sources

### Sequence Generator Source

``` properties

a1.sources = r1
a1.channels = c1
a1.sinks = s1

a1.sources.r1.type = seq
a1.sources.r1.channels = c1

a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.sinks.s1.type = logger
a1.sinks.s1.channel = c1

```

```shell

./bin/flume-ng agent -c ./conf -f ./conf/seq.conf -name a1 -Dflume.root.logger=INFO,console

```

### Exec Source

``` properties

a1.sources = r1
a1.channels = c1
a1.sinks = s1

a1.sources.r1.type = exec
a1.sources.r1.command = tail -f /root/var/data.log
a1.sources.r1.channels = c1

a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.sinks.s1.type = logger
a1.sinks.s1.channel = c1

```

```shell

./bin/flume-ng agent -c ./conf -f ./conf/exec.conf -name a1 -Dflume.root.logger=INFO,console

```

```shell

echo hello >> /root/var/data/log
echo flume >> /root/var/data/log

```

### Syslog TCP Source

```properties

a1.sources = r1
a1.channels = c1
a1.sinks = s1

a1.sources.r1.type = syslogtcp
a1.sources.r1.port = 5140
a1.sources.r1.host = localhost
a1.sources.r1.channels = c1

a1.channels.c1.type = memory 
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.sinks.s1.type = logger
a1.sinks.s1.channel = c1

```

```shell

./bin/flume-ng agent -c ./conf -f ./conf/syslogtcp.conf -name a1 -Dflume.root.logger=INFO,console

```

```
vi /etc/rsyslog.conf
```

```properties
*.* @@localhost:5140
```

```shell
logger hello flume
```

### Spooling Directory Source

Unlike the Exec source, this source is reliable and will not miss data, even if Flume is restarted or killed. In exchange for this reliability, only immutable, uniquely-named files must be dropped into the spooling directory. Flume tries to detect these problem conditions and will fail loudly if they are violated:


```properties

a1.sources = r1
a1.channels = c1
a1.sinks = s1


a1.sources.r1.type = spooldir
a1.sources.r1.spoolDir = /root/var/flumespool
a1.sources.r1.fileHeader = true
a1.sources.r1.channels = c1

a1.channels.c1.type = memory 
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

a1.sinks.s1.type = logger
a1.sinks.s1.channel = c1

```

```shell

echo hello >> /root/var/flumespool/data1.txt

```