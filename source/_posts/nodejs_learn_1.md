---
title: nodejs 学习(一)
category: js 
tag: nodejs
---

## 介绍

[nodejs 中文网](http://nodejs.cn)

2009 年Ryan Dahl 柏林宣布nodeJs 技术
优点: 基于事件轮询(event loop)的技术,构建于V8之上

```javascript
var http = require('http')
var server  = http.creatServe(function(req,res){
  res.writeHead('200')
  res.end('Hello World')
})
server.listen(80)
```

上面这段代码并不是一个玩具,相反,它是一个高性能的服务器.甚至在某些场景之下，比现在的Apach和nginx这样的服务器都性能优越.

<!-- more -->

## 服务器端js和客户端js的区别

javascript 是根据ECMAScript语言标准来实现的, 由于Node使用的是Google的V8，更接近ECMAScript的标准
V8的一些特性是一些浏览器不支持的, IE就更不用说了.

```javascript
npm version
```

查看nodejs 以及 V8的版本号

### 全局变量

global 和 客户端js的window一样,它上面的所有对象都可以被全局访问,命令行输入node,然后敲global 可以全部看到如下
其中所有全局执行上下文都在**process**对象中,就向在浏览器中只有一个window对象,在nodejs中只有一个process对象。

**console**,**setTimeout**,**setImmediate** 等都是一些非常重要的API.

```javascript
global
{ console: [Getter],
  ...
  global: [Circular],
  process:
    process{
      title: '',
      version: '8.9.1',
      ...
  node: '8.9.1',
  v8: '6.1',
  ...
  clearImmediate: [Function],
  clearInterval: [Function],
  clearTimeout: [Function],
  setImmediate: { [Function: setImmediate] [Symbol(util.promisify.custom)]: [Function] },
  setInterval: [Function],
  setTimeout: { [Function: setTimeout] [Symbol(util.promisify.custom)]: [Function] },
  module:
    Module: {
      id: '<repl>',
      exports: {},
      parent: undefined,
      filename: null,
      loaded: false,
      children: [],
      paths:
    require:
      { [Function: require]
      resolve: { [Function: resolve] paths: [Function: paths] },
      main: undefined,
      extensions: { '.js': [Function], '.json': [Function], '.node': [Function] },
      cache: {} } }
```

### 模块&包管理

nodejs 采用`npm`作为包管理机制,类似java中的maven
主要是这三个核心全局对象的用法: require,module,和export
**绝对模块**: 指Node通过其内部node_modules查找模块或者内置的例如fs这样的模块

`require('color')` 指的是安装路径为 `./node_modules/colors` 的模块

**相对模块**: 指的是相对于工作目录中的JavaScript文件,一般是自己写的模块。

module_a.js

```javascript
exports.name = '我是A模块'
exports.changeByThis = function(){
  this.name = 'A模块自己被自己的方法改变了this';
}
exports.changeByExport = function(){
exports.name = 'A模块自己被自己的方法改变了exports';
```

module_b.js

```javascript
var a = require('./module_a');
exports.name='我是b模块';
exports.changeA = function(){
  a.name = 'b 模块改变了a模块';
}
```

module_test.js

```javascript
var a  = require('./module_a')
var b  = require('./module_b')
console.log('aname:',a.name)
console.log('bname:',b.name)
a.changeByThis()
console.log('aname:',a.name)
a.changeByExport()
console.log('aname:',a.name)
b.changeA()
console.log('aname:',a.name)

// aname: 我是A模块
// bname: 我是b模块
// this {}
// aname: A模块自己被自己的方法改变了this
// aname: A模块自己被自己的方法改变了exports
// aname: b 模块改变了a模

```

**注意:**
[exports 和 module.exports 的区别](https://cnodejs.org/topic/5231a630101e574521e45ef8)

### 事件

浏览器端的js依赖DOM的api, 比如: `addEventListener`,`removeEventListener`,`dispathEvent`

Nodejs 中事件的基础是EventEmitter对象,事件是Nodejs非阻塞异步事件驱动重要体现.

> 所有能触发事件的对象都是 EventEmitter 类的实例。 这些对象开放了一个 eventEmitter.on() 函数，允许将一个或多个函数绑定到会被对象触发的命名事件上。 事件名称通常是驼峰式的字符串，但也可以使用任何有效的 JavaScript 属性名。
> 如: net.Server 对象会在每次有新连接时触发事件；fs.ReadStream 会在文件被打开时触发事件；流对象 会在数据可读时触发事件。

```javascript
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}
const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('触发了一个事件！');
});
server.once('connection', () => {// 只会触发一次
  console.log('首次调用！');
});
myEmitter.emit('event');
myEmitter.emit('connection');// 只会触发一次
myEmitter.emit('connection');
```

## 阻塞和非阻塞IO

所有关于Nodejs的讨论都使人们把关注点在了起高并发的能力上面,但是Node是如何给开发这提供一个构建高性能网络应用的能力的呢

```javascript
console.log('Hello')
setTimeout(function(){
  console.log('World')
},5000)
console.log('Bye')
// Hello
// Bye
// World

```

> 采用事件轮询意味着什么呢? 从本质上说, Node 会显注册事件，随后不断的询问内核这些事件有没有分发，当事件分发时，对应的回调函数就会被触发。如果没有事件触发，则继续执行其他代码，直到有新事件的时候，再去执行的对应的回调函数

其他语言的代码如下:

```php
print('Hello');
sleep(5);
print('World');

```

sleep一旦执行，执行会被阻塞一段指定的时间，并且会在阻塞时间未到设定时间之前不做任何操作，所以sleep是同步的。而Nodejs 的setTimeout只是注册了一个事件,程序继续执行,所以这个是异步的。

> Nodejs 并发实现也采用了事件轮询。所有向http,net这样原生模块中IO部分都采用事件轮询技术。Node使用事件轮询会触发一个和文件描述符相关的通知。文件描述符是抽象的句柄，存在对打开文件，socket,管道等的引用。本质上来说，当Node接收道从浏览器发来的Http请求时，底层的TCP连接会分配一个文件描述符，队友如果客户端向服务器发送数据，Node会受到文件描述符的通知，去触发相应的回调函数.

### 单线程

```javascript
var start = Date.now();
setTimeout(function(){
  console.log(Date.now()-start);
  for(var i=0;i<100000;i++){
    console.log();
  }
},1000)

setTimeout(function(){ 
  console.log(Date.now() - start);
},2000)
// 10436
```

为什么会这样, 事件轮询被JavaScript代码阻塞了。由于回调函数需要执行很长一段事件. 所以下一个事件轮询执行的事件就远远超过了2s
关键在于，在V8调用堆栈非常快的情况下，同一时刻无需处理多个请求.而Nodejs不会因为有数据库访问或者硬盘访问而挂起。

```javascript
http.createServer(function(req,res){
  database.getInformation(function(data){
    res.writeHead(200);
    res.end(data);
  });
})
```

当请求到达的时候,调用堆栈只有数据库调用。由于调用时非阻塞的。当数据库IO完成时。就完全取决于事件轮询合适在初始化新的调用堆栈。
当数据库响应时,内核会通知NodeJs事件轮询。Nodejs 现在可以继续处理其他事情了。

EventLoop 比较复杂,作为初学者就不敢在这里板门弄斧了,大家看之前两位大神的讨论:

[JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
[【朴灵评注】JavaScript 运行机制详解：再谈Event Loop](http://blog.csdn.net/lin_credible/article/details/40143961)