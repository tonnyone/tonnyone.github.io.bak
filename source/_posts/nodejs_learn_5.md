---
title: 一起学nodejs(写一个基于TCP/IP终端聊天系统)
category: js
tag: nodejs
---

![test](/uploads/tcpip.gif)
[源码地址](https://github.com/tonnyone/nodejs_practise/tree/master/tcp-ip)
<!--more-->

## TCP/IP 协议回顾

![三次握手四次挥手](/uploads/tcp-ip-process.png)

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

### server基本代码

```javascript
var net = require('net');
var fs = require('fs');

var HOST = '127.0.0.1';
var PORT = 9999;

net.createServer(function (sock) {

  console.log('connected: ' + sock.remoteAddress + ':' + sock.remotePort);

  sock.on('data', function (data) {
    console.log('DATA ' + sock.remoteAddress + ': ' + data);
    // sock.write('Hey client, You said "' + data + '"');
    sock.pipe(process.stdout)
    sock.end()
  });

  sock.on('close', function (data) {
    console.log('CLOSED: ' + sock.remoteAddress + ' ' + sock.remotePort);
  });

}).listen(PORT, HOST);

console.log('Server listening on ' + HOST + ':' + PORT);
```

### client 基础代码

```javascript
var net = require("net");
stdin = process.stdin;
stdout = process.stdout;

var HOST = "127.0.0.1";
var PORT = 9999;

var client = new net.Socket();
client.connect(PORT, HOST, function() {
  console.log('CONNECTED TO SERVER: ' + HOST + ':' + PORT);
  stdout.write('>> ')
  stdin.resume(); // 等待输入
  stdin.setEncoding("utf-8");
  stdin.on("data", function(input){
    if ("quit" == input.trim()) {
      client.destroy();
      stdin.destroy();
    } else {
      client.write(input);
    }
  });
});

client.on("data", function(data) {
  console.log("server: " + data);
  stdout.write('>> ')
});

client.on("close", function() {
  console.log("Connection closed");
});

```

### server端维护所有的客户client的关键代码

```javascript
 var userkey = conn.remoteAddress+':'+conn.remotePort;
  if(!users[userkey]){
    users[userkey] = {name: '匿名',conn:conn};
  }
  for (const userKey in users ) {
    if (users.hasOwnProperty(userKey)){
      const user = users[userKey];
      conn.write('\033[92m '+ user.name +': '+user.conn.remoteAddress+':'+user.conn.remotePort+'\033[39m \n');
    }
  }
```

### [完整代码请点击](https://github.com/tonnyone/nodejs_practise/tree/master/tcp-ip)
