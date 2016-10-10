+++
date = "2016-10-10T17:53:14+08:00"
title = "在本地单机部署Hadoop/Storm运行环境"
categories = ["Java"]
tags = ["Storm", "Kafka", "Java"]

+++

## 前言

由于要在小组内做一个关于Storm的分享，涉及到我负责开发的大数据项目，本着开放的原则，把我做的准备工作记录下来，提前发给可能参会的同事。  
要演示的项目目前为止用到了Apache的多个项目，包括Kafka, Storm, Hadoop(HDFS, Hive), ZooKeeper等，项目刚刚起步，很多基础设施还不完善，比如现在是在本地开发完成之后直接部署到线上环境的，这次演示可不能直接在线上环境做了，故而在本地的台式机上部署了一下**伪集群**，用来作为演示和以后开发测试用的环境。

## 准备工作

说明：

1. 以下工作全部基于Ubuntu 16.04。用其他的发行版或版本理论上应该都是可行的，可能有些命令需要微调。  
2. 所有需要执行的命令前面都有一个`$`，表示的是Bash的命令提示符。
3. 下载安装包时我使用了速度相对较快的国内镜像，如果你对此有任何异议，可以自行去中心镜像站点下载。
4. 默认情况下文中提到的所有如StormUI等控制后台访问的路径都是localhost，如果你需要从Linux主机外部访问，需要iptables放行相应的端口。在本文中，用的Web端口有三个，具体如下表：
    
    | 服务描述 | 端口号
    | :---|---:
    |Storm UI | 8080
    |Hadoop UI | 50070
    |Hadoop Applications | 8088

    以Ubuntu为例，需要执行以下命令
    
    ```bash
    $ sudo ufw allow 8080/tcp
    $ sudo ufw allow 50070/tcp
    $ sudo ufw allow 8088/tcp
    $ sudo ufw reload
    ```

    CentOS/RHEL 7.0以下可能需要直接操作iptables，7.0以上可以使用`firewall-cmd`进行类似的操作。

### 1. 创建独立的用户并赋予合适的权限

```
$ sudo useradd hadoop
$ sudo passwd hadoop
$ sudo chsh hadoop
$ sudo mkdir /home/hadoop
$ sudo chown -R hadoop:hadoop /opt/apache
$ sudo chown -R hadoop:hadoop /home/hadoop
$ su - hadoop
$ ssh-keygen
$ cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
```

FAQ：

1. 为什么需要chsh并且手动为用户创建家目录？  
    不知道为什么在 Ubuntu 环境中执行`useradd`命令之后并不会给新建的用户创建家目录和设置shell，可能是为了安全吧，要稍微麻烦一些。
2. 为什么不给新用户赋予root权限？  
    这里是测试环境倒还好，如果你需要搭建生产环境，记住千万不要给hadoop用户赋予root权限，它需要哪些目录的权限就单独赋予即可，它需要运行的端口都是1024以上的，都不需要root权限。如果需要，执行` sudo gpasswd -a hadoop sudo`即可。
3. 为什么要做一个密钥认证？  
    这是因为在后续的步骤中会有**从本地用户登录本地用户**的需求，也就是执行了`ssh hadoop@localhost`这个命令，如果不做密钥信任，会需要多次输入密码。
4. 为什么会有一个`/opt/apache`目录？  
    往下看。

### 1. 下载所需安装包

