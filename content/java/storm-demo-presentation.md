+++
title = "一个真实Storm应用源码解析"
draft = true
date = "2016-10-11T16:41:18+08:00"
categories = ["Java"]
tags = ["Java", "Storm"]
+++

## 前言

这里是Storm分享的内容。我自己也是初学者，这里抛砖引玉，希望大家多多指教。为简单起见，本应用用的是Java实现，没有用到Storm的多语言支持和更高层面的Trident Topology。源码详见[storm-demo](https://github.com/lovelock/storm-demo)。

<!--more-->
## 基本概念介绍

### 概述

Apache Storm是一个自由并且开源的**分布式实时**计算系统.它使得像Hadoop做批处理一样做**实时的**、**无限量**的**流数据**处理变得简单可靠.

![Apache Storm工作流程](http://storm.apache.org/images/storm-flow.png)

### 概念解释

#### 工作原理

![Apache Storm组件间关系](https://www.tutorialspoint.com/apache_storm/images/zookeeper_framework.jpg)

1. Nimbus  
    Nimbus是Storm集群的**主节点master node**。Storm集群中除Nimbus集群之外的所有节点叫做**工作节点worker nodes**。  
    Nimbus负责三项工作：  
    - 向worker nodes分发数据
    - 向worker nodes分配tasks
    - 监控失败

1. Supervisor  
    接受Nimbus的指令的节点叫做Supervisors（监工），它有多个worker process，并控制worker process完成Nimbus分配的tasks

1. Worker Process  
    Worker process执行指定Topology的tasks。**worker process自己并不实际执行tasks，而是创建executors并由executors执行指定的task。**一个worker process可以由多个executor。

1. Executor  
    Executor是由worker process创建的线程。一个executor执行一个或多个tasks，但只为**一个指定的Spout或者Bolt工作**。

1. Task  
    Task是实际的数据处理工作，所以它可能是一个Spout或者Bolt。

#### 外围服务

1. ZooKeeper  
    [ZooKeeper](http://zookeeper.apache.org/)是一个分布式的配置分发服务。Storm和Kafka都是无状态的，它们的工作需要外部服务为其维持状态，如Storm从Kafka中取数据时需要的partition编号和offset偏移量等诸如此类的信息。

1. Kafka  
    [Kafka](http://kafka.apache.org/)是一个分布式的消息系统。支持**点对点**和**发布-订阅**两种消息模式。在和Storm配合中，充当**数据来源**的角色。用[KafkaSpout](https://github.com/apache/storm/tree/master/external/storm-kafka)和Storm进行组合。

#### 工作流程

1. Tuple  
    Tuple是Topology中数据流的传输格式。它是**不可变的键值对组**。既然是键值对，就需要设置键和值，典型的设置方式如下：

    ```java
    // 设置键
     outputFieldsDeclarer.declare(new Fields("timestamp", "fieldvalues"));
    // 设置值
    collector.emit(tuple, new Values(timestamp, stringBuilder.toString()));
    ```

    这样就会得到一个形如`("timestamp": timestamp, "fieldvalues": xxxx")`这样的Tuple。

1. Spout  
    Spout是Topology的数据来源，输出的数据以Tuple的形式传入下一个Bolt。具体到本例中，KafkaSpout会把它接收到的数据以类似`(0: message)`这样的形式发射(emit)出来。所以，在KafkaSpout下游的Bolt需要这样获取整条数据：

    ```java
    String message = tuple.getString(0);
    ```

1. Bolt  
    Bolt是真正写处理逻辑的地方，比如在本例中，我们要把message中的每个字段提取出来，并根据需求从其中的domain字段中过滤出以`.api.ksyun.com`结尾的，然后把这个domain以`.`分割，取出index为0的部分，也就是第一段作为service名，并把service写入最终的输出结果中。

    一般情况下，要实现一个Bolt有几种方式  
    - 实现`IRichBolt`接口  
        因为这个比较低级，要实现的方法有很多，而大多数其实是不需要什么实现的，所以一般会用第二种方式
    - 集成`BaseRichBolt`类  
        这个基类实现了`IRichBolt`中定义的几个不常用的方法，让我们只需要关注重点的几个方法即可。在这种方式中，我们需要自己实现三个方法：  

        1. `public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector)`    
             这个方法**类似构造函数**，用来做一些准备工作，通常用于**把上游传来的collector赋值给成员变量**。

        2. `public void execute(Tuple tuple)`

            这是最核心的方法。它负责：  
            - 从上游传来的Tuple中读取感兴趣的字段
            - 把这些字段做一些处理后产生一组新的字段
            - 把这些值通过`OutputCollector::emit(new Values())`方法发射出去
            - 向上游发送`OutputCollector::ack(Tuple tuple)`或`OutputCollector::fail(Tuple tuple)`，以告知上游本次Tuple处理是否成功。
            
        3. `public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer)`

            前面已经说过，Tuple是数据交流的格式，这个方法就是用来定义发送到下游的Tuple的字段名的。

1. Topology  

    ![一个典型的Topology](http://storm.apache.org/releases/0.10.0/images/topology.png)

    上面这张图中有4个Topology，说明了几个问题： 
    - 一个Spout就是一个Topology的入口，从Spout分出几条线就有几个Topology
    - 一个Topology由一个Spout和若干个Bolt组成
    - Topology之间可以共享Spout或者Bolt

    创建一个Topology的典型过程：

    ```java
        TopologyBuilder topologyBuilder = new TopologyBuilder();
        topologyBuilder.setSpout(KAFKA_SPOUT_ID, kafkaSpout, 10);
        topologyBuilder.setBolt(CROP_BOLT_ID, new CropBolt(), 10).shuffleGrouping(KAFKA_SPOUT_ID);
        topologyBuilder.setBolt(SPLIT_FIELDS_BOLT_ID, new SplitFieldsBolt(), 10).shuffleGrouping(CROP_BOLT_ID);
        topologyBuilder.setBolt(STORM_HDFS_BOLT_ID, hdfsBolt, 10).fieldsGrouping(SPLIT_FIELDS_BOLT_ID, new Fields("timestamp", "fieldvalues"));
    ```

    从上面的代码可以看到，`TopologyBuilder`类通过`setSpout()`和`setBolt()`两个方法生动的反映了上图的工作流程。

## 源码分析

### `CropBolt.java`
    主要关注`execute`方法，它首先从上游发射来的Tuple中取出第一个字段，也就是整条消息作为一个字符串。根据对字符串的分析，我们知道该字符串是以`\t\t`作为字段间分隔符，以`:`作为键值分隔符的字符串，
### `SplitFieldsBolt.java`
### `LogStatisticsTopology.java`



