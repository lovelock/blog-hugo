+++
categories = ["Http"]
title  = "HTTP权威指南学习笔记[1]"
isCJKLanguage = true
draft=true
date = "2015-05-01T22:33:03+08:00"
tags = ["http"]
+++

1. Web服务器是Web资源的宿主。Web资源是Web内容的源头，最简单的Web资源就是Web服务器文件系统中的静态资源，但也可以是根据需要生成内容的软件程序。

2. 媒体类型，当Web浏览器从服务器中取回一个对象时，就会查看相关的MIME(Multipurpose Internet Mail Extension)类型，看自己是否直到如何处理这个对象。MIME类型是一种文本标记，表示一种主要的对象类型和一个特定的子类型，中间用一道斜杠来分隔。如'text/html','text/plain', 'image/jpeg', 'image/gif', 'video/quicktime', 'application/vnd.ms-powerpoint'等。

3. 每个Web服务器上的资源都有一个名字，这样客户端就可以说明它们感兴趣的资源是什么了。服务器资源被称为"统一资源标识符"（Uniform Resource Identifier, URI)。如[http://www.joes-hardware.com/specials/saw-blade.gif](http://www.joes-hardware.com/specials/saw-blade.gif)就是一个URI。而我们还会经常看到URL，或者说更经常见到URL，那它们两个是什么关系呢？首先，URL是URI的一个子集，因为URI还包括URN，而后者现在几乎没有用到，或者说它本身是一个很好的想法，但目前由于基础架构的原因还没有办法真正的实施，因此事实上我
们能够见到的有意义的URI实际上都是URL。看到这里我就很纳闷了，那些经常写文章里面总爱说URI让我这种只直到URL的人弄不明白的人是为什么呢？你自己到底清不清楚它的含义啊，如果说你要说明的一个资源定位符，那它肯定就是个普通的URL，何必让人觉得高大上非要说是URI呢？更何况说URI其实更不准确呢？

4. HTTP支持集中不同的请求命令。这些命令被称为HTTP方法，每个HTTP请求报文都会包含一个方法。最常见的5种方法是
    | GET | 从服务器向客户端发送命名资源 |
    | PUT | 将来自客户端的数据存储到服务端一个命名的资源去 |
    | DELETE | 从服务器删除命名资源 |
    | POST | 将客户端数据发送到一个服务器网关应用程序 |
    | HEAD | 仅发送命名资源响应中的HTTP首部 |

5. 状态码这个经常用，忘不了的就不在这里再写了。

6. 报文，可以用TELNET客户端去模拟HTTP请求的过程。具体过程如下图:
{% img /images/http.png %}

7. HTTP是应用层协议，HTTP无需操心网络通信的具体细节，它把联网的细节都交给了通用、可靠的因特网传输协议TCP/IP。
TCP提供了无差错的数据传输、按序传输、未分段的数据流。

8. Web的结构组件
    - 代理： 位于客户端和服务器之间的HTTP中间实体。
    - 缓存： HTTP的仓库，使常用的页面的副本可以保存在离客户端更近的地方。
    - 网关： 连接其他应用程序的特殊Web服务器。
    - 隧道： 对HTTP通信报文进行盲转发的特殊代理。
    - Agent代理： 发起自动HTTP请求的半智能Web客户端。
