+++
title  = "Unix文件属性小全"
isCJKLanguage = true
date = "2015-06-26T18:48:56+08:00"
+++


玩Linux的最经常碰到的问题中，文件属性（权限）的问题应当属相当多见的。而今天又碰到了一个新鲜事儿，属性中又出现了一个特殊字符`@`。

由于在搜索引擎中搜索特殊符号挺困难，我花了十几分钟才搜到相关的内容。原来是OS X上的文件独有的一个属性值，它代表该文件还有`extended`字段，而具体这个字段的内容是什么就要用`xattr -l`来查看了。

既然碰到了这个问题，我就索性总结一下这个问题。以下的测试基于CentOS 7，与OS X相关部分基于OS X 10.10.3。

在Linux文件系统中，文件的属性分为X种。

1. 文件类型

    分为三种：

    1. - 普通文件
    2. d 目录
    3. l 链接文件

2. rwx权限

    rwx权限分为三组，分别为**所属用户**，**所属组其他用户**，**其他用户**。

    这里要注意的是对一个目录而言，`wx`权限对普通文件来说很容易理解，即该用户是否有执行该文件的权限，但对目录而言就不好理解了。

    - 如果一个用户对一个目录有`w`权限，则表示他可以在该目录中新建或删除或修改文件。
    - 如果一个用户对一个目录有`x`权限，则表示他可以进入该目录。

```bash

[frost@localhost test]$ mkdir w_directory
[frost@localhost test]$ sudo chmod -w w_directory/
chmod: w_directory/: new permissions are r-xrwxr-x, not r-xr-xr-x
[frost@localhost test]$ cd w_directory/
[frost@localhost w_directory]$ touch testfile
touch: cannot touch ‘testfile’: Permission denied

```

> 给用户去掉`w`权限之后就无法对目录中的文件做任何修改了。

```bash

[frost@localhost test]$ sudo chmod -x x_directory
[sudo] password for frost:
[frost@localhost test]$ ls -l
total 0
dr-xrwxr-x. 2 frost frost 6 Jun 20 07:18 w_directory
drw-rw-r--. 2 frost frost 6 Jun 20 07:22 x_directory
[frost@localhost test]$ cd x_directory/
-bash: cd: x_directory/: Permission denied

```

> 给用户取消`x`权限之后该用户就无法进入该目录了。

3. 特殊权限

    分为三种：

    - Set UID
        当`s`这个标识出现在文件所属用户的`x`的权限位上时，例如

```bash

-rwsr-xr-x. 1 root root 27832 Jun 10  2014 /usr/bin/passwd

```

此时就被称为Set UID，简称为SUID的特殊权限，它具有下列的特性

+ SUID权限只对二进制程序(binary program)有效
+ 执行者对于该程序需要有`x`权限
+ 本权限仅在执行程序的过程中有效(run-time)
+ 执行者将具有该程序所属用户的权限

看起来还是不好理解，那么举个例子吧。

比如我拿到一个新的机器就喜欢给它装上最喜欢的zsh，装完之后就要修改我的默认shell。明确两点：

1. 默认shell的配置文件是`/etc/passwd`文件，我作为普通用户是没有这个文件的读写权限的
2. 修改默认shell的命令是`/usr/bin/chsh`，这个文件的属性如下

```bash

-rws--x--x. 1 root root 23856 May 12 09:43 /usr/bin/chsh

```

这个文件是属于root用户的，但当我执行这个文件时，就(暂时)拥有了root用户的权限，就可以修改这个文件了。
        其实这段细节是从《鸟哥的Linux私房菜》里学到的，但书中说的是用户自己修改密码的例子，我想举个不同的例子来说明同一个问题，但显然我这个例子没有鸟哥的例子好。因为作为对比，鸟哥还说了`cat`命令没有这个`SUID`权限，所以


```bash

[frost@localhost test]$ cat /etc/shadow
cat: /etc/shadow: Permission denied

```

会得到这个结果，这个例子好就好在即使是root用户也是没有`/etc/shadow`这个文件的读写权限的，只能强制写入，但`/etc/passwd`这个文件的权限就比前者宽很多了。说到这个程度，估计读者应该也能明白其中的原理了。

另外需要注意的是，SUID仅仅可以用在二进制程序上，不能用在脚本上。因为脚本也只是调用别的二进制程序。同时SUID对目录也是无效的。

- Set GID

    当`s`标识出现在文件所属组的其他用户的`x`位置上，则称为Set GID，即SGID。与SUID不同，SGID可以设置文件和目录。

    SGID具有如下的功能：

    - SGID对二进制程序有效
    - 程序执行者对该程序来说具有`x`权限
    - 执行者在执行的过程中将暂时获得所属组其他用户的权限

- Sticky Bit

    Sticky Bit，即SBIT只针对目录有效，而对普通文件无效。其作用是：

    - 使用者对此目录有`wx`权限
    - 使用者在该目录下新建文件或目录时，仅有自己与root才有权利删除该文件


换句话说，如果一个用户是一个目录的所属用户组的其他用户或其他用户的身份，并且具有该目录的`w`权限，如前所述，他可以随意修改或删除目录中的任何文件——即使该文件是属于别人的。但如果对该目录加上SBIT权限，则该用户就只能修改或删除属于他自己的文件了。


特殊权限的设置与普通权限大同小异。同样是用数字代替权限


数字 | 权限
---|---
4 | SUID
2 | SGID
1 | SBIT

在设置时将特殊权限置于普通权限的前面，比如要给一个目录设置SBIT权限

```bash

[frost@localhost ~]$ sudo chmod 1775 test
[frost@localhost ~]$ ls -l
drwxrwxr-t. 4 frost frost   42 Jun 20 07:22 test

```


4. 隐藏属性

    隐藏属性对于系统的安全性上很重要。相关的两个命令式`chattr/lsattr`。但是之前的Ext2/Ext3/Ext4文件系统对于隐藏属性有完整的支持，但RHEL 7之后，默认的文件系统变成了xfs，它只对隐藏属性提供部分支持，不过对于正常的使用来说也已经足够了。

    不常用的就不说了，可以从鸟哥的书中去查，最常用的就数`i`属性了，其次是`a`属性。

    其中，`i = imutable`，`a = append`，那么这个意思就已经很明显了，前者表示文件是不可修改的，即使是root也不可以，而后者表示只可以对该文件进行添加操作，而不能进行别的修改或者删除。这两个属性只有root用户才可以设置。

    在哪里会用到呢？比如你不希望有用户随便添加具有sudo权限的用户，那就可以给`/etc/sudoers`文件添加`i`属性，这时当不明所以的具有sudo权限的其他用户想再添加一个用户时就会提示`permission denied`了。

    这几个属性就不像上面的那些用`chmod`进行操作了，而是`chattr`，并且它们也没有对应的数字，只需要用`+a`, `-a`, `+i`, `-i`就可以了。


正常点的属性说的差不多了，下面聊聊OS X下独有的属性。那个可恶的`@`符号。

那么这个属性是怎么来的呢？正常情况下它会是一个一个的键值对，也就是一个属性对应一个值，但我们也可以在finder里右键显示简介，然后再注释里面加上一些内容，这时这个文件就会有一个`@`属性了。

如果一个tar包在Mac里有扩展属性，那到了Linux里解压的时候就会出现问题，要解决这个问题，可以先复制一份不带扩展属性的文件，然后再到Linux中解压。

```bash

cp -XR xxx.tar.gz xxx.linux.tar.gz

```



