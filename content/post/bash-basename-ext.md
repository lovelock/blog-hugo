+++
author = "author"
date = "2016-01-09T21:38:27+08:00"
description = "用Shell脚本获取不包含后缀的文件名"
draft = false
keywords = ["shell"]
title = "Shell脚本获取文件名"
topics = ["Bash"]
type = "post"

+++

我的需求是把一些原来后缀是markdown的文本重命名为md，这个需求很low了，但其实也很有技巧，你可以一个一个的重命名，没有问题，但作为一个对Unix大道甚为叹服的人，还是应该寻求更好的方法。

简单搜索了一下，发现了[StackOverFlow](https://stackoverflow.com/questions/2664740/extract-file-basename-without-path-and-extension-in-bash/2664746#2664746)上的答案。

```bash
$ s=/the/path/foo.txt
$ echo ${s##*/}
foo.txt
$ s=${s##*/}
$ echo ${s%.txt}
foo
$ echo ${s%.*}
foo
```

好吧，只看到这里就行了，问题已经解决。不是我不想继续研究[Bash参考文档](http://www.gnu.org/software/bash/manual/bashref.html#Shell-Parameter-Expansion)，只是实在没有这个必要了，我还要研究Symfony3，研究建立起Symfony的Bundles，然后搞点Python满足平时的处理日志、甚至是Web编程的需要。。。


