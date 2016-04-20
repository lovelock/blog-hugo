+++
title  = "PHP延迟绑定"
date = "2015-06-17T23:37:38+08:00"
topics = ["PHP"]
+++

在公司的代码里看到很多重复代码，每张表对应的类都有一个一样的静态方法，作用是获得数据库的连接。但显然这并不是一个好的解决方案，而应该充分利用面向对象的思想，利用继承的方式来更优雅的解决。

我想，当初作者在写这部分代码的时候可能也已经考虑到了这个问题，但很可能是对PHP的延后绑定（LSB）不了解，因此没有实施。

得到数据库连接的实例有多种方式，这里我们假设有一个方法`DB::mysqlConn($tableName);`

举个例子来说，假设我们设计一个基类`BaseConn`：

```php
class BaseConn
{
    protected static $conn;
    protected static $tableName;
    
    public function getInstance()
    {
        self::$conn = DB::mysqlConn($tableName);
        return self::$conn;
    }
}
```

按照计划，我们就可以写子类了：

```php
class ATableConn extends BaseConn
{
    protected static $tableName = 'atable';
    
    public static findById($id)
    {
        $query = "select * from " . self::$tableName . " where id = {$id}";
        return $this->getInstance()->exec($sql);
    }
}
```

这时一般都会认为没有什么问题了，因为我们已经在子类里设定了`$tableName`的值，这样在创建数据库连接的时候一定就是`atable`的连接了。

错！

因为用`self`关键字绑定到了编译时引用的属性或方法。`self`关键字指向的是父类，而且不会意识到子类，基本上，编译器会用所绑定的名称替换`self`关键字。

那么怎么解决呢？PHP5.3为一个本来就存在的关键字赋予了新的含义——`static`，它会在可能的最近时刻强迫PHP绑定到实现代码。[^static]

因此，把上面代码中`getInstance`方法中的`self`替换成`static`就可以了。

这个特性也就可以实现ActiveRecord了，不过这就是另外一个话题了。


        
[^static]: [PHP V5.3 用延后静态绑定搞活面向对象编程](http://www.ibm.com/developerworks/cn/opensource/os-php-53static/)
