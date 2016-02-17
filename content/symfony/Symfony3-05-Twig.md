+++
author = "frostwong"
date = "2016-01-11T23:52:45+08:00"
description = "Symfony3 框架"
draft = true
keywords = ["PHP", "Symfony3"]
tags = ["one", "two"]
title = "Symfony3 In Action——Twig"
topics = ["Symfony"]
type = "post"

+++

如果你是PHP程序员，那你一定知道模板引擎。虽然PHP本身就是一种模板语言，但还是几乎每天都不断的有新的模板引擎出现。

你最经常听到的应该是Smarty。但Smarty老矣，我们要学的是现代化的模板引擎——Twig。

首先要知道模板引擎是干什么用的，以及它的实现原理。

网页最开始是静态的，也就是全部用HTML写成，如果要改动上面的内容，就要修改HTML源码。这样显然很不方便。后面发现很多网页其实样式是一样或者差不多，至少是有可以重用的组件的，那HTML就很不方便了，我觉得主要有以下两点：
    
1. 无法嵌套，iframe是一种，但并不是像PHP的`include`那种
2. 样式和变量的耦合太紧，无法生成动态页面

别忘了，PHP本身就是一种模板语言，如果你愿意，完全可以用PHP的`echo`生成一个完整的动态页面，但这样也太土了，维护起来也复杂。

总结起来，后来出现的模板引擎就是为了解决上面三个问题而出现的，要有以下特点

1. 支持嵌套
2. 支持变量赋值
3. 富有表现力的语法

是时候祭出今天的主角——Twig了。先看下对应上面三个特点相对应的语法

1. 嵌套

    ```twig
    {% block content1 %}
        {% block content2 %}
        {% endblock %}
    {% endblock %}
    ```
    
2. 动态变量赋值

    ```php
    ...
    return $this->render('AppBundle:Resource:test.html.twig', array(
        'foo' => 'bar'
        ));
    ```
    
    ```twig
    {{ foo }}
    ```
    
3. 富有表现力的语法
    
    上面已经看到了，打印变量是`{{ }}`，其他逻辑结构是`{% %}`，而`{# #}`表示注释。
    
是不是已经被它吸引了？



    


