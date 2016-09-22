+++
categories = ["Mac"]
title  = "Mac使用VMware Fusion安装Ubuntu"
isCJKLanguage = true
date = "2015-08-31T19:37:00"
topics = ["Mac"]
tags = ["Mac"]
type="post"
+++


越来越觉得我用Mac和Windows也没有什么区别。

用Windows的时候是因为没有类Unix环境，所以需要在虚拟机里面搭建一些Web环境，然后用SecureCRT或者XShell登录进行操作。当然主要是用VirtualBox比较多，因为它比较轻量，功能也没有什么缺陷。非要说缺陷，我觉得VMware的NAT功能比VirtualBox要好。前者是建立了一个192.168开头的虚拟网段，虚拟机使用这个网段的IP，虚拟机可以和宿主机相互通信，并且在类似学校的需要认证才能上网的环境下是可以直接上网的。但VirtualBox就不行了，它
的NAT模式默认用的是10.网段的地址，如果要用宿主机访问虚拟机，就需要端口转发，说起来容易，但谁也不想去操作，否则只能用桥接模式了，但在需要认证的时候就不行了，尤其是某些帐号只能允许一个客户端在线时。

抛却Windows的诸多不便，用上了Mac却不想让一堆随时可能修改的开发环境把Mac环境搞乱，编程语言的那些东西还好，但各种服务就不想在Mac上搭建了，所以虚拟机还是唯一的出路了。PD总刷存在感，在Launchpad里面加上一堆难看的图标，受不了。有时需要带图像界面的Ubuntu，这时VirtualBox的性能就成了问题，最终找到了Vmware Fusion了，这才是一款踏踏实实的产品，VMware的产品一直都那么让人信赖。

闲言少叙，其实我要说的是在高分屏下如何解决Ubuntu糊掉的问题。

之前用PD的时候也看到这个方法，但没有用，这下在VMware Fusion上面可以。

很简单，在"All Settings/Display"中找到"Scale for menu and title bars"，把这个bar拉到2就可以了。当然如果你觉得2太大了，也可以选择1.5或者其他。我觉得2是最合适的，也正是正常大小，因为Mac屏幕的分辨率宽和高分别是普通分辨率的2倍，调成2倍就和在普通屏幕上一样大小了，但不会糊掉。

废话好多。
