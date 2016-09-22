+++
categories = ["Mac"]
title  = "OS X完全卸载JDK的方法"
isCJKLanguage = true
date = "2015-12-04T23:04:43"
topics = ["Mac"]
tags = ["Mac"]
type="post"
+++


刚用OS X时想着可能会用到Java，毕竟Jetbrains的产品都要依赖Java。但新版本的这些IDE都自带java-bundle了，所以就没有必要留着了。

看了Java官网的指南，发现无法删干净，而在[StackOverFlow](https://stackoverflow.com/questions/19039752/removing-java-8-jdk-from-mac/23092014#23092014)上发现了这条，完美！

```bash
sudo rm -rf /Library/Java/JavaVirtualMachines/jdk<version>.jdk
sudo rm -rf /Library/PreferencePanes/JavaControlPanel.prefPane
sudo rm -rf /Library/Internet\ Plug-Ins/JavaAppletPlugin.plugin
sudo rm -rf /Library/LaunchAgents/com.oracle.java.Java-Updater.plist
sudo rm -rf /Library/PrivilegedHelperTools/com.oracle.java.JavaUpdateHelper
sudo rm -rf /Library/LaunchDaemons/com.oracle.java.JavaUpdateHelper.plist
sudo rm -rf /Library/Preferences/com.oracle.java.Helper-Tool.plist
```

记录一下。
