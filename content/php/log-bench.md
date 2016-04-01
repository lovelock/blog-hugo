+++
author = "frostwong@gmail.com"
date = "2016-04-01T16:33:19+08:00"
description = "seaslog 和 error_log性能大比拼"
draft = false
keywords = ["error_log", "seaslog"]
tags = ["PHP", "log"]
title = "seaslog 和 error_log性能大比拼"

+++

今天花了点时间测试了开源项目[seaslog](http://neeke.github.io/SeasLog/)和PHP内置的error_log的性能。本文不涉及该扩展的安装和使用，如果对其不了解，可移步[官网](http://neeke.github.io/SeasLog/)。

> 项目的描述有语法问题"A effective ..."，我提醒了原作者，然而并没有被理会。。。

## 测试环境

项目 | 属性 
---|---
CPU | Intel(R) Xeon(R) CPU  E5520  @ 2.27GHz 8核
内存 | 48G
PHP | 7.0.1
Nginx | 1.2.7

## 测试代码

```php
<?php

error_log("I am testing performance of error_log" . PHP_EOL, 3, __DIR__ . '/error_log.log');
SeasLog::debug("I am testing performance of seaslog");
```

## 结果数据

每种方式测试5次，请求次数1000次，并发量分别是1, 10, 100, 1000。分别记录每次测试的QPS。

1\. error_log

并发量     |        1    |    2     |    3     |    4    |    5
---|---|---|---|---|---|
1 | 2430   |2579    |2685    |2484    |2622
10              | 7303    |7844    | 5892    | 11739   | 9002
100             | 11763   | 6107   | 6921    | 9258    | 11999
1000            | 324     | 891    | 324     | 889     | 883

2\. seaslog

并发量     |        1    |    2     |    3     |    4    |    5
---|---|---|---|---|---|
1               | 2147 |   2071  |  2130   | 2123    | 2039
10              | 7415 |   9438    | 6901    | 6445    | 6047
100             | 7770 |  9389    | 6852    | 5806    | 6483
1000            | 890  |  324     | 891     | 760     |322

## 测试结果图

![error_log性能测试结果](http://7xn2pe.com1.z0.glb.clouddn.com/errorlog.png)
![seaslog性能测试结果](http://7xn2pe.com1.z0.glb.clouddn.com/seaslog.png)

## 结果分析

1. error_log的性能总体优于seaslog，但并没有压倒性优势
2. 在并发较高时二者都出现急剧性能下降，程度相当。怀疑瓶颈已经不在写日志，而在Nginx的处理能力了（待验证）。

## 总结

1. seaslog使用起来更简单，不需要多层封装
2. error_log输出的格式比较单一，如果要加上日期、IP等信息，一定会引入很多PHP函数调用，导致性能损失。但seaslog在这方面就有很大的想象空间，在扩展中计算时间、获取IP、详细的debug信息都是可能的。不过现在并没有加入这些功能。
3. seaslog提供了类似PDO的插值方式，使用起来更方便
4. seaslog可以自定义配置多
5. 一个细节，error_log在type=3时并没有在message后面加上换行符，需要自行添加，也就是说每次都要有一个字符串拼接，这在seaslog中得到了改进

我之前看到有人说seaslog的日期格式太固定，因此我fork了一份代码加上了配置日期格式的功能，作者到现在都没有合并进主干呢。而且我觉得现在这个功能有些简陋，比如我前面说的第2点，请求的一些基本信息如果从扩展层面直接取到，就不需要再在外层调用PHP函数或通过超全局变量获取了，既简化了外层使用的方式，又提高性能。但作者貌似也没有继续增加功能的意思，可能作者并不想在扩展层面做太复杂的事情，要保持这个项目的简单、纯粹。

我还是自己再维护一份好了。


