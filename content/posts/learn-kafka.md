---
title: "kafka能做什么？kafka集群配置 (卡夫卡 大数据)"
date: 2020-04-25T13:28:12+08:00
description: "kafka能做什么？kafak如何使用,kafka集群配置 (卡夫卡 大数据)"
categories: ["大数据","消息中间件"]
tags: ["大数据","kafka","消息中间件"]
featured_image:
author: "ipoo"
KeyName : "kafka,kafka集群,kafka使用"
---
<!-- MarkdownTOC -->

- [什么是Kafka](#%E4%BB%80%E4%B9%88%E6%98%AFkafka)
	- [官网介绍：](#%E5%AE%98%E7%BD%91%E4%BB%8B%E7%BB%8D%EF%BC%9A)
	- [几个概念：](#%E5%87%A0%E4%B8%AA%E6%A6%82%E5%BF%B5%EF%BC%9A)
- [详细介绍 ：](#%E8%AF%A6%E7%BB%86%E4%BB%8B%E7%BB%8D-%EF%BC%9A)
- [操作kafka：](#%E6%93%8D%E4%BD%9Ckafka%EF%BC%9A)
- [kafka集群](#kafka%E9%9B%86%E7%BE%A4)
- [消息测试](#%E6%B6%88%E6%81%AF%E6%B5%8B%E8%AF%95)
- [问题检测](#%E9%97%AE%E9%A2%98%E6%A3%80%E6%B5%8B)

<!-- /MarkdownTOC -->


## 什么是Kafka 
### 官网介绍：
> ApacheKafka®是一个分布式流媒体平台。这到底是什么意思呢？
>
>	我们认为流媒体平台具有三个关键功能：
>	 1. 它可以让你发布和订阅记录流。在这方面，它类似于消​​息队列或企业消息传递系统。
>	 2. 它允许您以容错方式存储记录流。
>	 3. 它可以让您在发生记录时处理记录流。
### 几个概念：
 1.  Kafka作为一个或多个服务器上的集群运行。
 2. Kafka集群以称为主题的类别存储记录流。
 3. 每个记录由一个键，一个值和一个时间戳组成。
## 详细介绍 ：
[这篇博客写的很详细](https://blog.csdn.net/qq_37518574/article/details/105748930)
- 下载安装 
[官网下载地址](https://www.apache.org/dyn/closer.cgi?path=/kafka/2.5.0/kafka-2.5.0-src.tgz): `https://www.apache.org/dyn/closer.cgi?path=/kafka/2.5.0/kafka-2.5.0-src.tgz`
- 解压：`tar zxf kafka_2.11-1.0.0`
- 启动Zookeeper：测试可以采用kafka自带zookeeper
```java
	./bin/zookeeper-server-start.sh config/zookeeper.properties 
```
-  启动kafka：
`./bin/kafka-server-start.sh config/server.properties`

在默认的 kafka_2.11-1.0.0/config 目录下的server.properties文件中
默认端口为9092
## 操作kafka：
- 创建名为`test`的topic
```text
./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
- 创建生产者（生产消息）：
```text
./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```
![](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTgwMTE4MTQwMDQ2NzU5)
- 创建消费者（消费消息）：
```text
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```
![](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTgwMTE4MTQwMTE5MTM3)

## kafka集群 
- 创建配置文件 `server.properties`拷贝到其他机器

- 修改`server.properties`配置,修改为:
```text
    broker.id=1
    listeners=PLAINTEXT://:9093
    log.dir=/tmp/kafka-logs-1
```
broker.id:当前机器在集群中的唯一标识,每台服务器的broker.id都不能相同  

log.dirs:是kafka接收消息存放路径

- 启动 
```text
./bin/kafka-server-start.sh config/server.properties & 
```
- 创建topic 
```text
./bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic test2
```
- 查看topic
```text
./bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test2

Topic:test2	PartitionCount:1	ReplicationFactor:3	Configs:
	Topic: zhh-replicated-topic	Partition: 0	Leader: 2	Replicas: 2,0,1	Isr: 2,0,1
```

第一行表示汇总信息. 有1个分区, 3份备份

第二行表示每个分区的信息,对分区0,领导节点id是2, 备份到2,0,1.

leader 表示负责某分区全部读写的节点. 每个分区都会有随机选择的leader.

Replicas 表示需要复制到的节点, 不管是否活着.

Isr 表示(“in-sync” replicas), 正在同步的备份, 表示可用的活着的节点
## 消息测试

- 创建生产者
```text
./bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test2
```
- 创建消费者
```text
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test2 --from-beginning
```
## 问题检测
- 节点崩溃
```text
ps aux | grep server.properties    #查看节点（73370）是否已启用
kill -9 73370					   #杀死此节点
```
- zookeeper
 zookeeper打开后要打开新的连接操作kafka。


**有帮助请留言...**
<!-- 

扫码关注公众号《ipoo》
![ipoo](https://oss.ipooli.com/images/%E5%85%AC%E4%BC%97%E5%8F%B7code.jpg) -->