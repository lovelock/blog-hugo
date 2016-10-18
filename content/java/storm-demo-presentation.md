+++
title = "一个真实Storm应用源码解析"
date = "2016-10-11T16:41:18+08:00"
categories = ["BigData"]
isCJKLanguage = true
tags = ["Java", "Storm"]
+++

## 前言

这里是Storm分享的内容。我自己也是初学者，这里抛砖引玉，希望大家多多指教。为简单起见，本应用用的是Java实现，没有用到Storm的多语言支持和更高层面的Trident Topology。源码详见[storm-demo](https://github.com/lovelock/storm-demo)。


## 理论

### 概述

Apache Storm是一个自由并且开源的**分布式实时**计算系统.它使得像Hadoop做批处理一样做**实时的**、**无限量**的**流数据**处理变得简单可靠.

![Apache Storm工作流程](http://storm.apache.org/images/storm-flow.png)

### 概念解释

#### 工作原理

![Apache Storm组件间关系](https://www.tutorialspoint.com/apache_storm/images/zookeeper_framework.jpg)

1. Nimbus  
     Nimbus是Storm集群的**主节点master node**。Storm集群中除Nimbus节点之外的所有节点叫做**工作节点worker nodes**。  
     Nimbus负责三项工作：  
    - 向worker nodes分发数据
    - 向worker nodes分配tasks
    - 监控失败

2. Supervisor  
     接受Nimbus的指令的节点叫做Supervisors（监工），它有**多个worker process**，并控制worker process完成Nimbus分配的tasks

3. Worker Process  
     Worker process执行指定Topology的tasks。**worker process自己并不实际执行tasks，而是创建executors并由executors执行指定的task。**一个worker process可以由多个executor。

4. Executor  
     Executor是由worker process创建的线程。一个executor执行一个或多个tasks，但只为**一个指定的Spout或者Bolt工作**。

5. Task  
     Task是实际的数据处理工作，所以它可能是一个Spout或者Bolt。

#### 配套服务

1. ZooKeeper  
    [ZooKeeper](http://zookeeper.apache.org/)是一个分布式的配置分发服务。Storm和Kafka都是无状态的，它们的工作需要外部服务为其维持状态，如Storm从Kafka中取数据时需要的partition编号和offset偏移量等诸如此类的信息。ZooKeeper会综合分析Spout和Bolt发送来的ack或者fail请求来决定是否更新offset。如下图所示

    ![](http://ww3.sinaimg.cn/large/65e4f1e6jw1f8wd8tm94ij21kw097q5h.jpg)

2. Kafka  
    [Kafka](http://kafka.apache.org/)是一个分布式的消息系统。支持**点对点**和**发布-订阅**两种消息模式。在和Storm配合中，充当**数据来源**的角色。用[KafkaSpout](https://github.com/apache/storm/tree/master/external/storm-kafka)和Storm进行组合。

> 本文只关注Storm，有关ZooKeeper和Kafka的介绍，可以访问[官网](http://apache.org/)、[TutorialsPoint](http://www.tutorialspoint.com/)或本博客的其他相关文章。

#### 拓扑作业

1.  Tuple  
     Tuple是Topology中数据流的传输格式。它是**不可变的键值对组**。既然是键值对，就需要设置键和值，典型的设置方式如下：

    ```java
     // 设置键
     outputFieldsDeclarer.declare(new Fields("timestamp", "fieldvalues"));
     // 设置值
     collector.emit(tuple, new Values(timestamp, stringBuilder.toString()));
    ```

     这样就会得到一个形如`("timestamp": timestamp, "fieldvalues": xxxx")`这样的Tuple。

2.  Spout  
     Spout是Topology的数据来源，输出的数据以Tuple的形式传入下一个Bolt。具体到本例中，KafkaSpout会把它接收到的数据以类似`(0: message)`这样的形式发射(emit)出来。所以，在KafkaSpout下游的Bolt需要这样获取整条数据(其实这里是可配置的)：

    ```java
     String message = tuple.getString(0);
    ```
    对KafkaSpout而言，它也实现了多个方法，但我们这里只需要了解两个`ack`和`fail`。

    ![](http://ww3.sinaimg.cn/large/65e4f1e6jw1f8wdqv4tuqj218y0ji41l.jpg)

    这两个是回调方法，分别在ack unit向其发送ack或fail请求时被触发，一般而言，ack方法由于通知Kafka发送下一条数据，fail方法用于通知Kafka重发上一条数据。

    > 关于ack unit，可以理解为Storm中处理bolt的ack或fail请求的总线，它接收到相应的请求后，做简单的异或处理，并把结果发送给Spout。

3.  Bolt  
     Bolt是真正写处理逻辑的地方，比如在本例中，我们要做以下几件事：

    1.  把message中的每个字段提取出来，
    2.  从message的domain字段中过滤出以`.api.ksyun.com`结尾的，其他的舍弃
    3.  把domain字段的值以`.`分割，取出index为0的部分，也就是第一段作为service字段
    4.  把service最终输出Tuple的一个field写入输出结果

     一般情况下，要实现一个Bolt有几种方式  

    ​

    -   实现`IRichBolt`接口  
          因为这个比较低级，要实现的方法有很多，而其中多数的方法不需要做特殊处理，所以一般会用第二种方式
    -   集成`BaseRichBolt`类  
          这个基类实现了`IRichBolt`中定义的几个不常用的方法，让我们只需要关注重点的几个方法即可。在这种方式中，我们需要自己实现三个方法：  

        1.  `public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector)`    
              这个方法**类似构造函数**，用来做一些准备工作，通常用于**把上游传来的collector赋值给成员变量**。

        2.  `public void execute(Tuple tuple)`

             这是最核心的方法。它负责：  
            - 从上游传来的Tuple中读取感兴趣的字段
            - 把这些字段做一些处理后产生一组新的字段
            - 把这些值通过`OutputCollector::emit(new Values())`方法发射出去
            - 向上游发送`OutputCollector::ack(Tuple tuple)`或`OutputCollector::fail(Tuple tuple)`，以告知上游本次Tuple处理是否成功。

        3.  `public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer)`

             前面已经说过，Tuple是数据交流的格式，这个方法就是用来定义发送到下游的Tuple的字段名的。

4.  Topology  

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

主要关注`execute`方法，它首先从上游发射来的Tuple中取出第一个字段，也就是整条消息作为一个字符串。根据对字符串的分析，我们知道该字符串是以`\t\t`作为字段间分隔符，以`:`作为键值分隔符的字符串，所以可以写一个方法来用这种规则解析出消息中的所有字段，并把它放在一个HashMap里。

```java
    private HashMap makeMapOfMessage(String message) {
    String[] fields = message.split(ServerConfig.getFieldSeparator());
    HashMap<String, String> map = new HashMap<>();

    try {
        for (String field : fields) {
            String[] pair = field.split(ServerConfig.getPairSeparator(), 2);
            map.put(pair[0], pair[1]);
        }
    } catch (ArrayIndexOutOfBoundsException e) {
        LOG.warn("makeMapOfMessage failed {}", message);
        e.printStackTrace();
    }

    return map;
}
```

其中`ServerConfig`是一个工具类，它提供了简单的API，把处理逻辑和配置信息分离。在本例中我用的分隔符和实际项目并不一样，这个差别只需要在相应的properties配置文件中做修改即可。还需要注意异常处理，这个方法的返回值有可能是null，在调用该方法的地方需要做相应的判断。

在`execute`方法中，从上述方法的返回值取出关心的字段，并按需求解析出需要的`service`字段，并通过`collector.emit`发送给下游的Bolt。

这个方法中有三点需要注意：

1. 没有在`try`子句中调用`ack`方法
2. 没有在`catch`子句中调用`fail`方法
3. 在`finally`子句中调用了`ack`方法

因为我们catch住的这种情况，是只有在输入的数据不满足我们约定要求的情况下才会发生的，比如某些必要的字段不存在等，而这种情况在当前的Topology中是不需要处理的，并且也不需要重试，因此，不需要调用`fail`。同时，不管数据是否符合要求，我们都是需要通知Spout**这里的处理已经完成**这个信息的，所以在`finally`中调用`ack`。

### `SplitFieldsBolt.java`

这步的功能很简单，就是把前面传过来的所有字段用一个特定的分隔符连接起来，变成一行数据。只有一个特殊，也就是`service`字段，它不是直接取出来的，而是前面的Bolt通过一些处理得到的，所以这是`stringBuilder`需要处理的一种特殊情况。

最后把『时间戳』和『各个字段的值』发射给下游的`HdfsBolt`。

### `HdfsBolt`

`HdfsBolt`是Storm到HDFS的一个中转层，配置一些规则，把Storm输出的数据写入HDFS。其中比较重要的配置有：

1. `DelimitedRecordFormat` 要写入的字段 在本例中，『时间戳』只是用来划分目录的，所以不需要写入HDFS中
2. `CountSyncPolicy` 指定当内存中超过多少条数据时cache到磁盘中
3. `FileSizeRotationPolicy` 指定cache的文件超过多大时将文件写入文件系统，如果该值设置的较大，而数据流量又不太大的情况下，文件通常不会达到设置的值，因为当等待写入的文件未达到限制大小而先达到超时时间时，也会创建一个新的文件。
4. `DefaultFileNameFormat` 指定文件写入HDFS中的根目录和文件后缀
5. `Partitioner` 指定分块规则，在本例中，我们根据日志中`time_local`字段划分相应的消息应该写入的HDFS目录，比如`31/Aug/2016:13:08:12 +0800`，相应的记录就会写入`root/20160831/13`目录中。
6. `FsURL` 当然需要指定正确的`HDFS`服务。

> 其实我们的KMR对应的Storm 0.10.0是不支持HDFS 的partition的，这里我是把Storm最新版的2.0.0-SNAPSHOT中相应的代码移植过来用的。

### `LogStatisticsTopology.java`

前面讲过，这是拓扑作业的入口，这里指定了一条消息要通过的路径。需要注意的有以下几点：

1. `setSpout`和`setBolt`方法中的parallelism_hint(并行度建议)，前面说了，Spout和Bolt在Storm中是以executor的形式存在的，而这个值就是指定executor的数量。但又没有那么绝对，比如在KafkaSpout中，如果指定的Topic在Kafka中有10个partition，但这里的KafkaSpout指定了15个并行度，实际还是只有10个executor有意义，因为剩余的5个在前面10个都正常工作的情况下是分配不到任何数据的，由于ZooKeeper做了中间人，它是知道每个Topic有多少个partition的，所以这里设置多于partition数量的并行度也是不起作用的。
2. grouping类型 Storm目前支持4种分组形式。

  - 随机分组 等量tuples随机分发给执行bolt的所有workers
  - 字段分组 把指定字段值相同的分配给同一个task,在wordCount应用中比较重要
  - 广播分组 给每个executor发送一个这个tuple的副本
  - 全局分组 把所有数据分发给bolt的executor中id最小的**一个**
  - 无分组   目前基本等同于随机分组，会把tuple交给和它上游同一个线程内的下游bolt，以减少数据传递的开销

根据我们的需求，其实是不需要太关心分组的事情。

关于优化，有几个方面可以考虑：

1. 考虑到后面需要用Hive分析数据，如果产生很多小的文件，就会产生过多的Map过程，影响性能，可以考虑同一小时的文件交给同一个executor来写，因为每个executor会打开一个hdfs文件，但这样可能会导致并发数过少
2. 既然这样，可以减少executor的数量，比如现在是10个，可以改成5个，在不触发FileSizeRotationPolicy的情况下，把生成的文件数量减少了一半，也就把Hive查询时Map过程的数量减少了一半
3. 分析需求，如果没有按照小时分组的需求，可以直接删除这个级别，直接用天作为区分，这样，在不触发FileSizeRotationPolicy的情况下产生的文件数量会变成1/24,相应的Hive查询中Map的过程也会变成1/24.

## 总结

本文主要介绍了Storm的工作流程，以及其与Kafka和HDFS的配合来进行日志分析的工作流程，并简单介绍了一些需要注意的点。

我也是Storm的初学者，并没有太多的实际经验，这里只是演示了一个可以工作的Storm作业的实例，大家有什么问题可以及时与我交流，也希望大家不吝赐教。
