+++
categories = ["Java"]
date = "2016-09-22T14:15:41+08:00"
description = "使用maven创建和运行Java应用"
tags = ["Java", "Maven"]
title = "使用maven创建和运行Java应用"
+++

最近这段时间在研究Storm，虽然不是研究源码而是研究使用，也让我这个自认为会写Java HelloWorld的菜鸟感到了深深的无力感。尤其是打开一本书，上面第一行代码就执行报错时我的心情可想而知了。

### 1. 创建应用

因为我最近的使用场景是创建一个普通项目（区别于Web项目），所以直接用`mvn archetype:generate`根据提示如果默认一路点下来会生成一个简单应用的骨架。

> 注意：如果你在哪里看到`mvn archetype:create`这样的写法，而在你的机器上执行出错，不用怀疑，因为你看到的资料太老了，而你用的是新版的maven，按我的写法没有错。

![创建应用的过程][1]

如果需要生成webapp类型应用，比如一个基于SpringFramework的应用就不是831了，而是类似这样

```
mvn archetype:generate -DarchetypeArtifactId=maven-archetype-webapp\
                        -DinteractiveMode=false \
                        -DgroupId=com.unixera.webapp \
                        -DartifactId=spring-example
```
具体再到里面怎么写应用这里就不展开了，如果有时间的话会有相关的文章单独来介绍。

