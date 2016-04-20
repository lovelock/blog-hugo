+++
title  = "git取消证书校验"
isCJKLanguage = true
date = "2015-06-08T12:32:44+08:00"
topics = ["git"]
+++

在一些较老版本的Linux服务器上如果用Git的话可能会出现证书错误的问题，这是因为Curl自带的证书版本较老。但好像要更新证书bundle挺麻烦，刚刚看到一个很简单的解决方案，直接取消git的证书校验。

```
git config --global http.sslVerify false
```

一切变得美好起来了。
