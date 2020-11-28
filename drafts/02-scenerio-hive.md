### Setup Hive

- http://hive.apache.org/
- http://www.incodom.kr/%ED%95%98%EB%91%A1%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D
- Hive Tutorials: https://cwiki.apache.org/confluence/display/Hive/Tutorial

```bash
wget https://downloads.apache.org/hive/hive-2.3.7/apache-hive-2.3.7-bin.tar.gz
tar zxvf apache-hive-2.3.7-bin.tar.gz
sudo mv apache-hive-2.3.7-bin /usr/local/hadoop
```

- conf/hive-env.sh.template -> conf/hive-env.sh

```bash
cd /usr/local/hive
cp conf/hive-env.sh.template conf/hive-env.sh
```

- 수정

```bash
vi conf/hive-env.sh

# in hive-env.sh
HADOOP_HOME=/usr/local/hadoop
```

- 하이브 실행을 위한 필수 디렉터리 HDFS에 생성 및 설정
  
```bash
hdfs dfs -mkdir -p /user/hive/warehouse
hdfs dfs -mkdir -p /tmp/hive
hdfs dfs -chmod g+w /user/hive/warehouse
hdfs dfs -chmod 777 /tmp/hive
```

- .bashrc 수정

```bash
# in .bashrc
# Hive Setting
export HIVE_HOME=/usr/local/hive
export PATH=$HIVE_HOME/bin:$PATH
```

- 메타스토어 초기화

```bash
# in HIVE_HOME
./bin/schematool -initSchema -dbType derby
# hive 실행
./bin/hive
hive> show databases;
hive> quit;
```
  
- 데이터베이스 생성
  
```bash
CREATE TABLE kvlocalstore (key INT, value STRING);
LOAD DATA LOCAL INPATH '/usr/local/hive/examples/files/kv1.txt' INTO TABLE kvlocalstore;
```

- EXRERNAL TABLE

```bash
CREATE TABLE airline_lines (line STRING)
LOCATION '/user/hadoop/input' # airline 데이터를 업로드한 위치
SELECT * FROM airline_lines LIMIT 10;
```

- airline table 만들기

```bash
CREATE EXTERNAL TABLE airline(
    YEAR STRING,
    MONTH STRING,
    DAY_OF_MONTH STRING,
    DAY_OF_WEEK STRING,
    FL_DATE STRING,
    UNIQUE_CARRIER STRING,
    TAIL_NUM STRING,
    FL_NUM STRING,
    ORIGIN_AIRPORT_ID STRING,
    ORIGIN STRING,
    ORIGIN_STATE_ABR STRING,
    DEST_AIRPORT_ID STRING,
    DEST STRING,
    DEST_STATE_ABR STRING,
    CRS_DEP_TIME STRING,
    DEP_TIME STRING,
    DEP_DELAY STRING,
    DEP_DELAY_NEW STRING,
    DEP_DEL15 STRING,
    DEP_DELAY_GROUP STRING,
    TAXI_OUT STRING,
    WHEELS_OFF STRING,
    WHEELS_ON STRING,
    TAXI_IN STRING,
    CRS_ARR_TIME STRING,
    ARR_TIME STRING,
    ARR_DELAY STRING,
    ARR_DELAY_NEW STRING,
    ARR_DEL15 STRING,
    ARR_DELAY_GROUP STRING,
    CANCELLED STRING,
    CANCELLATION_CODE STRING,
    DIVERTED STRING,
    CRS_ELAPSED_TIME STRING,
    ACTUAL_ELAPSED_TIME STRING,
    AIR_TIME STRING,
    FLIGHTS STRING, 
    DISTANCE STRING,
    DISTANCE_GROUP STRING,
    CARRIER_DELAY STRING,
    WEATHER_DELAY STRING,
    NAS_DELAY STRING,
    SECURITY_DELAY STRING,
    LATE_AIRCRAFT_DELAY STRING
)
COMMENT 'airline csv data'
ROW FORMAT DELIMITED
    FIELDS TERMINATED BY ','
    LINES TERMINATED BY '\n'
    STORED AS TEXTFILE
LOCATION '/user/hadoop/input';

SELECT * FROM airline limit 10;

# " 가 문제다
DROP TABLE airline

# 2차 수정
CREATE EXTERNAL TABLE airline(
    YEAR INT,
    MONTH INT,
    DAY_OF_MONTH INT,
    DAY_OF_WEEK INT,
    FL_DATE DATE,
    UNIQUE_CARRIER STRING,
    TAIL_NUM STRING,
    FL_NUM STRING,
    ORIGIN_AIRPORT_ID STRING,
    ORIGIN STRING,
    ORIGIN_STATE_ABR STRING,
    DEST_AIRPORT_ID STRING,
    DEST STRING,
    DEST_STATE_ABR STRING,
    CRS_DEP_TIME STRING,
    DEP_TIME FLOAT,
    DEP_DELAY FLOAT,
    DEP_DELAY_NEW FLOAT,
    DEP_DEL15 STRING,
    DEP_DELAY_GROUP STRING,
    TAXI_OUT STRING,
    WHEELS_OFF STRING,
    WHEELS_ON STRING,
    TAXI_IN STRING,
    CRS_ARR_TIME STRING,
    ARR_TIME FLOAT,
    ARR_DELAY FLOAT,
    ARR_DELAY_NEW FLOAT,
    ARR_DEL15 STRING,
    ARR_DELAY_GROUP STRING,
    CANCELLED STRING,
    CANCELLATION_CODE STRING,
    DIVERTED STRING,
    CRS_ELAPSED_TIME STRING,
    ACTUAL_ELAPSED_TIME STRING,
    AIR_TIME STRING,
    FLIGHTS STRING, 
    DISTANCE STRING,
    DISTANCE_GROUP STRING,
    CARRIER_DELAY STRING,
    WEATHER_DELAY STRING,
    NAS_DELAY STRING,
    SECURITY_DELAY STRING,
    LATE_AIRCRAFT_DELAY STRING
)
COMMENT 'airline csv data'
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES(
    "separatorChar" = ",",
    "quoteChar" = "\""
)
LOCATION '/user/hadoop/input'
tblproperties('skip.header.line.count'='1');

SELECT * FROM airline LIMIT 10;
```

- 참고: hive airline : http://www.lifeisafile.com/flight-analysis/

```bash
# 1차 테스트
SELECT count(*) FROM airline WHERE year = 2012;
# -> 시간 많이 걸림
SELECT year, month, count(if(ARR_DELAY>0, "", NULL)) 
FROM airline 
GROUP BY year, month
ORDER BY year, month;
```