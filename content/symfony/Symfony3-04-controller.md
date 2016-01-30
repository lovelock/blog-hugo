+++
author = "frostwong"
date = "2016-01-11T23:51:43+08:00"
description = "Symfony3 框架 Controller"
draft = true
keywords = ["PHP", "Symfony3"]
title = "Symfony3学习第四章——Controller"
topics = ["Symfony"]
type = "post"
+++

在Symfony中，Controller是很薄的一层，逻辑不会很复杂，主要就是接受Request，用Request的参数取得资源，放在Response中返回给Client。

## 初识Controller

```php
<?php

namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Component\HttpFoundation\Response;

class HelloController extends Controller
{
    /**
     * @Route("/hello")
     */
    public function helloAction()
    {
        return new Response('Hello, world!');
    }
}
```

它做的事情很简单，就是返回给调用方一个`Hello, world!`字符串。在这个过程中又发生了什么呢？

## Controller工作流

1. 所有的Request都经过一个单文件入口，在这里是app_dev.php（生产环境是app.php）来启动应用。
2. Router读取Request的信息，找到匹配的Controller。
3. Controller中对应的Action执行并返回Response。
4. Http Headers和Response被返回给客户端。

## Controller中的Request

常见的GET请求通常会带一长串的query_string，这时候可以在Action的最后一个参数位置传入Request，从Request的实例中取出相应的参数。

```php
<?php

namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class HelloController extends Controller
{
    /**
     * @Route("/hello")
     */
    public function helloAction(Request $request)
    {
        $name = $request->query->get('name', 'world');
        return new JsonResponse(array('welcome' => "Hello, {$name}!",));
    }
}
```

那你肯定要说了，GET参数这么搞没问题，那POST参数呢？这里要提一点，POST和GET参数的『容器』是有约定俗成的叫法的，比如Request实例的`query`属性就是`$_GET`，而`request`属性就是`$_POST`。知道了这个，就可以理解

```php
<?php

namespace AppBundle\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\Controller;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class HelloController extends Controller
{
    /**
     * @Route("/hello")
     */
    public function helloAction(Request $request)
    {
        $name = $request->query->get('name', 'world');
        $foo = $request->request->get('foo', 'bar');
        return new JsonResponse(array('welcome' => "Hello, {$name} and {$foo}!",));
    }
}
```

> 注意：上面从Request中取值的方法`get`的第二个参数表示**如果没有传入相应参数时的默认值**。强调这一点是因为它**不是该值为空时的默认值**。也就是说，如果你的query_string是`?name=`或者`?name`，它都不会把`world`赋给`$name`，当且仅当query_string中没有出现`name`字段时才会。

## 重定向

经常会出现由于身份认证等原因跳转页面的情况，Symfony提供了很方便的方法，终于可以不用人肉`header('Location: http://blah.blah.com');`了.

1. 跳转到站内命名路由
    
    ```php
    return $this->redirectToRoute(
            'hello',
            array(
                'name' => 'frost',
                'foo' => 'blah',
                ),
            301
        );
    ```
    
    这里用到了`redirectToRoute`方法，该方法接受三个参数，
    
    * 路由名称
    * 参数列表
    * HTTP状态码（默认302）

2. 跳转到站外（外部URL）
    
    这个就简单暴力了，直接
    
    ```php
    return $this->redirect('http://google.com', 301);
    ```
    
    同样，`redirect`方法接受两个参数，
    
    * URL地址
    * HTTP状态码（默认302）

    注意这里就没有参数了，因为跳转到站外已经不受我们控制了，于情于理参数也不应该作为`redirect`方法的参数，顶多加在URL后面罢了。
    
## 渲染模板

终于到了有意思的时刻了。

前面讲了那么多，最终也不可能就给用户看这个啊，还是需要页面来展示我们的结果的。

Symfony默认使用优雅的Twig模板引擎，Twig也是由Sensio开发的。当然，既然Symfony的模块耦合这么松散，替换掉Twig用什么Blade（Laravel的默认模板引擎）、Smarty也不会很费力，但既然有那么好的Twig了，为什么还要换呢？

首先要知道，既然是模板，就需要渲染，也就是`render`，渲染的过程就是把Controller中的Response赋值给模板中相应的变量，让页面正确的显示。松散耦合的Symfony当然是把渲染页面的功能单拎出来了，而且用了一种很高级的技术——Service Container。这玩意是Symfony的精髓，后面会抽一节单独探讨这个话题。

简单来讲就是需要拿到一个服务，然后调用服务来渲染模板。

1. 基础写法
    
    ```php
        /**
     * @Route("/render", name="render")
     */
    public function renderAction(Request $request)
    {
        $name = $request->query->get('name', 'world');
        $foo = $request->request->get('foo', 'bar');

        return $this->get('templating')->renderResponse(
            '::render.html.twig',
            array(
                'name' => $name,
                'foo' => $foo,
            )
        );
    }
    ```
    
2. 简单写法
    
    得益于继承的Controller类，可以直接访问很多Service，同时它也提供了很多简写的方法。
    
    ```php
       /**
     * @Route("/render", name="render")
     */
    public function renderAction(Request $request)
    {
        $name = $request->query->get('name', 'world');
        $foo = $request->request->get('foo', 'bar');

        return $this->render(
            '::render.html.twig',
            array(
                'name' => $name,
                'foo' => $foo,
            )
        );
    }
    ```
    
显然我们会更喜欢这种简单的写法。如果你看它实现，其实还是Controller帮我们转调了一次。

## 报错

我们访问网站时经常会出现错误提示，什么404找不到、403没权限、405不支持的方法等等，而Symfony也为我们准备了方便的方式来处理这些问题。

比如

```php
if (!$product) {
    throw $this->createNotFoundException('Product not found');
}

return new Response('blah');
```

如果还需要其他的异常，自己去实现即可。

## 管理Session

Symfony封装了一个管理Session的对象，是用Cookie实现的（但我用了之后发现set值之后也没有看到浏览器的Cookie中有相应的cookie，不知道是不是文档有问题）。

```php
    /**
     * @Route("/session", name="session")
     */
    public function sessionAction(Request $request)
    {
        $session = $request->getSession();

        $session->set('username', 'google');

        return new Response($session->get('username'));
    }
```

## Flash Messages

可不要误认为这和Adobe的Flash有什么关系，它的意思是『一闪即逝』。

比如用户提交了表单，你要提示一下然后跳转到另外一个页面，这时用Flash Message就再合适不过了。

```php
$this->addFlash(
    'notice',
    'bing'
);

return $this->redirectToRoute('hello');
```

在`hello`的模板中

```twig
{% for flashmessage in app.session.flashbag.get('notice') %}
   {{ flashmessage  }}
   <br>
{% endfor %}
```

就能把Flash Message打印出来了。












