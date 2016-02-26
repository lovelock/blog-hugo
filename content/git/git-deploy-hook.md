+++
author = "frostwong@gmail.com"
date = "2016-02-21T15:37:20+08:00"
description = "Git自动部署钩子的使用"
draft = true
keywords = ["git", "hook"]
title = "使用Git钩子自动部署代码"
type = "post"

+++

前几天在[搬瓦工](https://bandwagonhost.com/)上买了个年费19.99刀的VPS，想着用来做一些研究，毕竟在公网上，做什么事情都更方便点。虽然机器的性能尚可，就是网络稍慢。不过好在发现速度慢时可以切换一下机房，数据也没有损失，一分钟即可搞定，这点简直是搬瓦工的杀手锏。

其实是前几天我去一家公司面试，面试官跟我提到他们之前用过Laravel，发现性能像屎一样，一般框架的性能影响可以忽略不计，性能消耗可能都在数据库IO上，但Laravel不一样，性能真的消耗在了框架上。。。。这就让我萌生了一个念头，自己测试一下到底我们敬爱的鸟哥写的Yaf，和我喜欢的Symfony及其精简版Silex还有号称为艺术家而生的框架Laravel及其精简版Lumen的性能。

环境部署我就不说了，直接上配置说明

![操作系统版本](http://7xn2pe.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-02-21%20%E4%B8%8B%E5%8D%883.49.05.png)

![PHP版本](http://7xn2pe.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-02-21%20%E4%B8%8B%E5%8D%883.47.48.png)

![PHP模块](http://7xn2pe.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-02-21%20%E4%B8%8B%E5%8D%883.48.04.png)

## Git自动部署

由于网络连接比较慢，在服务器上直接写代码是很浪费时间的，那我就想在本地写了代码，push到远端服务器之后，给git配上一个钩子，让它自己把最新的代码覆盖到Web目录。

对，就是钩子。

第一次对钩子的作用有概念还是Yaf的Plugin中的钩子，~~说白了就是回调函数~~。你可以指定在某个事件发生前或者发生后做某件事。

那根据需求，这个钩子应该在`/var/www/project`中执行`git checkout -f`命令。那总不能要cd到Web目录去执行吧，所以得先设置两个变量

    1. 要部署到的目录，在这里是`/var/www/project`
    2. git的remote地址

下面是具体的操作步骤。

1. 在服务器设置git server
    
    ```sh
    useradd -m -d /var/git git -s /usr/bin/git-shell
    cd /var/git
    mkdir project.git
    cd project.git
    git init --bare
    cd hooks
    echo 'git --work-tree=/var/www/project --git-dir=/var/git/project.git checkout -f' > post-receive
    chmod +x post-receive
    ```
    
2. clone 代码库到本地，开始工作

    ```sh
    git clone git@remoteserver:/var/git/project.git
    cd project.git
    touch a.test
    git add .
    git commit -m 'test git hook'
    git push origin master
    ```
    
3. 在服务器检查Web目录

    ```sh
    cd /var/www/project
    ls -l
    ```

这个过程需要注意**用户git需要有Web目录的写权限**。

基础搭好了，就可以继续了。

