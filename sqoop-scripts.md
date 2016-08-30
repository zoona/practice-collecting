# Sqoop Scripts

## Sqoop 설치

```
wget http://apache.tt.co.kr/sqoop/1.4.6/sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz
tar xvfz sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz
cd sqoop-1.4.6.bin__hadoop-2.0.4-alpha
```

## 테스트 환경

### dummy data 생성

```
http://generatedata.com
```

### MySQL 셋팅

mysql 접속
```
mysql -uroot
```
DB 생성
```
mysql> create database collecting
```

Table 생성 / 데이터 Insert
```
mysql -uroot collecting < user.sql
```

확인
```
mysql> mysql -uroot collecting
mysql> show tables;
mysql> select * from user;
```

## Import

### 특정 테이블 데이터 import 
```shell
./bin/sqoop import \
--connect jdbc:mysql://localhost/collecting \
--username root \
--table user \
--driver com.mysql.jdbc.Driver
--target-dir /sqoop/input/user
```
확인
```shell
hadoop fs -ls /
hadoop fs -ls /sqoop/input/user
```

### DB내 전체 테이블 Import 
```   shell
./bin/sqoop import-all-tables \
--connect jdbc:mysql://localhost/collecting \
--driver com.mysql.jdbc.Driver \
--username root \
--warehouse-dir /sqoop/input/
```

```   shell
./bin/sqoop import-all-tables \
--connect jdbc:mysql://localhost/collecting \
--driver com.mysql.jdbc.Driver \
--username root \
--exclude-tables user \
--warehouse-dir /sqoop/input/
```

### Mapper 개수 지정 

기존 파일 삭제
```
hadoop fs -rm -r /sqoop
```

```
./bin/sqoop import \
--connect jdbc:mysql://localhost/collecting \
--driver com.mysql.jdbc.Driver \
--username root \
--table user \
--target-dir /sqoop/input/user \
--num-mappers 10
```

### 일부 데이터, 조건에 맞는 데이터 가져오기 

--incremental 옵션 사용

```
./bin/sqoop import \
--connect jdbc:mysql://localhost/collecting \
--driver com.mysql.jdbc.Driver \
--username root \
--table user \
--target-dir /sqoop/input/user \
--num-mappers 10 \
--incremental append \
--check-column id \
--last-value 50
```

```
hadoop fs -getmerge /sqoop/input/user result.txt
```

특정 Query 조건에 맞는 데이터 가져오기
```
./bin/sqoop import \
--connect jdbc:mysql://localhost/collecting \
--driver com.mysql.jdbc.Driver \
--username root \
--target-dir /sqoop/input/user \
--query 'SELECT id, name, email, city FROM user WHERE $CONDITIONS and city like "S%"' \
--split-by city \
--num-mappers 10 
```

### 데이터 전송속도 높이기 --direct 옵션 
```
./bin/sqoop import \
--connect jdbc:mysql://localhost/collecting \
--driver com.mysql.jdbc.Driver \
--username root \
--table user \
--target-dir /sqoop/input/user \
--num-mappers 10 \
--direct
```

## Export

### Hadoop 내 user 데이터를 MySQL DB의 user 테이블로 데이터를 옮기는 기본 사용 예제

```
mysql> use collecting;
mysql> delete from user;
```

```
./bin/sqoop export \
--connect jdbc:mysql://localhost/collecting \
--driver com.mysql.jdbc.Driver \
--username root \
--table user \
--export-dir /sqoop/input/user
```

### --batch 옵션을 통한 Insert 속도 높이기
```
./bin/sqoop export \
-Dsqoop.export.records.per.statement=10 \
--connect jdbc:mysql://localhost/collecting \
--driver com.mysql.jdbc.Driver \
--username root \
--table user \
--export-dir /sqoop/input/user
```


### 기존 데이터셋 Updating
```
./bin/sqoop export \
--connect jdbc:mysql://localhost/collecting \
--driver com.mysql.jdbc.Driver \
--username root \
--table user \
--export-dir /sqoop/input/user \
--update-key id
```