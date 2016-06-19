+++
author = "frostwong"
date = "2016-06-18T07:04:22+08:00"
description = "理解PHP的Service Container"
draft = true
isCJKLanguage = true
keywords = ["Container", "Dependency Injection", "Service Container"]
tags = ["PHP"]
title = "理解PHP的Service Container"
type = "post"

+++

前段时间一直在花时间研究依赖注入的问题，现在虽然也没有研究的很透彻，但有了一些很模糊的概念，是时间把它总结一下了，既帮助自己理解，也能给现在还没有理解的同学们一点启发。

本文将从以下几点展开。

1. 什么是依赖注入？为什么要有依赖注入？
2. 什么是Service Container(服务容器）？它和Docker的Container有关系吗？又有什么用？
3. 以Pimple为例，通过解读源代码来分析实现一个简单的Service Container。

## 什么是依赖注入？为什么要有依赖注入？

举个最常见到的例子来讲，比如我们的代码分层中有业务逻辑层(Logic层）和数据库操作层(Dao层），那在L层如何调用D层的代码呢？多数情况下可能会像下面这样：

```php

<?php

class Sample
{
    private $daoSample;

    public function __construct()
    {
        $this->daoSample = new DaoSample();
    }

    public function setDao(DaoInterface $dao)
    {
        $this->daoSample = $dao;
    }
    
    public function newAccount($params)
    {
        $this->daoSample->addRecord($params);
    }
}

<?php

class Sample
{
    const TABLE_NAME = 't_Sample';

    public function addRecord($params)
    {
        echo __METHOD__ . "\n";
    }
}

```

有了这样的依赖注入，我们在写业务层代码的时候就可以这样：

```php
$logicSample = new Logic\Sample();
$logicSample->newAccount($params);
```

完全不用考虑用的是什么类型的数据库操在类了，只需要保证它能够完成任务即可。

这是个极端简化的例子，在实际的工作中，可能会遇到一个类依赖很多个类的情况，如果把每个依赖都写到构造函数里，也不是不可以，但维护起来也是挺麻烦的。  
有一个“看起来”更简单一点的实现：

```php
<?php

class Sample
{
    private $daoSample;

    public function __construct(DaoInterface $dao)
    {
        $this->daoSample = $dao;
    }

    public function setDao(DaoInterface $dao)
    {
        $this->daoSample = $dao;
    }
    
    public function newAccount($params)
    {
        $this->daoSample->addRecord($params);
    }
}

<?php

class Sample
{
    const TABLE_NAME = 't_Sample';

    public function addRecord($params)
    {
        echo __METHOD__ . "\n";
    }
}

```

但这样在调用的时候就需要这样：

```php

$daoSample = new Dao\Sample();
$logicSample = new Logic\Sample($daoSample);
$logicSample->newAccount($params);
```

需要额外维护底层工具类的初始化。看起来更复杂了，但我觉得“依赖注入容器”的思想恰恰是从这样的用法中通过更深层次的思考得到的。  

## 什么是Service Container(服务容器）？它和Docker的Container有关系吗？又有什么用？

我们把像`Dao\Sample`这样的类叫做“服务”吧，现在在使用这些服务之前需要自己来管理**服务的初始化**，那么有没有办法把这个工作托管出去呢？  
最直观的想法可能就是把这些”服务“放在一个容器里，需要的时候直接拿来用就行。那么问题又来了，还是需要先给它初始化，然后才能放在容器里，再多想一步，能不能让容器自己来管理里它的初始化？  

当我需要一个服务时，就取容器里取，如果该服务在容器中定义了，但还没有初始化，那容器就给它初始化，并把这个实例保存下来，下次再有别的地方使用该服务时直接拿来用即可。

这就是依赖注入容器的理想状态了吧。

它和Docker的容器当然没有任何关系。

虽然都叫容器，但它们解决的问题或者说痛点其实是不同的。依赖注入容器解决的是服务随取随用，Docker的容器本质上把一个服务相关的所有东西打包在一起，找到一个宿主机它就能在自己的空间内跑起来，它解决的问题是一次生成，到处运行的问题，从这点上看有点像Java的口号。

## 实现一个简单的Service Container

没错，我们是PHP开发者，那我们一定很喜欢数组，对不对？

好，那我们就来实现一个和数组用起来差不多的依赖注入容器。（主要参考了Pimple的实现）

### 功能设计
### API设计 

