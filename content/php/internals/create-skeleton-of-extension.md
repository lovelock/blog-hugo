+++
author = "frostwong"
date = "2016-04-09T22:04:18+08:00"
description = "PHP扩展实战"
draft = false
keywords = ["PHP", "扩展"]
tags = ["PHP", "扩展"]
title = "PHP扩展实战——扩展的骨架"
topics = ["PHP"]
type = "post"
isCJKLanguage = true

+++

前面啰嗦了这么多读者都要没有兴趣了。从现在起要真正开始PHP扩展开发阶段了。

首先来生成扩展的骨架。所谓骨架就是一个扩展需要的基本文件了。

## 获取PHP源码

截至目前，PHP最新源码是7.0.5。[下载链接](http://cn2.php.net/get/php-7.0.5.tar.bz2/from/this/mirror)

```bash
➜  projects wget http://cn2.php.net/get/php-7.0.5.tar.bz2/from/this/mirror -O php705.tar.bz2
--2016-04-09 10:18:39--  http://cn2.php.net/get/php-7.0.5.tar.bz2/from/this/mirror
Resolving cn2.php.net (cn2.php.net)... 202.108.35.194, 202.108.35.235, 202.108.35.237, ...
Connecting to cn2.php.net (cn2.php.net)|202.108.35.194|:80... connected.
HTTP request sent, awaiting response... 302 Found
Location: http://cn2.php.net/distributions/php-7.0.5.tar.bz2 [following]
--2016-04-09 10:18:39--  http://cn2.php.net/distributions/php-7.0.5.tar.bz2
Reusing existing connection to cn2.php.net:80.
HTTP request sent, awaiting response... 200 OK
Length: 14086522 (13M) [application/octet-stream]
Saving to: ‘php705.tar.bz2’

php705.tar.bz2                             100%[=======================================================================================>]  13.43M  4.49MB/s    in 3.0s

2016-04-09 10:18:42 (4.49 MB/s) - ‘php705.tar.bz2’ saved [14086522/14086522]

➜  projects md5sum php705.tar.bz2
b15e6836babcbf0aa446678ee38f896b  php705.tar.bz2
➜  projects echo b15e6836babcbf0aa446678ee38f896b
b15e6836babcbf0aa446678ee38f896b
➜  projects tar xjf php705.tar.bz2
➜  projects cd php-7.0.5/ext
```

终于来到了正题了。我现在也终于明白鸟哥为啥费劲写个生成Yaf最小化应用的脚本了，就是从写扩展的经历中得来的，既然可以帮用户做的更多，那就帮一下好了。

```bash
➜  ext ./ext_skel --extname=hylog
Creating directory hylog
Creating basic files: config.m4 config.w32 .gitignore hylog.c php_hylog.h CREDITS EXPERIMENTAL tests/001.phpt hylog.php [done].

To use your new extension, you will have to execute the following steps:

1.  $ cd ..
2.  $ vi ext/hylog/config.m4
3.  $ ./buildconf
4.  $ ./configure --[with|enable]-hylog
5.  $ make
6.  $ ./sapi/cli/php -f ext/hylog/hylog.php
7.  $ vi ext/hylog/hylog.c
8.  $ make

Repeat steps 3-6 until you are satisfied with ext/hylog/config.m4 and
step 6 confirms that your module is compiled into PHP. Then, start writing
code and repeat the last two steps as often as necessary.
```

这样，`ext_skel`就帮我们生成了一个名为`hylog`的扩展框架。

下面要介绍一下安装扩展的两种方式了，一种是直接编译进PHP，一种是接下来我们要讨论的这种，即动态加载的扩展。

什么是直接编译进PHP呢？

```bash
➜  ext cd hylog
➜  hylog ls
config.m4  config.w32  CREDITS  EXPERIMENTAL  hylog.c  hylog.php  php_hylog.h  tests
➜  hylog vim config.m4
```

会看到这样的几行

```m4
dnl If your extension references something external, use with:

dnl PHP_ARG_WITH(hylog, for hylog support,
dnl Make sure that the comment is aligned:
dnl [  --with-hylog             Include hylog support])

dnl Otherwise use enable:

dnl PHP_ARG_ENABLE(hylog, whether to enable hylog support,
dnl Make sure that the comment is aligned:
dnl [  --enable-hylog           Enable hylog support])
```

其中的`dnl`是注释，主要看`--with-hylog`和`--enable-hylog`。假定你来看本文，你一定自己编译过PHP了，如果没有，那先去整一遍再回来看吧：）
是这样的，我们在编译PHP的时候经常会碰到类似这种`--with[out]-blah=/path/to/foo`或者`--enable-blah`或者`--disable-blah`的选项吧。其实对编写扩展的我们来说，这两种都是可行的，并没有本质上的区别，只是一般用`--with`会带个路径，告诉PHP这个扩展依赖的外部库的路径，而`--enable`则表示该扩展是独立的，或者依赖的库在默认的搜索路径内。

那和我们说的两种安装方式有什么关系呢？不如我们就来真的安装一下看看效果吧。

看上面的注释，我们知道了需要把

```m4
dnl PHP_ARG_ENABLE(hylog, whether to enable hylog support,
dnl Make sure that the comment is aligned:
dnl [  --enable-hylog           Enable hylog support])
```

这段改成

```m4
PHP_ARG_ENABLE(hylog, whether to enable hylog support,
dnl Make sure that the comment is aligned:
[  --enable-hylog           Enable hylog support])
```

至于是喜欢用`--enable`还是喜欢用`--with`看个人喜好了，因为本例中并没有用到外部依赖，所以用`--enable`。

提醒一下，改完之后最好把当前的这个状态保存下来——创建一个git工作目录就好了。

```bash
➜  hylog git init
Initialized empty Git repository in /home/frost/projects/php-7.0.5/ext/hylog/.git/
➜  hylog git:(master) ✗ gst
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	.gitignore
	CREDITS
	EXPERIMENTAL
	config.m4
	config.w32
	hylog.c
	hylog.php
	php_hylog.h
	tests/

nothing added to commit but untracked files present (use "git add" to track)
➜  hylog git:(master) ✗ ga .
➜  hylog git:(master) ✗ gc -m 'init hylog'
[master (root-commit) 58e5e4a] init hylog
 Committer: frost <frost@debian.unixera.com>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 9 files changed, 409 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 CREDITS
 create mode 100644 EXPERIMENTAL
 create mode 100644 config.m4
 create mode 100644 config.w32
 create mode 100644 hylog.c
 create mode 100644 hylog.php
 create mode 100644 php_hylog.h
 create mode 100644 tests/001.phpt
 ```
 
为什么要这么做呢？其实主要是想让这个目录干净，因为待会儿执行了一些命令之后会生成很多文件，如果你想清除这些文件就变得很麻烦。但现在我只把这些文件`commit`了，待会儿生成文件后，如果我想删除，就可以用`git clean -df`，立即回到现在的状态。但关于`git`的操作，那就是另外一回事了（强烈推荐[廖雪峰的git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000))。
 
 
### 编译进PHP

