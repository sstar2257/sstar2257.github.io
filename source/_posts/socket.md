---
title: socket编程踩坑
date: 2019-04-25 15:27:29
updated: 2019-04-25 15:27:29
tags: [随笔, linux, net]
---

> 情景：在看一份古早的tinyhttpd代码时，进行client和server的通信测试时遇到一个问题，当客户端被关闭时（主动/被动），服务端有一定概率结束，同时没有任何异常打印。

解决方法：在代码的各个部分加了很多打印，发现每次异常关闭时，程序没有打印该有的退出信息，所以首先可以确定程序是非正常退出。其次，每次退出没有规律可循，试了很多次，有时候是一次就直接关闭，有时候却能正常执行多次。所以感觉应该不是应用程序级别的行为，也没有看到程序运行崩溃等消息，所以判断是更底层（内核）级别的问题。最后确定的是，每次程序退出，都是在调用send前后发生，所以从上面找原因，google之终于找到了。  
一句话总结：当往一个已经收到RST响应的套接字发数据时，系统会发送一个SIGPIPE信号给进程，进程对于SIGPIPE的默认行为是结束进程。  
所以简单的解决方法有两个：一个是优化代码不往已经关闭的套接字写数据，一个是捕获SIGPIPE信号替换掉默认的终止行为。捕获了信号之后，果然服务器就不会突然关闭了。还可以将send的第四个flags参数设置为MSG\_NOSIGNAL。    


