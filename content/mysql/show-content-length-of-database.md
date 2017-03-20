+++
date = "2017-03-20T10:53:05+08:00"
title = "查看MySQL数据库大小"
tags = ["mysql","小技巧"]
isCJKLanguage = true
categories = ["MySQL"]

+++

这两天要从另外一个部门迁移一部分数据过来，在考察迁移方案的时候注意到不知道对方的数据量有多少，这里查了一下资料，记录一下。

1. 首先进入`information_schema`库，这里存放了其他库的基本信息

   `use information_schema; `

2. 查看所有数据库的**总大小**

   `select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables;`

3. 查看指定数据库的大小

   `select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables where table_schema='home';`

4. 查看指定数据库中指定表的大小

   `select concat(round(sum(data_length/1024/1024),2),'MB') as data from tables where table_schema='home' and table_name='members';`

知道了数据库大小了，才好选择相应的数据库迁移方案。

参考: [用SQL命令查看Mysql数据库大小](http://www.frostsky.com/2011/08/mysql-query-size/)

