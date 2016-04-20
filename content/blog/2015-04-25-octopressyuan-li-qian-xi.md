+++
title  = "octopress原理浅析"
isCJKLanguage = true
date = "2015-04-25T21:35:12+08:00"
+++

在学习的过程中，总想把一些心得记录下来，也就想到写博客，首先想到的就是WordPress了，
虽然也尝试了多个不同的云主机，但总是觉得用起来不够爽，程序员嘛，总归要像写代码一样
写文章才够感觉，然后就找到了github pages。也就是这个github pages让我是又爱又恨，就
是整不明白这个原理。

而且，github pages目的是为项目写文档，而不是持续的写博客，octopress是在这个基础
功能上添加了一系列工具链，这就让我更迷糊了，octopress的文档看了很多次，总是觉得
糊里糊涂的，不知道怎么操作，终于让我找到了[生命之氢 octopress教程目录](http://shengmingzhiqing.com/blog/octopress-tutorials-toc.html/)，
感觉就像醍醐灌顶，像昨天明白了[PHP单入口模式](http://unixera.com/blog/20150425/phpdan-ru-kou-mo-shi.html)
一样，忍不住把这个理解过程记下来。废话有点多，言归正传。

首先要搞明白的有两点

1. 不需要按照[github](https://github.com) 官网的gh-pages教程进行那些操作。
2. 不需要把新建的仓库clone到本地，所有的操作都是在octpress的目录中进行的。

几乎每篇关于octopress这种类似的工具的教程里面总会提到github 官网的github pages
教程，我觉得这就是把人搞晕的罪魁祸首。

因为你根本**不需要**像按那篇教程一样进行到创建目录之后的步骤，相反，如果你做了，出现的错误
反倒会让人不知所措，提示你代码库中的代码比当前要提交的新，需要先pull下来，问题是你根本
没有本地代码，又何谓的更新呢？

下面是整个过程的梳理。
{% img /images/octopress.png %}

1. 把octopress 的代码clone到本地，然后执行下面两条命令
    - `gem install bundler` 安装bundler, 也就是用来安装gem的工具
    - `bundle install` 安装需要的依赖
    - `rake install` 这个很有意思，rake = ruby make，所以这个命令的意义就像make install 一样把
        所需而文件拷贝到对应的目录，而在现在这个场景下，就是安装默认的主题。

2. 经过第一步的操作，现在写博客所需要的工具已经备齐了，就差一个远端存储它的仓库和文章的内容了。

3. 这一步才需要在github新建一个代码仓库，这个仓库的名字必须是固定的格式，`git@github.com:username/username.github.io`，
或者`https://github.com/username/username.github.io`，这两种格式分别是使用ssh协议和https协议，
具体区别可以参考github上的介绍文章。个人喜好是使用ssh协议，虽然github不建议使用这种方式，因为
每次提交代码时不需要输入密码，但这样更方便不是吗。需要注意的一点是这时也不要在新建的仓库种
添加README.md文件，否则又要出现前面所说的代码新旧的问题了。

4. 这时才要到`octopress` 的目录中去执行各种命令了。

这时你可能还是像我一样有疑问，`octopress`的作用到底是什么呢？

答案是：把你写的md文件（可以理解成源文件，位于source目录）编译成html文件（可以理解成目标文件，位于_deploy目录），
然后把你的编译得到的文件`commit+push`到你的`username.github.io`仓库的`master`分支，也就是说你的
源文件没有被提交，所以这时需要
    - `git add .` 将octopress 目录中位于`source` 目录下的文件添加到你的`username.github.io`的仓库的版本库中
    - `git commit -m 'init source'` 提交上述文件
    - `git push origin source` 把文件`push`到仓库的`source` 分支中。

现在问题就一切都了然了，`rake generate`命令把你写的md文件编译成html，`rake deploy`把生成的html提交到master分支，
然后你需要做的就是把source提交到source分支了。

-- source 分支存储的是文章的md文件，而master分支存储的是编译后的文件。

说的可能有点罗嗦，但我认为这样是最不容易引起歧义的，我在学习的过程中不明白的地方都着重说了，也可能是
我自己的思维习惯和常人有所不同，有些东西感觉不太容易理解，但总之这下我能开心的写东西了。
