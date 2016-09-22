+++
categories = ["Git"]
title  = "github学习记录"
isCJKLanguage = true
date = "2015-04-29T00:31:18+08:00"
tags = ["git"]
topics = ["git"]
+++

> 这是我自己总结的一些比较容易迷惑的地方，更详细的教程推荐[廖雪峰的git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

## remote到底是什么？
首先要知道remote的意思是什么。remote显然是相对local而言的，那么你在local可以干什么呢？可以创建或删除分支，所以你在remote也可以进行同样的操作，那么其实remote的意思就是存在于服务器上的代码库喽。而对于github而言，它默认的名字是origin，所以最好也不要改了，因为一看到origin就知道是服务端的代码库了。

所以，当你想要在本地进行开发时就有了两种选择，在github的页面上建好一个repo之后：

1. 把目录记好，clone开本地再继续

2. 在本地执行命令，直接在

```bash
git remote add origin git@github.com:lovelock/hackvim.git
```

## 多个remote

我们都爱github,但国内的环境实在是太慢了，所以平时也可以用国内的平台，像coding.net，oschina等。这样，让一个工作目录可以同时被提交到多个远程目录。

```bash
git remote set-url --add origin git@coding.net:lovelock/hackvim.git # 增加remote
git remote set-url --delete origin git@coding.net:lovelock/hackvim.git 删除指定remote
```

需要注意的是上面两个命令有条件，如果现在本地代码仓库只有一个remote了，就不能再删除remote了。

## tag
每次提交虽然都会写上提交message，但这仍然替代不了版本号。git的版本号不像svn那样是一个数字，而是一个hash，人眼当然不能看到它的高低的，所以手动指定一个版本号就变得必要了。这个功能可以使用`git tag -a v0.1 -m 'version 0.1'`这样的命令来操作，注意这里的`-a`选项，它不是必选项，但很重要。因为如果忽略了这个选项，新建的标记就仅仅是一个标记，标记就是这次提交的一个代称。而如果加了`-a`选项，创建的就是一个标签对象，并且需要一个标签消息，
当执行这个命令后，就在git对象库中添加了一个新的对象，并且标签引用指向了一个标签对象而不是指向一个提交。

## 分支
现在我遇到的问题是我创建了一个[vim的配置文件](https://github.com/lovelock/hackvim.git)，在我本地的ArchLinux上用的是风生水起，那叫一个爽，可是在公司用的服务器上就没有那么爽了，因为很多环境不是我可以决定换的，有些插件用不了，但主要的配置还是想和主版本保持一致，怎么办呢？

新建一个分支。

```bash
git branch server
git checkout server

# 做一些适应server的修改

git add .
git commit -m 'add server branch'
git push origin server:server
```

如果有一天我不想要这个server分支了，那么可以用同样的方式

```bash
git push origin :server
```

就把远程的分支删除了。其实这是一种不准确的说法，看这个命令就知道了，它并不是真正的删除命令，而是把一个空分支推到了remote的server分支，所以实际上server分支就被删除了。

这样就在push的同时在远端也新建了server分支。

## 取消提交的文件

可能会出现这样的情形，比如我创建了一个PHP的项目，用了composer，在本地安装了很多依赖，然后我就直接添加所有文件，然后提交了，这样代码仓库就会变得很大，而且存了很多没有用的东西，大家都知道可以加个.gitignore文件来避免，那现在我已经提交了，再添加这个文件也没有用了。当然git不会那么傻，这都是可以解决滴

```
git rm -r --cached vendor
```

然后再提交，push，再看remote，是不是已经没有那些文件了？


上面虽然写了那么多，其实我并没有在生产环境也就是公司环境用过Git，到了新公司我所掌握的这些知识就捉襟见肘了。还是另开一篇写这些话题吧，算是这篇的进阶。


