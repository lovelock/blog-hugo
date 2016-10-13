+++
categories = ["BigData"]
date = "2016-10-12T23:36:09+08:00"
isCJKLanguage = true
draft=true
tags = ["zookeeper"]
title = "ZooKeeper原理简介和简单使用"

+++

## 什么是分布式应用

分布式应用运行在其上的一组系统被称为『集群』(cluster)，运行在集群中的每个机器被称为『节点』(node)。

### 分布式应用的优点

- 可靠性 单机或少量机器的故障不会导致整个系统不可用。
- 可扩展性 不用停机只需要做很少的配置就可以根据需求通过增加机器来提升系统的性能。
- 透明性 隐藏了系统的复杂性，对外值暴露单一的入口/应用。

### 分布式应用需要解决的难点

- 竞争条件 两个或多个机器都尝试去执行同一个任务，而该任务在任意时刻都应该只被一台机器执行。比如，共享的资源在某一时刻应该只能被一台机器修改。
- 死锁 两个或多个操作无限期的相互等待对方完成。
- 不一致性 数据的部分错误。


## ZooKeeper简介

ZooKeeper是一个**分布式的**、用来管理大量主机的**协调服务**。
在分布式环境中协调和管理一个服务是很复杂的工作，而ZooKeeper用简单的架构和API解决了这个问题，它用`fail-safe synchronization`机制解决了竞争和死锁的问题, 用`atomicity(原子性)`解决了数据的一致性问题。它屏蔽了分布式环境中的复杂性，让开发人员可以专注于核心应用功能的开发，而不用去关心分布式环境的太多细节。

### ZooKeeper提供的服务

- 名字服务 在一个集群内根据name找到主机，类似DNS服务
- 配置管理 集中管理某个节点的最新配置
- 集群管理 管理一个集群中某一节点的加入和离开
- 主节点选举 协调一个集群选举中一个新的主节点
- 加锁和同步服务 在数据被修改时给其加锁，这种机制可以帮助你在连接到其他如HBase的分布式服务时实现自动错误恢复
- 存放高可用数据 可以保证在一个或多个节点出故障时保证数据的可用性


### ZooKeeper的优点

- 简单的分布式协调过程
- 同步 服务器进程间的互斥和协作
- 有序的消息
- 序列化 用指定的规则编码数据。保证你的应用一致的运行。这种方式可以用在MapReduce中来协调对来执行线程
- 可靠性
- 原子性 数据传输要么成功要么失败，不存在中间状态

### ZooKeeper的架构

