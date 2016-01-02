+++
title  = "Python日志处理记"
date = "2015-06-06T10:31:09+08:00"
+++

事情是这样的，今天领导给了一个需求，处理日志。

1. 给了我一个文件，是一个文件列表ids.txt
2. 让我在当天所有的日志里找到这些文件所在的行，从而查找失败原因

首先我就想到了用Python 作匹配，当然由于刚来到公司，自己的工作环境配置的不顺手，期间又装了个tar, Python，下面说说我的思路，很朴素，但很管用。

1. 读取ids.txt中的所有行
2. 读取所有日志然后作匹配

于是写了下面的代码


```Python
#!/usr/bin/env python2

import glob
import re


lines = open('ids.txt', 'r')
logfiles = glob.glob('logs/*.log')

result = open('result.log', 'r+')

for line in lines:
    # 去除换行符
    pattern = re.compile(line)
    for logfile in logfiles:
        logfilehd = open(logfile, 'r')
        for logline in logfilehd.readlines():
            if pattern.search(logline):
                # 防止覆盖
                result.read()
                result.write(logline)
    # 为每个不同的id加一个分隔线，方便查看
    result.read()
    result.write(100 * '-')
```
看起来没有什么问题，但执行的时候无论如何都得不到我想要的结果。我注意到如果把在第一个`for in`里面的`line`打印出来，每行后面都有一个空行，所以想到是这个空行引起的，匹配的时候把空行也算在内了，而日志里面很明显是匹配不到这个空行的。

于是在第一个`for in`下面加上了一行
```
line = line.strip('\n')
```
去掉了换行符，于是一切OK。
