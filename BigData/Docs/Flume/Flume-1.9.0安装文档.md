---
layout: photo
title: Flume-1.9.0安装文档
date: 2020-06-05 23:28:09
tags: [Flume]
categories: [Bigdata,Flume]
excerpt: 。。。
---

## 一、写在最前
---
> CentOS版本： CentOS Linux release 7.9.2009 (Core)
> jdk版本： 1.8
> Hadoop版本： 3.1.3
> Kafka版本： kafka_2.12-3.0.0



## 二、下载
---

### 1. 浏览器下载
[Apache Flume官网下载](https://mirror-hk.koddos.net/apache/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz)
[Apache国内镜像下载](https://mirrors.tuna.tsinghua.edu.cn/apache/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz)

### 2. 服务器本地下载
```bash
#Apache Flume官网下载
wget https://mirror-hk.koddos.net/apache/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz
#Apache国内镜像下载
wget https://mirrors.tuna.tsinghua.edu.cn/apache/flume/1.9.0/apache-flume-1.9.0-bin.tar.gz
```



## 三、配置
---

### 1. 创建conf-file文件
> 注：推荐在Flume根目录下创建一个job文件夹，用于存放以后满足各种需求的conf-file文件

### 2. 配置环境变量
> 注：配置环境变量极为简单，这里不做赘述、



## 四、采集示例
---

> 注：Flume需要根据不同的业务配置不同的conf-flie，下面以三种不同类型的业务需求来了解 如何设置conf-file

### 1. Flume采集端口信息并打印到控制台

- 检查是否安装telnlt，如果没有则安装测试工具**telnlt**
```bash
sudo rpm -ivh telnet-server-0.17-59.el7.x86_64.rpm 
sudo rpm -ivh telnet-0.17-59.el7.x86_64.rpm
```

- 查看是否被占用，由于telnet的默认端口是44444
```bash
netstat -an | grep 44444
```

- 在job目录下创建**flume-telnet-logger.conf**文件
```properties	
# 定义组件
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# 配置Source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# 配置sink
a1.sinks.k1.type = logger

# 配置Channel
a1.channels.c1.type = memory
a1.channels.c+1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# 组装
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```
- 开启Flume进程（控制台运行）
```bash
#控制台运行
flume-ng agent -n a1 -c /home/TomFor/software/flume/conf/ -f /home/TomFor/software/flume/job/flume-telnet-logger.conf -Dflume.root.logger=info,console
```

> 注：
>  	1. -n			表示agent的名字，与配置文件中的每一行的开头相同
>  	2. -c			表示Flume默认的配置文件
>  	3. -f		    表示用户自定义的配置文件（随业务需求而变）


- 打开新的窗口简历Telnet通话
```bash
telnet localhost 44444
```
- 在telnet端输入数据，在Flume监听端就可以接收到，至此Flume监听端口信息采集数据到控制台完成


### 2. Flume采集日志文件到Kafka

- 编写conf-file文件**flume-kafka.conf**

```properties
#定义组件
a1.sources = r1
a1.channels = c1

#配置source
a1.sources.r1.type = TAILDIR
a1.sources.r1.filegroups = f1
a1.sources.r1.filegroups.f1 = /home/TomFor/applog/logapp.*
a1.sources.r1.positionFile = /home/TomFor/software/flume/data/taildir_position.json
#自定义拦截器
a1.sources.r1.interceptors =  i1
a1.sources.r1.interceptors.i1.type = org.charlie.flume.interceptor.HdfsInterceptor$Builder

#配置channel
a1.channels.c1.type = org.apache.flume.channel.kafka.KafkaChannel
a1.channels.c1.kafka.bootstrap.servers = bigdata1:9092,bigdata2:9092,bigdata3:9092
a1.channels.c1.kafka.topic = app_log
# 是否组装为事件（header + body）
a1.channels.c1.parseAsFlumeEvent = false

# 断点续传有序
a1.channels.c1.transactionCapacity = 1
# # 控制每个事务中最多传输的小文件数量，以避免出现过多小文件导致的性能问题
# #agent.sinks.sink1.batchSize

#组装
a1.sources.r1.channels = c1
```

- 开启Flume进程

```bash
#控制台运行
flume-ng agent -n a1 -c /home/TomFor/software/flume/conf/ -f /home/TomFor/software/flume/job/file_to_kafka.conf -Dflume.root.logger=info,console

#后台运行
nohup flume-ng agent -n a1 -c /home/TomFor/software/flume/conf/ -f /home/TomFor/software/flume/job/file_to_kafka.conf >/dev/null 2>&1 &
```

- 开启一个Kafka控制台消费者进程，当日志有新增数据时，控制台上是否有数据显示。若有数据显示，则Flume采集日志文件到Kafka完成


### 3. Flume采集Kafka数据到HDFS


### 4. Flume采集日志文件到HDFS

- 编写conf-file文件**flume-file-hdfs.conf**

```properties
# 定义组件
a2.sources = r2
a2.sinks = k2
a2.channels = c2

# 配置Source
a2.sources.r2.type = exec
a2.sources.r2.command = tail -F /opt/module/hive-2.3.3/logs/hive.log
a2.sources.r2.shell = /bin/bash -c

# 配置Sink
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

# 配置Channel
a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100

# 组装
a2.sources.r2.channels = c2
a2.sinks.k2.channel = c2
```

- 开启Flume进程

```bash
#控制台运行
flume-ng agent -n a2 -c /home/TomFor/software/flume/conf/ -f /home/TomFor/software/flume/job/flume-file-hdfs.conf -Dflume.root.logger=info,console
#后台运行
nohup flume-ng agent -n a2 -c /home/TomFor/software/flume/conf/ -f /home/TomFor/software/flume/job/flume-file-hdfs.conf >/dev/null 2>&1 &
```

- 检查日志有新增数据时，HDFS上是否出现新文件。若有新文件产生，则Flume采集日志文件到HDFS完成