![ZooKeeper架构图](http://ww3.sinaimg.cn/large/006y8mN6jw1f7alv0grqej30fw04zdgg.jpg)

| 概念 | 职责和作用 |
---|---
Client | Client定时向Server发送消息通知Server该Client是alive众泰，同时Server会返回Response给Client，如果Client发送Message后没有收到Response，则会自动重定向到其他Server
Server | ZooKeeper集群中的一个节点，提供给Clients所有的服务
Ensemble | 一个可以提供ZooKeeper服务的集群，如果要达到高可用性，至少需要三个节点
Leader | 节点故障时执行自动恢复的节点，启动时选举出的
Follower | 根据Leader的指示执行任务

### 层级的命名空间

![ZooKeeper的层级的命名空间](http://ww4.sinaimg.cn/large/006y8mN6jw1f7ayltmo57j30go0ce0t8.jpg)

层级结构中的每个节点叫做znode, 每个znode维护一个`stat`结构。这个`stat`仅仅提供一个znode的元信息，其中包括版本号(Version number)、行为控制列表(Action Control List, ACL)、时间戳(Timestamp)、数据长度(Data Lenght)。

下面来就实例看一下一个znode有哪些信息。

![](http://ww4.sinaimg.cn/large/006y8mN6jw1f8r0ho7jkhj31320h6ad1.jpg)

这样一看就很明显了吧。

### znode的类型

znode分为三种类型：

- 永久型 永久型节点当客户端断开连接之后仍然存在，默认情况下创建的节点都是永久型节点
- 临时型 只有client保持alive时才存在的节点叫临时节点，当client从ZooKeeper集群断开时，节点被自动删除。所以临时节点不允许有子节点。如果一个临时节点被删除了，下一个合适的节点会填充它的位置。临时节点在Leader的选取中起到重要作用。
- 顺序型 序列型节点可以是永久的也可以是临时的。当一个znode被创建为顺序型时，ZooKeeper在它原来的name后面加上十位的十进制数字。如果两个顺序型节点是并发创建的，ZooKeeper会保证两个节点的name不同。顺序型节点在锁和同步中起到重要作用。

### 会话（Sessions）
一个会话中的请求是按照FIFO的顺序执行的。当一个client连接上一个server，一个会话就创建成功了并且会生成一个session id给client。  
client会按一个时间间隔给server发送heartbeat来保证session的有效性。如果在一个session的生命周期内没有收到client的heartbeat，它就会认为这个client已经死掉了。  
Session超时通常用ms表示。当一个session不管由于什么原因结束时，在session中创建的临时节点都会被删除掉。

### Watches
Watches是用来保证client能在znode上的数据发生变化时收到通知的一种简单机制。client在读取znode的数据时可以设置一个watches给一个特定的znode，当这个znode上的数据或者它的子节点发生变化时都会触发watches给client发送通知。  
watches只会被触发一次，如果client还需要通知，那就需要另外一次的读取操作了。当一个client和server之间的会话过期时，它们之间的连接就断开了，同时watches也会被移除。  

### 工作流程

- client读取数据 client发送一个**读取请求**给ZooKeeper的一个节点，该节点根据请求中的path信息读取**自己数据库中的数据**返回znode的信息给client。所以读取操作在ZooKeeper集群中是很快的。
- client写数据 如果收到请求的是Follower，它会先把请求转发给Leader，由Leader再发送写请求给Followers。只有**大多数节点**正确响应时，写请求才会成功并且返回正确的返回码给client。否则写请求就会失败。这个严格的**大多数节点**被称为**Quorum（法定人数)**。

### ZooKeeper中的节点数量

- 当只有一个节点时，没有大多数
- 只有两个节点，一个出故障时，也没有大多数
- 当有三个节点，有一个出了故障，那2个就是大多数
- 当有四个节点，2个出故障了，那也是没有大多数的

所以，ZooKeeper集群的中节点的数量不要太多，不然写的性能会有下降。同时节点的数量是3/5/7这种奇数，而不要是偶数。

### 小结
看了上面的这么一大套理论，可能还是对ZooKeeper做的事情云里雾里，因为它做的事情太抽象了，好像实际它什么都没做，但又发现好像每个组件比如Kafka/Storm都要和ZooKeeper配合才能用。到底为什么呢？

上面讲过，ZooKeeper其实是一个配置分发服务，也就是具体的应用如Kafka和Storm都是**无状态**的，它本身为了保持**容错**的特性，而容错很重要的一项特性就是应用Down掉之后重启还要能从之前结束时的地方继续。既然是无状态，其实是**自己不存储状态**，那要实现的这个特性肯定是**需要知道**应用Down掉之前的状态的，那么好，我就把状态存在ZooKeeper里。

举个生动的例子，假设有一个很长的（水）槽，Kafka会每秒把一个玻璃球放在槽里，这样的结果就是最先放进去的玻璃球在最前面。而Storm就是**计数工**，（注意**不是搬运工**，因为它数了之后并不会真实的改变玻璃球的位置）极端一点这个游戏是在一个战场上，Storm随时会死掉，它怎么保证它的后来者来到之后马上知道它之前数到哪个位置了？自己当然是不可靠的，因为它死掉之后这个信息就丢失了，所以它**每数一个**就朝ZooKeeper大喊一声（发送写请求）告诉它数到哪个位置了，而Storm又是个健忘症（无状态），刚数完的自己就忘了，更不用说后面来的人了，那么当它把位置信息告诉ZooKeeper之后其实它和自己的后来者就没有区别了，因为不管是谁，在计数之前都需要先去ZooKeeper读取一下前面数到的位置。这样的好处就是每个Storm随时都可以死掉，只要能有新的应用随时可以起来即可。那么存到了ZooKeeper就万无一失了么？考虑前面ZooKeeper处理写请求的特点，它是把相同的信息在集群中所有的机器上都写了一份，即使其中的一台或几台宕掉了，除非在这几台重启之前仅剩的一台也宕掉了，服务是不受影响的。如果全宕掉了，那真的没办法了，你把整个机房的电源拔掉，肯定会丢数据的。

也可以理解为把状态和应用做的解耦。

那么问题来了，为什么是在战场上？为什么好好的一个应用会无缘无故的Down掉呢？这就要从分布式应用的特点说起了。我们知道以前的所谓大型机、小型机都是很大很昂贵的特殊机器，是区别于普通的硬件的，包括CPU、内存、硬盘都是特制的，所以一台机器上百万甚至千万的都很常见，这种机器宕机的几率很小，但如果宕机的话影响也会相当严重。所以可以说这些机器的费用里其实也包含了保险费，因为机器宕机导致的损失，供应商是要负责任的。

但现在不同了，现在是用大量普通（廉价）的机器组成集群来替代之前特殊的机器，既然是普通，那出错的几率当然就更高了，这也就是为什么诸如Storm这些系统在设计之初就特别注重**容错**和**无状态**了。

## ZooKeeper的使用

### 安装和配置

#### 安装
大数据的这套东西安装起来都是很简单，因为都是编译好的包，直接解压之后就可以以默认配置执行了。不过ZooKeeper有点特殊，因为它需要读取的配置文件是`conf/zoo.cfg`，而默认的发行包里面是有个`conf/zoo_sample.cfg`，不过好在只需要重命名一下即可。

```
[root@localhost zookeeper-3.4.8]# cp conf/zoo_sample.cfg conf/zoo.cfg
[root@localhost zookeeper-3.4.8]# bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /root/packages/zookeeper-3.4.8/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@localhost zookeeper-3.4.8]# bin/zkServer.sh status
Mode: standalone
```

> 注意这里的Mode，表示单点模式，区别于集群模式

#### 配置
前面只讲了基础配置，这样的配置是没法跑集群环境的，下面先从默认配置出发，一步一步搭建一个集群环境。
先贴默认配置：

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/tmp/zookeeper
clientPort=2181
#maxClientCnxns=60
#autopurge.snapRetainCount=3
#autopurge.purgeInterval=1
```

1. `tickTime`： ZooKeeper服务器或客户端与服务器之间维持心跳的时间间隔，也就是每个tickTime就会发送一条心跳
2. `dataDir` 顾名思义就是ZooKeeper保存数据的目录
3. `clientPort` ZooKeeper对外提供服务的端口，即客户端通过该端口与ZooKeeper通信
4. `initLimit` ZooKeeper集群中的Leader忍受Follower多少个心跳间隔不发送心跳。从这里的默认配置推算，10个心跳间隔，每个心跳间隔2秒钟，也就是当Leader经过2*10秒还收不到Follower的信条时就认为这个Follower已经挂了
5. `syncLimit` Leader和Follower之间发送消息时，请求和应答的时间长度，默认5，及10秒

从上面的描述就可以看到，从第4条开始就是集群需要的配置了，然而仅仅在每个机器上这样配置并不能变成一个集群，还需要一个重要的配置，形如

```
server.1=c1:2888:3888
server.2=c2:2888:3888
server.3=c3:2888:3888
```

其中的`server.n`中的n表示节点的编号，那么问题来了，编号从哪里定义呢？我觉得这个设计其实不太好，当然我也想不到更好的方式来解决这个问题了。我们还需要在`dataDir`中写入一个名为`myid`的文件，其中填写当前机器的编号，操作如下

```
cd /path/to/dataDir
echo 1 > myid
```

c1的位置是节点机器的hostname或者IP地址，这样写当然还是不行的，因为它们并不知道c1是什么鬼，所以还需要修改`/etc/hosts`，以我当前的本地集群为例，在该文件中添加

```
192.168.1.111 c1
192.168.1.110 c2
192.168.1.112 c3
```

2888是默认的Follower与Leader交换信息的端口，3888是用于选举Leader的端口，当Leader挂了，当然需要选举一个新的Leader来继续它未竟的事业了。

### 启动集群

这时可以在三台机器上同时执行`bin/zkServer.sh start`了。如果看到和前面一样的结果（注意把刚才已经启动的服务先关掉，执行`bin/zkServer.sh stop`），恭喜你成功了一半了。
这时再执行`bin/zkServer.sh status`你会惊奇的发现其中一台会显示

```
[root@localhost zookeeper-3.4.8]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /root/packages/zookeeper-3.4.8/bin/../conf/zoo.cfg
Mode: leader
```

另外两台显示

```
[root@localhost zookeeper-3.4.8]# bin/zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /root/packages/zookeeper-3.4.8/bin/../conf/zoo.cfg
Mode: follower
```

为了证明Leader节点是自动选举的，可以把Leader手动关掉，再分别看看另外两台的`status`。是不是有一个变成了Leader了？

> 几天前写这篇文章时没有问题，但今天继续编写Kafka部分时，因为之前重启过这几台虚拟机，IP变了，重启所有虚拟机之后发现虽然已经在本地启动了ZooKeeper服务，但执行`zkServer.sh status`时总是提示没有运行，而如果你再执行`start`指令，它又会提示说进程已经在运行了。根据[StackOverFlow的答案](http://stackoverflow.com/questions/29909191/zookeeper-it-is-probably-not-running)，把`/root/packages/zookeeper-3.4.8/bin`添加至`$PATH`中即可。

> ```
> $ vim /etc/profile.d/zookeeper.sh
> export ZK_HOME=/root/packages/zookeeper-3.4.8
> export PATH=$PATH:$ZK_HOME/bin
> ```
> 之后执行`source /etc/profile.d/zookeeper.sh`即可直接在系统的任何地方执行`zkServer.sh start`了。


通过动态的选举Leader节点，就解决了**主从系统的单点故障问题**。

### 简单使用

前面说了启动服务，细心的你可能还发现在bin目录里面还有一个zkCli.sh（请自动无视zkCli.cmd，因为那明显是给Windows用的，而我觉得也没有人会在Windows上跑这些服务），这就是ZooKeeper的命令行客户端。

而我要说的只有两个最简单的命令。

#### 1. `ls`
`ls`顾名思义就是查看指定path下的数据，前面我已经演示过了，要注意的一点是**如果`ls`后跟的是一个叶子节点，返回的结果是`[]`**，这时你应该很敏锐的意识到应该换用`get`来操作这个节点从而查看它的详细信息了。

#### 2. `get`
`get`当然就是用来查看指定节点的详细信息用的了。

ZooKeeper提供的借口当然远远不止这两个，但起码到目前为止我还没有用到需要自行调用ZooKeeper接口的地方。因为实际上ZooKeeper是一个很底层的服务，它是用来为Storm和Kafka这类系统提供服务的，而我们通常不直接使用它们。在前两天一次查问题的过程中，发现数据一直在重复写入HDFS，查到了一个症状是ZooKeeper中的offset从一次重启发布之后一直没有更新过，导致系统一直反复读取该时间点之后的数据。这期间也就只用了这两个命令，至于对各种语言的binding，这里就不多说了，如果你要使用ZooKeeper给你的应用提供服务，那也不是看我的这篇文章就能搞明白的：）

Happy Coding!


> 本文大量参考了[https://www.tutorialspoint.com/](https://www.tutorialspoint.com//zookeeper/index.htm)