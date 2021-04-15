# Kafka Compose Setup

https://github.com/conduktor/kafka-stack-docker-compose
```yaml
version: '2.1'

# 주키퍼와 카프카 서비스를 설정한다
services:
  # 3개의 노드로 된 주키퍼 클러스터를 구성한다.
  # 각각 서버에 zoo1, zoo2, zoo3이름으로 설정한다. 
  zoo1:
    image: zookeeper:3.4.9
    hostname: zoo1 # 서버에 맞게 수정필요
    ports:
      - "2181:2181"
      - "2888:2888"
      - "3888:3888"
    environment:
        ZOO_MY_ID: 1 # 서버에 맞게 수정필요
        ZOO_PORT: 2183
        ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
    volumes:
      - ./zk-multiple-kafka-multiple/zoo1/data:/data  # 서버에 맞게 수정필요
      - ./zk-multiple-kafka-multiple/zoo1/datalog:/datalog # 서버에 맞게 수정필요
    extra_hosts:
      - "zoo1:{cluster ip2}"
      - "zoo2:{cluster ip3}"
      - "zoo3:0.0.0.0"

  # 3개의 카프카 브로커를 구성한다.
  # Advertised listeners와 주키퍼 노드 포트를 등록한다.
  # 4개의 파티션을 구성한다.
  # 각각 서버에 kafka1, kafka1, kafka1이름으로 설정한다.
  kafka1:
    image: confluentinc/cp-kafka:5.2.1
    hostname: kafka1 # 서버에 맞게 수정필요
    ports:
      - "19092:9092"
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_LISTENERS: LISTENER_DOCKER_INTERNAL://kafka3:19092,LISTENER_DOCKER_EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: LISTENER_DOCKER_INTERNAL:PLAINTEXT,LISTENER_DOCKER_EXTERNAL:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: LISTENER_DOCKER_INTERNAL
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181,zoo2:2181,zoo3:2181"
      KAFKA_BROKER_ID: 1 # 서버에 맞게 수정필요
      KAFKA_NUM_PARTITIONS: 4
      KAFKA_LOG4J_LOGGERS: "kafka.controller=INFO,kafka.producer.async.DefaultEventHandler=INFO,state.change.logger=INFO"
    volumes:
      - ./zk-multiple-kafka-multiple/kafka1/data:/var/lib/kafka/data # 서버에 맞게 수정필요
    depends_on:
      - zoo3
    extra_hosts:
      - "zoo1:{cluster ip1}" # insert cluster IP
      - "zoo2:{cluster ip2}" # insert cluster IP
      - "zoo3:{cluster ip3}" # insert cluster IP
      - "kafka1:{cluster ip1}" # insert cluster IP
      - "kafka2:{cluster ip2}" # insert cluster IP
      - "kafka3:{cluster ip3}" # insert cluster IP
```
