+++
title  = "jQuery动态生成元素"
isCJKLanguage = true
date = "2015-05-05T23:14:20+08:00"
+++

这些天做的项目中频繁遇到类似的问题，比如根据ajax返回的结果动态生成一个table或者select的options。这里做一个简单的总结。

## 生成元素

1\. 生成table

<iframe width="100%" height="300" src="//jsfiddle.net/u8aknbe7/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

2\. 生成select

<iframe width="100%" height="300" src="//jsfiddle.net/frostwong/uvgpew9c/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

## 获取元素

1\. 获取select的值

这个问题当时花了我不少时间，我们知道，在一个select中，带有selected标签的那个是不用点开就可以看到的那个，解决这个问题就要从这里入手。如果在option列表里没有手工指定哪个是选中的，那么默认是第一个。

<iframe width="100%" height="300" src="//jsfiddle.net/frostwong/Lxb4sgpu/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

2\. 遍历一个table中tbody的所有元素

<iframe width="100%" height="300" src="//jsfiddle.net/frostwong/f68y04c7/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>


