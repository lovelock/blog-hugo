+++
categories = ["Ios"]
title  = "iOS8及以下使用NSURLConnectionDelegate实现网络请求"
isCJKLanguage = true
date = "2015-07-13T20:20:18"
draft = false
topics = ["iOS"]
tags = ["iOS"]
+++

花了点时间把iOS的网络请求算是有了一点肤浅的认识，在这个夜深人静的晚上记录以下。

首先，用到的类有`NSURLRequest`和`NSURLConnection`，遵循的协议`NSURLConnectionDelegate`。

`NSURLConnectionDelegate`在iOS9中已经废弃了，取而代之的是`NSURLSessionDelegate`，由于我这次适配的是iOS8.3，所以抽时间再写关于后者的用法。

简明的说一下流程：
![iOS发送网络请求简单流程图](http://7xn2pe.com1.z0.glb.clouddn.com/iOS-network.jpg)

发送请求这步其实经常会用到，但作为一款App，它需要的通常不是**让服务器做什么改变**，而是**从服务端获取信息**。因此，发送网络请求其实最重要的是怎样处理请求返回的结果。

在图中用等宽字体显示的函数就是`NSURLConnectionDelegate`定义的方法。

1. `connection:didReceiveResponse:`

    这个方法也就是对服务器的响应做一个判断，通知该类（这里不知道应该怎么描述，是通知谁？）发送的请求服务器端已经给出响应。这时需要做初始化的工作了。我发发送请求就是为了得到数据，那么数据存在哪里呢？从`connection:didReceiveData:`的函数定义可以看到返回的数据是在这个函数的参数中，就需要一个可以全局访问的变量来保存数据，以在其余的流程里继续使用。而初始化又有问题，如何保证你这个全局变量不是已经被赋值呢？
    
    判断是否为NULL，如果为NULL，则初始化一个新的变量，否则，将变量的程度设置为0，其实也就是清空变量中存储的值。
    
2. `connection:didReceiveData:`
    
    这个方法是对服务器返回的数据做处理。其实就是一个简单的赋值操作，将返回值赋给上面提到的变量。

3. `connectionDidFinishLoading`
    
    这个方法才是最重要的，顾名思义，它用来处理最终你获取到的数据要干什么。比如把它赋给某个Label显示出来了，比如再次发送一个请求了。这里我只是要把整个流程走通，所以只是简单的把它的返回值赋给了当前视图中的一个Label。
    
提一下一个小的技巧
如何在日志中看到请求返回的结果？

要知道，返回的结果是一个`NSData`数据类型，无法在NSLog函数里打印（其实是可以打印的，不过打出的结果是16进制的字符，人眼也看不懂），但如果你只是想比较一下两次的请求返回值不同，那么可以直接用

```
NSLog(@"%@", data);
```

而如果要具体的看到返回值的内容，则要做进一步的处理

```
NSString *strData = [NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
NSLog(@"%@", strData);
```

OK，简单的就是这样。


