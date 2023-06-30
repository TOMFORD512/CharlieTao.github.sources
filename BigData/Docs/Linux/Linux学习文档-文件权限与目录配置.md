---
layout: post
title: Linux学习文档-文件权限与目录配置
date: 2021-04-20 23:56:00
tags: [Linux]
categories: [Linux,文件、目录与磁盘]
excerpt: 。。。
---

## 前言

<!-- more -->

## 1. 文件属性

![图一（文件属性示意图）](https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/Linux/%E6%96%87%E4%BB%B6%E5%B1%9E%E6%80%A7%E7%A4%BA%E6%84%8F%E5%9B%BE.png?raw=true)

- 图一中，第1串字符代表这个文件的类型与权限
    
    - 第一个字符代表着个文件是目录、文件或链接文件等
        - [d]代表目录
        - [-]代表文件
        - [l]代表链接文件
        
        ![图二（文件类型与权限之内容）](https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/Linux/%E6%96%87%E4%BB%B6%E7%B1%BB%E5%9E%8B%E4%B8%8E%E6%9D%83%E9%99%90%E4%B9%8B%E5%86%85%E5%AE%B9.png?raw=true)
        
    - 如图二所示，后九位字符中每三个字符为一组，且均为【rwx】的组合。
    
    - 第一组为文件拥有者的权限，第二组为文件所属用户组的权限，第三组为其他用户的权限
    
    - 以第一组为例：
        
        [r]代表read，可读权限，权限值为4，权限排名最低
        
        [w]代表write，可写权限，权限值为2，权限排名居中
        
        [r]代表excute,可执行权限，权限值为1，权限排名最高

> 注：
>- rwx所在的位置或者说顺序是不会改变的，有权限就会显示字符，没有则用-表示
>- 此处要与第一个字符区分开
- 图一中，第2串字符表示有多少个文件名链接到此节点

- 图一中，第3串字符表示这个文件的拥有者账号名

- 图一中，第4串字符表示这个文件的所属用户组

- 图一中，第5串字符表示这个文件的大小（默认单位为Bytes）

- 图一中，第6串字符表示这个文件的创建日期或者是最近的修改日期，若文件的修改时间距今太久，那么只会显示年月日

- 图一中，第7串字符表示文件名
    
    文件名前面有[.]则表示改文件为隐藏文件，可用【ls】、【ls -a】及【ls -al】来查看
>注：
>- 对于文件夹来说，[x]原本代表执行权限，但如果某个用户对于某个目录没有[x]的权限，则该用户无法进入该目录



## 2. 如何修改文件属性与权限

| 命令|   作用 |
|:-:|:-:|
| chgrp | 修改文件所属用户组 |
| chown | 修改文件拥有者 |
| chmod | 修改文件的权限，SUID、SGID、SBIT等的特性|

### (1) 修改所属用户组

```bash
chgrp   [-R]  账号名   dirname/filename
-R: 递归修改
```
>注：
>- 目标用户组必须存在于/etc/group中才能修改成功，否则会报错
>- 修改文件拥有者亦是如此，目标用户必须存在于/etc/passwd中

### (2) 修改文件拥有者，chown

```bash
chown   [-R]    账号名称    文件或目录
chown   [-R]    账号名称：用户组名称    文件或目录
-R: 递归修改
```

### (3) 修改文件权限，chmod

#### ① 用数字修改

| 权限|   权限值 |
|:-:|:-:|
| r | 4 |
| w | 2 |
| x | 1|

```bash
#例：
owner = rwx = 4+2+1 = 7
group = rwx = 4+2+1 = 7
others = --- = 0+0+0 = 0

#因此，对应用数字修改权限的语法是：
chmod   -R  xyz 文件或目录
#-R: 递归修改

#常用的如果要将某个文件所有的权限都开通则为
chmod 777 .bashrc
```
>注：
>- 通常以vim编辑一个Shell文件的时候，他的权限通常是：-rw-rw-r--，也就是644，且此文件是不能够执行的,如果要把它变成可执行问文件，则需要执行命令：【chmod 744 test.sh】或者：【chmod a+x test.sh】，后者即为下面要介绍的用符号修改
>- 另外，如果有些文件你不想被其他人看到，可以执行：【chmod 700 text.sh】

#### ② 用符号修改

在修改之前，首先需要知道权限只有user、group、同任何人三种身份，分别用u、g、o来表示。此外，所有a代表all即全部的身份，因此修改方法如下图所示：

![符号类型修改文件权限](https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/Linux/%E7%AC%A6%E5%8F%B7%E7%B1%BB%E5%9E%8B%E4%BF%AE%E6%94%B9%E6%96%87%E4%BB%B6%E6%9D%83%E9%99%90.png?raw=true)

```bash
# 例1：当我们要设置一个文件的权限为【-rwxr-xr-x】时，可执行如下命令：
chmod u=rwx,go=rx .bashrc

# 例2：当我们不知道原来的文件属性，但是只想要增加.bashrc这个文件的每个人都可以写入得权限时，可执行如下命令：
chmod a+w .bashrc

#例3：当我想要将全部人的可执行权限去掉而不修改其他已存在的权限时，可执行如下命令：
chmod a-x .bashrc
```
