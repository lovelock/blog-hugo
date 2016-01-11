+++
title  = "PHP单入口模式"
date = "2015-04-25T17:07:18+08:00"
author = "frostwong"
type = "post"
description = "PHP单入口模式简介"
+++

单入口模式是现在很多项目遵循的模式，如WordPress等。
下面描述一下我对单入口模式的理解和一些简单的代码，一步一步构建一个完整的单入口模式的简单框架。

> 文中的代码只需要用PHP内建Web Server测试即可，没必要先搭建Nginx环境。

## 初见
首先，所谓单入口，即是所有的访问都要经过同一个入口，当然不可能所有的功能(方法)都写在这个文件中，那么这个文件最可能的作用其实是转发请求。根据传过来的参数的不同，去调用不同的类中不同的方法。

```php

/* filename index.php */
<?php
require_once __DIR__ . '/TestClass.php';
$op = $_GET['op'];
if ($op == 'echoparam') {
    TestClass::echoParam();
} else if ($op == 'addparams') {
    TestClass::addParams();
}

```

```php

/* filename TestClass.php */
<?php
class TestClass
{
    public static function echoParam()
    {
        $param = $_GET['param'];
        echo $param;
    }

    public static function addParams()
    {
        $param1 = $_GET['param1'];
        $param2 = $_GET['param2'];
        echo $param1 + $param2;
    }
}

```

> 请自行忽略上面代码中没有作参数验证，线上代码肯定是需要的。后面出现代码的地方请参考这句话。

这样已经可以简单的对外提供服务了，当然如果你想提供Restful风格的API的话，也可以利用Nginx的rewrite来实现。这个后面会做详细的解释。

这样简单的服务肯定是没有用的，近从这段代码出发，我们可以发现很多需要做的工作。

1. 如果提供的API增加，会导致`if else` 判断的数量极速增加，不要说用`switch case` 做代替，那治标不治本，问题没有出在这里。根本的解决方案有两种，要么a) 用配置文件， b) 自适应（会有很多字符串的处理）。多数人可能会选择第一种方案，因为配置文件嘛，多简单，写个xml再解析一下就好了嘛，干嘛要做那么高端的自适应呢？再留个坑后面填。

2. 这里获取参数的方法写成了`GET` ，那么如果是`POST` 了怎么办？很简单，挂掉了，也就是说这个API接口只支持`GET`请求，要让它支持`POST` 等其他方法就要让获取参数的方法透明，不管客户端用什么方法传过来参数我服务端都能正确的解析并给出正确的响应。

3. 可能很多人首先想到的就是这个了，对，`spl_autoloader` ，作为一个现代化的应用程序，PSR-4当然是要支持的。

