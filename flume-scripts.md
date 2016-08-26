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

../bin/flume-ng agent -c . -f ./seq.conf -name a1 -Dflume.root.logger=INFO,console

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

```sh

../bin/flume-ng agent -c . -f ./exec.conf -name a1 -Dflume.root.logger=INFO,console

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