1. 重新生成配置文件
	
	注意其中的`./buildconf --force`，之所以带`--force`是因为我们是在正式版的PHP源码中进行操作的，正常情况下使用这种方式编译的都是内建扩展，例如`PDO`这种，是PHP官方团队开发的，所以你非要用这种方式编译的话，就强制一下好了。
	
	```bash
	➜  hylog git:(master) cd ..
	➜  ext cd ..
	➜  php-7.0.5 ./buildconf --force
	Forcing buildconf
	Removing configure caches
	buildconf: checking installation...
	buildconf: autoconf version 2.69 (ok)
	rebuilding aclocal.m4
	rebuilding configure
	rebuilding main/php_config.h.in
	```

2. 查找变化
	
	刚刚的操作背后发生了什么呢？注意`rebuilding`的三行，那我们就挨个看看。分别在三个文件中搜索`hylog`关键字吧。
	在`aclocal.m4`中未找到变化。
	在`configure`中有大量变化，稍后介绍能看到的变化。
	在`main/php_config.h.in`中，增加了两行，用来取消`COMPILE_DL_HYLOG`的定义，表示该扩展不是动态加载。
	
	这时检查一下`configure --help`
	
	```bash
	➜  php-7.0.5 ./configure --help | grep hylog
	  --enable-hylog           Enable hylog support
	```

