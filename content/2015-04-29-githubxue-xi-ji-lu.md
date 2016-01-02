+++
title  = "github学习记录"
date = "2015-04-29T00:31:18+08:00"
+++

现在想说说github的使用。


## remote到底是什么？
首先要直到remote的意思是什么。remote显然是相对local而言的，那么你在local可以干什么呢？可以创建或删除分支，所以你在remote也可以进行同样的操作，那么其实remote的意思就是存在于服务器上的代码库喽。而对于github而言，它默认的名字是origin，所以最好也不要改了，因为一看到origin就知道是服务端的代码库了。

所以，当你想要在本地进行开发时就有了两种选择，在github的页面上建好一个repo之后：

1. 把目录记好，clone开本地再继续

2. 在本地执行命令，直接在

```bash
git remote add origin git@github.com:lovelock/hackvim.git
```

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

这样就在push的同时在远端也新建了server分支。
