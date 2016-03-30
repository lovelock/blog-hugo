+++
author = "frostwong@gmail.com"
date = "2016-03-30T16:08:03+08:00"
description = "检测浏览器是否安装了Flash"
draft = false
title = "检测浏览器是否安装了Flash"
tags = ["Flash", "JavaScript"]
type = "post"

+++

现在国内竟然那么多的视频网站都还不支持HTML5的视频播放，无外乎几个原因：

现在我们的浏览器也只是对Safari开启了默认HTML5，而其他浏览器都是**鼓励**用户使用Flash的，但我觉得这样不好。

下面的代码可以用来检测用户是否装了Flash。其实我不需要知道Flash的版本，因为任何的版本都应该是支持视频播放的。如果有些其他功能需要高版本的Flash，那也是用户安装了之后的事情了，API受限的或者有安全漏洞的话，浏览器会提示用户更新。

```javascript

function detectFlash() {
    if (navigator.mimeTypes.length > 0) {
        var flashAct = navigator.mimeTypes["application/x-shockwave-flash"];
        return flashAct != null ? flashAct.enabledPlugin != null : false;
    } else if (self.ActiveXObject) {
        try {
            new ActiveXObject('ShockwaveFlash.ShockwaveFlash');
            return true;
        } catch (oError) {
            return false;
        }
    }
}

```
