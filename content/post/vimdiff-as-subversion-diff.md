+++
author = "frostwong"
date = "2016-03-07T12:06:45+08:00"
description = "描述怎样自定义svn-diff"
draft = false
keywords = ["svn", "vim", "diff"]
tags = ["vim", "svn"]
title = "用vimdiff作为svn的diff工具"
type = "post"

+++

前后两家公司都是用Subversion做版本控制，虽说程序员自己有喜好，但仍然需要遵守公司的规定。我个人是比较偏好直接登录服务器进行开发的，所以得配置好各种好用的命令行工具。比如svn提交之前需要diff一下所做的修改，默认的diff工具简直没用什么卵用。屏幕那么短，上下显示不直观。

发现gist.github.com一直上不去，干脆就把它集成到我自己维护的[hackvim](https://github.com/lovelock/hackvim)里了，这里先加一个svn的配置，稍后回家把git的也加上。

用法：

1. 先切换到你想要放hackvim的目录，我喜欢~/Projects
2. git clone https://github.com/lovelock/hackvim
3. 如果你想用我的vim发行版，就看下项目的说明，否则直接拿出来utils/svnvimdiff.sh，放在你希望的位置
4. 进入~/.subversion
5. 打开config文件，修改`diff-cmd = /path/to/svnvimdiff.sh`
6. done

注意vimdiff.sh文件必须是可执行的。我在提交之前已经加了755，如果出错，请自行检查一下。




