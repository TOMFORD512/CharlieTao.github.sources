---
layout: photo
title: Scala学习文档——容器
date: 2020-06-05 09:02:57
tags: [Scala]
categories: [编程语言,Scala]
excerpt: 。。。
---
# 前言

# Collection （容器）

---

- 根据中元素的`组织方式`和`操作方式`，Scala中的容器Collection可以分为有序和无序容器、可变和不可变等不同的容器类别。

- Scala用了三个包来组成容器类：

  - **scala.collection**

  - **scala.collection.mutable**

    *包含了所有`可变`的容器（例：可变集合、可变映射）*

  - **scala.collection.immutable**

    *包含了所有`不可变`的容器（可变集合、可变映射）*

<!-- more -->

- 组织关系图

  - 下图显示了scala.collection包中所有的容器类。

    > 这些都是`高级抽象类`或`特质`。例如，所有容器的基本特质（trait）是Traverable特质。它为所有的容器定义了公用的foreach方法，用于对容器元素进行遍历操作

    ![](https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/Scala/Scala%E5%AE%B9%E5%99%A8/scala.collection.png?raw=true)
  
- 下图显示了scala.collection.immutable（`不可变`）包中所有的容器类
  
  ![](https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/Scala/Scala%E5%AE%B9%E5%99%A8/scala.collection.immutable.png?raw=true)
  
  
  
- 下图显示了scala.collection.mutable（`可变`）中的所有容器类
  
  ![](https://github.com/CharlieTao/CharlieTao.github.sources/blob/master/BigData/Pictures/Scala/Scala%E5%AE%B9%E5%99%A8/scala.collection.mutable.png?raw=true)
  
  

## 1. 列表（List）

---

列表是一种共享相同类型的`不可变`的对象序列，Scala的List定义在scala.collection.immutable包中

```scala
var strList = List("Bigdata", "Hadoop", "Spark")
```

> 注：
>
> 1. 不同于Java的java.util.List，scala的List一旦被定义其值就不能改变了，因此声明List时必须初始化！
> 2. 这里用var声明不是说声明的List类型的strList是可变的，而是指向是可变的

基本知识：

- 列表有头部和尾部的概念，可以分别使用head和tail方法来获取

- head返回的是列表第一个元素的值

- tail返回的是除第一个元素外的其他值构成的新列表（这里体现出列表具有`递归`的`链表结构`）

```scala
  var strList = List("Bigdata", "Hadoop", "Spark")
  
  #下面表达式返回“Bigdata”
  var str = strList.head
  
  #下面表达式返回List(“Hadoop”, "Spark")
  val list = strList.tail
```

  

## 2. 集合（Set）

---

## 3. 映射（Map）

---

## 4. 迭代器（Iterator）

---

## 5. 数组（Array）

---

## 6. 元组（Tuple）

---