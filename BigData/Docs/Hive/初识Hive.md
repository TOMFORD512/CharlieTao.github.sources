---
layout: post
title: 初识Hive
date: 2020-06-13 05:41:59
tags: [Hive]
categories: [Bigdata,Hive]
excerpt: 。。。
---

### 前言
---



### 什么是Hive？
---

1. 由Facebook实现并开源、基于Hadoop的**数据仓库的工具**
2. 提供HQL（Hive SQL）查询功能
3. Hive本质上是将SQL**翻译**转换成MapReduce任务执行



### 为什么使用Hive（MapReduce、Spark）？
---

1. MR在实现复杂逻辑查询的时候，成本高、难度大。
2. Spark对集群资源更高（基于内存）



### Hive内部架构
---

![Hive内部架构](https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/Hive/Hive内部架构.png?raw=true)

- Thrift Server：提供了一种让用户可以使用多种不同的语言来操作Hive

  <!-- Thrift 是 Facebook 开发的一个软件框架，可以用来进行可扩展且跨语言的服务的开发， Hive 集成了该服务，能让不同的编程语言调用 Hive 的接口 -->

- Compiler（编译解释器）：将HiveSQL转换为抽象语法树（AST），并将语法树编译为逻辑执行计划

- Optimizer（优化器） ： 对逻辑执行计划进行优化 

-  Executor（执行器）：调用**底层的运行框架**执行逻辑执行计划 