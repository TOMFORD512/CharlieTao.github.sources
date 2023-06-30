---
layout: post
title: CnetOS时间同步
date: 2020-06-13 05:31:01
tags: [CentOS]
categories: [Linux,CentOS]
password: 5201314
---


<!-- more -->

# 前言

>本文所述的CnetOS时间同步方法用于CDH安装时准备工作。以下是服务器的一些基本信息:
>1. 环境：自建虚拟机
>2. 系统：CentOS 7
>3. NTP服务器：阿里云（ntp.aliyun.com）
>4. 同步频率：开机自动同步


# NTP安装与配置

## 1. 下载
```bash
yum -y install ntp
```

## 2. 修改配置文件(/etc/ntp.conf）

```properties
#注释掉一下内容：
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst

#添加以下内容：
server ntp.aliyun.com
server ntp.aliyun.com
server ntp.aliyun.com
```

## 3. 开启服务

```bash
systemctl start ntpd
```

## 4. 设置NTP服务开机自启

```bash
systemctl enable ntpd
```

## 5. 将系统时钟同步到NTP服务器

```bash
ntpdate -u ntp.aliyun.com
```

## 6. 将硬件时钟与系统时钟同步

```bash
hwclock --systohc
```

>注：
>1. 本次同步的频率为开机自启，也可以设置Linux自带的crontab定时器定时执行。具体根据自身需求而定。
>2. 由于本次时间同步环境为自建虚拟机，时区在新建时就已经设置了；如果是线上服务器，可能会自动设置。具体后面会进一步补充。
>3. 更换时区命令：ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
>4. 查看当前时区命令： ll /etc/localtime