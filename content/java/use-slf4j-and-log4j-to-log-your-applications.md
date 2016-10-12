+++
categories = ["Java"]
date = "2016-09-22T14:16:06+08:00"
description = "是否看了好多文章仍是无法让slf4j正确的运行起来？来看一下吧"
tags = ["Java", "Log"]
title = "slf4j配合log4j来给你的应用打log"
topics = ["Java", "Log"]
+++

最近刚开始接触Java，被折腾的很难受。因为PHP随便找个环境都能执行，只需要把代码传上去就OK，而Java还需要编译、打包，用mvn执行命令，本来不多的功能就高出一下上百兆的包，然后传到服务器上。后来在运维同学的配合下整了一套只需要我把代码提交到Git仓库，系统自动打包并上传到需要执行的机器上的工作流，虽然比之前好了很多，但还是没有PHP开发的过程流畅。

<!--more-->

说了这么多，是因为Java调试起来不容易，或者说成本高。所以就需要尽量在提交代码之前打尽可能多的日志，帮助后面查找问题。找了一圈，发现最通用的Java日志库是slf4j(Simple Logging Facade for Java)。
这名字都不喜欢，什么破玩意儿。而且这玩意儿其实自己并不记录日志，而只是一个日志的Facade，也就是说它是用来**为其他真正执行记日志功能的类库提供一个标准接口**的。我也没有兴趣去研究它的代码，想想也是各种设计模式用的6到飞起了。
下面是记录一下怎么用最简单的方式把它添加到自己的项目中，以log4j为例。

1. 需要添加的dependencies

    ```xml
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.21</version>
    </dependency>

    ```
    后面两个依赖并不是必需的，只需要第一个即可。

2. 在需要记日志的类中添加这样的一些代码片段

    ```java
    package com.unixera.mvn;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;

    public class App 
    {
        private static Logger LOG = LoggerFactory.getLogger(App.class);
        public static void main( String[] args )
        {
            LOG.info("this is a log");
            LOG.debug("This is a log with some information {}", args.toString());
            System.out.println( "Hello World!" );
        }
    }
    ```

执行一下，这时候会发现下面的警告信息。

```
log4j:WARN No appenders could be found for logger (HelloWorld).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
```
虽然把我引导去了一篇FAQ，但看完我还是不知道是怎么回事。关于这个问题的答案一搜一大把，这也说明看了FAQ仍然不明白的人远远不止我一个。  
首先要知道这里appender的意思。它其实就是上面说到的真正执行记录日志工作的类库，在这里就是log4j。那它指出的问题就是log4j需要一个初始化配置，而我们并没有给它指定初始化配置。所以这个appender就不知道该怎们工作。  
其实这也很容易理解，毕竟要记日志，你不告诉它日志级别、记录日志的路径和格式，它怎么能帮你决定这些事情呢？  

『小声说两句，其实这也是我写Java的时候感受最深最痛苦的事情，Java太过于讲究设计模式了，把任务的职责分的太细，导致本来很小的一件事都要绕来绕去引入很多东西，虽然这保证了大量菜鸟写起来不容易出错，但也导致了开发效率的降低和开发者的心情问题。』

言归正传，最简单的办法是需要在main里面新建一个目录`resources`，新建文件log4j.properties，写入

```
log4j.rootLogger=TRACE, file, stdout

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd'T'HH:mm:ss.SSS} %-5p [%c] - %m%n


log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./log/javavillage.log
log4j.appender.file.MaxFileSize=10000KB
log4j.appender.file.MaxBackupIndex=10
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} [%t] %-5p:: %m%n
```

这段配置文件做了以下几件事：

1. 第一行设置了日志级别TRACE，意思是记录TRACE和高于TRACE级别的所有日志，因为TRACE是最低的，所以这里设置的是所有日志都会记录。什么意思呢？log4j有这几个日志级别：TRACE,DEBUG,INFO,WARN,ERROR,FATAL，这几个级别从左到右依次升高。也越来越不详细。也就是说，如果设置了日志级别是INFO，那么`LOG.trace()`和`LOG.debug()`这种语句会被忽略，而后面四中是会正常执行的。
2. 日志输出的位置，这里设置的是文件和标准输出
3. 分别设置了两种输出位置的一些属性，比如输出到标准输出应该是什么样的格式，用什么标准来做文本替换，用什么样的时间格式等等；至于输出到文件的情况就比较复杂了，因为还牵涉到文件的路径，文件大小上限等。具体这个项需要怎么配置我也没有详细的研究，以后如果需要再用到Java并且有需求再说吧。毕竟我不太喜欢Java。需要的可以参考这里[1]。

[1]: https://www.tutorialspoint.com/log4j/log4j_logging_files.htm