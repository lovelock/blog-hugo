+++
title  = "函数默认参数比较"
date = "2015-05-02T18:21:24+08:00"
+++

当一个函数的参数很多，而有很多需要传输的数据都具有相同的属性值，那么我们可能希望可以不传入某些默认的参数，这就导致了函数默认参数这个需求的诞生。

第一次注意到这个需求是在JavaScript中。它的实现是这样的：

```javascript
function defaultParamTest(p1, p2, p3, p4) {
    p3 = p3 ? p3 : default_value_3;
    p4 = p4 ? p4 : default_value_4;
    // function body
}
```

我不喜欢这样的实现，尝试着用我自己喜欢的方式，如下

```javascript
/* 错误的代码 */
functon defaultParamTest (p1, p2, p3=default_value_3, p4=default_value_4) {
    // function body
}
```

显然上面的是PHP的写法，JavaScript通不过。
既然说到了PHP的写法，那么就再写一段来展示一下。

```php
function defaultParamTest ($p1, $p2, $p3="default_value_3", $p4="default_value_4")
{
    // function body
}
```
这种方式用起来简单直观，符合常理。
但今天又注意到Python的写法和PHP又有不同，这一点让我很惊喜。先上示例代码：

```python
def default_param_test(p1, p2, p3 = 'default_value_3', p4 = 'default_value_4'):
    # function body
```
这样的写法你可能会认为和PHP的没有什么区别，其实区别是很大的，因为Python的这个默认参数可以省略第三个只要传入第四个，而PHP不可以。举例如下

```php
defaultParamTest($p1v, $p2v, $p4v);
```

```python
default_param_test(p1v, p2v, p4=p4v)
```
看出来区别了吧，PHP的是按顺序读入的参数，导致如果忽略第三个参数直接传入第四个参数，则函数会把你认为的第四个参数当成第三个,即使你显式的把参数列表写成`$p1v, $p2v, $p4=$p4v`也没有区别。而Python的参数列表是有键的，也就是索引，在调用方法的时候也可以再参数列表中加上索引，告诉函数我传入的是第四个参数而不是第三个。
得出的结论当然是Python的实现最灵活，用起来也最舒服。
