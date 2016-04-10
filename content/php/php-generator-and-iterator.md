+++
author = "frostwong@gmail.com"
date = "2016-04-05T12:05:46+08:00"
description = "description"
draft = false
keywords = ["generator", "iterator"]
tags = ["generator", "iterator"]
title = "使用PHP生成器和迭代器"
type = "post"

+++

从开始写PHP就知道迭代器这个东西，当时师傅告诉我用的挺少的，需要的时候再看也不晚，于是就没有放在意上。但他还说这其实也是区分高手和菜鸟的一个标志，那我还是研究一下好了：）

PHP程序员都知道我们最经常用的可能就是`foreach`这个大杀器了。得益于我们**万能的数组**，所以这个大杀器在多数场合都是可以直接用的，只要输入元素是数组类型即可——事实上并不是如此，`foreach`能遍历数组并不是因为它是数组，而是因为数组`implements`了`Iterator`接口。说白了就是只要告诉`foreach`遍历的规则，它就可以执行遍历，而和是否数组无关。

### Iterator

#### 解析

`Iterator`接口定义了5个方法，如果一个类要实现`Iterator`接口，当然就要实现这一套方法了。`Iterator`的原型如下

```php
Interface Iterator
{
	abstract public function current() : mixed;
	abstract public function key() : scalar;
	abstract public function next() : void;
	abstract public function rewind() : void;
	abstract public function valid() : boolean;
}
```

> 我在方法后面按照PHP7的新语法加了个返回值类型，其实这样写是不对的，但可以表明意思啦：）

详细说一下这几个方法要做的事情。

1. current()

	返回当前位置的**值**。

2. key()

	返回当前位置的**键**。
	
3. next()

	当前位置的**键**加1。
	
4. rewind()

	**键**回到第一个位置。
	
5. valid()

	返回当前的**键**是否是有意义的。如是否是`false`/`NULL`等。
	
#### 实例

还是来具体写个例子理解一下吧。通常写这种例子的作者都会举一个类，它的一个属性是个数组，然后实现`Iterator`的5个方法，来让这个类可以使用`foreach`，这个例子没意思，因为数组本身就带`current`、`key`这些方法。让我来举一个`pdo_mysql`从数据库中取数据的例子吧。

从数据库取出一一个数组，数组中的元素是`User`类的实例，我们需要`Users`类的方法，它又要有一些方法。所以，就产生了这样的用法了。这个例子可能有些牵强，但起码描述了一个使用场景，比单纯的迭代一个类的类型为数组的属性要有意义。

[代码地址](https://git.coding.net/lovelock/iterator_example.git)


### Generator

只有真正理解了`Iterator`才能再来谈`Generator`。

还是举例来说，前面已经说了一个比较复杂的例子，这里为了说明二者的区别，举个简单的例子。

假定有一个日志文件，1000000行吧，很大了？或许吧。现在我们要遍历这个文件，找到我们需要的东西。如果使用`Iterator`，可能需要这样

```php
<?php

class LinesIterator implements Iterator
{
	private $_fp;
	private $_currentLine;
	private $_lineNum;

	public function __construct($filename)
	{
		$this->_fp = fopen($filename, 'r');
		$this->_lineNum = 0;
	}

	public function current()
	{
		$this->_currentLine = fgets($this->_fp);
		return $this->_currentLine;
	}

	public function key()
	{
		return $this->_lineNum;
	}

	public function valid()
	{
		return $this->_currentLine === false;
	}

	public function next()
	{
		fgets($this->_fp);
		$this->_lineNum++;
	}

	public function rewind()
	{
	}

	public function __destruct()
	{
		fclose($this->_fp);
	}
}
```

需要遍历文件时，可以这样

```php
$file = new LinesIterator('file');

foreach ($file->current() as $line) {
	echo $line;
}
```

这没有问题，但也太复杂了吧！！！重点是即使我实现了这些，但还是无法随便定位到某一行（这需要`fseek`）。所以这种场景下，`Generator`出现了。

```php
function getLine($fileName)
{
	$fp = fopen($fileName, 'r');	

	while ($line = fgets($fp) !== false) {
		yield $line;
	}

	fclose($fp);
}
```

这样就简明多了。`Generator`的标志就是`yield`，这点在所有编程语言里都一样。

正常如果在用`yield`的地方用了`return`，那么代码执行到这里就结束了，下次再执行这个函数时，还是从头开始，我们永远得不到文件的第二行。那么怎么办呢？我的理解是`Generator`把这行内容返回的同时，也把文件句柄所在的指针向后移动了一个单位，下次再次执行该函数时，就会从上次的位置继续执行。

这个函数的功能和上面那个类的效果完全相同。

还有一点要提一下，`Generator`通常用来处理文件特别大的情况，比如上面这样，文件太大，如果直接用`file`读进来保存成为一个数组，很可能就会报错。而如果用`Generator`就没有这个问题了。
	




