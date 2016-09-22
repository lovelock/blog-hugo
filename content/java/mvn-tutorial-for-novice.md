+++
categories = ["Java"]
author = "frostwong"
date = "2016-08-28T14:29:16+08:00"
description = "Maven基础"
draft = false
isCJKLanguage = true
keywords = ["maven", "mvn", "Java"]
tags = ["Java"]
title = "给Java新手看的mvn指南"
type = "post"

+++

## 官方定义
Maven是基于项目对象模型，可以通过一小段描述信息来管理项目的构建、报告和文档的软件管理工具。

## 基本概念
Maven的使用过程中最经常用到的就是依赖管理了，一个依赖也就是一个包，是包含了几个属性的
- `groupId` 通常是公司域名的反写加上项目名，比如`com.unixera.mvndemo`
- `artifactId` 模块名，比如`project1`
- `version` 版本号，经常见到的是形如`1.0.0-SNAPSHOT`这种，即快照版本，还有`RELEASE`等。

## 规定
它规定的目录结构如下：
```
-src 
 -main
  -java
   -packagename
 -test
  -java
   -packagename
 -resource
```

## 基础命令
- `mvn compile` 编译项目
- `mvn test` 测试
- `mvn package` 打包
- `mvn clean` 删除已经生成的测试报告和字节码文件，其实就是删除target文件夹
- `mvn install` 安装jar包到本地目录中

上面这5个过程如果执行后面的，前面的也会自动执行。也就是说后面的命令是依赖前面的命令的。

## 经常遇到的问题
### 1. 间接依赖
A依赖B，B依赖C，那么A就间接的依赖了C，如果要显式的声明A不依赖C，可以在A的pom.xml中加入
```xml

<dependecy>
	<groupId>BgroupId</groupId>
	<artifactId>BartifactId</artifactId>
	<version>1.2.3</version>
	<exclusions>
		<exclusion>
			<groupId>CgroupId</groupId>
			<artifactId>CartifactId</artifactId>
			<version>1.2.3</version>
		</exclusion>
	</exclusions>
</dependecy>
```

### 2. 如何添加依赖
比如项目要使用servlet，那就去[全球中央仓库](http://search.maven.org/)查找包名和相应的版本号,如图所示![](http://ww4.sinaimg.cn/large/7853084cjw1f79btj2nb8j20fl0f9q4b.jpg)。从中复制Apache Maven下面的一段XML粘贴到相应的`<dependencies></dependencies>`中即可。

### 3. 变量的使用
有时在一个pom.xml文件中会看到有`${project.version}`这种写法，那一看就是一个引用，这个东西是在`<properties></properties>`中定义的，比如

```xml
<properties>
    <maven.compile.source>1.5</maven.compile.source>
    <maven.compile.target>1.5</maven.compile.target>
</properties>
```

这样定义了之后在后面就可以用`${maven.compile.source}`来引用了。

### 4. 一个项目下多个模块重复依赖一个包
下面着重说一下如何利用Maven的继承关系简化项目的POM配置。

#### 在项目的根目录下创建pom.xml
还以上面的项目名为例，比如root目录是project,则在project目录里创建pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.unixera</groupId>
	<artifactId>root</artifactId>
	<version>1.0.0-SNAPSHOT</version>
	<packaging>pom</packaging>
	
	<properties>
	    <project.version>1.0.0-SNAPSHOT</project.version>
	    <junit.version>4.10</junit.version>
	    <jmock.version>2.8.2</jmock.version>
	</properties>
	
	<dependencies>
	    <dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>${junit.version}</version>
			<scope>test</scope>
		</dependency>
		<dependency>
            <groupId>org.jmock</groupId>
            <artifactId>jmock</artifactId>
            <version>${jmock.version}</version>
        </dependency>
	</dependencies>
</project>
```

这样就声明了该项目需要依赖junit，那么里面的子项目就用`mvn archetype:generate`来交互式的生成，比如
```
Choose org.apache.maven.archetypes:maven-archetype-quickstart version:
1: 1.0-alpha-1
2: 1.0-alpha-2
3: 1.0-alpha-3
4: 1.0-alpha-4
5: 1.0
6: 1.1
Choose a number: 6: 6
Define value for property 'groupId': : com.unixera.mvndemo
Define value for property 'artifactId': : project1
Define value for property 'version':  1.0-SNAPSHOT: :
Define value for property 'package':  com.unixera.mvndemo: :
Confirm properties configuration:
groupId: com.unixera.mvndemo
artifactId: project1
version: 1.0-SNAPSHOT
package: com.unixera.mvndemo
 Y: :
[INFO] ----------------------------------------------------------------------------
[INFO] Using following parameters for creating project from Old (1.x) Archetype: maven-archetype-quickstart:1.1
[INFO] ----------------------------------------------------------------------------
[INFO] Parameter: basedir, Value: /Users/frost/IdeaProjects
[INFO] Parameter: package, Value: com.unixera.mvndemo
[INFO] Parameter: groupId, Value: com.unixera.mvndemo
[INFO] Parameter: artifactId, Value: project1
[INFO] Parameter: packageName, Value: com.unixera.mvndemo
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] project created from Old (1.x) Archetype in dir: /Users/frost/IdeaProjects/project1
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 35.492 s
[INFO] Finished at: 2016-08-28T14:16:28+08:00
[INFO] Final Memory: 13M/201M
[INFO] ------------------------------------------------------------------------
```
进入`project`目录，打开`pom.xml`，可以看到mvn生成的项目已经默认依赖了junit，那我们来修改一下让它依赖parent所定义的junit。
首先加上

```xml
<parent>
	<groupId>com.unixera.mvdemo</groupId>
	<artifactId>root</artifactId>
	<version>1.0.0-SNAPSHOT</version>
</parent>
```

这样就可以把junit的依赖放心的删掉了，因为它已经认了root做parent，parent的依赖就是它的依赖了。  
那如果parent的某些依赖它并不需要呢？可以在子项目中添加
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.unixera.mvndemo</groupId>
            <artifactId>root</artifactId>
            <version>1.0.0-SNAPSHOT</version>
            <exclusions>
                <exclusion>
                    <groupId>org.jmock</groupId>
                    <artifactId>jmock</artifactId>
                    <version>2.8.2</version>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
</dependencyManagement>
```
这样子项目就不依赖jmock模块了。

当然mvn的使用远远不止这些，这里记录一些目前使用到的，后面如果还继续回写Java的话会再更新一下。