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

### 8. 为用户赋予Root权限

```bash
#在执行某些命令时需要sudo并输入密码

#先切换到root用户
vim /etc/sudoers
#修改如下：

## Allow root to run any commands anywhere 
root    ALL=(ALL)       ALL
## Allows members of the 'sys' group to run networking, software, 
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL
#可以写用户或者用户所属组
%Worker ALL=(ALL) NOPASSWD:ALL
## Allows members of the users g
```



### 9. 服务器之间配置别名

```BASH
vim /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.0.101 bigdata1
192.168.0.102 bigdata2
192.168.0.103 bigdata3
```

### 10.服务器之间配置免密

<img src="https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/Linux/SSH%E9%85%8D%E7%BD%AE.png?raw=true" />

```BASH
#生成密钥对
ssh-keygen
#将公钥上传到需要配置免密的服务器，并以相同用户登录（默认是以相同用户登录），根据提示输入免密即可
ssh-copy-id CharlieTao@192.168.0.102
```

### 11. 服务器之间传输文件（配置完免密之后）

```BASH
scp -r package/ CharlieTao@bigdata2:/home/CharlieTao/
```

### 12. 配置JDK

- 前置工作：检查系统是否已自带JDK（一般最小版本安装是没有自带JDK的，其余版本建议先检测，有则卸载）

```bash
#检测
rpm -qa | grep -i java

#检测+删除（检测出来的结果可能有多个，直接每行遍历删除）
rpm -qa | grep -i java | xargs -n1 rpm -e --nodeps 
```

- 解压

```BASH
tar -zxvf jdk-8u45-linux-x64.tar.gz -C /home/CharlieTao/software/
```

- 配置环境变量

<img src="https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/Linux/Java%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E8%AE%BE%E7%BD%AE.png?raw=true" />
