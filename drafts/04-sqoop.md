### Sqoop

#### Install

- Sqoop Homepage: https://sqoop.apache.org/
    - Download > nearby mirror
    - 1.4.7 > tar.gz 링크 복사

```bash
# 다운로드 sqoop
wget https://downloads.apache.org/sqoop/1.4.7/sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz
# 압축 해제
tar zxvf sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz
# 이동
sudo mv sqoop-1.4.7.bin__hadoop-2.6.0 /usr/local/sqoop
```

- 환경 변수 등록

```bash
# in .bashrc
# Sqoop Setting
export SQOOP_HOME=/usr/local/sqoop
export PATH=$SQOOP_HOME/bin:$PATH

# 반영
source .bashrc

# 실행해 보자!
sqoop
```

- mariadb jdbc 드라이버
    - https://downloads.mariadb.org/connector-java/

```bash
# download jdbc driver
wget https://downloads.mariadb.com/Connectors/java/latest/mariadb-java-client-2.3.0.jar
```

- sqoop 스크립트 작성

```bash
# in sqoop_nation_import.sh
--username
hadoop
--password
hadoop
--connect
jdbc:mariadb://localhost:3306/tpch_1g
--table
nation
```

- sqoop 스크립트 실행

```bash
sqoop import --options-file sqoop_nation_import.sh
```

- Error 발생
  - Why? 시작하세요 하둡! p618

```bash
# sqoop_nation_import.sh 수정
# 아래 내용 추가
--split-by
n_regionkey

# 스크립트 다시 시작

# 완료 후 결과 확인
hdfs dfs -ls /user/hadoop/nation
hdfs dfs -cat /user/hadoop/nation/part-m-00000
```

- hive로 임포트하기

```bash
# 스크립트 작성
# sqoop-orders-import-hive.sh
--username
hadoop
--password
hadoop
--connect
jdbc:mariadb://localhost:3306/tpch_1g
--driver
org.mariadb.jdbc.Driver
--table
orders
--split-by
o_orderkey
--hive-home
/usr/local/hive
--hive-import
--hive-table
orders
--hive-overwrite
-m
2
```

- 스크립트 실행

```bash
sqoop import --options-file sqoop_orders_import_hive.sh
# 완료되면 하이브로 확인
hive

hive> show databases;
hive> use default;
hive> show tables;
hive> select * from orders LIMIT 10;
```

- TIP
    - 에러가 나면 HIVE_CONF_DIR 확인
    - 그래도 안되면 HIVE_CONF_DIR 내의 hive-common-*.jar를 $SQOOP_HOME/lib로 복사