---
title: javascript 异步编程-1.深入理解javascript事件
category: javascript 异步编程读书笔记
date: 2017-12-01 00:00:00
---

# 深入理解javascript事件

## 事件的调度

### 最常见的一个例子

```javaccript
for (var i = 1; i <= 3; i++) {
 setTimeout(function(){
     console.log(i);
 }, 0);
};
// 4 4 4
```

<!--more-->

> 循环结束后JavaScript的事件处理器才开始执行

### 关于setTimeout的理解

```javascript
var start = new Date;
setTimeout (function(){
    var end = new Date();
    console.log('Time elapsed:', end - start,'ms');
}, 500);
while (new Date - start; 1000) {};
// Time elapsed:
// 1010 ms
// Time elapsed:
// 1015ms
```

1. 按照多线程的思维定势，计500毫秒后计时函数就会运行。不过这要求中断欲持续整整一秒钟的循环。每次的结果都会稍有不同,但是总是大于1000ms.因为setTimeout回调在while循环结束运行之前不可能被触发.

1. 调用setTimeout的时候，会有一个延时事件排入队列。然后setTimeout调用之后的那行代码运行，接着是再下一行代码，直到再也没有任何代码。这时JavaScript虚拟机才会问：“队列里都有谁啊？”

1. 事件的易调度性是JavaScript语言最大的特色之一。像set-Timeout这样的异步函数只是简单地做延迟执行，而不是孵化新的线程。JavaScript代码永远不会被中断，这是因为代码在运行期间只需要排队事件即可，而这些事件在代码运行结束之前不会被触发。

## 异步函数的类型

> JavaScript环境提供的异步函数通常可以分为两大类：I/O函数和计时函数。
> 如果想在应用中定义复杂的异步行为，就要使用这两类异步函数作为基本的构造块。

### 异步的IO函数

1. 创造Node.js，并不是为了人们能在服务器上运行JavaScript，仅仅是因为Ryan Dahl想要一个建立在某高级语言之上的事件驱动型服务器框架。JavaScript碰巧就是适合干这个的语言。为什么？因为JavaScript语言可以完美地实现非阻塞式I/O。

1. 在浏览器端，Ajax方法有一个可设置为false的async选项（但永远、永远别这么做），这会挂起整个浏览器窗格直到收到应答为止。在Node.js中，同步的API方法在名称上会有明确的标示，譬如fs.readFileSync。编写短小的脚本时，这些同步方法会很方便。但是，如果所编写的应用需要处理并行的多个请求或多项操作，则应该避免使用它们。

