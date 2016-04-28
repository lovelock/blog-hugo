+++
title  = "JavaScript执行总结"
isCJKLanguage = true
date = "2015-04-28T22:03:47+08:00"
topics = ["javascript"]
keywords = ["async", "sync", "javascript"]
+++

在所有的场景下，都有一个**将来某个时刻**会执行的回调函数，这**将来某个时刻**就是我们所说的**异步流**。

异步执行会被推出**同步流**。也就是说，当同步代码正在执行时异步代码就不会执行。这就是JavaScript单线程的意义。

更具体地说，当JS引擎空闲时——没有在执行同步或者异步代码——它会轮询那些被触发的异步回调事件（例如到期时间(setTimeout)，接收到的网络响应(ajax)）并且一一的执行它们。这就是**事件循环**。

也就是说，回调代码有可能会在**所有同步代码**在它们各自的代码块中**执行完毕**之后才会执行。

简而言之，回调函数是被同步创建但异步执行的。除非你知道这个异步函数**确实**已经执行了，否则你不可以依赖一个异步函数的执行，如何做到呢？

这很简单，真的。依赖异步函数执行的逻辑应该在异步函数内部开始或调用。

```javascript

helloAsyncCat(function (result) {
    alert(result);
}

function helloAsyncCat(callback) {
    setTimeout(function () {
        callback('Nya');
    }, Math.random() * 2000);
}

```

上面的代码是一段简单的例子。
首先，需要为`helloAsyncCat`函数添加一个回调函数，它是在异步执行的内部执行的。
然后，在调用该函数的时候就可以用回调函数中取到的结果进行操作了。