诶，有点眼熟对不对？就是刚才在`ext/hylog/config.m4`中取消注释的内容。

3. 编译PHP

	既然要把它编译进来，那就加上`--enable-hylog`吧。
	
	```bash
	➜  php-7.0.5 ./configure --enable-hylog
	➜  php-7.0.5 make
	➜  php-7.0.5 sudo make install
	```

4. 查看已安装的扩展
	
	```bash
	➜  php-7.0.5 php -v
	PHP 7.0.5 (cli) (built: Apr  9 2016 11:08:08) ( NTS )
	Copyright (c) 1997-2016 The PHP Group
	Zend Engine v3.0.0, Copyright (c) 1998-2016 Zend Technologies
	➜  php-7.0.5 php -m
	[PHP Modules]
	Core
	ctype
	date
	dom
	fileinfo
	filter
	hash
	hylog
	iconv
	json
	libxml
	pcre
	PDO
	pdo_sqlite
	Phar
	posix
	Reflection
	session
	SimpleXML
	SPL
	sqlite3
	standard
	tokenizer
	xml
	xmlreader
	xmlwriter
	
	[Zend Modules]
	```
	
	现在可以看到我们新创建的扩展已经编译进PHP了——虽然它没有任何功能。可以再到`/usr/local/lib/php/extensions/no-debug-non-zts-20151012`中验证一下是不是真的没有`hylog.so`存在。
	
	所以如果不想用它了怎么办呢？你当然可以选择无视它，但最好还是卸载了吧，卸载的方法也很简单，
	
	
	```bash
	➜  php-7.0.5 ./configure --disable-hylog
	➜  php-7.0.5 make
	➜  php-7.0.5 sudo make install
	```

	看起来就是三行，其实要用很长时间，所以，像我们这样的第三方扩展开发者还是不要用这种方式比较好。

### 动态加载

动态加载方式是把每个扩展编译成一个单独的`.so`文件，然后在`php.ini`中加上`extension=hylog.so`，如果有配置就再加上一些配置。CLI的话就直接生效了，FPM环境下就要重启一下FPM了。我们这里只讨论CLI模式。

1. 第三方扩展安装的一般流程

	还记得我刚刚提到的执行某些命令后会生成很多文件吗？就是这里了。如果你还没有用`git`，我劝你现在用了。
	
	```bash
	➜  hylog git:(master) phpize
	Configuring for:
	PHP Api Version:         20151012
	Zend Module Api No:      20151012
	Zend Extension Api No:   320151012
	➜  hylog git:(master) ✗ ./configure
	➜  hylog git:(master) ✗ make
	➜  hylog git:(master) ✗ sudo make install
	Installing shared extensions:     /usr/local/lib/php/extensions/no-debug-non-zts-20151012/
	```

	好，到这里已经看到在独立编译动态扩展时，生成的`.so`文件是放在了这个目录下的。这时动态的好处就体现出来了。文件有了，至于你想不想用，只需要修改`php.ini`即可，不用任何重新编译。

2. 安装和卸载扩展

	前面说了，如果需要该扩展，编辑`/usr/local/lib/php.ini`，在最下面添加（安装）或删除（卸载）一行

	```ini
	extension=hylog.so
	```

3. 调试

	我可不敢保证代码一次就能成功，调试的时候要多次执行以上三个命令，所以可以创建一个`rebuild.sh`脚本，运行脚本重新编译并安装最新的版本。
	
	```bash
	./configure
	make
	sudo make install
	```

最好把它加入到`git`工作目录中。

扩展的安装就这些，下一节介绍PHP变量的基本类型。
	
	



