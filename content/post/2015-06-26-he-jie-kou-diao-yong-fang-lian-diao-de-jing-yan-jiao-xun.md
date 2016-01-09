+++
title  = "和接口调用方联调的经验教训"
date = "2015-06-26T10:22:18+08:00"
+++



一个同事做的接口，今天他请假没有来，就临时由我协助接口调用方联调。

这是一个很简单的接口。但浪费了近一天的时间。

我是第一次看这部分代码，得出的结论是该接口接受两个参数，然后进行后面的处理。但是对方调用时总是出错，我不明白这原因，于是让对方把调用的参数发给我，结果发现他传入了三个参数，而且有一个必须的参数意思一样但变量名不同，并且可选值也不同。

当时我就懵逼了。

我说这调用的不对啊！！！结果对方上来给我一个文档截图，说他是按照这个文档调用的。

这时我又懵逼了，为什么不事先给我这个文档?

我手指在键盘上飞舞，然后我这边再用相同的参数调用接口，OK，没有任何问题。但是对方不停的告诉我不行、不行还是不行。

究竟哪里出了问题？

而且由于同事的懒于配置，这个开发环境连日志都没法记录。然后我就往里面一步一步的跟，终于跟到了调用平台接口的地方也看不到什么问题。终于我把那个调用的方法外套了一层`try catch`，并把`Exception`的内容打印出来，真相终于大白。

原来是我用的开发机上没有调用这个平台接口的权限，而我用的我自己的账户测试，不需要再调用这个接口，但对方的却不是这样，因此这个地方通不过。

这时已经中午了，吃了饭下午再搞。

下午我想着配到仿真环境应该没有问题了，就给对方发过去了一个IP，让他配上这个IP再来测试。于是我胸有成竹的让他调用接口，但他仍然不通过。

这时我又懵逼了！！！

怎么可能！？？我自己取消了那个权限之后仍让可以正常调用，问题出在哪里呢？中间我跟他发了好几次IP，生怕他没有配好hosts。结果我查了一下accesslog发现他的请求并没有进入我部署代码的这台机器，我就问他，你到底有没有按我给你的IP配置hosts啊？他这时也自信满满的给我发来了一张**上午让他配置的环境**的ping的截图。

我简直不知道说什么好了。

难道他以为我发给他一个IP是发着玩的吗？尼玛不知道干啥的你问啊，何况我都已经告诉你了让你配上这个。。。。

于是他重新配置了hosts，所有问题都解决了。

真是一个曲折的事件，一个本来五分钟就能搞定的东西，从上午十点搞到了下午三点多。

从这件事上可以总结出几点联调接口的经验：

1. 一定要先确认双方用的环境是不是同一个(hosts配置)。
    即使业务日志无法获取，但起码要能看到accesslog，看一下对方的请求有没有发送到目标机器上。
2. 如果确认完之后仍然有问题，那么就一步一步的往里面跟，如果调用平台接口有问题，首先想到的就是权限问题。因为有些敏感操作并不是每台开发机都有权限的。如果因为这个浪费大量的时间，就不值得了。
3. 谨防破窗效应。这里是说原来的代码或者开发环境已经是千疮百孔了，调试等非常不方便，如果任由它这样发展下去，那它只能进一步的变坏。虽然没有大块的时间去整理它，但要慢慢地优化它，最终整个团队一定会发现这种变化，如果大家能一起践行，那代码的质量一定会越来越好。