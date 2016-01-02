+++
title  = "Symfony一对多关系的YAML配置"
date = "2015-11-01T16:11:21"
+++

# 用YAML方式配置对象关系
这段时间在学习Symfony，看了Symfony Book和Symfony Cookbook，现在准备自己实现一个简单的博客系统，做一些简单的设计。

## 分析对象关系
首先考虑会操作哪些对象。

1. 文章 Article
2. 用户 User
3. 评论 Comment
4. 分类 Category

### 文章
对文章来说，它包含的属性有

    - 标题
    - 概要
    - 正文
    - 作者
    - 创建时间
    - 更新时间

而作者是属于用户的，其属性不应该在Article对象中，分析文章和作者的关系，应该是一篇文章只能有一个作者，而一个作者可以写很多篇文章，这就是doctrine中的OneToMany和ManyToOne了。总感觉用Annotation的方式写这个关系不是那么明确，所以今天特意用yml配置文件的方式来实现这个概念。后面再详细说，这里要清楚的是对文章来说 

    1. 文章对评论 OneToMany
    2. 文章对作者 ManyToOne
    3. 文章对分类 ManyToOne

### 用户
对用户来说，包含的属性有

    - 用户名
    - 密码
    - 昵称
    - 角色

前面已经说过，用户可以创建多篇文章，因此

    1. 用户对文章 OneToMany
    2. 用户对评论 OneToMany

### 评论
对评论来说，包含的属性有

    - 标题
    - 内容
    - 作者
    - 创建时间

和文章一样，它和作者是多对一的关系

1. 评论对作者 ManyToOne
2. 评论对文章 ManyToOne

### 分类
对分类来说，只有一个属性（最简单的情况）

    - 分类名称

它和文章是一对多的关系

    1. 分类对文章 OneToMany

## 实现
这样，四个对象的关系就很明确了。具体到用yml来实现这些关系要注意的有两点。
以用户和文章的关系为例。

### OneToMany
在YourBundle/Resources/config/doctrine/User.orm.yml中，除了user表自身的属性配置之外，还要加上以下配置
```
    oneToMany:
        articles:
            targetEntity: Artcile
            mappedBy: user
```
为了便于理解这段的意思，我们可以给`oneToMany`加上一个主语——user，就可以这样理解了：

> 一个user对应多个articles，articles对应的Entity是Article，articles是被user映射的（这个map是doctrine定义的叫法，相应的是inversedBy，没法解释了。。。。）

这时对应的数据表是没有任何变化的，这只是声明了实体之间有这种关系，但并没有表现在数据库中。
那么，既然有了用户对文章是一对多，那么必然存在文章对用户是多对一，即，数据表的变化需要两方面的配置才能生效。

### ManyToOne
在YourBundle/Resources/config/doctrine/Article.orm.yml中，除了article表自身的属性外，还要加上如下配置
```
    manyToOne:
        user:
            targetEntity: User
            inversedBy: articles
            joinColumn:
                name: user_id
                referencedColumnName: id
```
同样，我们还给这段配置加上主语articles来理解

> articles对应一个userser对应的实体是User，对应User.orm.yml中的mappedBy: user的就是inversedBy: articles。注意另外一个对应，manyToOne下第一级的user是对应OneToMany中的mappedBy: user，而这里的inversedBy: articles对应User.orm.yml中的oneToMany下第一级的articles。下面就是最重要的改变数据表的配置了，在article表中插入一个名为user_id的列，而这一列对应
user表中的id

写完这些，再执行`app/console doctrine:generate:entities YourBundle`就会看到

```bash
Updating database schema...
Database schema updated successfully! "10" queries were executed
```
这样的输出了，再连上数据库看下，就会发现相应的表结构都已经变了。

## 总结和练习
如前面所说的，这四个对象之前的关系可不止上面写的这些，作为练习，请读者把其他的关系也用yml的方式配置出来吧。

本文所用的代码位于[symfony练习题]
