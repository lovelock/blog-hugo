+++
author = "frostwong"
isCJKLanguage = true
date = "2016-01-25T12:14:35+08:00"
description = "Firefox下日期解析行为兼容方案"
draft = false
keywords = ["JavaScript", "Firefox", "Date.parse()"]
title = "Firefox Date.parse()方法行为不一致的兼容方案"
topics = ["javascript"]
type = "post"

+++

事情是这样的，我有两个输入框，用Wdatepicker自定义了日期格式，格式当然是我们中国人最喜欢的Y-M-d H:i:s，然后需要确定后者（结束时间）不能小于前者（开始时间）

```javascript
// Firefox incompatible
var start = $("#start").val();
var end = $("#end").val();
retrun Date.parse(end) - Date.parse(start) > 0 ? true : false;
```

Chrome下测试没问题，当时负责的同事就上线了。（因为这个是内部用的，只有两三个同事用，所以规定一下大家都用Chrome没有问题）

后来发现Firefox下不管两个日期怎么填，总会提示不对，跟了一下发现`Date.parse(end)`和`Date.parse(start)`里面的两个值都是NaN，但在Chrome里却是正常的Unix时间戳。

这就诡异了，查看了[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date/parse)发现了

> ### Parameters

> #### dateString
A string representing an RFC2822 or ISO 8601 date (other formats **may be** used, but results may be **unexpected**).


> ##ECMAScript 5 ISO-8601 format support

> The date time string may be in ISO 8601 format. For example, "2011-10-10" (just date) or "2011-10-10T14:48:00" (date and time) can be passed and parsed. The UTC time zone is used to interpret arguments in ISO 8601 format that do not contain time zone information (note that ECMAScript 2015 specifies that date time strings without a time zone are to be treated as local, not UTC).



从上面可以看到，Firefox是支持RFC2822 or ISO 8601格式的，但其他格式就呵呵了，或许可以支持，但可能会得到意想不到的结果。

所以，我们可以认为Chrome在标准之外支持了一些非标准的写法，而Firefox却没有。

知道了问题所在，就好解决了。

在比较之前把时间里面的空格替换成"T"即可。

```javascript
// Firefox compatible
var start = $("#start").val();
var end = $("#end").val();

start = start.replace(/ /, 'T');
end = start.replace(/ /, 'T');
retrun Date.parse(end) - Date.parse(start) > 0 ? true : false;
```

问题解决。



