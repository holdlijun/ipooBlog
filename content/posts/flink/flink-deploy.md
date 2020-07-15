---
title: "Flink 集群搭建,Standalone,集群部署,HA高可用部署"
date: 2020-06-22T16:50:46+08:00
description: Flink 集群搭建,Standalone,集群部署,HA高可用三种模式部署方式
categories: ["Flink"]
tags: ["Flink"]
featured_image: "https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/uploads%2F2020%2F06%2F22%2Fxr2fPLjz_%E7%A8%BF%E5%AE%9A%E8%AE%BE%E8%AE%A1%E5%AF%BC%E5%87%BA-20200622-165815_%E7%9C%8B%E5%9B%BE%E7%8E%8B.png?Expires=1592816335"
author: "ipoo"
KeyName : "Flink,Standalone 部署,Flink 集群部署,Flink HA高可用部署,High Availability (HA)"
---


<!-- MarkdownTOC -->

- [基础环境](#%E5%9F%BA%E7%A1%80%E7%8E%AF%E5%A2%83)
	- [准备3台虚拟机](#%E5%87%86%E5%A4%873%E5%8F%B0%E8%99%9A%E6%8B%9F%E6%9C%BA)
	- [配置无密码登录](#%E9%85%8D%E7%BD%AE%E6%97%A0%E5%AF%86%E7%A0%81%E7%99%BB%E5%BD%95)
	- [下载Flink](#%E4%B8%8B%E8%BD%BDflink)
- [部署](#%E9%83%A8%E7%BD%B2)
	- [Standalone Cluster 单机模式](#standalone-cluster-%E5%8D%95%E6%9C%BA%E6%A8%A1%E5%BC%8F)
		- [启动](#%E5%90%AF%E5%8A%A8)
	- [集群模式](#%E9%9B%86%E7%BE%A4%E6%A8%A1%E5%BC%8F)
		- [修改配置文件](#%E4%BF%AE%E6%94%B9%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
		- [拷贝到其他两台机器](#%E6%8B%B7%E8%B4%9D%E5%88%B0%E5%85%B6%E4%BB%96%E4%B8%A4%E5%8F%B0%E6%9C%BA%E5%99%A8)
		- [启动集群](#%E5%90%AF%E5%8A%A8%E9%9B%86%E7%BE%A4)
	- [HA高可用模式](#ha%E9%AB%98%E5%8F%AF%E7%94%A8%E6%A8%A1%E5%BC%8F)
		- [下载hadoop依赖包](#%E4%B8%8B%E8%BD%BDhadoop%E4%BE%9D%E8%B5%96%E5%8C%85)
		- [修改`./conf/flink-conf.yaml` 配置文件](#%E4%BF%AE%E6%94%B9confflink-confyaml-%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6)
		- [启动hdfs](#%E5%90%AF%E5%8A%A8hdfs)
		- [启动Flink中zookeeper](#%E5%90%AF%E5%8A%A8flink%E4%B8%ADzookeeper)
		- [启动Flink](#%E5%90%AF%E5%8A%A8flink)
		- [查看](#%E6%9F%A5%E7%9C%8B)
		- [测试](#%E6%B5%8B%E8%AF%95)

<!-- /MarkdownTOC -->


# 基础环境

## 准备3台虚拟机

![](http://oss.ipooli.com/uploads%2F2020%2F06%2F22%2Fe79Dap6M_%E6%9C%AA%E5%91%BD%E5%90%8D1592806473.png?Expires=1592806501)

## 配置无密码登录
配置方法:[https://ipooli.com/2020/04/linux_host/](https://ipooli.com/2020/04/linux_host/)

并且做好主机映射。
## 下载Flink

https://www.apache.org/dyn/closer.lua/flink/flink-1.10.1/flink-1.10.1-bin-scala_2.11.tgz

并解压缩

# 部署

## Standalone Cluster 单机模式

### 启动
进入flink-1.10.1 文件夹内

直接执行:

```txt
./bin/start-cluster.sh
```
![image](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/uploads%2F2020%2F06%2F22%2FxNovWky1_%E6%9C%AA%E5%91%BD%E5%90%8D1592807732.png?Expires=1592807753)

## 集群模式

### 修改配置文件
进入flink-1.10.1 文件夹内

- 修改`./conf/flink-conf.yaml`

修改如下几个参数:
```txt
jobmanager.rpc.address: bigdata1
jobmanager.rpc.port: 6123
jobmanager.heap.size: 1024m
taskmanager.memory.process.size: 1568m
taskmanager.numberOfTaskSlots: 3
parallelism.default: 3
```

- 修改`./conf/masters` 配置master节点

修改为:
```txt
bigdata1:8081
```

- 修改`./conf/slaves` 配置slaves节点

修改为:
```txt
bigdata1
bigdata2
bigdata3
```

### 拷贝到其他两台机器

```txt

scp -r /home/admin/flink/ root@bigdata2:/home/admin/

scp -r /home/admin/flink/ root@bigdata3:/home/admin/

```

### 启动集群

在 bigdata1上执行:

```txt
./bin/start-cluster.sh
```

随后访问 http://bigdata1:8081/

可以看到有3个Task Managers，1个Job Manager 为bigdata1

![image](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/uploads%2F2020%2F06%2F22%2FnZmnxoBm_yanshi.gif?Expires=1592807359)

## HA高可用模式

两个JobManager,当主 JobManager 宕机之后，使用备用 JobManager ,等宕机的 JobManager 恢复之后，又变成备用

### 下载hadoop依赖包

- 对应hadoop版本下载

我使用的hadoop版本为:hadoop-2.6.5 对应依赖包:[下载](https://repo.maven.apache.org/maven2/org/apache/flink/flink-shaded-hadoop-2-uber/2.6.5-10.0/flink-shaded-hadoop-2-uber-2.6.5-10.0.jar)

其他版本:[下载](https://flink.apache.org/downloads.html)

- 把依赖包放在flink 的 lib 目录下
- 配置环境变量

```txt
vi /etc/profile
# 添加环境变量
export HADOOP_CONF_DIR=/home/admin/hadoop-2.6.5/etc/hadoop
# 环境变量生效
source /etc/profile
```
### 修改`./conf/flink-conf.yaml` 配置文件

修改如下几个参数
```txt
high-availability: zookeeper
high-availability.storageDir: hdfs://bigdata1/flinkha/
high-availability.zookeeper.quorum: bigdata1:2181
high-availability.zookeeper.path.root: /flink
state.checkpoints.dir: hdfs:///flink/checkpoints
state.savepoints.dir: hdfs:///flink/savepoints

```

### 启动hdfs

关于hadoop的配置文件与启动方式在这就不赘述了。

### 启动Flink中zookeeper 

进入Flink文件夹
```txt
./bin/start-zookeeper-quorum.sh
```

`jps` 查看是否启动

![](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/uploads%2F2020%2F06%2F22%2Fz2rFiGyC_zoo.png?Expires=1592814593)

### 启动Flink

在bigdata1中执行

```txt
./bin/start-cluster.sh
```

### 查看
分别打开访问:

http://bigdata1:8081/

http://bigdata2:8081/

两个页面都可以查看集群信息

![](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/uploads%2F2020%2F06%2F22%2FGROpHmFc_ha.gif?Expires=1592814816)

### 测试

- 我们可以 kill掉bigdata1机器上的Job Manager,然后备用(bigdata2)Job Manager也是可以使用的。

![](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/uploads%2F2020%2F06%2F22%2F0mPVE0qa_%E6%9C%AA%E5%91%BD%E5%90%8D1592815242.png?Expires=1592815282)

- 再启动bigdata1的Job Manager

```txt
./bin/jobmanager.sh start
```
![](https://bigdata-lijun.oss-cn-hangzhou.aliyuncs.com/uploads%2F2020%2F06%2F22%2FScghUBo8_%E6%9C%AA%E5%91%BD%E5%90%8D1592815642.png?Expires=1592815708)

小结:本篇介绍了Flink单机,集群,HA高可用三种部署方式。




扫码关注公众号《ipoo》
![ipoo](http://oss.ipooli.com/images/%E5%85%AC%E4%BC%97%E5%8F%B7code.jpg)