+++
title  = "CentOS7防火墙简单配置"
isCJKLanguage = true
date = "2015-07-12T17:46:00"
+++

防火墙这么个复杂的东西没有一个标准的配置方式真是太烦了，iptables的命令太繁琐，时间长了不用就忘记了，忘记可以记下来，可到了RHEL7竟然又加了个wrapper，叫什么firewall-cmd，我的天，你看看你取了个什么名字吧，不得不说红帽的人真是没有品位。

言归正传，通常我们的工作环境其实也打不到这个7，也不会接触这个东西。

打开一个端口
`sudo firewall-cmd --zone=public --add-port=80/tcp --permanent`
这就添加了一个80/tcp的端口
然后需要`reload`一下
`sudo firewall-cmd --reload`

说它是个wrapper是因为不管是这个`firewall-cmd`还是Ubuntu的`ufw`其实都是对`iptables`系列命令的一个封装，无非就是三步

1. 添加一条记录
2. 把这条记录写入内存
3. 如果下次开机后还要它生效，那就把它写入文件

相应的，`ufw`的操作如下


