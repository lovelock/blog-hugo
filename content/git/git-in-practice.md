+++
categories = ["Git"]
author = "frostwong"
date = "2016-05-13T22:30:18+08:00"
description = "实际工作中使用的git是什么样的"
draft = true
isCJKLanguage = true
keywords = ["git"]
tags = ["git"]
title = "多人协作场景下的git"
type = "post"

+++

之前虽然也写了点Git的使用方法，但都是在仅有我自己维护代码的情况下做的，只有master分支，而到了多人协作的场景下，就需要有新的知识补充了。

首先这个使用场景是这样的：

1. master分支就是它的原始功能，主干，所有人在分支上的代码在上线之前需要合并到master
2. 上线是把master再checkout一份
3. 每个人在自己的分支做开发

下面说下碰到的问题，以[learngit](https://coding.net/u/lovelock/p/learngit/git)这个项目为例。
    
刚接手一个新项目，clone一份吧
    
`git clone https://git.coding.net/lovelock/learngit.git`
    
这时项目目录是这样的
    
```bash
bash-4.3$ git branch
* master
bash-4.3$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working directory clean
```

我要开发的功能不能放在master里，所以需要新建一个分支，那可能会有人问为什么不能放在master里。我的理解是**要保持master分支的清洁**，这体现在每个人在自己分支的提交不会体现在master分支的git log结果里，这里只体现合并到master的信息，这样我们对每次合并做了什么就一目了然，不会出现多余的信息。当然另外一个重要的原因是防止代码冲突，其实解决冲突的还是人，这点就不多说了。

```bash
git checkout -b feature
bash-4.3$ git checkout -b feature
Switched to a new branch 'feature'
bash-4.3$ git branch
* feature
  master
bash-4.3$ git status
On branch feature
nothing to commit, working directory clean
```

从`git branch`的输出可以看到，我们现在已经位于feature分支里了。现在所进行的任何修改都将位于feature分支。

```bash
bash-4.3$ touch feature.txt
bash-4.3$ git status
On branch feature
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	feature.txt

nothing added to commit but untracked files present (use "git add" to track)
```

现在在新分支创建一个文件，再看status，现在这个文件没有被track，需要把它加进来。

好，那么问题来了，如果我不想把它加进来呢？我们先说前一种最常见的情况。其实提示信息已经教会我们要怎么做了。

```bash
bash-4.3$ git add feature.txt
bash-4.3$ git status
On branch feature
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   feature.txt
```

这里为了严谨，没有用我平时用的`git add .`（其实最主要的是切换到了bash，而没有用zsh提供的更方便的git操作)。现在这个新文件正在等待被提交。等等，括号里面的提示是什么东西？它告诉我怎样才能让git 不再track它？对，可以试一下。

```bash
bash-4.3$ git reset HEAD feature.txt
bash-4.3$ git status
On branch feature
Untracked files:
  (use "git add <file>..." to include in what will be committed)

	feature.txt

nothing added to commit but untracked files present (use "git add" to track)
```

这次的输出又和上上一次的一样了。好，既然已经回到了这个状态，下面看怎么让这个文件消失。这个需求我觉得主要用在要在一个项目中做一些编译工作，编译嘛，会产生很多多余的文件，安装完成后也不会消失，我想让这个项目重新回到干净的状态。

```bash
bash-4.3$ git clean -df
Removing feature.txt
bash-4.3$ git status
On branch feature
nothing to commit, working directory clean
```

现在工作目录又焕然一新了，如果并不想清除整个目录中的**unstaged**的文件，可以在这条命令的后面加上路径即可。

回到前面`git add feature.txt`后，执行

```bash
bash-4.3$ git status
On branch feature
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	new file:   feature.txt

bash-4.3$ git commit -m 'init commit'
[feature 4e31b7f] init commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 feature.txt
```

就是真正把这个文件提交到版本库了。这时用`git log`是可以查看到的。

```bash
commit 4e31b7fca7a95f5fb7a9c0ace9a94e02a9f18eba
Author: lovelock <frostwong@gmail.com>
Date:   Fri May 13 22:54:09 2016 +0800

    init commit
```




