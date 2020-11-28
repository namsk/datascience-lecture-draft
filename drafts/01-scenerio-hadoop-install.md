# Installing CentOS 7

## CentOS 7 Minimal

- 가상머신 구성
  - Networks
    - 어댑터 0: NAT
    - 어댑터 1: Host Only
      - 고급 : 무작위모드 모두 허용
  - Disk : 자동 구성(전체를 파티션으로 잡기)

```bash
# device 확인
ip addr

# root
cd /etc/sysconfig/network-scripts

# enp0s8 활성화
ifup enp0s8
ip addr
```

- ifcfg-enp0s8 수정
  
```bash
# ifcfg-enp0s8
BOOTPROTO=static
ONBOOT=yes
IPADDR=192.168.56.100
NETMASK=255.255.255.0
GATEWAY=192.168.56.1
NETWORK=192.168.56.0
```

- 네트워크 재시작

```bash
systemctl restart network
# 다시 enp0s8 확인
ip addr
```

- XShell에서 접속

```
# XShell
ssh 192.168.56.100
```

```bash
# 저장소 구성 내용 확인
yum repolist
# 업데이트
yum update
```

- -> livecodings / 02-linux-install-and-setup.md 진행

# Setup Hadoop

- java : 1.8

- Hadoop : 2.9.2
  - 공식 document: https://hadoop.apache.org/docs/r2.9.2/

JDK 설치

```bash
# 패키지명 검색
yum list java*jdk-devel
yum install java-1.8.0-openjdk-devel
# 설치 확인
java -version
javac -version
# jdk는 어디에 설치되어 있는가?
which javac
readlink -f /usr/bin/javac

# 링크 설정
cd /usr/lib/jvm
ln -s java-1.8.0-openjdk-1.8.0.272.b10-1.el7_9.x86_64/ jdk
```

JAVA_HOME 설정
```bash
# in /etc/profile
export JAVA_HOME=/usr/lib/jvm/jdk
```

JAVA_HOME 반영
```bash
source /etc/profile
echo $JAVA_HOME
```

PROTOBUF 설치

- 구글이 작성
- C로 개발
- 이기종간 명령어들에 대한 문자 체계가 깨지는 것을 방지하는 역할

```bash
yum -y install protobuf protobuf-devel
# 설치 확인
protoc --version
```

## 하둡 프로그램 운영 계정 생성

- root 계정으로 설치하는 것은 권장하지 않는다
- 하둡 운용을 위한 별도의 계정 생성(hadoop)

### hadoop 그룹 생성

```bash
groupadd hadoop
```

### hadoop 사용자 생성

```bash
useradd -g hadoop -G wheel -m hadoop
usermod -aG wheel hadoop # hadoop 사용자를 wheel 그룹에 포함
```

- g: 1차 그룹 지정
- G: 2차 그룹 지정 (하둡도 공유폴더를 이용하기 위한 virtualbox용 그룹)
- m: 사용자 디렉터리 생성

### hadoop 계정 패스워드 지정

```bash
passwd hadoop
```

### 사용자가 소속된 그룹 확인
```bash
groups hadoop
```

### 여기서부터는 hadoop 계정으로 진행합니다.

- hadoop 2.9.2 binary 링크 복사

```bash
wget https://downloads.apache.org/hadoop/common/hadoop-2.9.2/hadoop-2.9.2.tar.gz
# 압축 해제
tar zxvf hadoop-2.9.2.tar.gz
# 이동
sudo mv hadoop-2.9.2 /usr/local/hadoop
```

```bash
# .bashrc 수정
cd
vi .bashrc
```

```bash
# in .bashrc
export JAVA_HOME=/usr/lib/jvm/jdk
export HADOOP_HOME=/usr/local/hadoop
export PATH=$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```

```bash
source .bashrc
# 반영 내용 확인
echo $HADOOP_HOME
echo $PATH
```

### 공개키 설정

```bash
cd
ssh-keygen
ssh-copy-id -i /home/hadoop/.ssh/id_rsa.pub hadoop@192.168.56.100
# 계정 password 입력
# ssh hadoop@192.168.56.100 했을 때 비밀번호를 안물어보면 성공
```

### 하둡 실행 스크립트 수정

```bash
# in /usr/local/hadoop/etc/hadoop/hadoop-env.sh
export JAVA_HOME=/usr/lib/jvm/jdk
# 하둡 버전 확인
hadoop version
```

- [HADOOP 설정] : https://m.blog.naver.com/PostView.nhn?blogId=kgw1988&logNo=220949101567&proxyReferer=https:%2F%2Fwww.google.com%2F
### 가상 분산 모드에서 하둡 환경 설치

- http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html
- core-site.xml : 네임노드 관련 설정
  
```bash
# in /usr/local/hadoop/etc/hadoop/core-site.xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/usr/local/hadoop/temp</value>
  </property>
</configuration>
```

- hdfs-site.xml

