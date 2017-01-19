+++
draft = false
tags = ["PHP","语言特性"]
isCJKLanguage = true
date = "2017-01-19T15:21:43+08:00"
title = "关于PHP接口特性的一个发现"
categories = ["PHP"]

+++

这两天看一本PHP的进阶书，发现了一些之前没有注意的特性。比如PHP接口的设计方式和它对实现该接口的类的约束就和通常的语言(比如Java）不一样。

举一个简单的例子，要写一个配置管理类，这个类为了适配不同的配置文件格式，比如`ini`,`yaml`等，就需要一个接口来约束这些具体的实现。代码如下：

```php
<?php
  
  interface ConfigInterface
  {
    public function get($name);
  }
```

```php
<?php
  class IniConfig implements ConfigInterface
  {
    public function get($name)
      {
        xxxxx;
      }
  
  	public function fetch($name)
      {
        xxxxxx;
      }
  }
```

```php
<?php
  class YamlConfig implements ConfigInterface
  {
    public function get($name)
      {
        xxxxxx;
      }
  }
```

```php
<?php
  function check(ConfigInterface $config)
  {
    $config->fetch('foo');
  }
```

如果你按类似上面的结构写完执行你就会发现一个很神奇的特性，这段代码竟然是可以执行的（忽略我为了偷懒省略了具体实现吧）。

但是仔细想想，我在方法`check`里面用接口解耦的目的是什么呢？就是为了接受不同实现，而如果这些实现自己的独有的方法在这里都可以调用，这个约束的存在还有什么意义呢？所以，我理解的PHP的接口的作用仅仅限于两点：

- 规定每个实现一定要实现相应的方法
- 方便IDE进行自动提示和补全

也就是说，PHP的接口更多意义上是一个**约定**，而不是**规定**。