1. 下载Java  
    到[Java官网](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载适用于Linux的Java安装包（这里下载了jdk-8u101-linux-x64.tar.gz）。
2. 下载maven  
    点击[阿里云镜像站](http://mirrors.aliyun.com/apache/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.zip)下载最新版Maven。
3. 下载Apache的Storm Hadoop Kafka ZooKeeper  
    [点击下载Storm-0.10.2](http://mirrors.aliyun.com/apache/storm/apache-storm-0.10.2/apache-storm-0.10.2.tar.gz)  
    [点击下载Hadoop-2.6.4](http://mirrors.aliyun.com/apache/hadoop/core/hadoop-2.6.4/hadoop-2.6.4.tar.gz)  
    [点击下载Kafka_2.10-0.8.2.2](http://mirrors.aliyun.com/apache/kafka/0.8.2.2/kafka_2.10-0.8.2.2.tgz)  
    [点击下载ZooKeeper-3.4.6](http://mirrors.aliyun.com/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz)  

### 2. 创建所需目录

把所有和此项目相关的文件都放在统一的位置。

`$ sudo mkdir -p /opt/apache/{jdk, storm,hadoop,kafka,zookeeper}`

### 3. 解压所有安装包，将相应的安装包放在对应的位置

```
$ sudo mv jdk1.8.0_101/* /opt/jdk/.
$ sudo mv apache-storm-0.10.0/* /opt/apache/storm/.
$ sudo mv hadoop-2.6.4/* /opt/apache/hadoop/.
$ sudo mv kafka_2.10-0.8.2.2/* /opt/apache/kafka/.
$ sudo mv zookeeper-3.4.6/* /opt/apache/zookeeper
```

### 4. 配置环境变量

在~/.bashrc中添加下面的片段

```bash
export JAVA_HOME=/opt/jdk
export APACHE_HOME=/opt/apache
export STORM_HOME=$APACHE_HOME/storm
export ZK_HOME=$APACHE_HOME/zookeeper
export KAFKA_HOME=$APACHE_HOME/kafka

export HADOOP_HOME=$APACHE_HOME/hadoop
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_INSTALL=$HADOOP_HOME

export PATH=$PATH:$JAVA_HOME/bin
export PATH=$PATH:$STORM_HOME/bin
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export PATH=$PATH:$ZK_HOME/bin
export PATH=$PATH:$KAFKA_HOME/bin
```

之后执行`$ source ~/.bashrc`

#### 5. 安装Java

Apache的这些东西都是直接放在对的地方就可以运行的，但Java需要稍微的配置一下，因为可能你的机器上已经装了OpenJDK。

`$ sudo update-alternatives --install /usr/local/bin/java java /opt/jdk/bin/java 10000`

这时再执行`$ java -version`验证一下是否使用的时最新安装的JDK。

## 2. 启动服务

### 1. 启动ZooKeeper

```
$ cp $ZK_HOME/conf/zoo_sample.cfg $ZK_HOME/conf/zoo.cfg
$ zkServer.sh start
```

### 2. 启动Storm

```
$ storm nimbus
$ storm supervisor
$ storm ui
```

现在可以访问[Storm UI](http://localhost:8080)了。

### 3. 启动Kafka Broker

```
$ cd $KAFKA_HOME
$ bin/kafka-server-start.sh -daemon config/server.properties
$ bin/kafka-topics.sh --list --zookeeper localhost:2181
$ bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 10 --topic storm-demo-topic
Created topic "storm-demo-topic".
$ bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic storm-demo-topic
Topic:storm-demo-topic  PartitionCount:10       ReplicationFactor:1     Configs:
        Topic: storm-demo-topic Partition: 0    Leader: 0       Replicas: 0     Isr: 0
        Topic: storm-demo-topic Partition: 1    Leader: 0       Replicas: 0     Isr: 0
        Topic: storm-demo-topic Partition: 2    Leader: 0       Replicas: 0     Isr: 0
        Topic: storm-demo-topic Partition: 3    Leader: 0       Replicas: 0     Isr: 0
        Topic: storm-demo-topic Partition: 4    Leader: 0       Replicas: 0     Isr: 0
        Topic: storm-demo-topic Partition: 5    Leader: 0       Replicas: 0     Isr: 0
        Topic: storm-demo-topic Partition: 6    Leader: 0       Replicas: 0     Isr: 0
        Topic: storm-demo-topic Partition: 7    Leader: 0       Replicas: 0     Isr: 0
        Topic: storm-demo-topic Partition: 8    Leader: 0       Replicas: 0     Isr: 0
        Topic: storm-demo-topic Partition: 9    Leader: 0       Replicas: 0     Isr: 0
$ kafka-console-producer.sh --broker-list localhost:9092 --topic storm-demo-topic
$ kafka-console-consumer.sh --zookeeper localhost:2181 --topic storm-demo-topic --from-beginning
```

> 注意前面有些命令仅仅是为了测试kafka broker运行的情况。

### 4. 启动Hadoop(HDFS)

#### 1. 修改Hadoop的配置文件

在`$HADOOP_HOME/etc/hadoop`目录下有5个文件需要修改:

1. `core-site.xml`

    ```xml
    <configuration>
        <property>
            <name>fs.default.name</name>
            <value>hdfs://localhost:9000</value>
        </property>
    </configuration>
    ```

2. `hdfs-site.xml`
    
    ```xml
    <configuration>

        <property>
            <name>dfs.replication</name>
            <value>1</value>
        </property>

        <property>
            <name>dfs.name.dir</name>
            <value>file:///home/hadoop/hadoopinfra/hdfs/namenode</value>
        </property>

        <property>
            <name>dfs.data.dir</name>
            <value>file:///home/hadoop/hadoopinfra/hdfs/datanode</value>
        </property>

    </configuration>
    ```

3. `yarn-site.xml`
    
    ```xml
    <configuration>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
    </configuration>
    ```

4. `mapred-site.xml`

    ```xml
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
    </configuration>
    ```

5. `hadoop-env.sh`
    把hadoop-env.sh中的${JAVA_HOME}替换成路径,这里是`/opt/jdk`，因为貌似会找不到正确的`JAVA_HOME`


#### 2. 验证是否安装成功

```bash
$ hadoop namenode -format
$ start-dfs.sh
$ start-yarn.sh
```

执行上面三行语句，观察有没有明显的报错信息。

[检查HadoopUI](http://localhost:50070)运行是否正常。  
[检查Hadoop Applications](http://localhost:8088)(我自己取的名字)运行是否正常。  

至此已经搭建了一个可以运行的hadoop环境，可以移步这里查看关于Storm入门分享详情。
