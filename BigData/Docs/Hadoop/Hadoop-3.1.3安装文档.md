---
layout: post
title: Hadoop-3.1.3安装文档
date: 2020-06-17 08:50:21
tags: [Hadoop]
categories: [Bigdata,Hadoop]
excerpt: 。。。
---

# 前言

# 前提条件

1. 免密登录
2. 关闭防⽕墙
3. JDK的安装
4. Zookeeper的安装

>注：搭配的Spark版本是2.3.4

# 角色分配


| 机器名称         | bigdata001  | bigdata002  |  bigdata003 |
|:-:              |     :-:     |:-:          |:-:          |
|QuorumPeerMain   |       √     |  √          | √           |
| NameNode        |  √          |      √        |             |
| DataNode        |  √          |        √      |√              |
|  JournalNode    | √           |          √    |  √            |
| ResourceManager |  √          |           √   |             |
|  NodeManager    | √           |             √ |    √          |
| ZKFC            |  √          |         √     |             |


# 下载

## 1. 浏览器下载

- [**Hadoop官网下载**](https://archive.apache.org/dist/hadoop/common)

- [**国内镜像下载**](https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common)

## 2. 服务器下载

```bash
#官网镜像
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.1.3/hadoop-3.1.3.tar.gz

#国内镜像
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-3.1.3/hadoop-3.1.3.tar.gz
```

# 配置

## 1. 修改配置文件：

- apache-hadoop-3.1.3/etc/hadoop/core-site.xml

```xml
<configuration>
    <!-- 指定hdfs地址，高可用模式中是连接到nameservice -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://ns1</value>
    </property>

    <!-- 这里的路径默认是NameNode、DataNode、JournalNode等存放数据的公共目录，也可以单独指定 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/home/xuanfu01/software/hadoop/data/dfs</value>
    </property>

    <!-- 指定ZooKeeper集群的地址和端口。注意，数量一定是奇数，且不少于三个节点 -->
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>bigdata001:2181,bigdata002:2181,bigdata003:2181</value>
    </property>

	<!-- 代理用户 -->
	<property>
    	<name>hadoop.proxyuser.root.hosts</name>
    	<value>*</value>
	</property>
    
	<property>
    	<name>hadoop.proxyuser.CharlieTao.hosts</name>
    	<value>*</value>
	</property>
    
	<property>
    	<name>hadoop.proxyuser.CharlieTao.groups</name>
    	<value>*</value>
	</property>
</configuration>
```

- apache-hadoop-3.1.3/etc/hadoop/hadoop-env.sh

```bash
export JAVA_HOME=/home/xuanfu01/software/jdk
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_ZKFC_USER=root
export HDFS_JOURNALNODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_NODEMANAGER_USER=root
export YARN_RESOURCEMANAGER_USER=root
```

- apache-hadoop-3.1.3/etc/hadoop/hdfs-site.xml

```xml
<configuration>
    <!-- 指定副本数，不能超过机器节点数 -->
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>

    <!-- 为Namenode集群定义一个ServicesName -->
    <property>
        <name>dfs.nameservices</name>
        <value>ns1</value>
    </property>

    <!-- ServicesName包含哪些NameNode -->
    <property>
        <name>dfs.ha.namenodes.ns1</name>
        <value>nn1,nn2</value>
    </property>

    <!-- 名为nn1的NameNode的rpc地址和端口号，rpc用来和DataNode通讯 -->
    <property>
        <name>dfs.namenode.rpc-address.ns1.nn1</name>
        <value>bigdata001:9000</value>
    </property>

    <!-- 名为nn2的NameNode的rpc地址和端口号，rpc用来和DataNode通讯 -->
    <property>
        <name>dfs.namenode.rpc-address.ns1.nn2</name>
        <value>bigdata002:9000</value>
    </property>

    <!--名为nn1的NameNode的http地址和端口号，用来和web客户端通讯 -->
    <property>
        <name>dfs.namenode.http-address.ns1.nn1</name>
        <value>bigdata001:50070</value>
    </property>

    <!-- 名为nn2的NameNode的http地址和端口号，用来和web客户端通讯 -->
    <property>
        <name>dfs.namenode.http-address.ns1.nn2</name>
        <value>bigdata002:50070</value>
    </property>
        
    <!-- NameNode间用于共享编辑日志的Journal节点列表 -->
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
        <value>qjournal://bigdata001:8485;bigdata002:8485;bigdata003:8485/ns1</value>
    </property>

    <!-- 指定该集群出现故障时，是否自动切换到另一台NameNode -->
    <property>
        <name>dfs.ha.automatic-failover.enabled</name>
        <value>true</value>
    </property>

    <!-- Journalnode上用于存放edits日志的目录 -->
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>/home/xuanfu01/software/hadoop/data/dfs/journalnode</value>
    </property>

    <!--指定NameNode Dir的地址 -->
    <property>
        <name>dfs.name.dir</name>
        <value>/home/xuanfu01/software/hadoop/data/dfs/namenode</value>
    </property>

    <!--指定Datanode Dir的地址 -->
    <property>
        <name>dfs.data.dir</name>
        <value>/home/xuanfu01/software/hadoop/data/dfs/datanode</value>
    </property>

    <!-- 客户端连接可用状态的NameNode所用的代理类 -->
    <property>
        <name>dfs.client.failover.proxy.provider.ns1</name>
        <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>
    
    <!-- 一旦需要NameNode切换，使用ssh方式进行操作 -->
    <property>
        <name>dfs.ha.fencing.methods</name>
        <value>sshfence</value>
    </property>

    <!-- 如果使用ssh进行故障切换，使用ssh通信时用的密钥存储的位置 -->
    <property>
        <name>dfs.ha.fencing.ssh.private-key-files</name>
        <value>/home/xuanfu01/.ssh/id_rsa</value>
    </property>

    <!-- connect-timeout超时时间 -->
    <property>
        <name>dfs.ha.fencing.ssh.connect-timeout</name>
        <value>30000</value>
    </property>

    <!--开启Web HDFS -->
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
</configuration>
```

- apache-hadoop-3.1.3/etc/hadoop/mapred-site.xml

```xml
<configuration>
    <!-- 使用Yarn作为资源调度 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

- apache-hadoop-3.1.3/etc/hadoop/yarn-site.xml

```xml
<configuration>
    <!-- 启用HA高可用性 -->
    <property>
        <name>yarn.resourcemanager.ha.enabled</name>
        <value>true</value>
    </property>

    <!-- 指定ResourceManager的名字 -->
    <property>
        <name>yarn.resourcemanager.cluster-id</name>
        <value>yrc</value>
    </property>

    <!-- 使用了2个ResourceManager,分别指定ResourceManager的地址 -->
    <property>
        <name>yarn.resourcemanager.ha.rm-ids</name>
        <value>rm1,rm2</value>
    </property>

    <!-- 指定rm1的地址 -->
    <property>
        <name>yarn.resourcemanager.hostname.rm1</name>
        <value>bigdata001</value>
    </property>

    <!-- 指定rm2的地址 -->
    <property>
        <name>yarn.resourcemanager.hostname.rm2</name>
        <value>bigdata002</value>
    </property>

    <!-- 指定当前机器bigdata001作为rm1 -->
    <property>
        <name>yarn.resourcemanager.ha.id</name>
        <value>rm1</value>
    </property>

    <!-- 指定Zookeeper集群机器 -->
    <property>
        <name>yarn.resourcemanager.zk-address</name>
        <value>bigdata001:2181,bigdata002:2181,bigdata003:2181</value>
    </property>

    <!-- NodeManager上运行的附属服务，默认是mapreduce_shuffle -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>

    <!-- Hadoop ClassPath（执行hadoop classpath的结果） -->
    <property>
        <name>yarn.application.classpath</name>
        <value>
            /home/CharlieTao/software/hadoop/etc/hadoop:
            /home/CharlieTao/software/hadoop/share/hadoop/common/lib/*:
            /home/CharlieTao/software/hadoop/share/hadoop/common/*:
            /home/CharlieTao/software/hadoop/share/hadoop/hdfs:
            /home/CharlieTao/software/hadoop/share/hadoop/hdfs/lib/*:
            /home/CharlieTao/software/hadoop/share/hadoop/hdfs/*:
            /home/CharlieTao/software/hadoop/share/hadoop/mapreduce/lib/*:
            /home/CharlieTao/software/hadoop/share/hadoop/mapreduce/*:
            /home/CharlieTao/software/hadoop/share/hadoop/yarn:
            /home/CharlieTao/software/hadoop/share/hadoop/yarn/lib/*:
            /home/CharlieTao/software/hadoop/share/hadoop/yarn/*
        </value>
    </property>

    <!-- 客户端通过该地址向RM提交对应用程序操作 -->
    <property>
        <name>yarn.resourcemanager.address.rm1</name>
        <value>bigdata001:8032</value>
    </property>

    <!--ResourceManager对ApplicationMaster暴露的访问地址。ApplicationMaster通过该地址向RM申请资源释放资源等。 -->
    <property>
        <name>yarn.resourcemanager.scheduler.address.rm1</name>
        <value>bigdata001:8030</value>
    </property>

    <!-- RM WebUI访问地址,查看集群信息-->
    <property>
        <name>yarn.resourcemanager.webapp.address.rm1</name>
        <value>bigdata001:8088</value>
    </property>

    <!-- NodeManager通过该地址交换信息 -->
    <property>
        <name>yarn.resourcemanager.resource-tracker.address.rm1</name>
        <value>bigdata001:8031</value>
    </property>

    <!--管理员通过该地址向RM发送管理命令 -->
    <property>
        <name>yarn.resourcemanager.admin.address.rm1</name>
        <value>bigdata001:8033</value>
    </property>

    <property>
        <name>yarn.resourcemanager.ha.admin.address.rm1</name>
        <value>bigdata001:23142</value>
    </property>

    <property>
        <name>yarn.resourcemanager.ha.admin.address.rm2</name>
        <value>bigdata002:23142</value>
    </property>
    
    <property>
        <name>yarn.resourcemanager.address.rm2</name>
        <value>bigdata002:8032</value>
    </property>

    <property>
        <name>yarn.resourcemanager.scheduler.address.rm2</name>
        <value>bigdata002:8030</value>
    </property>

    <!-- 设置Yarn WebUI页面访问端口 -->
    <property>
        <name>yarn.resourcemanager.webapp.address.rm2</name>
        <value>bigdata002:8088</value>
    </property>

    <property>
        <name>yarn.resourcemanager.resource-tracker.address.rm2</name>
        <value>bigdata002:8031</value>
    </property>

    <property>
        <name>yarn.resourcemanager.admin.address.rm2</name>
        <value>bigdata002:8033</value>
    </property>
    
    <!-- 设置Yarn可调度的最大内存，防止执行Hive时MR跑不起来（java.io.InterruptedIOException） -->
	<property>
        <name>yarn.scheduler.maximum-allocation-mb</name>
        <value>30720</value>
    </property>
</configuration>
```

- apache-hadoop-3.1.3/etc/hadoop/workers

```
bigdata001
bigdata002
bigdata003
```

- apache-hadoop-3.1.3/etc/hadoop/hadoop-evn.sh

```BASH
#优雅的关闭NodeManager(默认HADOOP的PID和YARN的PID时存放在Linux的TEMP目录下，当时间Hadoop集群运行时间较长时，TEMP目录下的文件会被系统清理掉。因此使用stop-all.sh关闭集群时会找不到PID，从而报出nodemanager did not stop gracefully after 5 seconds的错误，所以在这里我们自定义PID的地址即可解决)
export HADOOP_PID_DIR=${HADOOP_HOME}/pid
#YARN_PID_DIR无需指定，与HADOOP_PID_DIR地址一致
```



## 2. 拷贝

- 将bigdata001的Hadoop⽂件夹拷⻉⾄其余机器上


```bash
 scp -r apache-hadoop-3.1.3/ xuanfu01@bigdata002:/home/xuanfu01/software/
 scp -r apache-hadoop-3.1.3/ xuanfu01@bigdata003:/home/xuanfu01/software/
```

- 在bigdata002服务器上的 yarn-site.xml 配置⽂件中做出如下修改

```xml
<property>
    <name>yarn.resourcemanager.ha.id</name>
    <value>rm2</value>
</property>
```

- 在bigdata003上删除上述配置项



## 3. 配置环境变量

> 在此不做过多赘述，以下为本⼈服务器的所有环境变量（/home/xuanfu01/.bashrc）

```bash
# .bashrc
# Source global definitions
if [ -f /etc/bashrc ]; then
. /etc/bashrc
fi
# User specific environment
export
PATH=$PATH:$JAVA_HOME/bin:$SCALA_HOME/bin:$ZOOKEEPER_HOME/bin:$HADOOP_HOM
E/bin:$HADOOP_HOME/sbin:$KAFKA_HOME/bin:$SPARK_HOME/bin:$FLUME_HOME/bin:$
HBASE_HOME/bin:
# Uncomment the following line if you don't like systemctl's auto-paging
feature:
# export SYSTEMD_PAGER=
# User specific aliases and functions
export JAVA_HOME=/home/xuanfu01/software/jdk
export SCALA_HOME=/home/xuanfu01/software/scala
export ZOOKEEPER_HOME=/home/xuanfu01/software/zookeeper
export HADOOP_HOME=/home/xuanfu01/software/hadoop
export KAFKA_HOME=/home/xuanfu01/software/kafka
export SPARK_HOME=/home/xuanfu01/software/spark
export FLUME_HOME=/home/xuanfu01/software/flume
export HBASE_HOME=/home/xuanfu01/software/hbase
```

<!--注：添加完环境变量之后记得source-->



## 4. 初始化及启动流程

<!--注：⾄此，Hadoop⾼可⽤集群已经安装完毕。以下将进⾏初始化操作，必须按照顺序进⾏！-->

### (1) 初始化HDFS

#### ① 启动JournalNode(三台都要启动)

```bash
hadoop-daemon.sh start journalnode
```

<!--注：JournalNode之所以需要三台都启动，是因为JournalNode的作用在于同步两台NameNode的元数据，而为了保证JournalNode自身的高可用特性，通常数量设置为（2N-1），其中N表示NameNode的数量，具体在这里不做赘述，后续文章会说明JournalNode及Hadoop其他进程的作用。因此要搭建高可用的Hdaoop集群，则JournalNode进程的最小个数为3-->

#### ② 格式化NameNode（bigdata001执行）

```bash
hdfs namenode -format
```

#### ③ 同步NameNode的元数据（bigdata002执行）

```bash
hdfs namenode -bootstrapStandby
```

#### ④ 格式化ZKFC（bigdata001）

```bash
hdfs zkfc -formatZK
```

注：至此，HDFS的格式化完毕。现将现有HDFS进程关闭，再重新启动Hadoop集群

#### ⑤ 关闭现有HDFS进程（bigdata001）

```bash
stop-dfs.sh
```



### (2) 启动HDFS（bigdata001）

```bash
startp-dfs.sh
```

<!--注：ZKFC与JournalNode进程无需单独启动，包含在startp-dfs.sh脚本里面-->



### (3) 检验HDFS是否启动成功

1. 在浏览器中分别输⼊ bigdata001:50070 、 bigdata002:50070 检查主从NameNode是否 为正常状态。如下所⽰：

ActiveNameNode
![bigdata001:50070](https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/Hadoop/ActiveNameNode.png?raw=true)

StandbyNameNode
![bigdata002:50070](https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/Hadoop/StandbyNameNode.png?raw=true)

2. ⾼可⽤测试（将bigdata001上的NameNode进程杀死，在Web⻚⾯上观察bigdata002的NameNode是否可以自行变更为Active状态）
3. 重新启动bigdata001上NameNode进程

```bash
hdfs --daemon start namenode
```



### (4) 启动Yarn（bigdata001）

```bash
start-yarn.sh
```

1. 在浏览器中输入bigdata002:8088，页面如下所示：

ResourceManager

![bigdata002:8088](https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/Hadoop/ResourceManager.png?raw=true)

2. ResourceManager不存在Active、StandBy一说，只能访问活动的那台服务器，如果你访问bigdata001:8088，则会自动跳转到bigdata002:8088页面上

<!--注：-->

​		<!--1. Yarn无需格式化，直接启动即可-->

​		<!--2. 这里采用的是逐一启动HDFS与Yarn服务，Hadoop支持一键启动：start-all.sh、一键关闭：stop-all.sh-->



## 5. 疑难解答

1. 在搭建集群过程中曾出现Active状态NameNode与StandBy状态的NameNode无法自动切换的现象
   - 日志：查看ZKFC的日志，发现日志中报错（bash: fuser: 未找到命令）
   - 解决方法：sudo yum install psmisc，关闭所有Hadoop进程，重新启动即可