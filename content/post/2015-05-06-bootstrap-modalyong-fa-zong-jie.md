+++
title  = "Bootstrap Modal用法总结"
date = "2015-05-06T23:15:41+08:00"
+++

这段时间前端后端同时写，对jQuery和Bootstrap用的比较多，说实话我对Modal很感兴趣，单纯的就是因为它好看，并且看起来友好，不像alert那么暴力，FF的实现还好，而Chrome的alert直接忽视所有其他操作，只能选择不再弹出，但只要选择了这个不再弹出，在重启这个页面之前（刷新没用）就不会再弹出了，真不明白这是什么逻辑。当然最重要的一点是Modal可以交互。

下面简单说说Modal的用法，当然基本上来自[官方文档](http://getbootstrap.com/javascript/)。

## 依赖
1. jQuery
2. Bootstrap

没有对它们进行安全版本测试，就现在而言，使用最新的版本总是没有问题的。

## 基本组成
1. `.modal` Modal的主体div
	-  通常我们会给它指定一个id，因为你总是会希望能更细粒度的控制一个modal的行为吧，况且文档中也说不可以使用叠加的modal，也就是说，同一时间一个页面上只能出现一个modal，如果直接使用`.modal` 来控制它的显示和隐藏，早晚会出现问题的
	- 还有一个属性是`role` ，这个属性很有意思，其实没有什么特别的用途，只是在阅读代码时让你知道它的作用是一个对话框，为什么需要这个呢？因为`<div>` 这个标签本身不能表达这个意思，比如`<select>` 标签，你一看就知道它一定是一个下拉选择器，但`<div>` 呢？所以，这个`role` 就是用来说明标签本身无法说明的一些内容，比较常见的还有在一个`.nav .nav-tabs` 中的`<li>` 标签中，会添加`role="option"` 这样的说明，就是告诉开发者这是一个功能的多个选项之一
	- 通常我们会喜欢加上一些动画，最常用的是`.fade` ，当然如果你更喜欢直接，也可以忽视它
	- `tabIndex="-1"` [关于这个属性的解释](http://jehiah.cz/a/tabindex)
2. `.modal-dialog` 关于这个没有找到更多的解释，我想大概是因为弹出的是一个对话框吧，没有见到更多的类型
	- 同样这个class也可以有叠加选项，`.modal-lg/.modal-md/.modal-sm` 分别表示大中小三种型号
3. `.modal-content` 当然就是modal的内容了
4. `.modal-header` 内容的第一层，头部
	- 可以在其中放置一个关闭按钮，比如右上角的一个`x` 等，为了诸如屏幕阅读器之类的用户辅助工具更好的工作，可以在这个关闭按钮上加上`aria-labelledby` 属性，值是`.modal-title` 的`id` ，当然也可以手动用`aria-label="myModalTitle"` 来指定，但动态的写法更方便，不是吗？
5. `.modal-title` 放置在`.modal-header` 中，modal的标题
6. `.modal-body`  modal的正文部分，需要显示的主要内容就在这个部分了
7. `.modal-footer` 通常用来放置确认/关闭等按钮
	- 当然我们需要一个关闭modal的按钮，这个按钮需要添加一个`data-dismiss="modal"` 的属性才能正确的完成功能，和上面那个`x` 按钮的功能是一样的

<iframe width="100%" height="300" src="//jsfiddle.net/frostwong/ve96c6fr/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

经过上面这几步，一个完整的modal对话框就完成了，但问题来了，怎么触发它呢？

下面来说一下modal的触发方式。

1. 按钮
	这个应该是最常用的方式。需要注意的有几点
	- `data-toggle="modal"` 指明这个按钮要触发的widget类型是modal，常见的还有
		```html
		data-toggle="modal"
		data-toggle="collapse"
		data-toggle="dropdown"
		data-toggle="tab"
	   ```
	- `data-target="#myModal"` 指明这个按钮要触发的modal的id是`myModal`
	有了这两个属性点击这个按钮时就可以触发这个modal了。

2. JavaScript方法
`$('#myModal').modal(options)`
关于`options`，有以下几个选项
	- backdrop，值可选布尔值(`true/false`)或`'static'` ，默认值是`true`，当设置为`false` 或`'static'` 时点击modal外的区域modal不消失，反之会消失
	- keyboard，点击`Escape` 键时modal是否消失，默认是`true`，可选`false`
	- show，初始化时modal是否显示，默认`false` 不显示
```javascript
$('#myModal').modal({
	backdrop: 'static',
	keyboard: false,
	show: true
});
```

## Modal方法
1. `$('#myModal').modal(options)` 上面已经说过了
2. `$('#myModal').modal('toggle')` 手动显示或隐藏modal
3. `$('#myModal').modal('show')` 手动显示modal
4. `$('#myModal').modal('hide')` 手动隐藏modal
5. `$('#myModal').modal('handleUpdate')` 在modal处于显示状态时，如果它的高度有变化，就需要执行这个方法（也仅有这个情况下需要执行这个方法），它会在右边产生一个滚动条，并且把内容向左跳一下。

<iframe width="100%" height="300" src="//jsfiddle.net/frostwong/ya6956pp/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

## Modal事件
1. `show.bs.modal` 当`show` 方法执行时，立即触发该事件
2. `shown.bs.modal` 当`show` 方法执行时，会等待CSS效果完成之后再触发该事件
3. `hide.bs.modal` 同1，`hide` 方法
4. `hidden.bs.modal` 同2， `hide` 方法

> 关于Modal还有使用Bootstrap网格系统的部分，但这个部分至少对目前来说用处不多，暂不提。
> `options` 中还包括一个`remote` 选项，因为已经被废弃，将来（4.0）版本中会移除，故对该主体和它相关的`loaded.bs.modal` 都未涉及。

