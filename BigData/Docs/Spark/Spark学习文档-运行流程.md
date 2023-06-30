---
title: Spark学习文档--运行流程
date: 2020-12-27 23:57:49
tags: [Spark]
categories: [Bigdata,Spark]
---

# 前言

<!-- more -->

# 基本概念

## 1. Application

```
    表示本次运次的应用程序
```

## 2. Driver

```
    表示main()函数，创建SparkContext。由SparkContext负责与CLusterManager通信，进行资源的申请、任务的分配和监控等。程序执行完毕后关闭SparkContext
```

## 3. Executor

```
    某个Application运行在Worker节点上的一个进程，该进程负责运行某些Task，负责将数据存在内存或者磁盘上。在Spark On Yarn模式下，其进程名称为CoarseGrainedExecutorBackend，一个CoarseGrainedExecutorBackend能运行Task的数据就取决于分配给它的CPU的个数
```

## 4. Worker

```
    集群中可以运行Application的节点。在Standalone模式中指的是通过slave文件配置的worker节点，在Spark On Yarn模式中指的就是NodeManager节点
```

## 5. Task

```
    在Executor进程中执行任务的工作单元，多个Task组成一个Stage
```

## 6. Job

```
    包含多个Task组成的并行计算，是有行动算子触发的
```

## 7. Stage

```
    每个Job会被拆分成很多组Task，作为一个TaskSet，其名称为Stage
```

## 8. DAGScheduler

```
    根据Job构建基于Stage的DAG，并提交Stage给TaskScheduler，其划分Stage的依据是RDD之间的依赖关系
```

## 9. TaskScheduler

```
    将TaskSet提交给Worker（集群）运行，每个Executor运行什么Task就是在此处分配的
```

>注：
>   有关Application、Driver、Job、Stage、Task之间的关系如下图所示：



# 运行流程

## 图解

## 文字说明
1. 构建SparkApplication的运行环境（启动SparkContext），SparkContext向资源管理器（Standalone、Mesos或者Yarn）注册并申请运行Executor资源

2. 资源管理器分配Executor资源并启动StandaloneExecutorBackend，Executor运行情况将随着心跳发送到资源管理器上

3. SparkContext构成DAG图，将DAG图分解成Stage，并把TaskSet发送给Task Scheduler；Executor向SparkContext申请Task

4. Task Scheduler将Task发放给Executor运行同时SparkContext将应用程序代码发放给Executor

5. Task在Executor上运行，运行完毕释放所有资源