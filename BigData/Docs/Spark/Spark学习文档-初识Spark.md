---
title: Spark学习文档--初识Spark
date: 2020-12-24 23:50:18
tags: [Spark]
categories: [Bigdata,Spark]
---

# 前言

# 概述

## 1. 什么是Spark？
Apache Spark用于大规模数据处理的统一分析引擎。

## 2. 特点

- 处理速度快
    - Apache Spark通过使用最先进的DAG调度器、查询优化器和物理执行引擎，实现了批处理和流数据的高性能

>这里有Spark与MR速度对比图

- 易用性好
    - Spark提供了80多个高级操作符，可以轻松构建并行应用程序

    - Spark不仅支持Scala编写应用程序，而且还支持Java和Python 等语言进行编写

- 通用性高

    - Spark生态圈即BDAS（伯克利数据分析栈）所包含的组件：Spark Core 提供内存计算框架、SparkStreaming的实时处理应用、Spark SQL 的即席查询、MLlib的机器学习和GraphX的图处理

    - 它们都是由AMP实验室提供，能够无缝地集成，并提供一站式解决平台

> 这里有Spark技术堆栈组成图

- 随处运行
    - Spark可以运行在Hadoop、Apache Mesos、Kubernetes、独立平台上，也可以运行在云上。它可以访问不同的数据源

    - 你可以在EC2、Hadoop YARN、Mesos或Kubernetes上使用它的独立集群模式运行Spark。访问HDFS、Alluxio、Apache Cassandra、Apache HBase、Apache Hive和数百个其他数据源中的数据
>这里有Spark支持的技术框架截图

## 3. Spark与MapReduce比较

Spark是通过借鉴Hadoop MapReduce发展而来的，继承了其分布式并行计算的优点，并改进了MapReduce明显的缺陷，具体体现在以下几个方面：

    (1) Spark把中间数据放在内存中，迭代式运算效率高。MapReduce中的计算结果是保存在磁盘上，这样势必会影响整体的运行速度，而Spark支持DAG图的分布式并行计算的编程框架，减少了迭代过程中数据的落地，提高了处理效率
    
    (2) Spark的容错性高。Spark引进了弹性分布式数据集（Resilient Distributed Dataset，RDD）的概念，它是分布式在一组节点中的制度对象集合，这些集合是弹性的，如果数据一部分丢失，则可以根据 “血统” （即允许基于数据衍生过程）对它们进行重建。另外，在RDD计算时可以通过CheckPoint来实现容错，而CheckPoint有两种方式，即CheckPoint Data和Logging The Updates，用户可以控制采用哪种方式来实现容错。
    
    (3)Spark更加通用。不像Hadoop只提供了Map和Reduce两种操作，Spark提供农的数据集操作类型有很多种，大致分为转换操作和行动操作两大类。转换操作包括 Map、Filter、FlatMap、Sample、GroupByKey、ReduceByKey、Union、Join、Cogroup、MapValues、Sort、和PartionBy等多种操作类型，行动操作包括Collect、Reduce、Lookup和Save等操作类型。另外，各个处理节点之间的通信模型不再像Hadoop只有Shuffle一种模式，用户可以命名、物化、控制中间结果的存储、分区等



# Spark生态系统

>注：以下说的组件均为重要内容，只做简单介绍，后面会一一整理专门的文档！

## Spark Core

- Spark Core提供了多种运行模式，不仅可以使用自身运行模式处理任务，如本地模式、Standalone，而且可以使用第三方资源调度框架来处理任务，如Yarn、Mesos等。相比较而言，第三方资源调度框架能够更细粒度管理资源

- Spark Core提供了有向无环图（DAG）的分布式并行计算框架，并提供内存机制来支持多次迭代计算或者数据共享，大大减少迭代计算之间读取数据的开销，这对于需要进行多次迭代的数据挖掘和分析性能有着极大的提升。另外，在任务处理过程中移动计算而非移动数据，RDD Partition可以就近读取分布式文件系统中的数据到各个节点内存中进行计算

- 在Spark中引入了RDD的抽象，它是分布在一组几点钟的只读对象结婚，这些集合时弹性的，如果数据集一部分丢失，则可以根据“血统”对它们进行重建，保证了数据的高容错性


## Spark Streaming
- Spark Streaming是一个对实时数据进行高吞吐、高容错的流式处理系统，可以对多种数据源（如Kafka、Flume、Twitter和ZeroMQ等）进行类似Map、Reduce和Join等复杂操作，并将结果保存到外部文件系统、数据库或应用到实时仪器盘。

- 相比其他的处理引擎要么只专注与流处理，要么只负责批处理（仅提供需要外部实现的流处理API接口），而Spark Streaming最大的优势是提供处理引擎和RDD编程模型可以同时进行批处理与流处理。

## Spark SQL

- Spark SQL是Spark用来操作结构化数据的程序包。通过Spark SQL，我们可以使用SQL或者Apache Hive版本的SQL方言（HQL）来查询数据。

- Spark SQL支持多种数据源，比如Hive表、Parquet以及JSON等。

- 除了为Spark提供了一个SQL接口，Spark SQL还支持开发者将SQL和传统的RDD编程的数据操作方式相结合，不论是使用Python、Java还是Scala，开发者都可以在在单个的应用中同时使用SQL和复杂的数据分析。

## MLlib

- MLlib是Spark中提供常见的机器学习功能的组件

- MLlib提供了包括分类、回归、聚类、协同过滤等，还提供了模型评估、数据导入等额外的支持功能。

- MLlib还提供了一些更底层的机器学习原语，包括一个通用的的梯度下降优化算法

## GraphX

- GraphX是Spark中用来操作图（比如社交网络的朋友关系图）的组件，可以进行并行的图计算。

- 与Spark SQL与Spark Streaming类似，GraphX也拓展了Spark的RDD API，能用来创建一个顶点和边都包含任意属性的有向图

- GraphX还支持针对图的各种操作（比如进行图分割的subgraph和操作所有顶点的mapVertices），以及一些常用图算法（比如PageRank和三角计数）