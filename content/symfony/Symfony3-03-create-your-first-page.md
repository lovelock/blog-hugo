+++
author = "frostwong@gmail.com"
date = "2016-01-10T22:02:07+08:00"
description = "Symfony3 框架"
draft = true
keywords = ["PHP", "Symfony3"]
tags = ["Symfony3", "PHP框架"]
title = "Symfony3学习第三章——创建第一个页面"
topics = ["Symfony"]
type = "post"

+++

实际上前面我们已经看到了一个正常的页面了。是这样的
![成功安装的Symfony3运行界面](http://7xn2pe.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-01-10%20%E4%B8%8B%E5%8D%8810.17.26.png)

这个默认的页面属于`AppBundle/Controller/DefaultController/indexAction`，可以在该方法的annotation中找到对应的配置。

**我不喜欢Annotation这种方式**。

Annotation看起来是把路由信息和代码写在一起，增强了代码的可读性，但在我看来完全是没有必要的，因为这样我还需要在写代码的时候注意Annotaion的编写，一不小心写错了找都不好找。或者说，它增强了代码和配置的耦合。xml就不考虑了，那么冗长的写法谁受得了，幸亏没有提供JSON的支持——即便支持我也不用，JSON的可读性太差。所以YAML就成了我的首选。在本系列文章中，我都会用YAML方式做配置，包括但不限于路由控制、数据表设置等。

不好的一点在于貌似官方还挺喜欢Annotaion，能下载到的Symfony Book、Symfony Cookbook都对YAML的介绍很少，默认都使用Annotation。然而这并不能阻挡我。

废话了那么多，进入正题吧。

## 配置文件的路径
路由的配置文件在`app/config/routing.yml`，`resource`行就指定了它要寻找的路由配置的位置，`type`行指定了要查找的路由配置格式，默认是annotation。

首先解释一下为什么会是`app/config/routing.yml`，当然是因为

```yaml framework:     #esi:             ~     #translator:      { fallbacks: ["%locale%"] }     secret:          "%secret%"     router:         resource: "%kernel.root_dir%/config/routing.yml"         strict_requirements: ~
```

那你还会问，`%kernel.root_dir`是在哪里配置的呢？问得好，这个设置可不在配置文件里，是在`Symfony\Component\HttpKernel\Kernel.php`中设定的，代码如下:

```php
protected function getKernelParameters()
    {
        $bundles = array();
        foreach ($this->bundles as $name => $bundle) {
            $bundles[$name] = get_class($bundle);
        }

        return array_merge(
            array(
                'kernel.root_dir' => realpath($this->rootDir) ?: $this->rootDir,
                'kernel.environment' => $this->environment,
                'kernel.debug' => $this->debug,
                'kernel.name' => $this->name,
                'kernel.cache_dir' => realpath($this->getCacheDir()) ?: $this->getCacheDir(),
                'kernel.logs_dir' => realpath($this->getLogDir()) ?: $this->getLogDir(),
                'kernel.bundles' => $bundles,
                'kernel.charset' => $this->getCharset(),
                'kernel.container_class' => $this->getContainerClass(),
            ),
            $this->getEnvParameters()
        );
    }
```

顺着这个一步步的往下追，就可以查到每个kernel参数的来源了。

刚才说到了路由配置，而且只想用yaml的格式，下面就改一下。

```yaml
app:
    resource: "@AppBundle/Resources/config/routing.yml"
```

这就表明会从`@AppBundle/Resources/config/routing.yml"`中读取路由配置了。当然在下面也可以直接加上更多配置——经过我的一番实验，发现在这里是无法引入第二个路由配置文件的，改文件不支持`import`，不支持多个`resource`。。。

注意，在`resource`中的配置优先级是比当前文件下面的高的。和在同一个文件中的配置一样，因为路由信息一旦匹配成功就不再往下找了。

## 路由规则
### 简单路由规则
一个最简化的可以执行的路由规则其实很简单，由path和defaults两部分构成。

以下面的路由规则为例：

```yaml
# app/config/routing.yml
_welcome:
    path:      /
    defaults:  { _controller: AppBundle:Main:homepage }
```

path是URI中的Path部分，完整的路径就是`http://symfony.dev/app_dev.php`，它被映射到`AppBundle/MainController/homepageAction`上。

### 带占位符的路由规则
在RESTful的API设计中，是避免用query string的，相应的，用`/`来分隔参数。我知道的有这样两种设计思路

1. 带参数名 `/param1/value1/param2/value2`
2. 不带参数名 `/value1/value2`

很明显，第二种方式更简洁，因为程序肯定是知道第几个参数的意义是什么的，但这样客户端传递参数时就失去了灵活性。当然，在我看来，一个设计稳定的接口才重要，只要不随便改动，是没什么问题的。

带占位符的路由规则

```yaml
show:
    path:     /show/{title}
    defaults: { _controller: AppBundle:Article:show }
```

这个路径会匹配所有/show/*，传进来的值会被当做参数`$title`传递到方法`AppBundle\ArticleController\showAction`中。例如，`/show/hello`，那么在`AppBundle\ArticleController\showAction`方法中拿到的`$title`就是字符串`hello`。

### 可选占位符的路由规则
上面介绍了带占位符的路由规则，那考虑这样一个场景，我像让`/show`显示`$page=1`的文章，而不需要指定`/show/1`，如果加了`/show/{page}`则按照给定的显示。可以类比PHP函数的默认参数，也很简单

```yaml
show:
    path:     /show/{page}
    defaults: { _controller: AppBundle:Article:page, page: 1 }
```

### 参数要求
上面两个例子会有冲突吗？当然会！比如`/show/2`表示要展示第二页，那会不会是标题是2的文章呢？要怎么判断？



