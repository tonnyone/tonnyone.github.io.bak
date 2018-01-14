---
title: 一起学nodejs(写一个基于TCP/IP的聊天系统)
category: js
tag: nodejs
---

## TCP/IP 协议回顾

![三次握手四次挥手](/uploads/tcp-ip-process.png)
<!--more-->

### TCP

- 面向连接的传输协议

> 可以这么理解,面向连接的意思是先连接了才能通信,UDP是无连接的,两者的区别是前者是打电话,后者是发短信

- 有序的

> TCP下层IP协议传输是基于数据包无序的,假设一条消息分为四部分的话，TCP先收到第一和第四部分，会等待第二第三部分

- 可靠的

> 当数据发送出去以后,发送方就会等待一个确认消息，如果过了指定事件未收到，发送方就会对数据进行重发

- 面向字节的

> TCP对字符以及字节是完全不知道的.

- 流控制

> 如果发送发和接收方的速度不匹配，TCP有流控制的机制保证发送和接收方数据的传输的平衡

- 拥堵控制

> TCP 会通过控制数据包的传输速率来保证数据包的延迟率和丢包率不会太高，以此确认服务的质量

适用场景:

- 文件传输
- HTTP

### UDP

- 无连接
- 不可靠
- 不保证有序
- 不会重发
- 无流量控制
- 无拥堵控制

适用场景:

- 包总量较小的通信
- 视频音频等多媒体通信
- 限定于LAN等特定网络中的通信
- 广播多播通信

### 基于TCP 和 UDP 的应用协议

![](/uploads/tcp_yingyong.png)

## nodejs 中 tcp/ip

Node HTTP 服务器是构建于NODE TCP服务器之上的 , 即 [http.Server](http://nodejs.cn/api/http.html)继承至 [net.Server](http://nodejs.cn/api/net.html) 模块