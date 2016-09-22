+++
categories = ["Javascript"]
author = "frostwong"
isCJKLanguage = true
date = "2016-02-28T21:40:04+08:00"
description = "教你如何使用Bootstrap插件DataTables"
draft = false
keywords = ["Bootstrap", "DataTables"]
tags = ["Bootstrap", "DataTables"]
title = "Bootstrap插件DataTables实现服务器端分页"
topics = ["javascript"]
type = "post"

+++

我们知道，PHP和MySQL实现分页的基本思路是limit，页面上有个page(第几页）和count(每次取出几个），PHP把它传递给后端MySQL，查询时在条件查询的基础上加上`limit {($page - 1) * $count, $count}`。

但是，自己实现PHP分页也是有不少繁琐的问题需要解决的，比如分页要最多显示几个，两头如何处理，样式设计等等，既然有了Bootstrap这个大框架，各种解决方案就层出不穷了。

废话不多说，今天要讲一下用Bootstrap的插件DataTables做一个带分页的表格。

DataTables官方文档给的例子是一次把数据取出，然后在前端做分页处理。这样的好处是减少了每次查询都要请求网络，坏处则是如果数据太多，前端肯定就扛不住了。文档也说了，如果数据量在10000以内，可以在前端做分页，100000以上就在服务端做，中间的量就衡量自己的数据和服务器的处理能力做决定了。

本文的基础是PHP内置Web Server和Bootstrap的模板SB-Admin2。

## 前端处理分页

最直观的方法就是把所有的数据自己组织好，用foreach组织`<tr><td>`，再加上一个

```javascript
$(document).ready(function () {
    $('#datatable-example').DataTable({
        responsive: true
    });
});
```

这样，插件就会帮你把你想要的效果展示出来了，再加上实时搜索，效果简直不能更赞。

但这在实际应用中用的应该还是比较少，毕竟稍微上点规模的公司需要处理的内容就不止10000那么少了。

## 后端处理分页

先要理解几个概念。

1. draw 请求的当前次数
2. start 返回结果的开始位置
3. length 返回结果的长度

我这里为了简单起见，把数组当做数据源了，根据传入的参数不同，返回数组的不同部分。

一般来说，返回的数据里需要这些字段

```json
{
  "draw": 1,
  "recordsTotal": 57,
  "recordsFiltered": 57,
  "data": [
    [
      "Airi",
      "Satou",
      "Accountant",
      "Tokyo",
      "28th Nov 08",
      "$162,700"
    ],
    ...
}
```

在页面上需要有

```html
<table class="table table-striped table-bordered table-hover" id="datatable-example">
    <thead>
        <tr>
            <th>Column1</th>
            <th>Column2</th>
            <th>Column3</th>
            <th>Column4</th>
            <th>Column5</th>
            <th>Column6</th>
        <tr>
    </thead>
</table>
```

然后把

```javascript
$(document).ready(function () {
    $('#datatable-example').DataTable({
        responsive: true
    });
});
```

替换成

```javascript
$(document).ready(function () {
    $('#datatable-example').DataTable({
        responsive: true,
        serverSide: true,
        processing: true,
        ajax: "ajax.php"
    });
});
```

其中，serverSide选项表示使用服务端分页，processing表示加载表格的时候会在表格上覆盖一个『加载中』的类似弹层，而ajax就是数据的来源了。具体代码见。。

那你可能要说了，我的数据返回的可是正宗的HashTable，我指明是HashTable是因为PHP中数组也是HashTable，指的是Python里的字典。该怎么办呢？

相应的，如果返回的每列数据是这样的(其实这才是从数据库中取出来的样子)

```json
{
      "first_name": "Jennifer",
      "last_name": "Chang",
      "position": "Regional Director",
      "office": "Singapore",
      "start_date": "14th Nov 10",
      "salary": "$357,650"
}
```

那只需要把

```javascript
$(document).ready(function () {
    $('#datatable-example').DataTable({
        responsive: true,
        serverSide: true,
        processing: true,
        ajax: "ajax.php"
    });
});
```

替换成

```javascript
$(document).ready(function () {
    $('#datatable-example').DataTable({
        responsive: true,
        serverSide: true,
        processing: true,
        ajax: "ajax.php",
        columns: [
            { "data": "first_name" },
            { "data": "last_name" },
            { "data": "position" },
            { "data": "office" },
            { "data": "start_date" },
            { "data": "salary" }
        ]
    });
});
```

就可以了。

这是数据库的最简单的情况，那如果我用了ORM，取出来的就是对象了，这就免不了『嵌套对象』了。不管是Python的数组（PHP中key是数组的）还是字典（PHP中key是字符串的），都是一样的处理方式，都是最直观的表示法，把

```javascript
$(document).ready(function () {
    $('#datatable-example').DataTable({
        responsive: true,
        serverSide: true,
        processing: true,
        ajax: "ajax.php",
        columns: [
            { "data": "first_name" },
            { "data": "last_name" },
            { "data": "position" },
            { "data": "office" },
            { "data": "start_date" },
            { "data": "salary" }
        ]
    });
});
```

替换成

```javascript
$(document).ready(function () {
    $('#datatable-example').DataTable({
        responsive: true,
        serverSide: true,
        processing: true,
        ajax: "ajax.php",
        columns: [
            { "data": "first_name" },
            { "data": "last_name.other" },
            { "data": "position.2" },
            { "data": "office" },
            { "data": "start_date" },
            { "data": "salary" }
        ]
    });
});
```

即可。

