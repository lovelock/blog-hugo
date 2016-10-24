+++
isCJKLanguage = true
tags = [
  "VitualBox",
  "Linux",
]
categories = [
  "Virtualization",
]
date = "2016-10-13T16:35:22+08:00"
title = "使用VBoxManage创建虚拟机"

+++

## 前言

最近没有产品需求，就深入的研究一下大数据吧，第一步先要搭建一个集群，前面已经写了一篇关于搭建“伪集群”的文章，还是希望更完整的理解这套东西，还是弄一套真正的集群吧。但是没有机器，就只能拿本地的台式机搞起来了。

本文主要介绍了如何使用VirtualBox命令行工具VBoxManage创建和维护虚拟机。官方文档中说到VBoxManage的功能是比GUI的VirtualBox要更完整的，但其实我也用不到那么完整的功能，我能想到的主要有以下几点，参照官方文档来逐个完成。

1. 创建一个虚拟机
2. 给虚拟机配置网络、CPU核心、内存、磁盘驱动器
3. 复制（clone）虚拟机

我使用的环境如下：
![](http://7xn2pe.com1.z0.glb.clouddn.com/machine.png)

##
> 看到这张图片，我想说一句关于字体的，Windows下绝对是Consolas最耐看；Linux下SourceCodePro最好看；Mac下命令行用Monaco，但IDE里面用Monaco最觉得不能认真写代码了，太花了，还是SourceCodePro比较正常一点，呼呼

下面开搞吧。

**一定要确认你的VirtualBox安装了Extension Pack， 如果没有马上根据你的发行版或者去官网下载之后安装，否则无法远程连接虚拟机。**

## 创建虚拟机

### 知识

#### 1. `VBoxManage createmedium`

首先要创建一块磁盘。

1. `--filename <name>` 创建的设备的名字
2. `--format VDI|VMDK|VHD` 创建的设备的格式，默认是vdi，当年我做云主机运维的时候还测试过各种虚拟化磁盘格式的性能，vdi的性能是所有可选项里面最快的，值得信赖
3. `--size <megabytes>` 创建的磁盘的大小，以M为单位

注意这个命令创建的磁盘是位于你当前所在的目录的，所以为了避免后面的问题，你最好在你想放在的位置执行这个命令。

#### 2. `VBoxManage createvm`

然后创建一个虚拟机。

1. `--name <name>` 指定虚拟机的名字，还会在`~/.config/VirtualBox/Machines`目录下创建同名的xml文件，如果该虚拟机被重命名，该xml文件也会被自动重命名。
2. `--basefolder <path>` 指定上述的Machines目录，如果指定了这个目录，新创建时还是会在这个目录下产生xml文件，但当虚拟机被重命名时，该xml不会被重命名。所以这里我觉得还是不要改为好，虽然通常也不会去重命名虚拟机。
3. `--groups <group>` 指定虚拟机组，总是从`/`开始，可以嵌套，默认是`/`
4. `--ostype <ostype>` 指定虚拟机的操作系统类型，具体支持的操作系统类型可以使用`VBoxManage list ostypes`来查看。
5. `--uuid <uuid>` 指定虚拟机的UUID，这个id在宿主机的命名空间内必须是唯一的，如果指定了虚拟机组，则在组内必须是唯一的，如果不指定，会自动生成，所以这个其实也每必要指定。

默认情况下，这个命令只会创建一个xml文件，而并不会把虚拟机注册到系统中，可以使用`--register`选项或者单独执行`VBoxManage register <uuid>`来执行注册。

#### 3. `VBoxManage storagectl`

1. `<uuid|vmname>` 指定要操作的虚拟机，可以使用前面创建时指定的名字，或者自动生成的uuid
2. `--name <name>` 要创建的控制器的名字
3. `--controller` 控制器，这个我也不太懂，一般电脑上是ACHI，这里就选IntelAHCI吧
4. `--add` 添加的控制器类型，因为我们要创建的是磁盘驱动器，所以选择sata

> 这里其实是瞎说的，我也不太懂电脑硬件，这些概念不了解，就照着熟悉的来吧。需要创建两种类型的设备控制器，一个是磁盘控制器，用来管理硬盘，一个是光驱，用来管理ISO文件。这个很容易理解，这一步是创建控制器，而而这控制的东西，磁盘是前面创建的，iso是先前下载好的。

#### 4. `VBoxManage storageattach`

1. `<uuid|vmname>` 指定要操作的虚拟机，可以使用前面创建时指定的名字，或者自动生成的uuid
2. `--storagectl <name>` 这就是上面那个`storagectl`命令时的`--name`选项指定的参数了
3. `--port` 端口号
4. `--device` 设备号
5. `--type` 设备类型
6. `--medium` 指定创建磁盘文件，即vdi文件


把创建的磁盘驱动器和虚拟机、磁盘连接起来。

#### 5. `VBoxManage list hdds`

这时就可以查看注册过的磁盘了。


### 操作

```bash
// 创建磁盘
$ VBoxManage createmedium disk --filename CentOS7.vdi --size 50000
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Medium created. UUID: 1cc3b870-7180-4eea-8263-f82a783d1478

// 创建虚拟机配置
$ cd ~/cluster
$ VBoxManage createvm --name CentOS7 --ostype RedHat_64 --register
Virtual machine 'CentOS7' is created and registered.
UUID: 4afd6d6d-9cee-4efe-89a6-b752644711f0
Settings file: '/home/hadoop/VirtualBox VMs/CentOS7/CentOS7.vbox'

// 创建磁盘控制器
$ VBoxManage storagectl CentOS7 --add sata --controller IntelAHCI --name "SATA Controller"

// 绑定磁盘控制器
$ VBoxManage storageattach CentOS7 --storagectl "SATA Controller" --port 0 --device 0 --type hdd --medium CentOS7.vdi

// 创建光盘驱动器
$ VBoxManage storagectl CentOS7 --name "IDE Controller" --add ide

// 绑定光盘控制器
$ VBoxManage storageattach CentOS7 --storagectl "IDE Controller" --port 0 --device 0 --type dvddrive --medium ~/Downloads/CentOS-7-x86_64-Minimal-1511.iso

// 设置网络连接方式为桥接
$ VBoxManage modifyvm CentOS7 --nic1 bridged --bridgeadapter1 eno1 --vrde on --vrdeaddress 0.0.0.0 --vrdeport 5010 --memory 1024 --cpus 1

$ VBoxManage startvm CentOS7 --type=headless

```

> 注意：在我的操作系统下，执行createvm会在~/VirtualBox VMs/目录下生成`CentOS7.vbox`文件，其实就是一个xml文件。而所谓的注册操作，就是在`~/.config/VirtualBox`目录下生成一个VirtualBox.xml文件，里面有注册过的虚拟机的信息，类似下图所示：

![](http://7xn2pe.com1.z0.glb.clouddn.com/virtualbox.png)


## 连接虚拟机

> 这里我又要牢骚两句，网上有些人啊，觉得用命令行就是为了装13，根本不从问题的出发点去考虑。我为什么要用VirtualBox的命令行来安装虚拟机？图形界面不是更简单么？那是因为我的工作站没有图形界面啊！有些人上来就说连接你新创建的虚拟机要用`rdesktop -N localhost:3389`，简直是bullshit，我要是用带图形环境的工作站，就根本就不用费那么大力气搞这个了。

言归正传，现在有了两种选择，Windows可以用自带的“远程桌面连接”应用进行连接，需要注意的是，以上面的命令为例，在填写主机时就需要写"192.168.159.3:5010"（其中IP是我用的工作站的IP，具体根据你的实际情况写），如果是Linux桌面就用`rdesktop -N 192.168.159.3:5010`。至于Mac我好像也没有找到可以用的。

## 复制虚拟机

毕竟资源不是无限的，咱们创建虚拟机建集群也不能太浪费资源。我的理解是，如果要赋值虚拟机，最好用"link"形式，也就是说，复制虚拟机的快照，系统通过两个虚拟机的diff来区分二者。具体到VirtualBox的操作是这样的

1. 为模板虚拟机创建一个snapshot
2. 复制snapshot并命名

这两步就可以创建一个以前面的虚拟机为基础的虚拟机，而系统又不会占用两套存储空间，也步要求它有多高的性能，只要能说名问题即可。

先查看一下当前有哪些虚拟机（注意：snapshot是另外一种实体，查看vm的命令是查看不到snapshot的）
![](http://ww1.sinaimg.cn/large/65e4f1e6gw1f8v1dqo36oj20yw05etat.jpg)
看一下指定的虚拟机是否已经有snapshot(这里只是不想给已经有snapshot的虚拟机再创建新的，其实是没有问题的)
![](http://ww1.sinaimg.cn/large/65e4f1e6gw1f8v1ffm5tcj216002omy7.jpg)
创建新的snapshot并查看是否创建成功
![](http://ww1.sinaimg.cn/large/65e4f1e6gw1f8v1ig2xsuj21kw07en04.jpg)

`VBoxManage clonevm Debian-original --options link --name Debian-cluster-01 --register`

这样就克隆了一个新的虚拟机，并且注册到VirtualBox中，下面启动新的虚拟机的步骤就和前面直接创建新虚拟机一样了。 

