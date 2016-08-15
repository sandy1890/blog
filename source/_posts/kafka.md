---
title: kafka基础操作
date: 2016-04-25 21:37:56
categories:
    - 服务端
---

# kafka

## 启动

```
./bin/kafka-server-start.sh  config/server.properties
```

## 创建topic

```
./bin/kafka-topics.sh  --zookeeper 127.0.0.1:2181 --create --topic lingxiao-helloworld --replication-factor 1 --partitions 1
```

## 生产者

```
./bin/kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic lingxiao-helloworld
```

## 消费者

```
./bin/kafka-console-consumer.sh --zookeeper 127.0.0.1:2181 --topic lingxiao-helloworld --from-beginning
```
