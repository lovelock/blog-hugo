+++
categories = ["Java"]
date = "2016-09-22T14:15:41+08:00"
description = "使用maven创建和运行Java应用"
draft = true
tags = ["Java", "Maven"]
title = "使用maven创建和运行Java应用"
+++

1. 创建应用

因为我最近的使用场景是创建一个普通项目（区别于Web项目），所以直接用`mvn archetype:generate`根据提示如果默认一路点下来会生成一个简单应用的骨架。

> 注意：如果你在哪里看到`mvn archetype:create`这样的写法，而在你的机器上执行出错，不用怀疑，因为你看到的资料太老了，而你用的是新版的maven，按我的写法没有错。

如果需要生成webapp类型应用，比如一个机遇SpringFramework的应用就不是831了，而是类似这样

```
mvn archetype:generate -DarchetypeArtifactId=maven-archetype-webapp\
                        -DinteractiveMode=false \
                        -DgroupId=com.unixera.webapp \
                        -DartifactId=spring-example
```

2. 打包应用

一般需要把应用和它的依赖打到同一个包里面，所以就需要配置jar-with-dependencies
然后配置一下execute single

3. 执行应用

`java -cp $CLASSPATH -jar /path/to/jar-with-dependencies.jar`

mvn生成的还需要在pom.xml中配置主class




