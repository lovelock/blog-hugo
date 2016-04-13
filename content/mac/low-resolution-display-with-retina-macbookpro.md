+++
author = "frostwong"
isCJKLanguage = true
date = "2016-01-20T14:03:19+08:00"
description = "description"
draft = false
keywords = ["rMBP", "1080P"]
title = "rMBP连接1080P显示器显示效果优化"
type = "post"

+++

喜大普奔地，公司发了13寸MacBook Pro和Dell 24寸显示器。

然而，没有转接头，自己买。

然而，分辨率太渣，同时看两个显示器眼要瞎。

搜了一大通，发现了这个方法。

```
defaults -currentHost delete -globalDomain AppleFontSmoothing
defaults -currentHost write -globalDomain AppleFontSmoothing -int 1
defaults -currentHost read -globalDomain AppleFontSmoothing
```

其中值 `1` 可以根据自己的喜好调节，我这里调成1感觉效果比之前好了一点。如果不想要了就用第一条删除预设。

可以用iTerm做测试，执行之后退出应用，再打开，对比效果。


