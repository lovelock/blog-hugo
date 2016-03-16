+++
author = "frostwong"
date = "2016-02-01T11:55:20+08:00"
description = "如果你对Yaf的用法迷迷糊糊，请点击"
draft = false
keywords = ["Yaf", "文档"]
tags = ["Yaf", "解惑"]
title = "又一份Yaf文档——写给正在迷惑的你"
topics = ["Yaf文档"]
type = "post"

+++

终于把困扰我很久的问题搞定了，好开心。趁着记忆还热乎，一定要把它记下来。

前几天还和同事抱怨，用Yaf框架的人那么多，但我们身边竟然没有一个人对它的用法很清楚的。真是有些悲哀。几个同事都说鸟哥写的Yaf文档看不明白。

言归正传，就我自己的学习过程来看，如果你要自己搭建一个Yaf环境，遇到的问题可能主要有以下这些：

1. 配置Nginx的rewrite规则
2. 命名空间怎么用
3. 目录结构设置
4. 插件的使用
5. 单Controller多Action配置
6. 多模块配置
7. 更多配置文件

下面就这些问题一一给出解答。

## 配置Nginx的rewrite规则

让我们直接忽略Apache和Lighttpd吧，默认大家都用Nginx。

### Yaf路由规则

如果你看过官方文档，那4种路由规则我就不说了，只说最简单也是默认的`Yaf_Route_Static`。

读了Yaf源码的同学会发现，其实这个规则就是解析`request_uri`，用`/`把它分开，然后用每一部分去匹配Module/Controller/Action/Param。举个例子吧，假设PATH=/foo/bar/doge，更通俗一点，如果你的域名是`http://yaf.dev`，那么这个例子中的完整URL就是`http://yaf.dev/foo/bar/doge`。

路由规则做了以下动作：

1. 解析URL，得到PATH部分
2. 认为`foo`是Module，去`application.modules`配置中找`Foo`（不区分大小写）
3. 如果找到了`modules/Foo`，则继续认为`bar`是Controller，查找`modules/Foo/BarController`；没找到则会认为`foo`是Controller，下面同4
4. 如果找到了`modules/Foo/BarController`，则继续认为`doge`是Action，查找`modules/Foo/BarController/dogeAction`

这样是不是很清晰了？

## 不要这样配

老湿老湿，你不是要说Nginx的rewrite规则怎么配吗，怎么在这讲起了路由规则？

**我是要告诉你不要听那些自作聪明的人(没错就是我)把rewrite规则配错!!!**

也就是说

```
rewrite ^/(.*)  /index.php/$1 last;
```

是绝对正确的，不要看谁谁谁说这不对而写成

```
rewrite ^/(.*)  /index.php?$1 last;
```

这才是错误的。

我的配置:

```nginx
server {
    listen 80;

    root /var/www/yaf.ubuntu.com/public;

    index index.php index.html;

    server_name yaf.ubuntu.com;

    location = /favicon.ico {
        access_log off;
        error_log off;
        log_not_found off;
    }

    if (!-e $request_filename) {
        rewrite ^/(.*\.(js|ico|gif|jpg|png|css|bmp|html|xls)$) /public/$1 last;
        rewrite ^/(.*)  /index.php/$1 last;
    }

    location ~ \.php {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/usr/local/var/run/php-fpm-www.sock;
    }
}
```

上面这个配置主要是加上了对静态文件的支持，如果没有多出来的配置，你会经常在日志中看到找不到Favicon.ico.php找不到的500报错。

## FPM配置

那好，路由规则明白了，rewrite规则好了，毕竟我是PHP脚本啊，得有FPM吧。通常来说，Nginx会给你一个默认的配置，以Debian为例，用apt安装的Nginx自带的default配置`location ~ \.php$`段如下（已删除注释，我喜欢用unix socket，不服来打我啊）

```
location ~ \.php$ {
	include snippets/fastcgi-php.conf;
	fastcgi_pass unix:/var/run/php7.0-fpm.sock;
}
```

这样没问题，但也仅限于`http://yaf.dev/xxxx.php`这种请求。别的都匹配不到啊，别忘了，你请求URL可不是这样的。

所以，只要它是个请求，都要让它可以经过FPM。改成这样

```
location / {
	include snippets/fastcgi-php.conf;
	fastcgi_pass unix:/var/run/php7.0-fpm.sock;
}
```

好了，把这两部分保存一下，重启一下Nginx服务`sudo systemctl restart nginx`，访问一下，哈，爽。

### 静态文件怎么办？

这样配置，访问Action什么的是没问题了，但如果要访问这个域名下的静态文件，css/js啥的，就有问题了，这就需要在上面的`location /`段前面再添加一些匹配到这些静态文件的段，让它找到对应的文件。这里就不再赘述。

## 命名空间怎么办？

### 配置

新时代的我们当然想用命名空间了，起码我是不想写那么长的类名。

可喜的是Yaf只需要一条配置就可以爽快的支持PSR-4规范的命名空间。
可惜的是Yaf的这条配置是全局的，如果你的应用和别的应用公用PHP配置，而别人不用，那就没办法了。

这条配置就是

```ini
;php.ini
yaf.use_namespace=1
```

### 举例

我们都需要Http类吧，假设我写个HttpFoundation\Request类，可以在`library`下新建`HttpFoundation`目录，在其中新建`Request.php`。

```php
// HttpFoundation/Request.php
namespace HttpFoundation;

class Request
{
	public function get()
	{
		echo "I am going to send a GET request";
		exit;
	}
}
```

这样就注册了一个`HttpFoundation\Request`类，在Controller中使用它只需要这样

