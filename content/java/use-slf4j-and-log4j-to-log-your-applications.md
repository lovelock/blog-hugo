+++
categories = ["Java"]
date = "2016-09-22T14:16:06+08:00"
description = "是否看了好多文章仍是无法让slf4j正确的运行起来？来看一下吧"
draft = false
tags = ["Java", "Log"]
title = "slf4j配合log4j来给你的应用打log"
topics = ["Java", "Log"]

+++

需要添加的dependencies

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.21</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.21</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>

```

后面两个依赖并不是必需的，只需要第一个即可
这时候会发现无穷无尽的warn

```
log4j:WARN No appenders could be found for logger (HelloWorld).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
```

但可惜的是即使看了这篇FAQ，也还是不知道是怎么回事

最简单的办法是需要在main里面新建一个目录`resources`，新建文件log4j.properties，写入

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


todo 需要研究一下`log4j.appender.file.File`的根目录是哪里，看起来好像是classpath，但写一个点就会变成当前目录（项目根目录）
再重新编译，happy coding
