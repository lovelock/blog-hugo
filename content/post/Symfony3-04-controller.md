+++
author = "frostwong"
isCJKLanguage = true
date = "2016-01-11T23:51:43+08:00"
description = "Symfony3 框架 Controller"
draft = false
keywords = ["PHP", "Symfony3"]
title = "Symfony3 In Action——Controller"
topics = ["Symfony"]
type = "post"
+++

在Symfony中，Controller是很薄的一层，逻辑不会很复杂，主要就是接受Request，用Request的参数取得资源，放在Response中返回给Client。

> 注意，以下说的Controller，没有区分Controller类和Action，请自行根据上下文判断。

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
     *@Route("/hello")
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
     *@Route("/hello")
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
     *@Route("/hello")
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
     *@Route("/render", name="render")
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
     *@Route("/render", name="render")
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
    
显然我们会更喜欢这种简单的写法。如果你看它实现，其实还是Controller帮我们转调了一次。可以看到，不管是基础写法还是简单写法，最终都是把一个数组传递给了模板引擎，这样在模板中就可以获取到这些值。Twig的语法这里暂且不说。

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
     *@Route("/session", name="session")
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
    'bingo'
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

## Request

在Symfony中，我们可以很方便的访问Request和Response。了解了HTTP协议的同学都知道Request/Response是包括Header和Content（可选）的，看下面的常用方法吧。

```php
$request->isXmlHttpRequest(); // 是否是Ajax请求
$request->query->get('param', 'default'); // 获取GET参数，并设默认值
$request->request->get('param', 'default'); // 获取POST参数，并设置默认值

$request->headers->get('Content-Type');
$request->getContentType();
```

倒数第二个方法是通过HeaderBag获取Header中的字段，这种写法获取的是请求Header中的Content-Type，传过来的是什么，获取到的就是什么。而最后一种方法则不一样，看代码可知，返回的是经过格式化的简化格式。比如如果在请求中指定`Content-Type:application/json`，前者获取到的是`application/json`，而后者是`json`。但需要保证MIME type是合法的，否则后者获取到的就是null了。


## Response

和Request一样，Response也是包含Header和Content两部分。我们经常用的

```php
return new Response('blah-blah');
```

其实就是返回Content是`blah-blah`，Status Code是200的相应。那么Header在哪里呢？对，HeaderBag。

```php
$response = new Response('blah', 201); // 默认是200
$response->headers->set('Content-Type', 'application/json');

return $response;
```

可以把第二行注释掉看看会有什么效果。

> 默认的Content-Type是`text/html`，响应是字符串没有问题。但如果设置了`application/json`，那响应就需要时键值对了。相应的需要修改如下
> 
> ```php
> $response = new Response(json_encode(array('I am a key' => 'I am a value')));
> $response->headers->set('Content-Type', 'application/json');
> return $response;
> ```

## 创建静态页面

这里所说的静态页面是指『没有参数传入的页面』，也就是说不会因为你传入任何参数而改变的页面。前面说了，Controller的作用就是接受Request，搞一通之后把数据写入Response再返回给客户端。那既然是静态页面，我还需要创建Controller吗？

是，也不是。

本质上这个Controller还是存在的，只不过是Symfony已经创建好了的，不需要自己创建了而已。

既然没有了Controller，也就没有地方写Annotation了，在routing.yml中可以这样写

```yaml
static_page:
    path: /static
    default:
        _controller: FrameworkBundle:Template:template
        template: static/sample.html.twig
```

写到这里就出现了另一个问题，既然你Symfony官方推荐我们用Annotation，那明显这样的需求用Annotation是实现不了的啊，如果要让两种方式并存应该怎么办？

事实证明是可以的，前面说过，在`app/config/routing.yml`中跟节点的名字是没有意义的，既然我可以指定一个用Annotation的节点，当然就可以再指定一个用yml的节点，就像这样

```yaml
app:
    resource: "@AppBundle/Controller/"
    type: annotation

yaml:
    resource: "@AppBundle/Resources/config/routing.yml"
    type: yaml
```

这样就可以随心所欲的在两种形式之间切换了（要小心顺序问题）。

## 『跳』到另一个Controller

前面介绍过`redirectToRoute`方法了，是做了一个302/301跳转，用户的浏览器会刷新一下，那有没有办法让它不刷新呢？

当然有。用`forward`方法。这个方法比`redirectToRoute`方法少了两点限制：

1. 用户浏览器不用刷新
2. 要跳转到的Controller不用配置路由

关于第二条，想不明白的话再往前找找。

```php
/**
 *@Route("/forward", name="_forward")
 */
public function forwardAction()
{
   $response = $this->forward(
       'AppBundle:Hello:render',
       array(
           'name' => '大众化',
           'foo' => 'bar',
       ));

   return $response;
}
```

## 验证CSRF token

说实话我工作到现在还没有遇到过这个事儿。CSRF token是用来进行表单安全验证的，如果用Symfony的Form组件，默认就有，否则就需要自己在表单中加上CSRF token，然后服务端接收到的时候再验证一下了。

```php
public function validAction(Request $request)
{
   $submitted_token = $request->request->get('token_id');

   if ($this->isCsrfTokenValid('token_id', $submitted_token)) {
       // do something dangerous 
   }
   
   return new Response('haha');
}
```


最后想说一点，我写的这系列文章一定要自己动手实践才行，很多问题你不自己实践是发现不了的。官方文档的风格我很喜欢，但有些问题并没有解释的很清楚，或者要翻看具体组件的文档太麻烦，这时候自己尝试一下就会有很好的体验喽。

下一篇谈谈Request对象。














