---
title: javascript 异步编程-2.分布式事件
category: javascript 异步编程读书笔记
date: 2017-12-05 00:00:00
---

# 分布式事件

在本章中，你将会学到如何使用PubSub（Publish/Sub-scribe，意为“发布/订阅”）模式来分发事件。
沿着这个思路，我们会看到PubSub模式的一些具体表现：Node的EventEmitter对象、Backbone的事件化模型和jQuery的自定义事件。
在这些工具的帮助下，我们能解嵌套那些嵌套式回调，减少重复冗余，最终编写出易于理解的事件驱动型代码。

<!--more-->

## PubSub 模式

### 浏览器端

从JavaScript诞生之日起，浏览器就允许向DOM元素附加事件处理器,形如 ```link.onclick = clickHandler;``` 
一个事件添加多个处理器必须这么处理

```javascript
link.onclick = function() {
  clickHandler1.apply(this, arguments); 
  clickHandler2.apply(this, arguments);
};
```

### 服务端

nodejs 包含了一个一般性的PubSub实体,EventEmitter。nodejs 中几乎所有的 I/O源都式EventEmitter对象。 文件流，http服务器，甚至应用进程本身。

```javascript
['room', 'moon', 'cow jumping over the moon'].forEach(function (name) {
    process.on('exit', function () {
        console.log('Goodnight, ' + name);
    });
});

emitter.on('evacuate', function(message) {  console.log(message);});
emitter.emit('evacuate', 'Woman and children first!');
```

EventEmitter对象的所有方法都是公有的，但一般约定只能从EventEmitter对象的“内部”触发事件。也就是说，如果有一个对象继承了EventEmitter原型并使用了this.emit方法来广播事件，则不应该从这个对象之外的其他地方再调用其emit方法。


### 构建自己的PUB/SUB
如下:

```javascript
PubSub = {
    handlers: {}
}
PubSub.on = function (eventType, handler) {
    if (!(eventType in this.handlers)) {
        this.handlers[eventType] = [];
    }
    this.handlers[eventType].push(handler);
    return this;
}
PubSub.emit = function (eventType) {
    var handlerArgs = Array.prototype.slice.call(arguments, 1);
    for (var i = 0; i < this.handlers[eventType].length; i++) {
        this.handlers[eventType][i].apply(this, handlerArgs);
    }  return this;
}
```

PubSub模式的实现这么简单，需要添加监听事件的时候,只要将监听器推入数组的末尾即可.
触发事件的时候再循环遍历所有的事件处理器.
这里我们相当于实现了PUB/SUB的核心部分，移除事件和附加一次性事件处理器功能暂时没有实现.

JQUERY 库里面到处都有几个不同的Pub/Sub实现，于是决定再Jquery1.7将他们抽象为$.Callbacks
还有的Pub/Sub负责解析字符串时实现一些特殊的功能，
如: 

- jquery "click.tbb" "hover.tbb" 通过unbind.("tbb") 就可以同时解绑两个事件
- BackBone的All事件
- jquery 和 BackBone 支持用空格隔开多个事件来同时绑定和触发多种事件类型

### 同步性

尽管Pub/Sub 模式是一项处理异步事件的重要技术,但它内部和异步没有一点关系.

```javascript
$('input[type=submit]').on('click', function () {
    console.log('foo');
}).trigger('click');
console.log('bar');
// for
// bar
```

这证明了click事件的处理器因trigger方法而立即被激活。事实上，只要触发了jQuery事件，就会不被中断地按顺序执行其所有事件处理器。

如果事件按顺序触发了过多的处理器，就会有阻塞线程且导致浏览器不响应的风险。更糟糕的是，如果事件处理器本身触发了事件，还很容易造成无限循环。

```javascript
$('input[type=submit]').on('click', function () { $(this).trigger('click');  //堆栈上溢！});
```

回想本章开头提到的文字处理程序的例子。用户按键时，需要发生很多事情，其中某些事还需要复杂的计算。全部做完这些事之后再返回事件队列，只会制造出响应迟钝的应用。

解决办法: 就是对那些无需即刻发生的事情维持一个队列，并使用一个计时函数定时运行此队列中的下一项任务。

```javascript
var tasks = []; 
setInterval(function () { 
    var nextTask; 
    if (nextTask = tasks.shift()) {
         nextTask();
    };
}, 0);
```

## 事件化模型

只要对象带有PubSub接口，就可以称之为**事件化对象**。特殊情况出现在用于存储数据的对象因内容变化而发布事件时，这里用于存储数据的对象又称作模型。模型就是MVC（Model-View-Controller，模型-视图-控制器）中的那个M。MVC三层架构设计模式在最近几年里已经成为JavaScript编程中最热点的主题之一。MVC的核心理念是应用程序应该以数据为中心，所以模型发生的事件会影响到DOM（即MVC中的视图）和服务器（通过MVC中的控制器而产生影响）。

