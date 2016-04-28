+++
title  = "xhprof方便的插入要检测的代码"
isCJKLanguage = true
type = "post"
date = "2015-11-25T22:10:51"
+++


本文只是给自己搞的一个小封装做个入口。

一次给同事写的一个接口做重构，上线以后发现性能恶化严重，导致了线上的严重问题，我马上想到用xhprof查一下问题，结果由于用的是Yaf框架，里面对一些目录结构做了错误的判断，导致用起来很不方便。所以在问题解决之后我想着把这个xhprof的web目录做成一个独立的vhost，这样用起来就方便了，由于和主体项目所使用的框架无关，也不会受到不良影响。

这里只留个地址，看项目的README吧。有希望能继续改进的小伙伴可以联系我。
[github](https://github.com/lovelock/xhprof-web.git)
[coding.net](https://git.coding.net/lovelock/xhprof-web.git)
