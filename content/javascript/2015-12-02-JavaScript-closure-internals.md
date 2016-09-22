+++
categories = ["Javascript"]
title  = "JavaScript闭包深究"
isCJKLanguage = true
date = "2015-12-02T21:35:12+08:00"
keywords = ["JavaScript", "Closure"]
tags = ["JavaScript", "Closure"]
topics = ["javascript"]
+++


## 问题来源

刚刚看到微信公众号推送的一篇文章, [大部分人都会做错的经典JS闭包面试题](http://www.cnblogs.com/xxcanghai/p/4991870.html),里面出现了一道很有意思的题目，让我有兴趣自己尝试一下。

题目是这样的

```javascript
function fun(n, o) {
	console.log(o);
	
	return {
		fun: function (m) {
			return fun(m, n);
		}
	};
}
```

问题是下面这三行的输出分别是什么。

> 注意不是问a, b, c的值，是输出。

```javascript
var a = fun(0);  a.fun(1);  a.fun(2);  a.fun(3);//undefined,?,?,?
var b = fun(0).fun(1).fun(2).fun(3);//undefined,?,?,?
var c = fun(0).fun(1);  c.fun(2);  c.fun(3);//undefined,?,?,?
```

答案在最后，如果有兴趣可以先尝试自己分析一下。这道题的答案并没有争议，但我看了原作者的分析之后，感觉到可能有点问题，于是找到了经常翻阅的《JavaScript the Good Parts》再来参详一遍第四章吧，大概翻了一下发现这本书并没有太详细的介绍这部分，于是找了一篇博文[^refer1]，详细的研究一下。

函数(function)在JavaScript中是一等公民，老道说它是几乎完美的——当然也存在瑕疵，当你用过它变态的`this`特性以后，肯定会对这句话的严谨表示赞同。

我们知道，在JavaScript中创建对象时，对象字面量会连接到`Object.prototype`。而函数对象会连接到`Function.prototype`。那么`Function.prototype`又连接到谁呢？答案还是`Object.prototype`。因为在JavaScript中，函数、数组、对象(指我们经常见到的JSON对象)都是对象(特指原型链的对象)。

## 探究
根据《JavaScript the Good Parts》的说法，函数的定义有4个要素。

```javascript
// 创建一个名为add的变量，并把一个函数赋值给它

var add = function (a, b) {
	return a + b;
};
```

函数字面量包括4个部分：

1. 保留字`function`
2. 函数名
3. 包围在圆括号内的一组参数
4. 包围在花括号中的一组语句，它是函数的主题，在函数调用时被执行

从上面函数的定义可以看到，这个`add` 并不是函数字面量的一部分。而作为函数字面量的函数名却可以不存在，这也就是匿名函数。

重点来了：

**函数字面量可以出现在任何允许表达式出现的地方。函数也可以被定义在其他函数中。一个内部函数除了可以访问自己的参数和变量，同时也能自由访问把它嵌套在其中的父函数的参数与变量。通过函数字面量创建的函数对象包含一个连到外部上下文的连接。这被称为闭包(closure)。它是JavaScript强大表现力的来源。**

### 函数的定义方式

先来看一下JavaScript定义函数的各种方法

```javascript
function A() {}     // 函数声明
var B = function () {}; // 函数表达式
var C = (function () {}); // 带有分组操作符的函数表达式
var D = function foo() {}; // 命名的函数表达式
var E = (function () {
            return function () {};
        }());           // 返回函数的立即执行的函数表达式
var F = new Function (); // 函数构造器
var G = new Function () {}; // 特殊情况，对象构造器
```

不看不知道，竟然还有那么多种方式，老实说，我只有过第1、2、5这三种，并且没有区分的那么细致。下面详细的剖析一下这种方式的使用场景。

#### <a name="函数声明"></a>函数声明
##### <a name="作用域提升"></a>作用域提升
这种方式是见到的最多的声明函数的方式了，而且这种方式会引发『作用域提升』(Hoisting)，不知道是不是这么翻译，但我一说下面的你就知道是什么意思了。

如果这么写

```javascript
A();
function A() {
    console.log('foo');
}
```

你可能会担心出现`A is not a function`这种错误吧？但是这里不会有，一切都会正常运行。因为在执行这段代码时，解释器会把它当做下面的这样来执行

```javascript
function A() {
    console.log('foo');
}
A();
```

这样当然不会有问题了。

这里说到变量的作用域提升，你可能会想到下面这种例子，

```javascript
console.log(A);

var A = 'foo';
```

这时也会发生一个变量提升，但不同的是，这段代码等同于下面的

```javascript
var A;

console.log(A);

A = 'foo';
```

所以，这段代码的输出会是`undefined`，而不是`A is not defined`的错误信息。也就是说，这种方式的变量作用域提升只会把变量的**声明**提升，而不会把**定义**提升。

变量声明和函数声明方式的不同了，上面说了，函数在JavaScript里是可以赋值给变量的，那么用函数表达式定义的函数也是一个变量，本质上会和上面**变量作用域提升**出现同样的效果。即

```javascript
B();

var B = function () {
    console.log('foo');
};
```

并不会有输出，并且会报错，`B is not a function`，因为它等同于

```javascript
var B;

B();

B = function () {
    console.log('foo');
};
```

在执行函数B的时候，B只是一个变量声明，解释器并不知道它是一个函数或者一个普通的变量，但看报错结果`B is not a function`就可以知道，在执行函数B的时候，解释器是知道B的存在的，也就是B已经被声明过，只是它不知道它是函数而已，否则报的错就是`B is not defined`了。

##### 不能在`if`或其他条件语句中使用函数声明
也就是说，不能这样

```javascript
if (true) {
    function foo() {
        return 'foo';
    }
} else {
    function foo() {
        return 'bar';
    }
}
```

当我想在[JSbin](http:jsbin.com)中执行这段代码时，它报出了以下错误
![条件语句中用函数声明报错](http://7xn2pe.com1.z0.glb.clouddn.com/b_Screen%20Shot%202015-12-05%20at%2010.20.25%20PM.png)

建议的方式是这样的

```javascript
var foo;

if (true) {
    foo = function () {
        return 'foo';
    };
} else {
    foo = function () {
        return 'bar';
    };
}
```

**永远不要**这样写，不同的浏览器可能会表现出不同的行为。

##### 必须有函数名

声明一个没有赋值的函数是不能不带函数名的，也就是，无法这样声明以下函数

```javascript
function () {
    return 'blahblah';
}
```

![用函数声明的方式声明不带函数名的函数报错信息](http://7xn2pe.com1.z0.glb.clouddn.com/b_Screen%20Shot%202015-12-05%20at%2010.24.19%20PM.png)

#### <a name="函数表达式"></a>函数表达式

`var B = function () {};`

函数表达式和函数声明的方式很类似，但不同的是函数被赋值给了一个变量。要知道函数在JavaScript里并不是primitive的，也就是说它是可以再分解的，但这并不妨碍函数是JavaScript世界的一等公民，它可以被赋值给变量，可以作为其他函数的返回值，可以作为参数传递给其他变量也可以存储在其他数据结构中。

##### 匿名函数

虽然函数赋值给了一个变量，但这个函数仍然是一个匿名函数，因为在函数定义的4个要素里，缺少了『函数名』这一要素。

##### 变量的作用域提升
这在[上一节](#作用域提升)中已经讲过，这里略过。

#### 带有分组操作符的函数表达式
这种写法很应该很少见，或者说只会出现在测试题中了，因为它除了会带来迷惑之外并没有什么卵用（如果真有实际用处，请一定告诉我）。

可以这样理解这种写法。
当你写

```javascript
function () {}
```

的时候，它是一个『函数声明』，函数声明没有函数名是不正确的。而当你给它加上一对括号，那么它就成了一个『函数表达式』，还记得吗？函数表达式是可以赋值给变量的。

这时候就可以把`(function () {})`理解成一个没有被赋值的函数表达式。它的存在是没有意义的，就像你写了一个这样的表达式`'foo';`，它能有什么意义？对，它是一个字符串，谁都知道它是一个字符串，但它并没有被赋值给任何一个变量，换言之，在依赖『引用计数』做垃圾回收的语言里（没有研究过JavaScript是不是这种），它在执行时是会被回收的——因为它并没有被引用过。只有当`var C = (function () {});`时才有意义，而这时它和`var C = function () {};`的意义是一样的。

#### 带有变量名的函数声明
 
`var D = function foo() {}` 这种写法简直综合了[函数声明](#函数声明)和[函数表达式](#函数表达式)，搞得更让人摸不到头脑了。

##### 函数名只在函数内部可见
在这种情况下，函数名在函数外部是不可见的，例如

```javascript
var D = function foo(){
  console.log(typeof foo);
};
D();                       // function
console.log(typeof foo);   // undefined
```

##### 需要递归执行时很有用
想一下它的应用场景，在函数内部可以调用自己，但外部不需要知道它的实现机制。外部调用它仍然可以用变量名，然而调用变量名一次，却可以递归的调用函数内部的实现，是不是很酷炫呢？

```javascript
var countdown = function a(count){
  if(count > 0) {
    count--;
    return a(count);  // we can also do this: a(--count), which is less clear
  }
  console.log('end of recursive function');
}
countdown(5);
```

##### debug时很有用
这个嘛，看你有没有需要了，因为匿名的函数表达式是没有名字的，那再函数内部调用自己的时候也就无法记录被调用的函数名了，但如果函数表达式也有了名字，那事情就变得很奇妙了。

##### JScript的实现有坑
[kangax](http://kangax.github.io/nfe/)指出，IE的ECMAScript实现——JScript对这一特性的实现是有坑的，所以，如果有对这方面的需求的话，最好回避这个功能。

#### 立即执行的函数表达式
`var E = (function () { return function () {}; }())`

要理解这一特性，先要把它做的事情分解

1. 执行一个函数
2. 函数的返回值是函数
3. 将返回的函数赋给一个变量

为了深入浅出的理解这种做法，可以先尝试一种已经接受的方式，

```javascript
var foo = function () {
    return 'bar';
}

var output = foo();
console.log(output);
```

很显然，这段代码的输出就是`bar`。没什么好说的。

但是经过上面的讨论，我们知道了
`var foo = function () { return function () {}; }`是等价于`var foo = (function () { return function () {}; })`的，那么同时加个括号呢？对，
就变成了`(function () { return function () {}; })()`。到这里，这种写法是不是就很好理解了呢？

**<mark>注意</mark>**这个特性很重要，尤其在模块模式下隐藏一些信息时很有用。最常见的jQuery就用到了这种方法。

#### 构造器
`var F = new Function();`

这是一种很老的并且已经不推荐使用的写法。而且，根据老道的推荐，所有直接使用构造器，也就是使用了`new`关键字的方式都不推荐使用。这是因为他是一个强迫症——好吧，原因如下：

> JavaScript的`new`运算符创建一个继承于其运算数的原型的新对象，然后调用该运算数，把新创建的对象绑定给`this`。这给运算数（应该是一个构造器函数）一个机会在返回给请求者前自定义新创建的对象。
> 如果忘记了使用`new`运算符，得到的就是一个普通的函数调用，并且`this`被绑定到全局对象，而不是创建新的对象。这意外着当你的函数尝试去初始化新成员属性时它将会污染全局变量。这是一件非常糟糕的事情（老道非常不愿意看到JavaScript语言里的变量名的污染）。而且既没有编译时警告，也没有运行时警告。
> 按照惯例，打算域`new`结合使用的函数应该以首字母大写的形式命名，并且首字母大写形式应该只用来命名那些构造器函数。这个约定帮助我们进行区分（然而我还没有看到我修改的代码里真的遵守这个约定的），便于我们发现那些JavaScript语言自身经常忘记但却会带来昂贵代价的错误。
> **一个更好的应对策略就是根本不去使用`new`关键字。**

##### 使用方法

```javascript
var F = new Function ('arg1', 'arg2', 'console.log(arg1 + ", " + arg2)');
F('foo', 'bar'); // 'foo, bar'
```

##### 避免使用`new`操作符
##### 怪癖

[MDN开发文档](https://developer.mozilla.org/en/JavaScript/Reference/Functions_and_function_scope#Function_constructor_vs._function_declaration_vs._function_expression)指出了另一个这种方式不好的地方，它不能正确的创建闭包。


```javascript
function foo () {
    var bar = 'bar';
    
    var first = new Function('console.log(typeof bar')');
    first(); //undefined
    
    var second = function () {
        console.log(typeof bar);
    };
    second(); // string
}

foo();
```

从结果可以看到，使用构造器创建的函数并不能正确的创建闭包，所以，这种方式还是不要用了。

#### 特殊情况
说它是特殊情况是因为虽然用了`function`关键字，但它并没有意义。

`new function() {}`创建一个对象并且调用这个匿名函数来作为它的构造函数。如果函数返回的是一个对象，它的结果就是一个对象，否则就会从头创建一个对象并且函数在这个新函数的上下文中执行。（好绕，回头但针对这个问题再另起一篇）。

如果看到这了，你还没有整明白文章开头的那道题，那我也没有办法了。

答案：
//a: undefined,0,0,0
//b: undefined,0,1,2
//c: undefined,0,1,1

[^refer1]: [Different Ways of Defining Functions in JavaScript (This Is Madness!)](http://davidbcalhoun.com/2011/different-ways-of-defining-functions-in-javascript-this-is-madness/)