```bash
# in /usr/local/hadoop/etc/hadoop/hdfs-site.xml
<configuration>
  <property>
    <!-- 복제본의 개수-->
    <name>dfs.replication</name>
    <value>1</value>
  </property>
  <property>
    <!-- 네임노드 데이터 디렉터리 -->
    <name>dfs.namenode.name.dir</name>
    <value>/usr/local/hadoop/data/dfs/namenode</value>
  </property>
  <property>
    <!-- 네임노드 체크포인트 -->
    <name>dfs.namenode.checkpoint.dir</name>
    <value>/usr/local/hadoop/data/dfs/namesecondary</value>
  </property>
  <property>
    <!-- 데이터노드 디렉터리 -->
    <name>dfs.datanode.data.dir</name>
    <value>/usr/local/hadoop/data/dfs/datanode</value>
  </property>
</configuration>
```

- mapred-site.xml : 맵리듀스 프레임워크 관련 설정
  - mapred-site.xml.template를 복사하여 수정한다

```bash
# cp mapred-site.xml.template mapred-site.xml
# in mapred-site.xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

- yarn-site.xml : YARN 설정 파일

```bash
# in yarn-site.xml
<configuration>
<!-- Site specific YARN configuration properties -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
</configuration>
```

### 하둡 실행

- 네임노드 초기화
  
```bash 
hadoop namenode --format
# 실행
start-all.sh
# 확인
jps
```

```bash
# 50070(웹 콘솔)을 방화벽에 추가
sudo firewall-cmd --permanent --zone=public --add-port=50070/tcp
sudo firewall-cmd --permanent --zone=public --add-port=50075/tcp
sudo firewall-cmd --permanent --zone=public --add-port=8042/tcp
sudo firewall-cmd --permanent --zone=public --add-port=8088/tcp

# 참고: 포트 삭제: --remove-port
# 참고: 방화벽 규칙 확인 : firewall-cmd --list-all
sudo systemctl restart firewalld  
```

### tip

```bash
hdfs dfsadmin -report # HDFS 사용량 확인
```

### Hadoop 맛보기

```bash
cd $HADOOP_HOME
hdfs dfs -mkdir /example
hdfs dfs -copyFromLocal README.txt /example
hdfs dfs -ls /example
```

- wordcount 프로그램 실행

```bash
cd $HADOOP_HOME/share/hadoop/mapreduce
hadoop jar hadoop-mapreduce-examples-2.9.2.jar wordcount /example/README.txt /output

# 결과 확인
hdfs dfs -ls /output
hdfs dfs -cat /output/part-r-0000 | more

# 결과를 로컬로 가져오기
hdfs dfs -get /output/part-r-0000 WordCount.txt
```

### MapReduce

- Generally MapReduce paradigm is based on sending the computer to where the data resides!
- MapReduce program executes in three stages, namely map stage, shuffle stage, and reduce stage
  - Map Stage : The map or mapper's job is to processthe input data. Generally the input data is in the form of file or directory and is stored in the Hadoop file system(HDFS). The input file is passed to the mapper function line by line. The mapper processes the data and creates several small chunks of data.
  - Reduce Stage : This stage is the combination of the Shuffle stage and the Reduce stage. The Reducer's job is to process the data that comes from the mapper. After processing, it produces a new set of output, which will be stored in the HDFS.
- During a MapReduce job, Hadoop sends the Map and Reduce tasks to the appropriate servers in the cluster.
- The framework manages all the details of data-passing such as issuing tasks, verifying task completion, and copying data around the cluster between the nodes.
- Most of the computing takes place on nodes with data on local disks that reduces the network traffic.
- After completion of the given tasks, the cluster collects and reduces the data to from an appropritate result, and sends it back to the Hadoop Server.


### 실행 결과 확인

### 미국 항공 데이터

- https://docs.microsoft.com/ko-kr/azure/hdinsight/hdinsight-hadoop-r-scaler-sparkr
- https://packages.revolutionanalytics.com/datasets/AirOnTime87to12/

```bash
wget https://packages.revolutionanalytics.com/datasets/AirOnTime87to12/AirOnTimeCSV.zip --no-check-certificate

mkdir datasets
cd datasets
mv AirOnTimeCSV.zip ./datasets

zip -FF AirOnTimeCSV.zip --out ReparedCSV.zip
unzip ReparedCSV.zip
```

- 참고 skip header hadoop mapreduce : https://stackoverflow.com/questions/33979242/how-to-skip-reading-the-file-header-in-hadoop-mapreduce

```bash
hdfs dfs -mkdir input
hdfs dfs -mkdir output
```

# DepartureDelayCount App 만들기

```bash
# 일단 테스트
hadoop jar myhadoop-0.0.1.jar myhadoop.DepartureDelayCount input/airOT2010*.csv output
# 완료 후 확인
hdfs dfs -cat output/part-r-00000
# 전체를 대상으로 다시 진행
hdfs dfs -rm -r -f output
hadoop jar myhadoop-0.0.1.jar myhadoop.DepartureDelayCount input output
```

# python hdfs 모듈 사용 예

# ArrivalDelayCount App 만들기 실습


# Tip

https://datacadamia.com/db/hadoop/hdfs/ui

- hdfs 설정 알아내기
  
```bash
hdfs getconf -confKey 키
```

- yarn node list

```bash
yarn node -list -all
```


### TODO

- 또 다른 세팅 방법
  - https://lukelab.tistory.com/12