老式的JavaScript依靠输入事件的处理器直接改变DOM。新式的JavaScript先改变模型，接着由模型触发事件而导致DOM的更新。在几乎所有的应用程序中，这种关注层面的分离都会带来更优雅、更直观的代码。

### 模型事件的传播
正如我们知道的，JavaScript确实没有一种每当对象变化时就触发事件的机制。因此请记住，事件化模型要想工作的话，必须要使用一些像Backbone.js之set/get这样的方法。

```javascript
style.set({font: 'Palatino'});     // 触发器警报！
style.get('font');                 // 结果为"Palatino"
style.font = 'Comic Sans';         // 未触发任何事件
style.font; // 结果为"Comic Sans"style.get('font');
// 结果仍为"Palatino"
```

将来也许无需如此，前提是名为Object.observe的EC-MAScript提案已经获得广泛接纳。

### 事件的循环和嵌套式变化

从一个对象向另一个对象传播事件的过程提出了一些需要关注的问题。
如果每次有个对象上的事件引发了一系列事件并最终对这个对象本身触发了相同的事件，则结果就是事件循环。
如果这种事件循环还是同步的，那就造成了堆栈上溢，

然而在很多时候，变化事件的循环恰恰是我们想要的。最常见的情况就是双向绑定——两个模型的取值会彼此关联。
假设我们想保证x始终等于2 * y。

```javascript
var x = new Backbone.Model({
    value: 0
});

var y = new Backbone.Model({
    value: 0
});

x.on('change:value', function (x, xVal) {
    y.set({
        value: xVal / 2
    });
});

y.on('change:value', function (y, yVal) {
    x.set({
        value: 2 * yVal
    });
});
```

你可能觉得当x或y的取值变化时，这段代码会导致无限循环。但实际上它相当安全，这要感谢Backbone中的两道保险。

- 当新值等于旧值时，set方法不会导致触发change事件。
- 模型正处于自身的change事件期间时，不会再触发change事件。

很明显，在Backbone中维持双向数据绑定是一个挑战。而另一个重要的MVC框架，即Ember.js，采用了一种完全不同的方式：双向绑定必须作显式声明。一个值发生变化时，另一个值会通过延时事件作异步更新。于是，在触发这个异步更新事件之前，应用程序的数据将一直处于不一致的状态。(此处后来者**angularjs，vue都借鉴与此**)

多个事件化模型之间的数据绑定问题不存在简单的解决方案。
在Backbone中，有一种审慎绕过这个问题的途径就是silent标志。
如果在set方法中添加了{silent:true}选项，则不会触发change事件。
因此，如果多个彼此纠结的模型需要同时进行更新，一个很好的解决方法就是悄无声息地设置它们的值。
然后，当这些模型的状态已经一致时，才调用它们的change方法以触发对应的事件。

事件化模型为我们带来了一种将应用状态变化转换为事件的直观方式。Backbone及其他MVC框架做的每件事都跟这些模型有关，这些模型的状态变化会触发DOM和服务器进行更新。要想掌控客户端JavaScript应用程序与日俱增的复杂度，运用事件化模型存储互斥数据是伟大长征的第一步。(作者此处非常有远见)

### Jquery 的自定义事件

自定义事件是jQuery被低估的特性之一，它简化了强大分布式事件系统向任何Web应用程序的移植，而且无需额外的库。在jQuery中，可以使用trigger方法基于任意DOM元素触发任何想要的事件。

```javascript
$('#tabby, #socks').on('meow', function() { 
   console.log(this.id + ' meowed');
});
$('#tabby').trigger('meow'); // "tabby meowed"$('#socks').trigger('meow'); // "socks meowed"
```

冒泡技术。只要某个DOM元素触发了某个事件（譬如'click'事件），其父元素就会接着触发这个事件，接着是父元素的父元素，以此类推，一直上溯到根元素（即document），除非在这条冒泡之路的某个地方调用了事件的stopPropagation方法。

你是否也知道jQuery自定义事件的冒泡技术?
举个例子，假设有个名称为“soda”的span元素嵌套在名称为“bottle”的div元素中，
代码如下。

```javascript
$('#soda, #bottle').on('fizz', function() { 
     console.log(this.id + ' emitted fizz');
});
$('#soda').trigger('fizz');
//soda emitted fizz
//bottle emitted fizz
```

这种冒泡方式并非始终受人欢迎，幸运的是，jQuery同样提供了非冒泡式的triggerHandler方法。