+++
categories = ["Linux"]
date = "2017-02-09T14:39:09+08:00"
title = "认识正向代理和反向代理"
tags = ["proxy","reverse-proxy"]
isCJKLanguage = true

+++

这两个概念平时经常会提到，但我自己好像对它们两个的区别也不是很明白。刚才找了[一篇文章](http://freeloda.blog.51cto.com/2033581/1288553)，看起来讲述的挺清楚。这里我再用自己的语言描述一下以加深印象。



> 一句话解释：正向代理是为了让防火墙内的用户通过墙外的代理访问原本不可访问的服务；反向代理是为了让防火墙外的用户访问墙内的服务。

## 正向代理

正向代理就是我们平时提到最多的『代理』，而有科学上网需求的同学可能对这个概念更熟悉。原本无法访问墙外资源的我们买或租一台位于墙外的机器，搭上代理，就可以通过代理来访问『不存在的网站』了。

为了描述简单，比如我的代理服务器地址为myproxy.com，我要通过它访问twitter.com。

在这个使用场景下，我在浏览器输入的地址仍是`https://twitter.com`，因为本地配置了代理的客户端，其中配置了代理规则，当命中这个规则时，访问该地址的请求就会通过`myproxy.com`去访问，代理服务器去目标服务器取回我想要的数据后返回给我。

正如前面所说，**客户端必须要有一定配置才能使用正向代理**。因为要保证本来要发往目标服务器的地址会被代理服务器『劫持』。如果我说的不够明白，自行代入一下影梭的用法，就知道我在说什么了。

## 反向代理

正向代理我们个人使用比较多，而反向代理可能就更多的出现在工作中了。我理解有两种情况需要用到反向代理。以VPC（Virtual Private Cloud）为例，如果不太了解VPC的特点，可以简单的理解为真实提供服务的机器都位于一个对外不可见的网络环境中，要让外部客户端访问其中的资源，有两种方法：

1. 给内部机器分配外部可以访问的IP地址
2. 使用『负载均衡』技术

其中第一种也就是偶尔会听到的『绑定多网卡』，一个机器有一个内网地址和一个外网地址。内部地址用于访问内网资源，比如云主机提供商通常会有自己的『镜像源』，从外部访问时很慢或者根本不能访问，而用内网访问相当于直连，速度快到飞起。服务器说到底还是要为外部提供服务，所以它还需要一个外部地址供外部的客户端访问。

这种只需要按照相应的规则配置就可以了，我也只是了解，因为我接触不到这一层。

接触更多的是『负载均衡』，也叫Load-Balance，简称LB。在VPC的网络环境下，它肯定是需要有两个IP的。我本地没有VPC的环境，下面的实验基于本地的Docker环境。

#### 实验条件

1. Ubuntu 16.04 with Docker 1.12.3
2. `sudo docker pull nginx`官方Nginx Docker镜像

#### 实验步骤

##### 直接访问（通过端口转发）

1. 本地建一个目录，用于存放网站资源文件。如`$HOME/Public/web1/index.html`，文件内容如下：

   ```html
   <html>
   	<head>
   		<title>Web1</title>
   	</head>
   	<body>
   		<h1>Content of Web1</h1>
   	</body>
   </html>
   ```

2. `docker run --name web1 -v /home/docker/Public/web1:/usr/share/nginx/html:ro -d -p 8081:80 nginx` 这句命令做了以下几件事：

   1. 给docker容器指定名字web1
   2. 设置一个『共享目录』，从容器中读取`/usr/share/nginx/html`目录实际是**以只读方式**读取宿主本地的`/home/docker/Public/web1`目录。
   3. 设置本地端口8081映射到容器的80端口
   4. 启动Nginx容器

3. 我的宿主机器IP是`192.168.159.3`，所以可以访问`http://192.168.159.3:8081`来检查一下是否成功了

   ![](https://ww2.sinaimg.cn/large/006tKfTcly1fck90lvtwxj30uq04sq3i.jpg)

等等，这种方式绝对是不可接受的。我虽然不用关心容器的IP，但它竟然用了这么个端口，难道还要让用户记住端口号？

##### 反向代理

1. 配置宿主机的Nginx（类比上面提到的VPC中的LB，因为它既能被外部访问，同时也能访问容器实例）

   ```nginx
   server {
   	server_name proxy.ubuntu.com;

   	location / {
   		proxy_pass http://localhost:8081;
   	}
   }
   ```

   上面的配置做了以下几件事：

   1. 给宿主机配置了一个server_name，这个在测试环境不太需要，这里配置了只是因为我的宿主机已经有了好几个vhost。

   2. location段指定当访问`http://proxy.ubuntu.com/`时会将请求重定向到`http://localhost:8081`。这里省略了`listen 80;`主要是懒，因为本来就是默认的。

   3. 通过`http://proxy.ubuntu.com`访问，配置hosts什么的不用我说了吧

      ![](https://ww4.sinaimg.cn/large/006tKfTcly1fck9or3xq6j30js05qdga.jpg)

通过上面的两个例子，我们已经看到了什么是反向代理和它的实际作用了。但实际上它更重要的应用是『负载均衡』。比如一个服务由2台服务器提供，假设两台服务器配置一样，再起两个容器实例吧，这次要把访问日志打出来。首先要看一下容器里默认的Nginx配置：

![](https://ww4.sinaimg.cn/large/006tKfTcly1fck9uzyl4zj317u0410uu.jpg)

执行`docker cp 3bcbb1c5df2b:/etc/nginx/nginx.conf nginx.conf`就把其中一个容器实例的`/etc/nginx/nginx.conf`拷贝到本地的`./nginx.conf`文件了。内容如下：

```nginx

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

> 你也许知道Docker中的服务是不能以daemon方式运行的，那这里为什么没有`daemon off;`的配置呢？因为它在Dockerfile的CMD段指定过了。。。

看倒数第二行我们知道了具体的server配置是位于`/etc/nginx/conf.d`中的，也就是我们俗称的vhosts配置。根据经验，它一定是包含一个`default.conf`的，同样的方式把它拿出来:

`docker cp 3bcbb1c5df2b:/etc/nginx/conf.d/default.conf default.conf`

```nginx
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/log/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

好，从这里我们就看到了Nginx的访问日志所在路径，稍微修改一下：

```nginx
server {
    listen       80;
    server_name  localhost;

    access_log  /var/log/nginx/log/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```

这样就确定了日志的位置。执行下面的指令:

```bash
docker run --name web5 -v /home/docker/Public/web2:/usr/share/nginx/html:ro -v /home/docker/etc/custom.conf:/etc/nginx/conf.d/default.conf -v /home/docker/etc/log/web5.log:/var/log/nginx/host.access.log -d -p 8085:80 nginx
docker run --name web6 -v /home/docker/Public/web2:/usr/share/nginx/html:ro -v /home/docker/etc/custom.conf:/etc/nginx/conf.d/default.conf -v /home/docker/etc/log/web6.log:/var/log/nginx/host.access.log -d -p 8086:80 nginx
```

这样就起了两个容器实例。然后在宿主机的Nginx配置的http段加入以下配置：

```nginx
upstream web_proxy {
		server localhost:8085;
		server localhost:8086;
}
```

然后在`proxy.ubuntu.com`的vhosts中配置

```nginx
server {
	server_name proxy.ubuntu.com;

	location / {
		proxy_pass http://web_proxy;
	}
}
```

重启宿主Nginx就可以见证一个最简单的Nginx负载均衡了。

打开`http://proxy.ubuntu.com/`，每次出现的页面都是这样：

![](https://ww3.sinaimg.cn/large/006tKfTcly1fckby59r15j30mm04wdgb.jpg)

可以多刷新几次，关注下两个log文件的变化，你会发现两个log文件总是一样的，比如这次访问web5，下次一定是web6，偶数次之后两个总是相等的。这是因为在上面upstream的设置中，没有配置weight（权重）的情况下，默认两个server的权重是一样的。

关于Nginx负载均衡的具体分配策略问题，另外开一篇文章来讨论。