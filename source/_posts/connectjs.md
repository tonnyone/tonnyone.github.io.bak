---
title: 一起学nodejs(connect)
category: js
tag: nodejs
---

## 中间件(middleware)的概念

后端的中间件的概念常常指与业务无关的将业务和底层解耦的组件，比如后端常说的消息中间件 `kafka`,'rabbitmq'.
在前端目前最为常见的在处理http请求的过程的组件,比如我们要介绍的connect.js, 后来的nodejs 框架 `expressjs`(expressjs3.* 依然是connectjs的封装),`koa` 等都借鉴了中间件的实现,最开始在angularjs1.x的时候,第一次看到$http模块拦截器的实现也是这种模式.其实这概念没有严格的定义,感觉这里应该叫做插件比较合适.

## connect

`connnect`是一个基于`HTTP`服务的工具集，它的核心功能就是提供了一种`中间件`的代码组织方式来与请求和响应做交互.
下面是connnect最常见的使用方式,github上有[大量的中间件](https://github.com/senchalabs/connect#middleware)简化http处理的过程

```javascript
var connect = require('connect');
var http = require('http');

var app = connect();

// gzip/deflate outgoing responses
var compression = require('compression');
app.use(compression());

// store session state in browser cookie
var cookieSession = require('cookie-session');
app.use(cookieSession({
    keys: ['secret1', 'secret2']
}));

// parse urlencoded request bodies into req.body
var bodyParser = require('body-parser');
app.use(bodyParser.urlencoded({extended: false}));

// respond to all requests
app.use(function(req, res){
  res.end('Hello from Connect!\n');
});

//create node.js http server and listen on port
http.createServer(app).listen(3000);
```

### 如何编写一个中间件

connect 的api非常少,一个`use()` 一个`listen()`,use方法用来添加中间件,listen则调用http模块的方法 `http.creatServer().listen()`;

#### 使用静态文件服务中间件,下面两种写法相同

写法1：

```javascript
var connect = require('connect');
var serveStatic = require('serve-static');

var app = connect();
app.use(serveStatic(__dirname,{'index': ['index.html', 'index.htm']}));
app.listen(3000);
```

写法2：

```javascript
var connect = require('connect');
var http = require('http');
var serveStatic = require('serve-static');

var app = connect();
app.use(serveStatic(__dirname,{'index': ['index.html', 'index.htm']}));
http.createServer(app).listen(3000);
```

#### 中间件的执行顺序

添加了三个中间件,按照添加顺序依次执行,每个中间件的`next`对应下一个中间件,那么第三个`middleWare`执行`next`的时候对应哪个中间件呢?
每次添加next()防止还有其他中间件,如果没有的话,则中间件链中断.实际上第三个`middleWares`中的`next()`和没执行效果是相同的
如果去掉middleWare2中的next() 调用,输出结果则是`12success`

```javascript
var connect = require('connect');

var app = connect();
app.use(middleWare1)
app.use(middleWare2)
app.use(middleWare3)
app.listen(3000);

function middleWare1(req,res,next){
  res.write('1')
  next();
  res.end('success');
}

function middleWare2(req,res,next){
  res.write('2')
  next();
}

function middleWare3(req,res,next){
  res.write('3')
  next();
}
// 页面输出123success
```

#### 模拟一下自己实现一个connect,只实现核心功能

```javascript

```

## 参考链接

- [connect](https://github.com/senchalabs/connect)
- [中间件是什么？如何解释比较通俗易懂](https://www.zhihu.com/question/19730582)
- [connect-middleware](https://stackoverflow.com/questions/5284340/what-is-node-js-connect-express-and-middleware)