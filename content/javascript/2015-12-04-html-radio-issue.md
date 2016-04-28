+++
title  = "HTML中的input[type=radio]的一个小发现"
isCJKLanguage = true
date = "2015-12-04T23:12:52"
topics = ["javascript"]
tags = ["HTML", "tags"]
keywords = ["radio"]
+++

昨天在写一个表单时，一个无意的错误让我发现了HTML radio类型的input的一个特性。

在一个表单`<form></form>` 中，相同name的input[type=radio]只有最后一个才会正常的渲染checked。

也就是说，如果一个表格中有很多条目，这时每一列都会有相同的name，这时你会发现及时每个条目的input[type=radio]都有正确的checked属性，但并不能看到它被选中——只有最后一行正常。

起初我还以为这是个bug，仔细一想，其实这是更合理的方式。

因为，在一个表单中，每个不同的name就是一个参数，如果同一个name的参数出现了多次，服务端也会以最后一个为准。如此看来，这样的实现反倒会让开发者在参数输入就发现问题。
