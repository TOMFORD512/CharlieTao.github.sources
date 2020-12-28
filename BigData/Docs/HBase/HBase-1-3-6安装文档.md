---
layout: photo
title: HBase-1.3.6安装文档
date: 2020-06-07 22:09:46
tags: [HBase]
categories: [Bigdata,HBase]
password: 5201314
---

# 前提条件
1. Zookeeper的安装
2. Hadoop的安装
3. JDK的安装

>注：
>1. 虽然HBase内部存在Zookeeper，但是业内不推荐这种做法。建议使用外部自行安装的Zookeeper进行管理，以便好的集成大数据其他组件。
>2. HBase与Hive相似，数据存储在HDFS上，因此需要提前搭建好Hadoop

<!-- more -->

# 角色分类
|  机器名  | bigdata001  |  bigdata002 | bigdata003  |
|:-:|:-:|:-:|:-:|
| QuorumPeerMain  |  √ | √  | √  |
| HMaster  |   | √(Backup   )  | √(Active)  |
| HRegionServer  | √  |  √ |  √ |


# 下载

## 1. 自行下载源码，编译（Git）
## 2. 浏览器下载

- [HBase官网下载](https://apache.website-solution.net/hbase/hbase-1.3.6/hbase-1.3.6-bin.tar.gz)

- [国内镜像下载](https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/hbase-1.3.6/hbase-1.3.6-bin.tar.gz)

## 3. 服务器下载

```bash
#官网镜像
wget https://apache.website-solution.net/hbase/hbase-1.3.6/hbase-1.3.6-bin.tar.gz

#国内镜像
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hbase/hbase-1.3.6/hbase-1.3.6-bin.tar.gz
```

# 配置

- 在conf/hbase-site.xml⽂件中添加以下内容

```xml
<configuration>
    <!-- 开启集群模式 -->
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>

    <!-- 元数据在HDFS上的地址 -->
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://bigdata1:9000/hbase</value>
    </property>

    <!-- Zookeeper连接地址 -->
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>bigdata1,bigdata2,bigdata3</value>
    </property>

    <!-- Zookeeper元数据地址 -->
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/home/worker/software/zookeeper/zkData</value>
    </property>

    <!-- 文件存放地址 -->
    <property>
        <name>hbase.tmp.dir</name>
        <value>/home/worker/software/hbase/tmp</value>
    </property>
</configuration>
```

- 更改conf/ hbase-env.sh ⽂件

```bash
#Java环境变量：
export JAVA_HOME=/home/worker/software/jdk

#禁用HBASE内置Zookeeper：
export HBASE_MANAGES_ZK=false

#HBASE PID地址（若果不添加以下内容，启动和停止HBASE时会自动去tmp目录下寻找改文件，报错）
export HBASE_PID_DIR=/home/worker/software/hbase/conf/hadoop/pids
```

- 更改conf/regionservers⽂件

```properties
#在该文件中添加你的集群中即将充当HRegionserver角色的机器名
bigdata1
bigdata2
bigdata3
```

- 创建backup-masters⽂件

```properties
#在改文件中添加你的集群中即将充当backup HMaster角色的机器名
bigdata2
```

- 分发HBASE整个⽬录到各个机器上（scp）
- 配置HBASE环境变量
>环境变量的配置较为简单，这里不做过多赘述。集群的整个环境变量配置在Hadoop集群搭建中已给出。

# 启动与关闭

- 在被认定为HMaster的机器上执⾏以下命令

```bash
#启动
start-hbase.sh

#关闭
stop-hbase.sh
```

# 检验
浏览器中输⼊http://bigdata3:16010、http://bigdata2:16010查看Master以及Backup Master
>注：HBASE默认Web UI端⼝号为16010，可在配置⽂件中更改。若显⽰正常，则安装完成。

HMaster页面如下：
![](https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/HBase/HBASEMaster.png?raw=true)

Backup HMaster⻚⾯如下：
![](https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/HBase/BackupMaster.png?raw=true)