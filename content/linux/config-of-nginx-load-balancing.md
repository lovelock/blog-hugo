+++
date = "2017-02-09T18:00:47+08:00"
title = "Nginx负载均衡分配策略详解"
isCJKLanguage = true
categories = ["Linux"]
tags = ["nginx","load-balancing"]

+++

上篇文章配置的负载均衡是『轮询模式』，所以两台后端机器分到的请求数总是一样的。这里就其他分配策略做个详解。

## 分配策略

### 1. `weight`

![](https://ww1.sinaimg.cn/large/006tKfTcly1fcl7d8xu4fj30d503aaa9.jpg)

把前面的配置改成这样，意思是8086端口的这个server被hit的几率会是8085的2倍，实际测试结果如下：

![](https://ww1.sinaimg.cn/large/006tKfTcly1fcl7ee0sglj308802sjrf.jpg)

因为这里面包含前面没有配置weight的数据，所以不太准确，但能说明问题。

### 2. `ip_hash`

这个的作用是把用户的IP映射到固定的一台后端服务器上，可以解决session的问题。这个看起来很不错，但还是无法解决灰度测试的问题。我理解的灰度测试最好是针对一批用户，因为你不能让用户在移动App已经购买的东西到了Web登录后发现没有了吧。所以最好还是根据用户的ID进行Hash。

![](https://ww4.sinaimg.cn/large/006tKfTcly1fcld8wy07zj30e10460sw.jpg)

### 3. `least_conn`

这个功能可以让在有需要长时间才能完成的请求时让请求的分配更公平。简言之就是它可以控制不让单独的一台服务器负载太高，而是会根据其负载动态的分配请求。

![](https://ww4.sinaimg.cn/large/006tKfTcly1fcldjacqwtj30bs03fdg0.jpg)

### 4. `consistent hash`

可以根据请求的URL进行Hash，也就是当用户请求同一个URL时永远都会分配到同一台后端服务器，如果后端服务器提供缓存服务这个功能就很有用了。详细用户见[Nginx Wiki](https://www.nginx.com/resources/wiki/modules/consistent_hash/)，默认是没有编译这个模块的，可以自行编译安装。

### 5. `down`

标记某台应用服务器暂时不参与负载均衡。

![](https://ww2.sinaimg.cn/large/006tKfTcly1fcleinzabwj30dp0370sv.jpg)

### 6. `backup`

这个标记和`down`正好相反，它是在其他所有应用服务器全部繁忙或者处于`down`状态时才会被启用，所以这台机器的压力会最轻。

## 健康状态检查

Nginx会自动把返回报错信息的应用服务器标记为『故障』，然后就不再把新的请求分配给它了，这个相关的配置有:

1.   `max_fails` 报错多少次会被标记为**failed**

2.    `fail_timeout` 超时多少次会被标记成**failed**

     ​