1. 有些I/O函数既有同步效应，也有异步效应。举例来说，在现代浏览器中操纵DOM对象时，从脚本角度看，更改是即时生效的，但从视效角度看，在返回事件队列之前不会渲染这些DOM对象更改。[jsfiddle示例](http://jsfiddle.net/TrevorBurnham/SNBYV/)。

console.log是异步的吗 ?

```javacript
var obj = {};
console.log(obj);
obj.foo = 'bar';
```

怎么会这样 ?

1. WebKit的console.log并没有立即拍摄对象快照，相反，它只存储了一个指向对象的引用，然后在代码返回事件队列时才去拍摄快照。
1. Node的console.log是另一回事，它是严格同步的，因此同样的代码输出的却为{}。

### 异步的计时函数

#### 浏览器端

```javascript
var fireCount = 0;
var start = new Date;
var timer = setInterval(function() {
    if  (new Date-start > 1000) {
           clearInterval(timer);
           console.log(fireCount);
            return;
        }
        fireCount++;
    },
0);
// 246
// 251
```

- 大约为200次/秒。这是Chrome、Safari和Firefox等浏览器的平均值。

- 在Node环境下，此事件的触发频率大约能达到600次/秒。（若使用setTimeout来调度事件，重复这些实验也会得到类似的结果。）作为对比，

- 如果将setInterval替换成简单的while循环，则在Chrome中此事件的触发频率将达到400万次/秒，而在Node中会达到500万次/秒！ 

- 这是怎么回事？最后我们发现，setTimeout和setInterval就是想设计成慢吞吞的！事实上，HTML规范（这是所有主要浏览器都遵守的规范）推行的延时/时隔的最小值就是4毫秒.

- 在Node中，**process.nextTick** 允许将事件调度成尽可能快地触发。对于笔者的系统，process.nextTick事件的触发频率可以超过10万次/秒。

- 一些现代浏览器（含IE9+）带有一个requestAnimation-Frame函数。此函数有两个目标：一方面，它允许以60+帧/秒的速度运行JavaScript动画；另一方面，它又避免后台选项卡运行这些动画，从而节约CPU周期。在最新版的Chrome浏览器中，甚至能实现亚毫秒级的精度。

- setTimeout和setInterval就是些不精确的计时工具。在Node中，如果只是想产生一个短时延迟，请使用pro-cess.nextTick。在浏览器端，请尝试使用垫片技术（shim）：在支持requestAnimationFrame的浏览器中，推荐使用reques-tAnimationFrame；在不支持requestAnimationFrame的浏览器中，则退而使用setTimeout。

## 异步函数的编写

> JavaScript中的每个异步函数都构建在其他某个或某些异步函数之上。凡是异步函数，从上到下（一直到原生代码）都是异步的！

> 遗憾的是，要想确认某个函数异步与否，唯一的方法就是审查其源代码。有些同步函数却拥有看起来像是异步的API，这或者是因为它们将来可能会变成异步的，又或者是因为回调这种形式能方便地返回多个参数。

### 间或异步函数
> 有些函数某些时候是异步的，但其他时候却不然。举个例子，jQuery的同名函数（通常记作$）可用于延迟函数直至DOM已经结束加载。但是，若DOM早已结束了加载，则不存在任何延迟，$的回调将会立即触发。

```html
$(function() {  
    utils.log('Ready');
});
//utils.js
window.utils = {  
    log: function() { 
       if (window.console) 
       console.log.apply(console, arguments);
    }
};
<script src ＝"application.js"></script>
<script src ＝"util.js"></script>
```

这段代码运行得很好，但前提是浏览器并未从缓存中加载页面（这会导致DOM早在脚本运行之前就已加载就绪）。如果出现这种情况，传递给$的回调就会在设置utils.log之前运行，从而导致一个错误。

### 缓存型异步函数

### 异步递归和回调存储

> 请避免异步递归。仅当所采用的库提供了异步功能但没有提供任何形式的回调机制时，异步递归才有必要。如果真的遇到这种情况，要做的第一件事应该是为该库写一个补丁。或者，干脆找一个更好的库。

### 返回值与回调的混搭

## 异步错误处理

如果从异步回调中抛出错误，会发生什么事？

```javascript
setTimeout(function A() { 
     setTimeout(function B() { 
        setTimeout(function C() {
           throw new Error('Something terrible has happened!');
        },0); 
    }, 0);
}, 0);
//Error: Something terrible has happened!
//    at C (script 1:94)
```

上述应用的结果是一条极其简短的堆栈轨迹。，A和B发生了什么事？为什么它们没有出现在堆栈轨迹中？这是因为运行C的时候，A和B并不在内存堆栈里。这3个函数都是从事件队列直接运行的。基于同样的理由，利用try/catch语句块并不能捕获从异步回调中抛出的错误。

总的来说，取用异步回调的函数即使包装上try/catch语句块，也只是无用之举。（特例是，该异步函数确实是在同步地做某些事且容易出错。例如，Node的fs.watch(file,callback)就是这样一个函数，它在目标文件不存在时会抛出一个错误。）正因为此，Node.js中的回调几乎总是接受一个错误作为其首个参数，这样就允许回调自己来决定如何处理这个错误。举个例子，下面这个Node应用尝试异步地读取一个文件，还负责记录下任何错误（如“文件不存在”）。

```javascript
var fs = require('fs');
fs.readFile('fhgwgdz.txt', function(err, data) {
  if (err) {
  return console.error(err);
  };
  console.log(data.toString('utf8'));
}
```

客户端的例子

```javascript
$.get('/data',
 { success: successHandler, 
 failure: failureHandler}
);`
```

不管API形态像什么，始终要记住的是，**只能在回调内部处理源于回调的异步错误**

### 未捕获的异常处理

#### 浏览器环境
现代浏览器会在开发人员控制台显示那些未捕获的异常，接着返回事件队列。要想修改这种行为，可以给window.onerror附加一个处理器。如果windows.onerror处理器返回true，则能阻止浏览器的默认错误处理行为。

```javascript
window.onerror = function(err) {  return true;  //彻底忽略所有错误};
```

在成品应用中，会考虑某种JavaScript错误处理服务，譬如:[Errorception](https://errorception.com/)。Errorception提供了一个现成的windows.on-error处理器，它向应用服务器报告所有未捕获的异常，接着应用服务器发送消息通知我们。

#### NodeJs 环境

在Node环境中，window.onerror的类似物就是process对象的uncaughtException事件。正常情况下，Node应用会因未捕获的异常而立即退出。但只要至少还有一个uncaughtExcep-tion事件处理器，Node应用就会直接返回事件队列。

```javascript
process.on('uncaughtException', function(err) {  console.error(err);  });//避免了进程停止
```
但是，自Node 0.8.4起，uncaughtException事件就被废弃了。据其文档所言，
> 对异常处理而言，uncaughtException是一种非常粗暴的机制，它在将来可能会被放弃……请勿使用uncaughtException，而应使用Domain对象。

```javascript
var myDomain = require('domain').create();
myDomain.run(function () {
    setTimeout(function () {
        throw new Error('Listen to me!')
    }, 50);
});
myDomain.on('error', function (err) {
    console.log('Error ignored!');
});
```

源于延时事件的throw只是简单地触发了Domain对象的错误处理器。

### 抛出还是不抛出

如果抛出那些自己知道肯定会被捕获的异常呢？这种做法同样凶险万分。2011年，Isaac Schlueter（npm的开发者，在任的Node开发负责人）就主张try/catch是一种“反模式”的方式。
Schlueter提倡完全将throw用作断言似的构造结构，作为一种挂起应用的方式——当应用在做完全没预料到的事时，即挂起应用。Node社区主要遵循这一建议，尽管这种情况可能会随着Domain对象的出现而改变。

### 嵌套式回调的解嵌套

```javascript
function checkPassword(username, passwordGuess, callback) {
    var queryStr = 'SELECT * FROM user WHERE username = ?';
    db.query(queryStr, username, function (err, result) {
        if (err) throw err;
        hash(passwordGuess, function (passwordGuessHash) {
            callback(passwordGuessHash === result['password_hash']);
        });
    });
}
```

这段代码有什么问题呢？目前为止，没有任何问题。它能用，而且简洁明了。但是，如果试图向其添加新特性，它就会变得毛里毛躁、险象环生，比如去处理那个数据库错误，而不是抛出错误（请参阅1.4.3节）、记录尝试访问数据库的次数、阻塞访问数据库，等等。


```javascript
function checkPassword(username, passwordGuess, callback) {
    var passwordHash;
    var queryStr = 'SELECT * FROM user WHERE username = ?';
    db.query(qyeryStr, username, queryCallback);

    function queryCallback(err, result) {
        if (err) throw err;
        passwordHash = result['password_hash'];
        hash(passwordGuess, hashCallback);
    }

    function hashCallback(passwordGuessHash) {
        callback(passwordHash === passwordGuessHash);
    }
}
```

这种写法更啰嗦一些，但读起来更清晰，也更容易扩展。由于这里赋予了异步结果（即passwordHash）更宽广的作用域，所以获得了更大的灵活性。
按照惯例，请避免两层以上的函数嵌套。关键是找到一种在激活异步调用之函数的外部存储异步结果的方式，这样回调本身就没有必要再嵌套了。


