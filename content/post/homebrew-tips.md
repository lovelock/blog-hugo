+++
author = "frostwong"
isCJKLanguage = true
date = "2016-02-17T22:58:19+08:00"
description = "Homebrew的一个小坑"
draft = true
keywords = ["homebrew", "curl"]
title = "homebrew 和 curl 的一个坑"
type = "post"

+++

用Homebrew的时间也不短了，老是碰到各种各样的Curl报错信息，猜想可能是由于系统的Curl版本低造成的，因为这些错误主要和SSL有关。但是由于Curl在OS X是个比较关键的组件，我总是不敢用『家酿』的去替换系统的。今天终于忍不住了，执行`brew link curl --force`。然后`brew update && brew upgrade`。

世界终于清静了，从来没有觉得那一行行#组成的进度条那么的可爱。

所以我总结了一个道理，当你用homebrew安装curl时，它会提示你curl是个系统组件，在homebrew里，curl是个keg-only的包，也就是不会自动link到PATH中去，而且如果你要手动link，还得指定`--force`选项。这其实更多是一个免责声明，告诉你尽量不要这么做——当然我还是有信心让你不出问题的呦。作者真鸡贼。
