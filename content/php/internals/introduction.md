+++
author = "frostwong@gmail.com"
date = "2016-04-04T22:37:18+08:00"
description = "通过编写扩展深入理解PHP"
draft = false
keywords = ["PHP", "扩展"]
tags = ["PHP", "扩展"]
title = "PHP扩展实战——背景介绍"
type = "post"

+++

这段时间其实在看C语言，但看来看去也不知道学了这些知识用在哪里。作为一名PHP程序员，想要进阶，当然得懂PHP的运行原理。那么，编写扩展就成了认识PHP的必经之路。而PHP的扩展当然是用C写的，这样，也给自己刚刚学的C语言找到了练手的项目。

那么写个什么项目好呢？最好简单一些，但也要能涵盖编写一个完整功能的扩展的方方面面。忘了在哪里看到了[SeasLog](https://github.com/Neeke/SeasLog)，感觉这个项目很符合我的期望，于是上它的issue列表里面找到了一个功能请求，作者还没有着手做，那我就顺手帮忙做了吧，前几天已经被作者合并了，让我的信心也倍增。所以，我决定自己也再写一个类似的东西，主要目的是通过做一个完整的项目，各个击破PHP扩展编写过程中的所有问题。

言归正传，先列上参考文献列表好了。

1. [PHP internals](http://www.phpinternalsbook.com/)
2. [PHP扩展开发及内核应用](https://github.com/walu/phpbook)
3. [PHP at the core: A Hacker's Guide](http://php.net/manual/en/internals2.php)
4. [Yaf源码](https://github.com/laruence/yaf)
5. [PHP源码](http://www.php.net/downloads.php)
6. [Understanding PHP7 Internal articles](https://github.com/laruence/php7-internal)

## 声明：

本系列所描述的PHP扩展相关知识大部分基于PHP 7.0.x，与PHP 5.x不完全兼容，因为列表里的前三个文献都是讲PHP5的，所以我在编写本文时也碰到了不少兼容性问题，都是通过看Yaf源码和PHP源码搞定兼容性的，我没有提到的地方还请在留言中指出。我当然也希望我的这份绵薄之力能帮助弥补目前PHP7相关文档严重不足的情况。