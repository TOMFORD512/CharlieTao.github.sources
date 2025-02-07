---
layout: photo
title: MySQL8安装文档
date: 2023-08-01 21:37:42
tags: [MySQL]
categories: [数据库,MySQL]
excerpt: 。。。
---

### 一、写在最前
---
> CentOS版本： CentOS Linux release 7.9.2009 (Core)
> MySQL版本： mysql80-community-release-el7-3

#### MySQL各组件介绍
- mysql-community-client-5.7.18-1.el7.x86_64.rpm 客户端
- mysql-community-devel-5.7.18-1.el7.x86_64.rpm 开发库
- mysql-community-embedded-5.7.18-1.el7.x86_64.rpm 嵌入式
- mysql-community-server-5.7.18-1.el7.x86_64.rpm 服务端
- mysql-community-libs-5.7.18-1.el7.x86_64.rpm 共享库
- mysql-community-test-5.7.18-1.el7.x86_64.rpm 测试组件



### 二、卸载MySQL
---

```bash
# 检查是否已经安装过mysql：
rpm -qa | grep mysql

# 如果环境中有遗留mysql则执行删除命令：
rpm -e --nodeps mysql-xxxxxxxxx

# 查询遗留的mysql设置或命令，执行两条命令：
whereis mysql
find / -name mysql

# 如通过上述两条命令发现有遗留，则执行清除命令，将所有查到的mysql都删除干净：
rm -rf  xxx xxx
```

### 三、安装
---

#### 1. RPM包安装

```bash
# 1.安装RPM包
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm

# 2.安装依赖
yum -y install mysql80-community-release-el7-3.noarch.rpm

# 3.安装MySQL
yum -y install mysql-community-server

# 如果安装server的时候提示未安装公钥，则需要手动导入
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022

# 检查RPM包的数字签名是否有效和可信
# rpm --checksig  /var/cache/yum/x86_64/7/mysql80-community/packages/mysql-community-client-plugins-8.0.33-1.el7.x86_64.rpm
```

![MySQL安装目录](https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/MySQL/MySQL安装目录.png?raw=true)

#### 2. tar.gz包安装


### 四、启动
---

```bash
# 启动
sudo systemctl start mysqld.service

# 开机自启
sudo systemctl enable mysqld.service

# 查看状态
sudo systemctl status mysqld.service

# 停止
sudo systemctl stop mysqld.service

# 重启
sudo systemctl restart mysqld.service
```



### 五、登录
---

```bash
# 首次登录的的时候不知道root用户的密码，查看生成的初始密码
grep "password" /var/log/mysqld.log

# 使用拿到的临时密码登录mysql
mysql -uroot -p
```
```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'bigdata3Tsk@';
```


### 六、其他
---

#### 1. MySQL JDBC驱动下载

```bash
# 下载链接驱动
wget https://downloads.mysql.com/archives/get/p/3/file/mysql-connector-java-8.0.16-1.el7.noarch.rpm

# 安装驱动，默认路径：/usr/share/java/mysql-connector-java.jar
sudo rpm -ivh mysql-connector-java-8.0.16-1.el7.noarch.rpm
sudo yum install mysql-connector-java-8.0.16-1.el7.noarch.rpm
# 以上两种安装方式选其一
```


#### 2. 允许使用root用户访问MySQL数据库（不推荐）

```sql
GRANT ALL ON *.* TO 'root'@'%';

-- 如果提示不允许改，执行以下语句
UPDATE user SET host='%' WHERE user='root';

-- 对MySQL中的权限进行更改（如使用 GRANT、REVOKE等语句）时，这些更改将在授权表中进行更新。然而，这些更改并不会立即生效，直到执行FLUSH PRIVILEGES命令
FLUSH PRIVILEGES;
```

#### 3. 允许使用普通用户访问MySQL数据库

- 添加用户

```sql
-- 1.只允许本机连接
CREATE USER '新用户名'@'localhost' identified by '密码';

-- 2.允许所有ip连接（用通配符%表示）
CREATE USER '新用户名'@'%' identified by '密码';
```

- 授权用户

```sql
-- 格式：
GRANT PRIVILEGES ON DataBaseName.TableName TO ‘username’@‘host’

-- 给admin用户授权study数据库中所有表的所有操作权限
GRANT ALL ON study.* TO 'admin'@'%';

-- 给admin用户授权所有库中所有表的所有操作权限
GRANT ALL ON *.* TO 'admin'@'%';

-- PRIVILEGES表示要授予什么权力，例如可以有 select ，insert 等，如果要授予全部权力，则填ALL
-- databasename.tablename表示用户的权限能用在哪个库的哪个表中，如果想要设置所有的数据库所有的表，则填*.*，*是一个通配符，表示全部。
-- ’username‘@‘host’：表示授权给哪个用户、哪台机器
```

#### 4. 改MySQL的密码策略（不推荐）

```sql
-- 查看密码粗略
SHOW VARIABLES LIKE 'validate_password%';

-- 修改密码策略(第一次登进去，需要改一次密码，不然不让你设置密码策略)
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'Tsk612473:';

-- 密码策略的强度要求
set global validate_password.policy=LOW;

-- 密码的最小长度
set global validate_password.length=4;

-- 退出重启
```



#### 5. MySQL开启binlog功能

```bash
# 修改/etc/my.cnf
[mysqld]

# 数据库id
server-id = 1

# 启动binlog，该参数的值会作为binlog的文件名
log-bin=mysql-bin

# binlog类型
binlog-format=row

# 启用binlog的数据库，需根据实际情况作出修改
binlog-do-db=gmall

# 设置完之后重启MySQL服务
sudo systemctl restart mysqld.service
```

MySQL binlog模式

- Statement-based：基于语句，Binlog会记录所有写操作的SQL语句，包括insert、update、delete等
  - 优点： 节省空间
  - 缺点： 有可能造成数据不一致，例如insert语句中包含now()函数

- Row-based：基于行，Binlog会记录每次写操作后被操作行记录的变化
  - 优点：保持数据的绝对一致性
  - 缺点：占用较大空间

- mixed：混合模式，默认是Statement-based，如果SQL语句可能导致数据不一致，就自动切换到Row-based