### 2. 打包应用
经过上面的`mvn archetype:generate`命令之后，创建的目录类似这样
![根目录结构](http://ww3.sinaimg.cn/large/006y8lVajw1f84hsklj4oj30g003ejrn.jpg)
如果再往深了看，是这样的
![树状结构](http://ww3.sinaimg.cn/large/006y8lVajw1f84huul3ihj30jq0j8ab7.jpg)
其中最需要注意的就是`pom.xml`这个文件，关于maven的简单使用可以参考我之前写的[给Java新手看的mvn指南](http://unixera.com/java/mvn-tutorial-for-novice/)

写完了应用当然是希望执行它，我们知道Java程序是需要编译成字节码之后才能执行的，当你学Java的HelloWorld时一般是告诉你

```
javac HelloWorld.java
java HelloWorld
```

这没有问题，但问题是当我们用maven管理一个项目时当然就不能这么去操作了，就像写C代码时直接用gcc编译和写Makefile使用make来管理项目是一样的道理。

#### 普通的jar包
1. `mvn compile`  
这时可以尝试在根目录里执行`mvn compile`看看结果。
![](http://ww3.sinaimg.cn/large/006y8lVajw1f84hzh3knjj31bk0l6dmm.jpg)
这相当于执行`javac`，只不过根据mvn的默认配置，它把编译生成的class文件放在了指定的位置，那它究竟生成了哪些文件呢？
![](http://ww1.sinaimg.cn/large/006y8lVajw1f84i209hshj30xu0x241u.jpg)
可以看到，它并没有生成我们希望的**可以发布的jar包**。
2. `mvn package`  
我们知道，其实jar包的本质就是zip，是把项目执行需要的资源全部打包（此处不准确，后面会谈）在一起发布的方式。而`mvn package`执行的就是打包的过程。
![](http://ww1.sinaimg.cn/large/006y8lVajw1f84i6llp9cj31di194k5p.jpg)
这里的输出结果也验证了之前的文章中提到的说法。

> `compile test package install`是一套流程，执行后面的命令时会重复执行前面的命令

再来看`mvn package`生成了哪些文件（只看target目录即可）
![](http://ww3.sinaimg.cn/large/006y8lVajw1f84ibl3z03j30qm0xw78g.jpg)
重点关注红线标注的jar包，这是我们需要的。
![](http://ww4.sinaimg.cn/large/006y8lVajw1f84iezqgjrj313c0h0jvy.jpg)
这就是jar包中的所有东西。你可能知道执行一个jar包需要的命令时`java -jar xxxx.jar`，然并卵，这时执行这个是会出错的。解决方法后面会说。
![](http://ww4.sinaimg.cn/large/006y8lVajw1f84iggkm9nj30vi02at9g.jpg)

#### 使用mvn生成Main-Class

下面来解释一下为什么简单的打包并不能生成可以直接执行的jar包。
首先需要了解一点Manifest的知识[2]。简单来说，就是一个jar包需要Manifest文件中包含指定的内容才可以执行。那我们根据前面的经验，用`mvn package`生成的jar包中是包含`META-INF/MANIFEST.MF`的，实际上jar包运行需要的就是这个所谓的Manifest文件。
打开它看一下
![](http://ww3.sinaimg.cn/large/006y8lVajw1f88hw52bhvj30kg06caax.jpg)

根据Java SE官方文档[3], 一个Manifest文件中至少需要包含`Main-Class`字段才可以使之使用`java -jar`命令执行。

那我们来试着修改一下
1. 把jar包解压：
![](http://ww2.sinaimg.cn/large/006y8lVagw1f88i2b79rzj30zq0d00wv.jpg)
2. 修改`META-INF/MANIFEST.MF`文件，改成
![](http://ww3.sinaimg.cn/large/006y8lVajw1f88i3lfb2fj30pc08c0u5.jpg)
（其中红框中的部分根据自己的实际报名填写）
3. 重新打包
![](http://ww4.sinaimg.cn/large/006y8lVajw1f88id79j1pj311a0cejv9.jpg)
4. 再来执行一下
![](http://ww4.sinaimg.cn/large/006y8lVajw1f88ig6mioaj30ew020q32.jpg)
没错，成功了。

所以我们知道了直接生成的jar包不能执行是因为Manifest中缺少了指定Main-Class的指令。那么既然我们使用了mvn，依赖了pom.xml，那mvn当然是能帮我们直接解决这个问题的，不然要自己每次解压、修改再压缩得累死了。

根据Maven官方文档[4], 在`pom.xml`中添加plugin配置即可
```xml
<dependencies>
...
</dependencies>

<build>
  <plugins>
      <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jar-plugin</artifactId>
      <version>2.6</version>
      <configuration>
        <archive>
          <manifest>
            <mainClass>com.unixera.mvn.App</mainClass>
          </manifest>
        </archive>
      </configuration>
    </plugin>
  </plugins>
</build>

</project>
```
再执行`mvn package`生成的jar包的Manifest中就会带有Main-Class的信息了。

#### jar-with-dependencies

现在我们终于有了一个可以正常工作的类了，不妨给它添加一个依赖吧。比如我在另外一篇文章中提到的slf4j[5]。具体做法可以参考该文章。这里只谈打包的问题。现在我们的pom.xml主体部分应该是这样的：
![](http://ww3.sinaimg.cn/large/006y8lVajw1f88leultmzj30tm10k43x.jpg)
我在App.java中添加了slf4j的用例。按照之前的做法，仍然
![](http://ww3.sinaimg.cn/large/006y8lVajw1f88lgk9cxxj31cs1787fv.jpg)
![](http://ww4.sinaimg.cn/large/006y8lVajw1f88lgxno8yj311u090jv3.jpg)

怎么回事？明明已经引入了需要的包了，为什么会找不到类呢？
问题还是出在jar包的打包方式上。
有PHP使用经验并且用过Composer的同学可能知道，Composer管理依赖的方式非常简单，就是把你的项目需要的代码下载到你项目的目录中，然后通过Composer的Autoload功能把它们加载到使用它们的地方。
但Java不是这样，或者说mvn不是这样。
首先，mvn把所有的依赖的包都放在了用户的根目录下，默认是`$HOME/.m2/repository`，在这个目录下可以看到各个vendor的各种版本的包。但是在目前我们配置的这种模式下，打成的jar包并不会包含这些东西，而且在执行jar包时，也不会去相应的目录去查找需要的包。
**其实就是动态加载**。和写C时用到的.so文件是一个道理。如果希望我们的程序能到处能运行的话，最好把它的依赖都打成.a文件，然后和项目代码打成一个完整的包，所有依赖的类库都在同一个包里就不存在这种问题了。
所以需要引入一个新的打包方式`jar-with-dependencies`。

根据mvn文档[6]，需要引入一个新的过程——Assembly。所以需要新的插件，在pom.xml中添加
![](http://ww3.sinaimg.cn/large/006y8lVajw1f88lybcex8j30xm0ditb1.jpg)
再重新执行上面的过程，可以运行了。

### 3. 执行应用

执行应用就很简单了，直接从java -help就可以获取很多信息，就不展开说了。

`java -cp $CLASSPATH -jar /path/to/jar-with-dependencies.jar`

## 总结
本文旨在引导读者一步一步创建可执行的jar包，包括工程骨架的创建，依赖管理，打包方式，重点谈了下打包方式对生成的jar包功能的影响。引用了不少官方文档，本文只是简略的描述了一下整个过程，更详细的配置可以追随我引用的文档继续研究。

可能有读者会问为什么我用截图而不是代码块的方式来演示，其实我是怕你们偷懒哈哈哈。

[1]: http://ww1.sinaimg.cn/large/006y8lVajw1f840c2rryxj31ey100n83.jpg
[2]: https://docs.oracle.com/javase/tutorial/deployment/jar/manifestindex.html
[3]: https://docs.oracle.com/javase/tutorial/deployment/jar/appman.html
[4]: https://maven.apache.org/shared/maven-archiver/examples/classpath.html#Make
[5]: http://unixera.com/java/use-slf4j-and-log4j-to-log-your-applications/



