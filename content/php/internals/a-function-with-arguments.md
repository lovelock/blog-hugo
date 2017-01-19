+++
draft = true
tags = ["PHP","扩展"]
isCJKLanguage = true
date = "2017-01-17T15:49:06+08:00"
title = "带有参数的函数"
categories = ["PHP"]

+++

没有参数的函数可以说用的比较少，这次我们来研究一下有参数的函数。

## `zend_parse_parameters`函数

我看到的解析参数的方法就是`zend_parse_parameters`，它本身有几个参数值得分别解释一下：

1. `ZEND_NUM_ARGS() TSRMLS_CC` 显然空格前面表示的是参数的个数，而后面这个就说来话长了，英文好的直接看[这篇文章](http://blog.golemon.com/2006/06/what-heck-is-tsrmlscc-anyway.html)，其实我也没有看的很明白，抽时间一字一句的翻译一下。

2. 格式化字符串，类似PHP的函数`printf()`的第一个字符串格式化参数，诸如`\s`等。具体如下：

   | 格式化字符串 | 代表的类型                               | 对应C中的数据类型                   |
   | ------ | ----------------------------------- | --------------------------- |
   | `b`    | Boolean                             | `zend_bool`                 |
   | `l`    | Integer                             | `long`                      |
   | `d`    | Floating point                      | `double`                    |
   | `s`    | String                              | `char*, int`（前者接收指针，后者接收长度） |
   | `r`    | Resource                            | `zval*`                     |
   | `a`    | Array                               | `zval*`                     |
   | `o`    | Object instance                     | `zval*`                     |
   | `O`    | Object instance of a specified type | `zval*, zend_class_entry*`  |
   | `z`    | Non-specified zval(任意)              |                             |
   | `Z`    | zval**                              |                             |
   | `f`    | function/method name                |                             |

   ​

3. 接收参数用的本地变量的引用，`zend_parse_parameters`的参数数量是不定的，在第二个参数后面可以跟理论上无数个参数。

## 举例

1. 简单数据类型

   ![](https://ww4.sinaimg.cn/large/006tKfTcly1fbtqud7r2pj31ga0rydlw.jpg)

直接在PHP中执行`hello_stranger(1000000);`会输出![](https://ww1.sinaimg.cn/large/006tKfTcly1fbtqvg8v0jj30ka01kdfz.jpg)

你可能会想知道第二个红框内的NULL到底是怎么回事，后面会做讲解，这里直接写成NULL是没有问题的。

2. 多个参数

   前面说了，`zend_parse_parameters`的参数数量是不定长的，所以只需要根据需要在后面添加更多参数即可。本来想写一个带`string`类型参数的例子，结果花了很久也无法编译成功，后来想想觉得可能是PHP7不兼容的问题，干脆把代码拿来编译成PHP5的版本，果然通过了，

   ![](https://ww2.sinaimg.cn/large/006tNbRwly1fbuwu3xeyjj31kw0kggpw.jpg)

   对其中吧定义写在下面的做法不要惊奇，因为我把声明写在头文件中了。至于PHP7的版本应该怎么写，只能待我把PHP5研究透了再来继续写了。