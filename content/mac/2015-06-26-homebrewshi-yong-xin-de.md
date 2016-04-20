+++
title  = "homebrew使用心得"
isCJKLanguage = true
date = "2015-06-26T10:25:07+08:00"
topics = ["Mac"]
tags = ["homebrew"]
+++

今天同事问我在Mac OS X上安装软件的问题，我看到是下载的过程出了问题，并且出在证书上，想起来之前看到说curl在某个版本之后就把证书和二进制文件打包在一起了，就想着让她更新一下curl试试看。我之前在自己的机器上也用brew安装过curl，但并没有注意到其实安装的版本默认并没有生效。

这里要引出的一种安装方式就是`keg-only`。

看到这我就想插一句，国外的程序员们可真会玩儿，能给软件起很有意思的名字，连其中的组件都很有意思。比如这里，homehrew默认是把软件安装在`/usr/local/Cellar`目录中的，而`Cellar`的意思是**酒窖**。如果你了解过`FreeBSD`的目录树结构，就知道它这么做的意义了，是要把同一个来源的文件组织在一起，不扰乱整个目录树。相应的，`keg-only`中的`keg`的意思是**酒桶**。就让我试着用平实的语言来解释一下：
Homebrew安装软件分为[两种情况](http://stackoverflow.com/questions/4691403/keg-only-homebrew-formulas)。
 1. 系统没有自带的 
   这个没什么好说的，因为如果系统没有带，我们安装完相应的软件之后就自动的将编译好的二进制文件软链接到`PATH`中，这样才会生效。
 2. 系统自带的
   如果系统自带的有这个软件，那么问题就不好办了，是直接覆盖呢，还是应该给用户一些选择？本着上面说过的原则，尽量少的影响原来的目录树，那么它在安装完二进制文件之后并没有做建软链的那一步操作，这就是所谓的`keg-only`的意思了。


那么，如果我要更新curl，而OS X发行时本就是带着curl的，应该怎么办呢？当然作者已经替我们想到了这一点，

```
brew link curl --force
```
这样，它就会把需要的二进制链接到`PATH`中，要注意这时这些路径中是存在相应的二进制文件的，正是homebrew不敢确定是不是要直接帮我们做这些操作，才给我们提供了这个命令。

那么如果你之前已经安装了不少软件了，发现有好几个没有建软链，难道要手工的一个一个执行？当然不用，可以参考下面的[脚本](http://apple.stackexchange.com/questions/123900/is-there-a-quick-way-to-relink-my-homebrew-kegs)

```bash
ls -1 /usr/local/Library/LinkedKegs | while read line; do
    echo $LINE
    brew unlink $LINE
    brew link $LINE --force
done
```

上面的脚本是利用了homebrew将`LinkedKegs`目录设置成了存放所有二进制文件的路径的特点，将其全部取出，删除全部软链，而后重新建立软链。

其实也可以将`ls -1 /usr/local/Library/LinkedKegs`替换成`brew list -1`也可。

另外别的诸如`brew install`，`brew uninstall`的方法就不说了。

我不是一个Python爱好者，只是一个用户，偶尔写点Python，但即便这样也会遭遇版本问题，`brew install python3 python2`一站式解决所有问题。

多说一点，现在很多语言都开发了包管理工具，我知道的有pip、npm、composer、gem，甚至我最不了解的Java，像Maven估计也是用来做这种事情的。

这其实就是在帮**操作系统或者说发行版减负**。之前没有这些语言的包管理器的时候都是需要系统给打包，比如各种pip的包，那现在，它就成了跨平台的，每个发行版只需要维护Python的包，只要用户有了Python就有了pip，有了pip就有了无限的可能。

但我发现周围根本没有人用这些东西，虽然Mac的普及率很高，但大部分人都没有听过这些东西，我们的PHP程序员本来也不必关心别的语言的东西，但了解一些提高效率的东西还是很有必要的。
