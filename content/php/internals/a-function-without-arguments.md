+++
date = "2017-01-17T15:48:42+08:00"
title = "没有参数的函数（区别于类的方法）"
draft =false
tags = ["PHP","扩展"]
isCJKLanguage = true
categories = ["PHP"]

+++

距离上一次写PHP扩展相关的内容已经很久很久了，这两天又想着写一个真正意义上的扩展了，所以又要重新学习了。

先从写一个最简单的函数说起，从我现在的理解来说这个函数是全局的。比如我要实现一个最简单的`helloworld`函数。

> 这里说一个小插曲，最后不要用dash(-)作为扩展名字的一部分，会出现乱七八糟的麻烦。

如图所示，![](https://ww4.sinaimg.cn/large/006tNc79ly1fbtoytl1g7j31ks0jswjf.jpg)

需要注意的是`PHP_FUNCTION(helloworld)`这段需要放在`const`这段前面，因为相当于前面是定义了`helloworld`这个函数名，后面是把它注册到『可用函数列表』中，如果都没有定义，怎么注册呢，对吧？

至于编译的细节，请看我之前的文章。