---
layout: photo
title: Hive-3.1.2安装文档
date: 2023-07-29 16:53:09
tags: [Hive]
categories: [Bigdata,Hive]
excerpt: 。。。
---

### 一、写在最前
---
> CentOS版本： CentOS Linux release 7.9.2009 (Core)
> jdk版本： 1.8
> Hadoop版本： 3.1.3



### 二、下载
---
#### 1. 官网下载
> 源码也是在这个连接下面下载

- [Apache Hive官网下载](https://mirror-hk.koddos.net/apache/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz)
- [Apache Hive国内镜像下载](https://mirrors.tuna.tsinghua.edu.cn/apache/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz)

#### 2. 服务器下载

```bash
#官网下载
wget https://mirror-hk.koddos.net/apache/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz

#国内镜像下载
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz
```


### 三、配置

#### 1. 前提条件：[MySQL安装](https://charlietao.github.io/2023/08/01/MySQL8安装文档/)
> 虽然Hive内置数据库管理元数据，但是一般都是单独配置到MySQL中，方便管理

#### 2. 拷贝JDBC驱动到Hive的lib目录下

```bash
# 这里安装的MySQL版本是5，所以JDBC驱动也需要对应，如果是mysql8的话同理，驱动下载参考MySQL安装一文
cp mysql-connector-java-5.1.46.jar $HVIE/HOME/lib/
```

#### 3. 配置hive-site.xml文件

```xml
<configuration>
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://bigdata2:3306/hive_metastore?createDatabaseIfNotExist=true&amp;useUnicode=true&amp;characterEncoding=UTF-8</value>
    </property>
    
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>
    
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>TomFor</value>
    </property>
    
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>tomfor</value>
    </property>
    
    <!-- <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/user/hive/warehouse</value>
    </property>
    
    <property>
        <name>hive.metastore.schema.verification</name>
        <value>false</value>
    </property>
    
    <property>
    <name>hive.server2.thrift.port</name>
    <value>10000</value>
    </property>

    <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>hadoop102</value>
    </property>

    <property>
        <name>hive.metastore.event.db.notification.api.auth</name>
        <value>false</value>
    </property>
    
    <property>
        <name>hive.cli.print.header</name>
        <value>true</value>
    </property>

    <property>
        <name>hive.cli.print.current.db</name>
        <value>true</value>
    </property> -->
</configuration>
```

#### 4. 配置hive-log4j2.properties
> 按需配置，目的是做好日志管理

#### 5. 创建Hive元数据库

```sql
-- 这里的数据库名称应该要和hive-site.xml数据库链接地址中的数据库名称保持一致
CREATE DATABASE hive_metastore;
```

#### 6. 创建外部访问元数据的用户并授权
> 外部访问是指：DBeaver、Java等

```SQL
-- 创建用户
CREATE USER 'TomFor'@'%' IDENTIFIED BY 'tomfor';

-- 授予用户相应的权限。
GRANT ALL PRIVILEGES ON *.* TO 'TomFor'@'%';

-- 刷新权限
FLUSH PRIVILEGES;

-- 注意这里的用户最好和Hadoop的用户保持一致
-- 因为如果想要使用DBeaver连接Hive的时候，如果你要操作HDFS上的文件，并且连接Hive的用户和Hadoop的用户不一样，会有问题。
-- 解决方法：
--   （1）把DHFS路径的权限改掉
--   （2）用户名与Hadoop安装用户保持一致
-- 使用root用户连接Hive Metastore: GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
```

#### 7. 初始化Hive元数据库

```bash
schematool -initSchema -dbType mysql -verbose
```

#### 8. 修改元数据库字符集
>发现乱码时再改也可

```sql
-- 字段注释
ALTER TABLE COLUMNS_V2 modify column COMMENT VARCHAR(256) character set utf8;

-- 表注释
ALTER TABLE TABLE_PARAMS modify column PARAM_VALUE mediumtext character set utf8;
```

#### 9. 添加环境变量$HIVE_HOME

#### 10. 其他
- 若出现日志Jar包冲突

```bash
mv $HIVE_HOME/lib/log4j-slf4j-impl-2.10.0.jar log4j-slf4j-impl-2.10.0.jar.bak
```



### 四、运行

#### 1. 测试

```bash
# 启动Hive客户端
hive
```
```sql
-- 查看Hive库、表
hive (default)> show databases;
OK
default
```

#### 2. 使用DBeaver连接Hive

```bash
# 开启元数据服务
hive --service metastore
nohup hive --service metastore >/dev/null 2>&1 &

# 开启hiveserver2
hive --service hiveserver2
nohup hive --service hiveserver2 >/dev/null 2>&1 &

# 日志管理
hive --service metastore --hiveconf hive.root.logger=INFO,DRFA --hiveconf hive.log.dir=/home/TomFor/log/hive/metastore --hiveconf hive.log.file=hive.log.<date>
hive --service metastore --hiveconf hive.log4j.file
```



### 五、Hive相关服务启动脚本

```bash
#! /bin/bash

# ================Hive启动相关服务的脚本=======================

METASTORE_LOG_DIR="/home/TomFor/log/hive/metastore"
METASTORE_LOG_FILE="hive_metastore_operate_$(date +'%Y-%m-%d').log"

HIVESERVER2_LOG_DIR="/home/TomFor/log/hive/metastore"
HIVESERVER2_LOG_FILE="hive_server2_query_$(date +'%Y-%m-%d').log"

HIVE_HOST="bigdata2"
HIVE_USER="TomFor"

# 信息日志
function log_info(){
  content="[INFO] $(date '+%Y-%m-%d %H:%M:%S') $@"
  echo -e "\033[32m"${content} "\033[0m"
}

# 警告日志
function log_warn(){
  content="[WARN] $(date '+%Y-%m-%d %H:%M:%S') $@"
  echo -e "\033[33m"${content} "\033[0m"
}

# 错误日志
function log_error(){
  content="[ERROR] $(date '+%Y-%m-%d %H:%M:%S') $@"
  #[ $LOG_LEVEL -le 3  ] && echo -e "\033[31m" ${content} "\033[0m"
  echo -e "\033[31m"${content} "\033[0m"
}

# 检测文件夹或文件是否存在
check_folder_or_file_exists() {
    if [[ $2 == "f" ]]; then
        f_flag=$(ssh $1 "test -f $3 && echo \"1\" || echo \"0\"")
        if [[ $f_flag == "1" ]]; then
            echo "1"
        elif [[ $f_flag == "0" ]]; then
            echo "0"
        fi
    elif [[ $2 == "d" ]]; then
        d_flag=$(ssh $1 "test -d $3 && echo \"1\" || echo \"0\"")
        if [[ $d_flag == "1" ]]; then
            echo "1"
        elif [[ $d_flag == "0" ]]; then
            echo "0"
        fi
    fi
}


# 程序主体
log_info "欢迎使用Hive启停服务脚本"
echo -e "\033[32m"请输入你要执行的操作： "\033[0m"

operate_actions=("开启metastore服务" "开始hiveserver2服务" "同时开启metastore、hiveserver2服务" "关闭metastore服务" "关闭hiveserver2服务" "同时关闭metastore、hiveserver2服务" "退出")
select action in "${operate_actions[@]}";
do
    case $action in
        "开启metastore服务" ) {
            result_floder=$(check_folder_or_file_exists "$HIVE_HOST" "d" "${METASTORE_LOG_DIR}")
            result_file=$(check_folder_or_file_exists "$HIVE_HOST" "f" "${METASTORE_LOG_DIR}/${METASTORE_LOG_FILE}")
            if [[ $result_floder == "1" ]]; then
              if [[ $result_file == "1" ]]; then
                  flag=$(ssh $HIVE_HOST "ps -ef | grep org.apache.hadoop.hive.metastore.HiveMetaStore |grep -v grep | awk '{print \$2}'")
                  if [[ $flag == "" ]]; then
                      echo "正在开启开启metastore服务..."
                      ssh $HIVE_HOST "nohup hive --service metastore > "${METASTORE_LOG_DIR}/${METASTORE_LOG_FILE}" 2>&1 &"
                      echo -e "\033[33m"hive metastore服务已经启动! "\033[0m"
                  else
                      echo -e "\033[33m"hive metastore服务已经在运行了! "\033[0m"
                  fi
              else
                  echo -e "\033[32m"检测到日期为今天的日志文件不存在，正在为您创建..."\033[0m"
                  ssh $HIVE_HOST "touch ${METASTORE_LOG_DIR}/${METASTORE_LOG_FILE}"
                  echo "正在启动metastore..."
                  ssh $HIVE_HOST "nohup hive --service metastore > "${METASTORE_LOG_DIR}/${METASTORE_LOG_FILE}" 2>&1 &"
                  echo -e "\033[33m"hive metastore服务已经启动! "\033[0m"
              fi
            else
              echo -e "\033[32m"检测到存放日志文件的路径不存在，正在为您创建路径以及日志文件..."\033[0m"
              ssh $HIVE_HOST "mkdir -p ${METASTORE_LOG_DIR}"
              echo -e "\033[32m"日志路径已创建完毕"\033[0m"
              ssh $HIVE_HOST "touch ${METASTORE_LOG_DIR}/${METASTORE_LOG_FILE}"
              echo -e "\033[32m"日志文件已创建完毕"\033[0m"
              echo "正在启动metastore..."
              ssh $HIVE_HOST "nohup hive --service metastore > "${METASTORE_LOG_DIR}/${METASTORE_LOG_FILE}" 2>&1 &"
              echo -e "\033[33m"hive metastore服务已经启动! "\033[0m"
            fi
        };; 
        "开始hiveserver2服务" ) {
            echo "开始hiveserver2服务"
        };;
        "同时开启metastore、hiveserver2服务" ) {
            echo "同时开启metastore、hiveserver2服务"
        };;
        "关闭metastore服务" ) {
            echo "正在关闭metastore服务..."
            flag=$(ssh $HIVE_HOST "ps -ef | grep org.apache.hadoop.hive.metastore.HiveMetaStore |grep -v grep | awk '{print \$2}'")
            if [[ $flag == "" ]]; then
                echo -e "\033[33m"metastore服务未在运行中，请确认后再执行： "\033[0m"
            else
                ssh $HIVE_HOST "kill -9 $flag"
                echo -e "\033[32m"metastore服务已经关闭"\033[0m"
            fi
        };;
        "关闭hiveserver2服务" ) {
            echo "关闭hiveserver2服务"
        };;
        "同时关闭metastore、hiveserver2服务" ) {
            echo "同时关闭metastore、hiveserver2服务"
        };;
        "退出" ) {
            echo "Bye Bye"
            break
        };;
        * ) {
            echo "\033[31m"Invalid option: $REPLY"\033[0m"
        };;
    esac
done

# 启动hiveserver2
# nohup hive --service metastore > "${HIVESERVER2_LOG_DIR}/${HIVESERVER2_LOG_FILE}" 2>&1 &

# LOG_DIR="/path/to/logs"
# LOG_FILE="hive.log"

# Get yesterday's date in YYYY-MM-DD format
# YESTERDAY=$(date -d "yesterday" +"%Y-%m-%d")

# Rename yesterday's log file with date suffix
# mv "${LOG_DIR}/${LOG_FILE}" "${LOG_DIR}/${LOG_FILE}.${YESTERDAY}"

# Create a new log file
# touch "${LOG_DIR}/${LOG_FILE}"

# Start Hive Metastore service
# nohup hive --service metastore > "${LOG_DIR}/${LOG_FILE}" 2>&1 &
```