4. 可能有人看到了，我在写`require_once` 语句时用的是`__DIR__` 而不是通常会见到的`dirname(__FILE__)` ，这是考虑到PHP 5.3也发布了多年了，是时候在利用它之后添加的新特性了。具体有哪些可以参考[PHP 5.3的新特性](http://koda.iteye.com/blog/490515)。

5. 项目复杂时，当然要用Unit Test了。

6. 关系到数据库时可能又会关系到ORM了。

下面将对上面的提出的问题逐步解决。

`if-else` 过度复杂的问题，就像前面说的，写个xml就解决了，看代码

```php

/* filename config.xml */
<?xml version='1.0'?>
<classmap>
    <action name="echoparam" class="TestClass" method="echoParam"></action>
    <action name="addparams" class="TestClass" method="addParams"></action>
</classmap>

```

> name 是对外暴露的接口名称，class 是该接口所属的类，而method是实际调用的方法，也就是说name是method的别名。

有了xml就需要解析它，要解析就要先获取它，那当然是`simplexml_load_file` 方法了。

```php

/* filename ActionMapLoader.php */
<?php

class ActionMapLoader
{
    protected $map;

    public function __construct($mapFile)
    {
        $this->map = simplexml_load_file($mapFile);
    }

    public function getMap()
    {
        return $this->map;
    }
}

```

只需要把xml文件作为`ActionMapLoader.php` 构造函数的参数传进去，实例化后调用`getMap()` 方法获得的就是xml表示的关系的对象了。


```

SimpleXMLElement Object
(
    [action] => Array
        (
            [0] => SimpleXMLElement Object
                (
                    [@attributes] => Array
                        (
                            [name] => echoparam
                            [class] => TestClass
                            [method] => echoParam
                        )

                )

            [1] => SimpleXMLElement Object
                (
                    [@attributes] => Array
                        (
                            [name] => addparams
                            [class] => TestClass
                            [method] => addParams
                        )

                )

        )

)

```

可以看到，最外层的标签`<classmap></classmap>` 是作为一个容器存在的，它的每一个子标签都是这个容器的子元素。取得了传入的`op` 参数之后就可以根据这个关系取得类名和方法名了。


```php

/* filename Controller.php */
<?php

class Controller
{
    protected $mapFile;

    public function __construct($mapFile)
    {
        $this->mapFile = $mapFile;
    }

    public function getActionInfo ($actionName)
    {
        $map = new ActionMapLoader($this->mapFile);
        foreach ($map->getMap()->action as $action) {
            if (strcasecmp($actionName, $action['name']) == 0) {
                return new ActionInfo($actionName, $action);
            }
        }
    }

    public function process()
    {
        $actionName = $_GET['op'];
        $actionInfo = $this->getActionInfo($actionName);

        $class = $actionInfo->class;
        $method = $actionInfo->method;

        (new $class())->$method();
    }
}

```

`Controller` 类是整个转发过程的核心，它根据传入的`op` 调用相应的类中的方法，然后调用该方法。那么它是如何获取到这个类的属性和方法的呢？


```php

/* filename ActionInfo.php */
<?php

class ActionInfo
{
    public $name;
    public $class;
    public $method;

    public function __construct($actionName, $action)
    {
        $attrs = reset($action);

        $this->name = $actionName;
        $this->class = $attrs['class'];
        $this->method = $attrs['method'];
    }
}

```

这个类用于从`ActionMapLoader` 类中获取某个类的所有属性和方法。
这样整个流程就走通了。

`index.php` 就只需要将xml文件传给`Controller` 类，剩下的事情就交给`Controller` 类去处理了。

```php

<?php
require_once __DIR__ . '/TestClass.php';
require_once __DIR__ . '/ActionMapLoader.php';
require_once __DIR__ . '/ActionInfo.php';
require_once __DIR__ . '/Controller.php';
require_once __DIR__ . '/config.xml';

$mapFile = 'config.xml';

(new Controller($mapFile))->process();

```

上面的代码中值得注意的一点是`if (strcasecmp($actionName, (string)$action['name']) == 0)` ，这里没有用`if ($actionName == $action['name'])` 这样的写法，是为了忽略调用方法的大小写。

这下对这个过程可是很清晰了。下面就到了第二点，请求方法写死了。那么有什么方法可以既支持`POST` 方法又支持`GET` 方法呢？当然是让程序自己去判断而不要我们我们人为的判断了。


```php

/* filename Request.php */
<?php
class Request
{
    public function getValue($key)
    {
        switch ($_SERVER['REQUEST_METHOD']) {
        case 'GET':
            $request = $_GET;
            break;
        case 'POST':
            $request = $_POST;
            break;
        }

        return $request[$key];
    }
}

```

这个方法自动适应请求方式，返回正确的结果。

你一定也看到了，每添加一个类，都需要在入口文件中添加一条`require_once` 语句，这样的操作现在看来当然是不可接受的。
这就需要命名空间和`autoloader` 了。现在的这个复杂度还没有到要分目录的时候，所以只需要用根命名空间去引用类就可以了。

单元测试和ORM的话题太大了就不放在这里说了。代码可以到[PHPSingleEntryDemo](https://github.com/lovelock/PHPSingleEntryDemo)去查看。

另外说一点，文中提到了用Nginx的rewrite方法，其实很简单，但我还是希望再开一个专题来说这个，所以本文中代码的测试只需要用PHP内建的Web Server就可以测试了。
简单的使用方法是


```bash

cd PHPSingleEntryMode
php -S localhost:8080 -t .

```

然后就可以用`curl 'localhost:8080/index.php?op=echoparam&param=teststring` 进行测试了，当然如果你愿意用浏览器也是没有问题的。


