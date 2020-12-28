---
layout: photo
title: Flume-1.9.0安装文档
date: 2020-06-05 23:28:09
tags: [Flume]
categories: [Bigdata,Flume]
password: 5201314
---
# 前提条件

## JDK的安装

> 注：Flume安装的前提条件取决于Flume的Sink类型。如果为KafkaSink则需要安装Kafka，如果为HDFS Sink则需要安装Hadoop，以此类推。

<!-- more -->

# 下载

---

## 1. 浏览器下载

[Apache Flume官网下载](https://mirror-hk.koddos.net/apache/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz)

[Apache国内镜像下载](https://mirrors.tuna.tsinghua.edu.cn/apache/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz)

## 2. 服务器本地下载

```bash
#Apache Flume官网下载
wget https://mirror-hk.koddos.net/apache/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz
	
#Apache国内镜像下载
wget https://mirrors.tuna.tsinghua.edu.cn/apache/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz
```

# 配置

---

## 1. 创建conf-file文件

> 注：推荐在Flume根目录下创建一个job文件夹，用于存放以后满足各种需求的conf-file文件

## 2. 配置环境变量

> 注：配置环境变量极为简单，这里不做赘述

## 3. 示例

> 注：Flume需要根据不同的业务配置不同的conf-flie，下面以三种不同类型的业务需求来了解 如何设置conf-file

### （1）Flume采集端口信息，并打印到控制台

- 安装测试工具**telnlt**（如果机器没有安装）

```bash
sudo rpm -ivh telnet-server-0.17-59.el7.x86_64.rpm 
sudo rpm -ivh telnet-0.17-59.el7.x86_64.rpm
```

- 查看我们之后用到的端口是否被占用

```bash
netstat -an | grep 44444
```

- 编写conf-file文件（在job目录下创建**flume-telnet-logger.conf**文件，写入以下内容）

```properties	
# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c+1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

- 开启Flume进程（控制台运行）

```bash
flume-ng agent --conf conf/ --conf-file */flume/job/flume-telnet-logger.conf --name a1 -Dflume.root.logger==INFO,console
```

> 注：
>  	1. --conf			表示Flume默认的配置文件
>  	2. --conf-flie		表示用户自定义的配置文件（随业务需求而变）
>  	3. --name			表示agent的名字，与配置文件中的每一行的开头相同



- 打开新的窗口简历Telnet通话

```bash
telnet localhost 44444
```

- 在telnet端输入数据，在 flume 监听端就可以接收到，至此Flume监听端口信息采集数据到控制台完成

	

### （2）Flume采集日志文件到HDFS

- 编写conf-file文件**flume-file-hdfs.conf**

```properties
# Name the components on this agent
a2.sources = r2
a2.sinks = k2
a2.channels = c2
# Describe/configure the source
a2.sources.r2.type = exec
a2.sources.r2.command = tail -F /opt/module/hive-2.3.3/logs/hive.log
a2.sources.r2.shell = /bin/bash -c
# Describe the sink
a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://hadoop201:9000/flume/%Y%m%d/%H
#上传文件的前缀
a2.sinks.k2.hdfs.filePrefix = logs-
#是否按照时间滚动文件夹
a2.sinks.k2.hdfs.round = true
#多少时间单位创建一个新的文件夹
a2.sinks.k2.hdfs.roundValue = 1
#重新定义时间单位
a2.sinks.k2.hdfs.roundUnit = hour
#是否使用本地时间戳
a2.sinks.k2.hdfs.useLocalTimeStamp = true
#积攒多少个Event才flush到HDFS一次
a2.sinks.k2.hdfs.batchSize = 1000
#设置文件类型，可支持压缩
a2.sinks.k2.hdfs.fileType = DataStream
#多久生成一个新的文件
a2.sinks.k2.hdfs.rollInterval = 600
#设置每个文件的滚动大小
a2.sinks.k2.hdfs.rollSize = 134217700
#文件的滚动与Event数量无关
a2.sinks.k2.hdfs.rollCount = 0
#最小冗余数
a2.sinks.k2.hdfs.minBlockReplicas = 1
# Use a channel which buffers events in memory
a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100
# Bind the source and sink to the channel
a2.sources.r2.channels = c2
a2.sinks.k2.channel = c2
```

- 开启Flume进程

```bash
#控制台运行
flume-ng agent --conf conf --conf-file */flume/job/flume-file-hdfs.conf --name a2 -Dflume.root.logger=INFO,console
#后台运行
nohup flume-ng agent --conf conf --conf-file */flume/job/flume-file-hdfs.conf --name a2 1>/dev/null 2>&1 &
```

- 检查日志有新增数据时，HDFS上是否出现新文件。若有新文件产生，则Flume采集日志文件到HDFS完成



### （3）Flume采集日志文件到Kafka（实时）

- 编写conf-file文件**flume-kafka.conf**

```properties
a1.sources = r1
a1.sinks = k1
a1.channels = c1
# Describe/configure the source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /home/xuanfu/logs/user/iag-video-info.log
# Describe the sink
#a1.sinks.k1.type = logger
a1.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink
a1.sinks.k1.topic = video_record
a1.sinks.k1.brokerList = 172.16.221.82:9092,172.16.221.80:9092,172.16.221.81:9092
a1.sinks.k1.requiredAcks = 1
a1.sinks.k1.batchSize = 20
# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100
# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

- 开启Flume进程

```bash
#控制台运行
flume-ng agent --conf conf --conf-file */flume/job/flume-kafka.conf --name a1 -Dflume.root.logger=INFO,console
#后台运行
nohup flume-ng agent --conf conf --conf-file */flume/job/flume-kafka.conf --name a1 1>/dev/null 2>&1 &
```

- 开启一个Kafka控制台消费者进程。检查日志有新增数据时，控制台上是否有数据显示。若有数据显示，则Flume采集日志文件到Kafka完成