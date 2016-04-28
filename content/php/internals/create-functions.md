+++
author = "frostwong"
date = "2016-04-14T22:12:56+08:00"
description = "PHP扩展实战"
draft = true
isCJKLanguage = true
keywords = ["PHP", "扩展"]
tags = ["PHP", "扩展"]
title = "为PHP添加全局函数"
type = "post"

+++

这几天一直没有抽出空来写，都快忘了。

下面开始写第一个函数。注意这里我说**函数**而不是**方法**，是有意要和**类的方法**做区分。因为这里的**函数**是全局的。

不能忘了咱们的功能设计，这里我需要一个这样的名叫`format_log`函数：

1. 接收一个字符串，用`sprintf`格式化以后传递给`str`
2. 接收若干个字符串

首先要思考几个问题：

1. 怎么在PHP的命名空间中注册这个函数？
2. 怎么接收参数？

## 在PHP命名空间中注册一个函数

找到这里

```c
/* {{{ hylog_functions[]
 *
 * Every user visible function must have an entry in hylog_functions[].
 */
const zend_function_entry hylog_functions[] = {
    PHP_FE(confirm_hylog_compiled,  NULL)       /* For testing, remove later. */
    PHP_FE_END  /* Must be the last line in hylog_functions[] */
};
/* }}} */
```

注意注释，“所有用户可见的函数都需要在`hylog_functions[]`中有一个入口”。所以，我在里面添加一行，变成这样

```c
/* {{{ hylog_functions[]
 *
 * Every user visible function must have an entry in hylog_functions[].
 */
const zend_function_entry hylog_functions[] = {
    PHP_FE(confirm_hylog_compiled,  NULL)       /* For testing, remove later. */
    PHP_FE(format_log, NULL)
    PHP_FE_END  /* Must be the last line in hylog_functions[] */
};
/* }}} */
```

先不用管第二个参数`NULL`，现在只需要知道它和目标函数要接收的参数有关即可，后面会详细讲述。现在函数`format_log`已经注册进了PHP了，但入口又在哪里呢？顺着`hylog_functions`找，

```c
/* {{{ hylog_module_entry
 */
zend_module_entry hylog_module_entry = {
	STANDARD_MODULE_HEADER,
	"hylog",
	hylog_functions,
	PHP_MINIT(hylog),
	PHP_MSHUTDOWN(hylog),
	PHP_RINIT(hylog),		/* Replace with NULL if there's nothing to do at request start */
	PHP_RSHUTDOWN(hylog),	/* Replace with NULL if there's nothing to do at request end */
	PHP_MINFO(hylog),
	PHP_HYLOG_VERSION,
	STANDARD_MODULE_PROPERTIES
};
/* }}} */
```

看到花括号中的第三个变量。刚才我们说了，所有用户可见的函数需要写在`hylog_functions`里，现在又把它打包传给`zend_module_entry`类型的变量了。至于后面的几个宏，其中`PHP_MINIT`和`PHP_MSHUTDOWN`是一对，表示**模块**的启动和关闭，所谓模块，其实就是扩展了；`PHP_RINIT`和`PHP_RSHUTDOWN`是一对，表示**请求**的开始和关闭；`PHP_MINFO`用来控制`phpinfo()`或`php -i`的输出中关于本扩展的信息；`PHP_HYLOG_VERSION`就是本扩展的版本号了。这些后面会详述(TODO)。至于`STANDARD_MODULE_PROPERTIES`，就是一个内置了宏了，暂时也不用管它。(TODO问问鸟哥）

那最关键的一步还没做呢，实现该方法。

PHP为我们提供了一个宏来定义函数，`ZEND_FUNCTION`或`PHP_FUNCTION`，它两个本质上是一样的，不过既然到现在生成的骨架里面还在用`PHP_FUNCTION`，那我们姑且就保持一致吧。


```c
PHP_FUNCTION(format_log)
{
	char str[] = "hylog";
	
	php_printf("Hello, %s\n", str);
}
```

把它写在和`confirm_hylog_compiled`相同的那块区域。现在执行`rebuild.sh`，然后`php -r 'format_log();`应该就能看到`Hello, extenstion`了。

等等，我们说好的是要接收输入参数的，怎么现在又是一个没有参数的函数？没关系，刚才只是为了验证一下方法是否走的通，现在我们来考虑接收参数。

## 接收参数的函数

