# Logstash Scripts

## Install Logstash
```
wget https://download.elastic.co/logstash/logstash/logstash-2.3.4.tar.gz
tar xvfz logstash-2.3.4.tar.gz
cd logstash-2.3.4
```

```
bin/logstash -e 'input { stdin { } } output { stdout {} }'
hello
```

## Install Elasticsearch

```
curl https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/tar/elasticsearch/2.3.5/elasticsearch-2.3.5.tar.gz
tar xvfz elasticsearch-2.3.5.tar.gz
cd elasticsearch-2.3.5
```

```
./bin/elasticsearch -Des.insecure.allow.root=true
```

## Install Kibana
```
curl https://download.elastic.co/kibana/kibana/kibana-4.5.4-linux-x64.tar.gz
```

## Skeleton

```properties
# The # character at the beginning of a line indicates a comment. Use
# comments to describe your configuration.
input {
}
# The filter part of this file is commented out to indicate that it is
# optional.
# filter {
#
# }
output {
}
```

## Sample Data

```shell
mkdir data && cd data
wget https://download.elastic.co/demos/logstash/gettingstarted/logstash-tutorial.log.gz
gunzip logstash-tutorial.log.gz
tail logstash-tutorial.log
```

