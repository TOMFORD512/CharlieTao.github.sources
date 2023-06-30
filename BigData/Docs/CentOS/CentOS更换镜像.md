---
layout: post
title: CnetOS更换镜像
date: 2020-06-13 05:23:52
tags: [CentOS]
categories: [Linux,CentOS]
password: 5201314
---

<!-- more -->
# 前言

>由于新建虚拟机时，使用系统自带的CentOS镜像在使用yum命令下载各种软件的速度不太理想，因此本文用于记录更换原始镜像为国内镜像的步骤。以下是服务器信息：
>1. 环境：自建虚拟机
>2. 系统：CentOS 7
>3. 镜像：阿里云

<!-- more -->

# 前提条件

## 安装wget/curl

```bash
yum -y install wget

yum -y install curl
```

# 配置

## 1. 备份原本的镜像

```bash
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

## 2. 下载镜像并写入指定文件中

- 以下两种方式均可下载目标镜像

```bash
#使用Wget命令
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

#使用curl命令
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

>注：-o后面路径+指定文件名，作用是将目标镜像中的内容写到指定目录的指定文件中


## 3. 更新镜像源

```bash
#清除缓存
yum clean all

#生成缓存
yum makecache
```

>注：网上有一篇博客说要将跟换后的CentOS-Base.repo中的所有http开头的更改为https，作用暂时未知（可能网络安全），后续了解后，会进一步补充。