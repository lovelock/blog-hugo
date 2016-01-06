+++
title  = "PHP设置cookie"
date = "2015-06-19T12:30:13+08:00"
+++

之前一直没有处理过cookie，刚才小小的看了一下，貌似很简单，大概记一下。

首先，客户端要向服务端发送一个请求，服务端接收到请求之后做一下身份校验什么的，然后就可以给客户端种cookie了。

```php
setcookie("key", "value", time()+expire);
```

`expire`是cookie的过期时间，上面的例子用了两部分来说明它，time()获取当前的时间戳，那么`expire`就是你希望这个cookie在当前时刻之后再存活的时间长短。

那么如何手动删除cookie呢？答案是无法直接删除cookie。但可以设置其过期，这样就间接的把它删除了。那怎样设置过期呢？`setcookie`的第三个参数设置的小于当前时间就可以了。

那怎么用cookie呢？

当你把cookie种到了客户端的机器上，它会保存在默认的域下，当客户端访问这个域下的资源时，发送的请求中会带着所有的cookie，然后在服务端用超全局变量`$_COOKIE['key']`就可以直接访问它了。

demo代码如下：

```php
<?php
$username = $_GET['username'];
$passwd = $_GET['passwd'];

if (/* 身份校验成功 */) {
    setcookie("username", $username, time() + 3600);
    echo "cookie set";
}
```

```php
<?php
$username = $_COOKIE["username"];
echo "username: " . $username;
```
