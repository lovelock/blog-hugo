+++
categories = ["Linux"]
author = "frostwong"
isCJKLanguage = true
date = "2016-01-21T16:21:46+08:00"
description = "Debian删除了所有桌面相关的包之后无法上网了"
draft = false
keywords = ["Linux", "Debian", "网络故障","Network", "Debian"]
tags = ["Linux"]
topics = ["Linux"]
title = "修复因为过量卸载引起的Debian无法上网"
type = "post"

+++

之前在公司的开发机上装了Debian + Gnome3，后来发了rMBP就基本用不上了，把桌面环境卸载了当服务器用吧，但卸载完之后发现无法上网了，忙的一直没有时间整它，今天抽个10分钟搞一搞。

我猜大概是这样的，某些服务——依赖Gnome3桌面环境，被卸载了，所以开机后不会自动启动网络了。

我之前的经验大概是`ifup / ifconfig eth0 up/ dhclient`之类的命令，但发现都不好使了。上Debian的Wiki上看了下，修改

```
# /etc/network/interfaces
auto eth0
iface eth0 inet dhcp
```

重启一下就好了。

我不理解的是为什么之前的方法都失效了，而且改了非要重启才能生效。

很久不研究Linux的这些东西了，出了问题竟然束手无策。但我还是相信，通常Linux上的问题都能通过修改某些文件解决，不像Windows那样改注册表，那我真是无能为力了。

