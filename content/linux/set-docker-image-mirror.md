+++
title = "为Docker配置国内加速镜像"
categories = ["Linux"]
tags = ["docker"]
isCJKLanguage = true
date = "2017-02-09T10:31:45+08:00"

+++

今天想着复习一下Nginx的反向代理，又懒得再配置多台机器了，正好手里有已经安装了Docker的台式机，于是就想用Nginx的Docker image来做实验了。

由于众所周知的原因，我们在圈内访问[docker官方镜像](https://hub.docker.com/)是很慢的，好在包括[DaoCloud](https://www.daocloud.io/)和[阿里云](https://dev.aliyun.com/search.html)为我们提供了而苏稳定的镜像服务。废话不多说，下面总结了一下使用国内镜像加速Docker镜像下载的方式。

## 获取专属的加速地址

上面提到的两个镜像服务提供者都是需要注册才能用的，注册后你会拿到一个形如`http://78a2f85b.m.daocloud.io`的地址（阿里的我没有尝试，应该是类似的），这就是专属你的加速地址了。说是专属，但其实并没有身份认证，任何人都是可以直接用的，除非你要把自己创建的镜像提交到DaoCloud才会做身份认证。

## 本地配置

DaoCloud是搞了一个配置脚本来为我们自动搞定这个工作的，怎奈不知道为什么这个脚本一直没有更新，对于用systemd的发行版来说已经不适用了。对于使用systemd的发行版，有以下两种方法实现加速：

### 1. 修改service文件

安装docker后，会在`/lib/systemd/system`目录下生成一个`docker.service`的文件，对于Ubuntu 16.04内容如下：

```nginx
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network.target docker.socket
Requires=docker.socket

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd://
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process

[Install]
WantedBy=multi-user.target
```

找到其中的`ExecStart=/usr/bin/dockerd -H fd://`行，改成`ExecStart=/usr/bin/dockerd -H fd:// --registry-mirror=http://78a2f85b.m.daocloud.io`。

因为你手动修改了service文件，当然要执行以下命令了:

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

如果还想查看一下docker的运行状态，就再执行一下`sudo systemctl status docker`即可。

### 2. 修改daemon.json文件

默认是没有这个文件的，可以自行创建：

```bash
sudo vim /etc/docker/daemon.json
```

文件内容如下：

```json
{
    "registry-mirrors": [
            "http://78a2f85b.m.daocloud.io"
        ],
    "insecure-registries": []
}
```

然后执行`sudo systemctl restart docker`。



## 总结

执行上面两种方法中的一种，即可享受国内良心厂商带给我们的加速服务了。注意我在本文中没有区分一个说法：镜像。有的地方镜像指的是Docker 的Image，有的地方是指加速服务的mirrors，请读者自行区分。