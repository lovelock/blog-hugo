+++
isCJKLanguage = true
tags = [
  "hdfs",
  "hadoop",
  "kafka"
]
draft = false
categories = [
  "bigdata",
]
date = "2017-01-11T14:27:46+08:00"
title = "清理HDFS的脏数据"

+++

周末同事在群里用由我维护的Hive里查数据，发现完全对不上了，我简单查了一下，发现是预计下一期上的日志格式提前上了，而我的Topology还没有跟上，导致数据字段完全错乱。我马上停掉相应的Topology，避免产生更多的脏数据，周一到公司在给新版的Topology做了严格测试准备上线。

接下来面对的就是怎么把脏数据清除掉，把被错误处理的数据重新处理一遍，并且后续的数据不受影响。

## 步骤

### 1. 找到脏数据的offset，最好向前移动一些，以保证所有脏数据都能被处理

在数据处理节点（我们这里是core节点）上找到kafka-broker的安装目录，执行

```bash
sh bin/kafka-simple-consumer-shell.sh --topic online_userbehaviortrack --offset -2 --broker-list 10.68.160.52:6667,10.68.160.53:6667,10.68.160.54:6667 --print-offsets --partition 9
```

上面的命令会输出类似 

```
next offset = 3381
{"message":"2017-01-11T14:40:55+0800`1484116855552`
```

的结果，从中可以很详细的看到每条message的offset，结合管道和less就可以根据时间找到出现脏数据的offset，然后确定一个时间点，比如下午2点，在每个partition中分别找到2点之前的最后一个offset，找个地方记下来。

### 2. 到ZooKeeper中找到每个对应的Partiton，用步骤1的结果覆盖Partiton中的offset字段

因为我的KafkaSpout是这么写的`SpoutConfig kafkaSpoutConfig = new SpoutConfig(brokerHosts, topic, "/" + topic, client_id);`，所以在我的ZooKeeper中我应该去`/mytopic/mytopologyname/`中找到所有的partiton，这个需要根据你自己的代码来确定。

以partition_9为例，结果如下：

```
[zk: localhost:2181(CONNECTED) 2] get /mytopic/fe_analyze/partition_9
{"topology":{"id":"c75ac929-1a8e-4958-a637-ead9a95436ca","name":"mytopologyname"},"offset":3384,"partition":9,"broker":{"host":"kmr-9a387314-gn-6bb9c438-core-1-002.ksc.com","port":6667},"topic":"mytopic"}
cZxid = 0x1017bd837
ctime = Mon Dec 26 15:06:19 CST 2016
mZxid = 0x101cbb4ec
mtime = Wed Jan 11 14:49:40 CST 2017
pZxid = 0x1017bd837
cversion = 0
dataVersion = 2173
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 223
numChildren = 0
```

因为我们只需要设置partiton的offset值，而它接收一个JSON类型的值，所以就需要这样

```
[zk: localhost:2181(CONNECTED) 4] set /mytopic/mytopologyname/partition_9 {"topology":{"id":"c75ac929-1a8e-4958-a637-ead9a95436ca","name":"fe_analyze"},"offset":3386,"partition":9,"broker":{"host":"kmr-9a387314-gn-6bb9c438-core-1-002.ksc.com","port":6667},"topic":"mytopic"}
cZxid = 0x1017bd837
ctime = Mon Dec 26 15:06:19 CST 2016
mZxid = 0x101cbba18
mtime = Wed Jan 11 14:54:29 CST 2017
pZxid = 0x1017bd837
cversion = 0
dataVersion = 2176
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 223
numChildren = 0
```

把其中的offset的值替换成在步骤1中拿到的partiton 9的offset值。

### 3. 写脚本删除HDFS中对应Topology2点以后的所有数据

我的设计是每天分成24个partition，类似(yyyymmddhh=2017011010)这种，所以就可以写个类似这样的脚本：

```bash
#!/usr/bin/env bash

DAY=$(date -d "-1 day" +%Y%m%d)

for HH in {00..23}
do
    hdfs dfs -rm -r -f /path/to/topology/yyyymmddhh=${DAY}${HH}
done
```
这个脚本可以删除前一天的所有24个partiton的数据。

### 4. 因为新的Topology对应的Hive table也有变化，所以需要先修改Hive的表结构

Hive表结构的修改和MySQL的差不多，不过还是有些差别：比如新添加的列只能放在所有真实列的最后，partiton伪列的前面。

`alter table xxxx add columns (string coln, string colm);`

注意是**`columns`**而不是`column`。而且也不能指定after哪个column。

### 5. 重新添加所有受影响的partiton到Hive table

因为我们这里用的是external table（当我提到HDFS的时候你就应该知道了），所以所有新数据对应的partiton都应该删除后重新添加，以yyyymmddhh=2017011020为例，

- 删除partition `alter table xxxx drop if exists partiton (yyyymmddhh= 2017011020);`
- 添加partition `alter table xxxx add if not exists partition (yyyymmddhh= 2017011020);`

这样就可以对新加进来的数据进行搜索了。你可以试一下，如果少了这步操作，在新的Hive table里新加的列的值会全部是NULL。

## 总结

步骤很多，但操作还是挺简单的，而且需要细心，不能出错，尤其这里每一步都是线上的操作，一次微小的出错都可能酿成事故。


1. 找到每个partiton出现脏数据时的offset
2. 把当前每个partiton的offset设置成出现脏数据时的offset
3. 清除出错的脏数据
4. 修改Hive table（如果需要）
5. 删除并重新添加受影响的partition(如果需要)
