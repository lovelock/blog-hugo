+++
author = "frostwong@gmail.com"
date = "2016-04-05T12:05:46+08:00"
description = "description"
draft = false
keywords = ["generator", "iterator"]
tags = ["generator", "iterator"]
title = "使用PHP生成器和迭代器"
type = "post"

+++

从开始写PHP就知道迭代器这个东西，当时师傅告诉我用的挺少的，需要的时候再看也不晚，于是就没有放在意上。但他还说这其实也是区分高手和菜鸟的一个标志，那我还是研究一下好了：）

PHP程序员都知道我们最经常用的可能就是`foreach`这个大杀器了。得益于我们**万能的数组**，所以这个大杀器在多数场合都是可以直接用的，只要输入元素是数组类型即可——事实上并不是如此，`foreach`能遍历数组并不是因为它是数组，而是因为数组`implements`了`Iterator`接口。说白了就是只要告诉`foreach`遍历的规则，它就可以执行遍历，而和是否数组无关。




