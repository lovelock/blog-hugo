+++
date = "2017-02-28T14:59:40+08:00"
title = "Slim 获取Postman发送的PUT请求的body"
categories = ["http"]
tags = ["slim","php", "postman"]
isCJKLanguage = true

+++

刚才用Postman测试一个接口发现用**form-data**格式发送PUT请求给Slim应用时，在应用中用`$request->getParsedBody()`获取不到请求的body，搜索到了[http://stackoverflow.com/questions/23761425/get-put-params-with-slim-php](StackOverFlow页面)，  简单说就是一般`form-data`格式用来上传文件，而一般情况下应该用`x-www-form-urlencoded`，改一下这里的发送body的格式就可以了。
