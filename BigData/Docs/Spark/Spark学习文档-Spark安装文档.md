---
title: Spark学习文档--Spark安装文档
date: 2020-12-25 23:53:08
tags: [Spark]
categories: [Bigdata,Spark]
excerpt: 。。。
---

## 写在最前
>- Spark版本： 3.0.3
>- Hadoop版本： 3.1.3
>- Scala版本： 2.11.12

## 下载及解压
```bash
tar -zxvf spark-3.0.1-bin-hadoop3.2
```

## 配置

### 1.配置spark-env.sh
```bash
cp spark-env.sh.template spark-env.sh

# jdk地址
JAVA_HOME=/data/java/jdk1.8.0_11
# hadoop安装的完的etc目录,hadoop我安装的是3.1.2版本的
HADOOP_CONF_DIR=/data/hadoop/hadoop-3.1.2/etc/hadoop
# zookeeper信息配置，知道所在节点地址信息
SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=bigdata1:2181,bigdata2:2181,bigdata3:2181 -Dspark.deploy.zookeeper.dir=/spark3.0"
```
### 2.配置slaves

```bash
cp slaves.template slaves

# 在slaves中添加自己的集群主机名,把里面的localhost删掉。
# 以下配置的是worker节点的Host，Spark集群中不需要配置Master的Host
bigdata1
bigdata2
bigdata3
```

### 3.拷贝至其他Worker节点
```bash
scp -r software/spark bigdata2:/home/TomFor/software/
scp -r software/spark bigdata3:/home/TomFor/software/
# 或者写一个简单的Shell命令，通过for循环执行也可
for A in {2..3}; do scp -r software/spark bigdata$A:/home/TomFor/software/;done
```

### 4.配置Spark On Yarn
在Hadoop集群的`yarn-site.xml`中添加以下配置
```properties
<property>
  <name>yarn.nodemanager.pmem-check-enabled</name>
  <value>false</value>
</property>
<property>
   <name>yarn.nodemanager.vmem-check-enabled</name>
   <value>false</value>
</property>
```

### 5.配置历史服务器
```bash
# 配置历史服务器的原因在于： 可以查看整个Spark任务的执行情况，是否出现数据倾斜等；否则当Spark任务执行完了之后就无法查看了
# 修改spark default.conf文件，配置日志存储路径

# 开启eventLog功能
spark.eventLog.enabled true
# 指定历史日志文件存放地址，这里是配置的HDFS，也是推荐配置。因为放在本地不利于多集群间的共享
spark.eventLog.dir hdfs://ns1:8020/spark-history-server
# 这里主要是指定启动historyServer的机器的Host，也可以指定一个或多个
spark.yarn.historyServer.address= bigdata3:18080
# WebUI
spark.history.ui.port=18080
```
>注意：
>- 配置完之后需要启动或者重启Hadoop集群，并且需要提前创建eventLog.dir：hadoop fs -mkdir /directory

```bash
# 修改spark-env.sh文件 , 添加日志配置
export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080 -Dspark.history.fs.logDirectory=hdfs://ns1:8020/spark-history-server -Dspark.history.retainedApplications=30"
```

## 启动

### 1.启动Spark集群
>由于配置了Spark On Yarn，所以这里要先启动Hadoop集群

```bash
# 在决定好的机器上执行以下命令，即可在本机上启动一个Master进程，以及在配置好节点上启动的Worker进程
# 因为Spark和Hadoop的启动命令都是start-all.sh，因此这里可以改成start-all-spark.sh
start-all-spark.sh
# 通常需要再启动一个stand by状态的Master进程，保证高可用。所以在另外的一台机器上执行以下命令
start-master.sh
```

### 2. 启动Spark的历史服务进程
```bash
# 因为之前配置了启动historyServer的机器的Host（spark.yarn.historyServer.address），所以这里在Bigdata3上执行以下命令
start-history-server.sh
```

## 测试
### 1.测试Master的高可用
### 2.测试history-server