```php
use Yaf\Controller_Abstract;
use HttpFoundation\Request;

class FooController extends Controller_Abstract
{
	public function barAction()
	{
		$request = new Request();
		$request->get();
		exit;
	}
}
```

### 再远一点？

是不是感受到了鸟哥的高瞻远瞩？其实我们还可以想的更多一点，既然可以自己写个HttpFoundation包了，你有没有想到Composer？因为多数时候这些基础类库都有现成的，用Composer安装就好，既然Yaf已经提供了性能好于autoloader的Yaf_Loader，何乐而不用呢？你甚至都可以引入第三方的ORM，比如Doctrine，第三方模板引擎Twig——对，没错，我就是Symfony的粉丝。

## 目录结构设置

读一下Yaf源代码就特别清晰的看到默认的目录设置了。

```
├── application
│   ├── Bootstrap.php
│   ├── controllers
│   │   ├── Another.php
│   │   └── Index.php
│   ├── library
│   │   ├── Helper
│   │   └── HttpFoundation
│   │       └── Request.php
│   ├── models
│   ├── modules
│   │   └── Foo
│   │       └── controllers
│   │           └── Foo.php
│   ├── plugins
│   └── views
├── conf
│   ├── app.ini
│   └── db.ini
└── public
    └── index.php
```

1. application

	这是应用的主目录。

	想让应用能最简单的跑起来，得有Controller，也就是controllers目录，这里面放的是IndexModule的东西。

	底层用到的一些类库，放在library里。

	数据库操作放在models里。

	多模块放在modules里。像上面的树图里一样，每个module里还有相应的Controller。

2. conf

	存放配置文件。

	我看多数时候是把它放在application里的，但我更倾向于把它放在和application同级目录下。其中的app.ini是Yaf框架的基础配置，里面需要包含Yaf的『唯一一个』必选配置项`application.directory`。

3. public

	存放index.php和静态文件。

	这里是用户可以直接访问的文件。静态文件放在这里最合适不过了。

## 插件的使用

应用上线后Nginx的配置就不太好改了，但我们可以随意修改代码啊，所以如果需要对路由规则做一些修改，可以写个插件。下面以Route插件为例，介绍插件的使用。

### 新建插件

在`application/plugins`目录里新建`Route.php`。

```php
use Yaf\Plugin_Abstract;
use Yaf\Request_Abstract;
use Yaf\Response_Abstract;

class RoutePlugin extends Plugin_Abstract
{
	public function routerStartup(Request_Abstract $request, Response_Abstract $response)
	{
		// some logic here
		$request->setModule('foo');
		$request->setController('bar');
		$request->setAction('doge');
	}
	....
}
```

### 注册插件

```php
// Bootstrap.php

use Yaf\Bootstrap_Abstract;
use Yaf\Dispatcher;

class Bootstrap extends Bootstrap_Abstract
{
	public function _initPlugin(Dispatcher $dispatcher)
	{
		$dispatcher->regiserPlugin(new RouterPlugin());
	}
	...
}
```

这时候你可以试一下，所有请求都会被路由到FooModule/BarController/dogeAction了。

## 单Controller多Action配置

这个问题困扰我最久，最不理解的就是为什么要一个Controller里面只有一个indexAction，然后其他的路由还需要传一个action参数来自己做路由。其实默认的Yaf_Route_Static自己就支持这种写法的。只要保证`application.dispatcher.defaultRoute`的值为空或`static`即可。

## 多模块配置

在目录结构设置一节中已经说过怎么创建多模块。那么多模块有什么用呢？

假设我们要做一个后台，不同模块是需要不同的访问权限的，该怎么办？我的想法是这样，做一个权限控制的plugin，先检查用户身份，然后`$request->getModule()`，如果要访问的是该用户不具有权限的模块，就给跳到一个403页。

## 配置分节

假设这样一个场景，你线上和线下资源配置肯定是不一样的，但又有些是一样的，怎么办？还以`db.ini`为例

```ini
[db]
adapter=pdo_mysql

[prod:db]
host="DB_HOST"
port="DB_PORT"

[dev:prod]
host="DB_HOST_DEV"
port="DB_PORT_DEV"
```

这种情况下可以写一个工具类，因为我觉得这个Yaf\Config\Ini提供的API并不太好用，先要初始化才能用。我做一层封装，不成想竟然发现了很方便的从生产环境/开发环境/测试环境的方法。工具类如下:

```php

namespace Your\Name\Space;
use Yaf\Config\Ini;

class Conf
{
    public static function get($key)
    {
        $filename = PATH/TO/CONF . '/' . explode('.', $key) . '.ini';

        if (is_file($filename) && is_readable($filename)) {
            $config = (new Ini($filename, ENV))->get($key);
            if (is_a($config, 'Yaf\Config\Ini')) {
                $config = $config->toArray();
            }
        } else {
            $config = null;
        }

        return $config;
    }
}
```

上面这个工具类的作用是支持`Your\Name\Space\Conf::get('foo.bar.good');`的方式取值。如果配置文件是这样的

```ini
[test]
foo.bar.good = 'good foo'
foo.bar.better = 'better foo'

[prod:test]
foo.bar.better = 'best foo'
```

那用`Your\Name\Space\Conf::get('foo.bar');`取出的就是包含`good`和`better`的一个数组，如果是`Your\Name\Space\Conf::get('foo.bar.good');`这样，就是`good foo`这个字符串。

重点是对生产环境的切换。注意实例化`Ini`类的时候的那个`ENV`变量，你可以在`/public/index.php`中`define`这个常量，然后又从一个公共的工具类中取出配置，所以只需要在修改`index.php`里面的`ENV`的定义，就可以方便的在各种环境之间切换了。
