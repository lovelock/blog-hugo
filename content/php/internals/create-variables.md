+++
date = "2017-01-18T18:08:53+08:00"
title = "创建变量(PHP5.x扩展)"
categories = ["PHP"]
tags = ["PHP","扩展"]
isCJKLanguage = true

+++

这里记录一下如何在PHP5的扩展中创建变量，包括局部变量和全局变量。

## 必备知识

1. PHP内部有符号表的概念，其中局部变量存放在指针`active_symbol_table`中，而全局变量存放在非指针（真实值）`symbol_table`中。
2. 使用`MAKE_STD_ZVAL`宏创建变量。
3. 使用`ZVAL_xxxx`宏为创建的变量赋值，当然也可以不赋值，而只是声明。
4. 使用`ZEND_SET_SYMBOL`宏设置变量设置成全局还是局部。

### 1. 局部变量

```c
zval *new_var;
MAKE_STD_ZVAL(new_var);
ZVAL_LONG(new_var, 2000);
ZEND_SET_SYMBOL(EG(active_symbol_table), "aVar", new_var);
```

上面的代码会创建一个名为`$aVar`的**局部变量**，它的值是2000。



### 2. 全局变量

```c
zval *new_var;
MAKE_STD_ZVAL(new_var);
ZVAL_LONG(new_var, 2000);
ZEND_SET_SYMBOL(&EG(symbol_table), "aVar", new_var);
```

上面的代码会创建一个名为`$aVar`的全局变量，它的值是2000。在PHP中没有什么是一个宏实现不了的，如果有，那就两个————所以，你看创建一个全局变量要那么多字符，干脆再包装一个宏算了，于是就可以把最后一行替换成

```c
ZEND_SET_GLOBAL_VAR("aVar", new_var);
```

![](https://ww3.sinaimg.cn/large/006tNbRwly1fbuyfmvy90j30z403q0tk.jpg)