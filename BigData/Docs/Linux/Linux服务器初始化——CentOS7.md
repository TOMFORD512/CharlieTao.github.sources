# Linux服务器初始化—CentOS7

### 1. 查看服务器的CentOS版本

```BASH
#二者皆可
tail /etc/redhat-release
cat /etc/redhat-release
```

### 2. 更改服务器名称

```bash
#输入以下命令，而后直接输入服务器名称
vim /etc/hostname
```

### 3. 更改网络设置

```bash
vim /etc/sysconfig/network-scripts/ifcfg-ens33
```

更改以下几点

```BASH
#1.设置静态IP
BOOTPROTO=static
#2.
ONBOOT=yes
#3.给定固定IP
IPADDR=192.168.0.101
#4.子网掩码
GATEWAY=192.168.0.1
#5.网关
NETMASK=255.255.255.0
#6.
DNS1=8.8.8.8
DNS2=8.8.4.4
```

### 4. 更换默认镜像（设置Yum源）

注：详情请参考[CnetOS 更换镜像](https://charlietao.github.io/2020/06/13/CentOS更换镜像/)

### 5. 按需更改防火墙状态

```BASH
#1.查看防火墙状态
systemctl status firewalld
#2.关闭(开启)防火墙
systemctl stop firewalld/systemctl start firewalld
#4.禁用防火墙服务
systemctl disable firewalld
#注：如果需要永久关闭，需要先关闭再禁用防火墙服务
```

### 6. 扩展（使用Telnet连接虚拟机）

### 7. 创建用户

```BASH
#优先创建用户组
groupadd Worker
#创建用户并制定用户组
useradd CharlieTao -g Worker
#创建用户之后应立即更新密码（输入以下命令后输入两次密码则密码设置成功）
passwd CharlieTao
```

### 8. 服务器之间配置别名

```BASH
vim /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.0.101 bigdata1
192.168.0.102 bigdata2
192.168.0.103 bigdata3
```

### 9.服务器之间配置免密

<img src="./Pictures/SSH配置.png" style="zoom:75%;">

```BASH
#生成密钥对
ssh-keygen
#将公钥上传到需要配置免密的服务器，并以相同用户登录（默认是以相同用户登录），根据提示输入免密即可
ssh-copy-id CharlieTao@192.168.0.102
```

### 10. 服务器之间传输文件（配置完免密之后）

```BASH
scp -r package/ CharlieTao@bigdata2:/home/CharlieTao/
```

### 11. 配置JDK

- 解压

```BASH
tar -zxvf jdk-8u45-linux-x64.tar.gz -C /home/CharlieTao/software/
```

- 配置环境变量

<img src="./Pictures/Java环境变量设置.png" style="zoom:75%;">
