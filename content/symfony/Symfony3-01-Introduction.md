+++
author = "frostwong"
date = "2016-01-10T13:07:27+08:00"
description = "Symfony简介"
draft = false
keywords = ["PHP", "Symfony3"]
tags = ["Symfony3", "PHP框架"]
title = "Symfony3 In Action——Introduction"
topics = ["Symfony"]
type = "post"

+++

断断续续也看了好几次Symfony框架了，但总没有研究深入，即便如此，每次再看的时候都能比上一次理解的更深入一些。

这次决定把其他的学习都先放一放，深入的研究一下Symfony的设计思想，还有相关的Bundles,Components等等，达到熟练掌握的程度。

言归正传，Symfony是什么？

Symfony是一个『大而全』的PHP框架，一个标准的Symfony安装就几乎可以满足你所有的需求了，包括一个高端的ORM框架doctrine、高效而安全的模板引擎Twig、简单但表达能力强的配置方案yaml、资源管理工具assets等等。如果你说这些东西单拎出来也很强大，我为什么要用它呢？因为Symfony把它们高度解耦，保证单个Bundle的独立性，但又用强大的Service Container把它们强力的结合在一起。如果你知道Laravel，可能也已经知道了Laravel的作者其实是Symfony的粉丝。Symfony的官网还有一篇最初的作者写的『如何写一个PHP框架』，我甚至在想Laravel是不是就是看了这篇教程才写出来的。

不管怎样，Symfony为PHP开发者提供了一个完整的开发环境，让我们很快就能开发出需要的产品。有人会说这种重量级的框架会让程序运行变的缓慢，但在公司用了号称『性能最好的PHP框架Yaf』之后，我觉得我还是更喜欢PHP写的框架（我还没有接触过Phalcon，可能比Yaf好用一点）。为了获得更好的性能带来的是引入外部组件困难，不支持最新的编程范式（PSR），升级维护不容易。作为开发者，我更倾向于用『开发者友好』而不是『机器友好』的框架，这可能也是国外Ruby比较流行的原因吧。

本系列文章一方面是自己学习的总结，也为后来人提供一些捷径，不用去读冗长的官方文档了。

