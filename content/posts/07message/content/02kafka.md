---
title: "kafka基础"
date: 2020-10-20T14:06:25+06:00
description: kafka基础
menu:
  sidebar:
    parent: message
    name: kafka基础
    identifier: message2
    weight: 2
---

### Topic特性

1. 通过partition增加可扩展性：线上改partion数，rebalance ，会照成性能抖动。
2. partition有序达到高吞吐
3. partition多副本增加容错性



### kafka单机

1. 安装 http://kafka.apache.org/downloads

2. 修改配置

   ```shell
   cd kafka_2.13-2.7.0
   # 打开 listeners=PLAINTEXT://localhost:9092
   vim config/server.properties
   # 启动zookeeper
   bin/zookeeper-server-start.sh config/zookeeper.properties 
   # 启动kafaka
   bin/kafka-server-start.sh config/server.properties
   ```



3. 命令测试

   ```shell
   # 创建topic
   mokernetdeMac-mini:kafka_2.13-2.7.0 mokernet$ bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic testf  --partitions 4 --replication-factor 1
   Created topic testf.
   
   # 查看
   bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic testf
   
   # 消费者从头开始消费
   bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic testf
   
   # 生产者生产
   bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic testf
   
   # 生产者性能测试 100万条数据 每条1000byte 限流100万条
   bin/kafka-producer-perf-test.sh --topic testf --num-records 1000000 --record-size 1000 --throughput 1000000 --producer-props bootstrap.servers=localhost:9092
   
   # 消费者性能测试   消费100万条数据  一个线程
   bin/kafka-consumer-perf-test.sh --bootstrap-server localhost:9092 --topic testf --fetch-size 1048576 --messages 1000000 --threads 1
   ```



### kafka集群

1. 修改各个节点的三个属性配置如下：

   ```shell
   # 复制新的配置文件
   cp config/server.properties config/server-1.properties
   cp config/server.properties config/server-2.properties
   ```

   ```properties
   # 修改 id  端口 数据文件目录
   # config/server-1.properties:
     broker.id=1
     listeners=PLAINTEXT://:9093
     log.dir=/tmp/kafka-logs-1
   ```

   ```properties
   # 修改 id  端口 数据文件目录
   # config/server-2.properties:
     broker.id=2
     listeners=PLAINTEXT://:9094
     log.dir=/tmp/kafka-logs-2
   ```

2. 启动

   ```
   bin/kafka-server-start.sh config/server.properties &
   bin/kafka-server-start.sh config/server-1.properties &
   bin/kafka-server-start.sh config/server-2.properties &
   ```

3. 测试

   ```shell
   # 创建topic test42 4个patition 2个副本
   bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic test42  --partitions 4 --replication-factor 2
   # 查看
   mokernetdeMac-mini:kafka_2.13-2.7.0 mokernet$ bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic test42
   Topic: test42   PartitionCount: 4       ReplicationFactor: 2    Configs: 
           Topic: test42   Partition: 0    Leader: 1       Replicas: 1,2   Isr: 1,2
           Topic: test42   Partition: 1    Leader: 2       Replicas: 2,0   Isr: 2,0
           Topic: test42   Partition: 2    Leader: 0       Replicas: 0,1   Isr: 0,1
           Topic: test42   Partition: 3    Leader: 1       Replicas: 1,0   Isr: 1,0
           
   # 消费者从头开始消费
   bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic test42
   
   # 生产者生产
   bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test42
   
   # 容错性测试 ，关闭id=1的broker
   ps aux | grep server-1.properties
   > mokernet         30302   0.0  1.9  7323756 405020 s006  S+   10:31PM   0:16.87 /usr/bin/java -Xmx
   kill -9 30302
   
   #查看 如下，rebalance
   mokernetdeMac-mini:kafka_2.13-2.7.0 mokernet$ bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic test42
   Topic: test42   PartitionCount: 4       ReplicationFactor: 2    Configs: 
           Topic: test42   Partition: 0    Leader: 2       Replicas: 1,2   Isr: 2
           Topic: test42   Partition: 1    Leader: 2       Replicas: 2,0   Isr: 2,0
           Topic: test42   Partition: 2    Leader: 0       Replicas: 0,1   Isr: 0
           Topic: test42   Partition: 3    Leader: 0       Replicas: 1,0   Isr: 0
   ```

   

### Kafka connect



### kafak stream



