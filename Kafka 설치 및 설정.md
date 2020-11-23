# Kafka 설치 및 설정

## 설치

### 1. Download (All Node)
```
# wget https://downloads.apache.org/kafka/2.6.0/kafka_2.13-2.6.0.tgz
# tar -zxvf kafka_2.13-2.6.0.tgz
```

### 2. Zookeeper 설정 (All Node)
```
# cd kafka_2.13-2.6.0
# vi config/zookeeper.properties
```
```
    dataDir=/tmp/zookeeper
    clientPort=22181
    maxClientCnxns=0
    admin.enableServer=false

    initLimit=5
    syncLimit=2

    server.1=172.27.0.51:2888:3888
    server.2=172.27.0.52:2888:3888
    server.3=172.27.0.53:2888:3888
```

## 설치

### 3. Kafka 설정 (All Node)
```
# vi config/server.properites
```
```
    # master 서버
    broker.id=1
    listeners=PLAINTEXT://:29092
    advertised.listeners=PLAINTEXT://172.27.0.51:29092
    zookeeper.connect=172.27.0.51:22181, 172.27.0.52:22181, 172.27.0.53:22181

    # slave1 서버
    broker.id=2
    listeners=PLAINTEXT://:29092
    advertised.listeners=PLAINTEXT://172.27.0.52:29092
    zookeeper.connect=172.27.0.51:22181, 172.27.0.52:22181, 172.27.0.53:22181

    # slave2 서버
    broker.id=3
    listeners=PLAINTEXT://:29092
    advertised.listeners=PLAINTEXT://172.27.0.53:29092
    zookeeper.connect=172.27.0.51:22181, 172.27.0.52:22181, 172.27.0.53:22181
```

## 설치

### 4. Myid 파일 생성 (kafka 설정 broker.id 값과 동일하게)
```
# master 서버
# mkdir /tmp/zookeeper
# echo 1 > /tmp/zookeeper/myid

# slave1 서버
# mkdir /tmp/zookeeper
# echo 2 > /tmp/zookeeper/myid

# slave2 서버
# mkdir /tmp/zookeeper
# echo 3 > /tmp/zookeeper/myid
```

### 5. Zookeeper 및 kafka 서버 구동 (All Node)
```
# sh bin/zookeeper-server-start.sh config/zookeeper.properties &
# sh bin/kafka-server-start.sh config/server.properties &
```

### 6. Topic 생성
```
# bin/kafka-topics.sh --create --zookeeper 172.27.0.51:22181,172.27.0.52:22182,172.27.0.53:22183 --replication-factor 3 --partitions 3 --topic sampleTopic
```

### 7. Topic list
```
# sh bin/kafka-topics.sh --list --zookeeper 172.27.0.51:22181,172.27.0.52:22182,172.27.0.53:22183
```

### 8. Topic 상세 확인
```
# sh bin/kafka-topics.sh --describe --zookeeper 172.27.0.51:22181,172.27.0.52:22182,172.27.0.53:22183 --topic sampleTopic
```