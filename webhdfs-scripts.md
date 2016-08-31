# WebHDFS SCripts

## Tool

### Advanced REST client

https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo

### CURL

comand line tool for transferring files with URL syntax

## Document

Apache Hadoop 2.7.2 WebHDFS REST API
http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/WebHDFS.html

Operations
http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#Operations

## LISTSTATUS

curl -i "http://localhost:50070/webhdfs/v1/?op=LISTSTATUS"
curl -i "http://localhost:50070/webhdfs/v1/tmp?op=LISTSTATUS"


## MKDIR

curl -i -X PUT "http://localhost:50070/webhdfs/v1/tmp/webhdfs?op=MKDIRS"

## GETFILESTATUS

curl -i "http://localhost:50070/webhdfs/v1/tmp/webhdfs?op=GETFILESTATUS"

## CREATE

curl -i -X PUT "http://localhost:50070/webhdfs/v1/tmp/webhdfs/README?op=CREATE"

curl -i -X PUT -T LOCALFILE "http://localhost:50075/webhdfs/v1/tmp/webhdfs/README?op=CREATE&namenoderpcaddress=sandbox.hortonworks.com:8020&createflag=&createparent=true&overwrite=false"

## OPEN

curl -i "http://localhost:50070/webhdfs/v1/tmp/webhdfs/README?op=OPEN"

curl -i "http://localhost:50075/webhdfs/v1/tmp/webhdfs/README?op=OPEN&namenoderpcaddress=sandbox.hortonworks.com:8020&offset=0"