# Sqoop Scripts

## Sqoop 설치

## Import

### 특정 테이블 데이터 import 
```
sqoop import \
--connect jdbc:mysql://mysql.example.com/mydb \
--username sqoop \
--password sqoop \
--table acclog \
--target-dir /teststore/input/acclog
```

### DB내 전체 테이블 Import 
```   
sqoop import-all-tables \
--connect jdbc:mysql://mysql.example.com/mydb \
--username sqoop \
--password sqoop \
--exclude-tables cities,countries \
--warehouse-dir /teststore/input/
```

### Mapper 개수 지정 

```
sqoop import \
--connect jdbc:mysql://mysql.example.com/mydb \
--username sqoop \
--password sqoop \
--table acclog \
--target-dir /teststore/input/acclog \
--num-mappers 10
```

### 일부 데이터, 조건에 맞는 데이터 가져오기 
#### --incremental 옵션 사용
```
sqoop import \
--connect jdbc:mysql://mysql.example.com/mydb \
--username sqoop \
--password sqoop \
--table acclog \
--target-dir /teststore/input/acclog \
--num-mappers 10 \
--incremental append \
--check-column seq \
--last-value 8910000
```

#### 특정 Query 조건에 맞는 데이터 가져오기
```
sqoop import \
--connect jdbc:mysql://mysql.example.com/mydb \
--username sqoop \
--password sqoop \
--target-dir /teststore/input/acclog \
--query 'SELECT seq, inputdate, searchkey \
FROM acclog
WHERE seq > 10' \
--num-mappers 10 
```

### 데이터 전송속도 높이기 --direct 옵션 
```
sqoop import \
--connect jdbc:mysql://mysql.example.com/mydb \
--username sqoop \
--password sqoop \
--table acclog \
--target-dir /teststore/input/acclog \
--num-mappers 10 \
--direct
```

## Export

### Hadoop 내 acclog 데이터를 MySQL DB의 acclog 테이블로 데이터를 옮기는 기본 사용 예제
```
sqoop export \
--connect jdbc:mysql://mysql.example.com/mydb \
--username sqoop \
--password sqoop \
--table acclog \
--export-dir /teststore/input/acclog
```

### --batch 옵션을 통한 Insert 속도 높이기
```
sqoop export \
-Dsqoop.export.records.per.statement=10 \
--connect jdbc:mysql://mysql.example.com/mydb \
--username sqoop \
--password sqoop \
--table acclog \
--export-dir /teststore/input/acclog
```


### 기존 데이터셋 Updating
```
sqoop export \
--connect jdbc:mysql://mysql.example.com/mydb \
--username sqoop \
--password sqoop \
--table acclog \
--export-dir /teststore/input/acclog \
--update-key seq
```