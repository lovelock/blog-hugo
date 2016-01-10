+++
author = "frostwong@gmail.com"
date = "2016-01-10T13:19:42+08:00"
description = "Symfony3应用安装"
draft = false
keywords = ["PHP", "Symfony3"]
tags = ["Symfony3", "PHP框架"]
title = "Symfony3学习第二章——应用安装"
topics = ["Symfony"]
type = "post"
+++


## 安装操作系统

操作系统当然需要Linux，不过如果非要在Windows也不是不可以，只是麻烦一点，我没有尝试。至于发行版，我的观点是个人的开发或者测试环境Debian是最合适的，我的习惯是用Debian的Sid也就是unstable版，软件足够新也足够多，能够很方便的满足各种需求。不建议在实体机上安装，最好用VirtualBox等虚拟机软件安装，如果是用VirtualBox则需要选择桥接的网络模式，如果用VMware Workdstation的话就不必了，Nat就可以了。安装过程这里不再详述，最小化安装即可。本文基于Virtualbox + Vagrant + Debian Sid。如果对Vagrant不了解，可以移步[这里](http://lovelock.github.io/2015/11/03/vagrant%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E/)。

## 安装PHP

我用的是最新的PHP7，相关的包如下图
![PHP7.0安装截图](http://7xn2pe.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-01-10%20%E4%B8%8B%E5%8D%881.25.27.png)

> 注意，在我写这篇文章时还没有官方的phpredis和phpmemecached的包，需要自行编译安装。

## 安装Nginx

`sudo apt-get install nginx`

## 安装composer

又到了问候GFW的时候了，不要考虑用官方的安装脚本了，最简单粗暴的方式就是直接[下载](https://getcomposer.org/composer.phar)下来`composer.phar`这个包。然后

```
cd ~/Downloads
sudo chmod +x composer.phar
sudo ln -s /home/frost/Downloads/composer.phar /usr/local/bin/composer
```
最关键的，要配置上[Packagist/Composer中国全量镜像](http://pkg.phpcomposer.com)提供的镜像地址，建议直接 

```
composer config -g repositories.packagist composer http://packagist.phpcomposer.com
```
即可。

## 安装Symfnoy3

```bash
$ sudo curl -LsS https://symfony.com/installer -o /usr/local/bin/symfony
$ sudo chmod a+x /usr/local/bin/symfony
```

执行一下`symfony`，如果安装正常的话就能看到它的帮助信息了。

## 新建Symfony3应用

首先选定一个目录，我的做法是在Debian虚拟机的`/var/www/symfony.dev`目录，然后回到宿主机，修改`Vagrantfile`，添加一行

```ruby
config.vm.synced_folder "./Symfony.dev", "/var/www/symfony.dev"
```

注意，如果这样配置，一定要在`Vagrantfile`同级目录下，新建一个Symfony.dev目录。在当前目录下执行

```bash
$ vagrant halt
$ vagrant up
$ vagrant ssh
```

就又回到了虚拟机了。而且这时你可以在宿主机上用你喜欢的IDE或者边编辑器了。回到`/var/www/symfony.dev`目录，执行

```bash
$ symfony new symfony.dev
```

就会在当前目录下新建一个名为symfony.dev的Symfony项目。

## 配置PHP-FPM和Nginx

Symfony项目可能由于安全上的考虑，使用了依赖操作系统实现的ACL，我也不想关心这个ACL是什么，但如果想简单一点让你的程序马上可以运行，就需要避免这个陷阱。我的做法是修改`/etc/php/7.0/fpm/pool.d/www.conf`中的下列字段

```conf
user = frost
group = frost
listen = /run/php7.0-fpm.socket
listen.owner = frost
listen.group = frost
listen.mode = 0660
```

同时修改`/etc/nginx/nginx.conf`中的

```conf
user frost;
```

注意其中的用户相关的地方我都改成了frost，也就是我在虚拟机中用的用户名。首先要明确，这两个地方的用户设置必须是一致的，通常来说，这样的配置加上相应的vhost设置，应该就可以让Web应用工作了，但对于Symfony这样还不行，或者说因为我用了Vagrant这样还不行。

因为是一个共享目录，所以权限的设置上可能有一些限制，Symfony3要求`symfony.dev/var`目录的权限是777，以便可以正常的写入日志和缓存等文件，但如果我要把它设置成777，宿主机就不乐意了。于是乎就只能让我的当前用户运行Nginx和FPM，这样他们自然对这个目录就有了写权限，变相的解决了这个问题。

注意生产环境可不能这么乱来，千万不要用可以登录的用户名运行这种程序。一般用nobody或者www-data。

```conf
server {
	listen 80;

	root /var/www/symfony.dev/symfony.dev/web;

	index index.php index.html;

	server_name symfony.dev www.symfony.dev;

	location / {
		try_files $uri /app.php$is_args$args;
	}

	location ~ ^/(app_dev|config)\.php(/|$) {
		fastcgi_pass unix:/run/php7.0-fpm.socket;
		fastcgi_split_path_info ^(.+\.php)(/.*)$;
		include fastcgi_params;
		fastcgi_param  SCRIPT_FILENAME  $realpath_root$fastcgi_script_name;
		fastcgi_param DOCUMENT_ROOT $realpath_root;
	}
    # PROD
	location ~ ^/app\.php(/|$) {
		fastcgi_pass unix:/run/php7.0-fpm.socket;
		fastcgi_split_path_info ^(.+\.php)(/.*)$;
		include fastcgi_params;
		fastcgi_param  SCRIPT_FILENAME  $realpath_root$fastcgi_script_name;
		fastcgi_param DOCUMENT_ROOT $realpath_root;
		internal;
        }

        error_log /var/log/nginx/symfony_error.log;
        access_log /var/log/nginx/symfony_access.log;
}
```

注意其中的`fastcgi_pass`一定要和你实际的设置一样。具体查看`/etc/php/7.0/fpm/pool.d/www.conf`。

这时一切都已经准备就绪了，但最好还是再用`bin/symfony_requirements`这个工具检查一下是否所有需求都已经满足了，不然后面可能还会遇到各种各样的问题。值得一提的是刚刚从2.8升级到3.0的时候就已经看到了一个小bug，在执行完这个命令之后，如果全部满足，会提示『满足了Symfony2的运行条件』，明显是忘了改，但现在已经3.0.1了，这个bug还没有修复。。。

上面的配置中可以看到我用的是symfony.dev这个域名，在宿主机上配一下hosts，就以访问[http://symfony.dev](http://symfony.dev)了。这时候你会发现是不能正常执行的，因为在`web/app_dev.php`中制定了如果不是从`127.0.0.1`来的请求就拒绝。所以直接把那段删了就行。

好了可以进行下一步了。

