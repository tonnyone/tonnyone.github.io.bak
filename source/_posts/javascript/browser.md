---
title: javascript 异步编程-5.异步脚本加载
category: javascript 异步编程读书笔记
date: 2017-12-20 00:00:00
---

# 异步脚本加载

```javascript
<script src = "allMyClientSideCode.js"></script>
```

放在哪儿? 

- head标签里面

用户在脚本加载完毕之前一直处于“白屏死机”状态

- 放到body 后面

<!--more-->

原本应该进行客户端渲染的地方却散布着不起作用的控件和空空如也的方框。

完美解决问题的办法
完美解决这个问题需要对脚本分而治之：那些负责让页面更好看、更好用的脚本应该立即加载，而那些可以待会儿再加载的脚本稍后再加载。
HTML5之async/defer属性的作用，以及两个流行的脚本加载库：yepnope和Require.js。

## Script 标签再认识

经典型和非阻塞型

### 阻塞型脚本

script 标签被称为阻塞型标签
现代浏览器看到阻塞型\<script\>标签时，会跳过阻塞点继续读取文档及下载其他资源（脚本和样式表）。但直到脚本下载完毕并运行之后，浏览器才会评估阻塞点之后的那些资源。

因此，如果网页文档的\<head\>标签里有5个阻塞型\<script\>标签，则在所有这5个脚本均下载完毕并运行之前，用户除了页面标题之外看不到任何东西。不仅如此，即便这些脚本运行了，它们也只能看到阻塞点之前的那部分文档。如果想看到<body>标签中正等待加载的那些好东西，就必须给像document.onreadys-tatechange这样的事件绑定一个事件处理器。

基于上述原因，现在越来越流行把脚本放在页面\<body\>标签的尾部。这样，一方面用户可以更快地看到页面，另一方面脚本也可以主动亲密接触DOM而无需等待事件来触发自己。对大多数脚本而言，这次“搬家”是个巨大的进步。

但并非所有脚本都一样。在向下搬动脚本之前，请先问自己3个问题。

- 该脚本是否有可能被\<body\>标签里的内联JavaScript直接调用？
- 该脚本是否支持老式浏览器识别HTML5元素？Modern-izr支持，也正因为这个原因，最佳实践项目的典范HTML5 Boilerplate直接在文档顶部包含了Modernizr。
- 该脚本是否会影响已渲染页面的外观？Typekit宿主字体就是一个例子。如果把Typekit脚本放在文档末尾，那么页面文本就会渲染两次，即读取文档时即刻渲染，脚本运行时再次渲染。

### 脚本延迟执行

理想情况下，脚本的加载应该与文档的加载同时进行，并且不影响DOM的渲染。这样，一旦文档就绪就可以运行脚本，因为已经按照\<script\>标签的次序加载了相应脚本。

如果大家已经读到这里了，那么一定会迫不及待地想写一个自定义Ajax脚本加载器以满足这样的需求！不过，大多数浏览器都支持一个更为简单的解决方案。

```javascript
<script defer src = "deferredScript.js">
```

添加defer（延迟）属性相当于对浏览器说：“请马上开始加载这个脚本吧，但是，请等到文档就绪且所有此前具有defer属性的脚本都结束运行之后再运行它。”在文档\<head\>标签里放入延迟脚本，既能带来脚本置于\<body\>标签时的全部好处，又能让大文档的加载速度大幅提升！

有什么不足？并非所有浏览器都支持defer。，甚至最新版的Opera都忽略了这个属性。[参见](https://caniuse.com/#search=defer)

如果想确保自己的延迟脚本能在文档加载后运行，就必须将所有延迟脚本的代码都封装在诸如jQuery之$(document).ready之类的结构中。这是值得的，因为差不多97%的访客都能享受到并行加载的好处，同时另外3%的访客仍然能使用功能完整的JavaScript。

使用defer属性把bodyScript.js换成deferredScript.js，于是

```html
<html><head>
  <!-- metadata and stylesheets go here --> 
   <script src="headScripts.js"></scripts>
   <script defer src="deferredScripts.js"></script>
   </head>
   <body>  <!-- content goes here --></body>
</html>
```

请记住deferredScripts的封装很重要，这样即使浏览器不支持defer，deferredScripts也会在文档就绪事件之后才运行。如果页面主体内容远远超过几千字节，那么付出这点代价是完全值得的。

### 脚本的完全并行化

如果你是斤斤计较到毫秒级页面加载时间的完美主义者，那么defer也许就像是淡而无味的薄盐酱油。你可不想一直等到此前所有的defer脚本都运行结束，当然也肯定不想等到文档就绪之后才运行这些脚本，更别提为了照顾Opera还使用什么$(doc-ument).ready了。你就是想尽快加载并且尽快运行这些脚本。

这也正是现代浏览器提供了async（异步）属性的原因

```javascript
<script async src = "speedyGonzales.js">
<script async src = "roadRunner.js">
```

如果说defer让我们想到一种静静等待文档加载的有序排队场景，那么async就会让我们想到混乱的无政府状态。前面给出的那两个脚本会以任意次序运行，而且只要JavaScript引擎可用就会立即运行，而不论文档就绪与否。因此，抛开对速度的渴求，我们有什么理由用async呢？

async不像defer那样得到广泛的支持，因此很少有用户会注意到性能的提升。同时，由于异步脚本会在任意时刻运行，它实在太容易引起海森堡蚁虫之灾了（脚本刚好结束加载时就会蚁虫四起）。

但是，对那些一心一意搞独立的脚本来说，async确实是一次不大但很重要的胜利。获取一个负责添加反馈小部件的第三方脚本，再获取一个负责添加技术支持聊天框的第三方脚本，怎么样？页面没有它们也运行得很好，而且也不在乎它们谁先运行谁后运行。因此，对这些第三方脚本使用async属性，相当于一分钱没花就提升了它们的运行速度.

“如果我对同一个脚本既用defer又用async，会怎么样呢？”答案是，在那些同时支持这两个属性的浏览器中，async会覆盖掉defer。由于defer有着更广泛的支持，而且具有async的主要优势（允许在下载脚本的同时进行DOM的渲染），因此我们建议尽量使用defer代替async。

```html
<html>
<head>
  <!-- metadata and stylesheets go here -->
    <script src="headScripts.js"></scripts>
    <script src="deferredScripts.js" defer></script>
</head>
<body> 
<!-- content goes here -->  
<script async defer src="feedbackWidget.js"></script>
<script async defer src="chatWidget.js"></script>
</body>
```

## 可编程的脚本加载

yepnope和Require.js。

### 直接加载脚本

在浏览器API层面，有两种合理的方法来抓取并运行服务器脚本。

- 生成Ajax 请求bing用eval函数响应处理 
- 向Dom 插入script标签

后一种方法更好，因为浏览器会替我们操心生成HTTP请求这样的事。再者，eval也有一些实际问题：泄漏作用域，调试搞得一团糟，而且还可能降低性能。因此，要想加载名为feature.js的脚本，我们应该用类似下面这样的代码来插入\<script\>标签。

```javascript
var head = document.getElementsByTagName('head')[0];
var script = document.createElement('script');
script.src = '/js/feature.js';
head.appendChild(script);
```

我们如何才能知道脚本何时加载结束呢？我们可以给脚本本身添加一些代码以触发事件，但如果要为每个待加载脚本都添加这样的代码，那也太闹心了。或者是另外一种情况，即我们不可能给第三方服务器上的脚本添加这样的代码。

HTML5规范定义了一个可以绑定回调的onload属性。

```javascript
script.onload = function(){}
```
不过，IE8及更老的版本并不支持onload，它们支持的是onreadystatechange。某些浏览器在插入\<script\>标签时还会出现一些“灵异事件”。而且，这里甚至还没谈到错误处理呢！为了避免所有这些令人头疼的问题，笔者在此强烈建议使用脚本加载库。

### yepnope的条件加载

[yepnope](http://yepnopejs.com/)
yepnope是一个简单的、轻量级的脚本加载库（压缩后的精简版只有1.7KB），其设计目标就是真诚服务于最常见的动态脚本加载需求。yepnope可以独立使用，也可以作为Modernizr特性检测库的一部分。

yepnope最简单的用法是，加载脚本并对脚本完成运行这一事件返回一个回调。

```javascript
yepnope({
  load: 'oompaLoompas.js',
  callback: function() {
    console.log('Oompa-Loompas ready!'); 
  }
});
```

下面我们要用yepnope来并行加载多个脚本并按给定次序运行它们。举个例子，假设我们想加载Back-bone.js，而这个脚本又依赖于Underscore.js。

```javascript
yepnope({ load: ['underscore.js', 'backbone.js'],
    complete: function () {}
})
```

请注意，这里使用了complete（完成）而不是callback（回调）。其差别在于，脚本加载列表中的每个资源均会运行call-back，而只有当所有脚本都加载完成后才会运行complete。

** 条件加载**

```javascript
yepnope({
    test: Modernizr.touch,
    yep: ['touchStyles.css', 'touchApplication.js'],
    nope: ['mouseStyles.css', 'mouseApplication.js'],
    complete: function () {}
});
```

yepnope的标志性特征是条件加载。给定test参数，yepnope会根据该参数值是否为真而加载不同的资源。举个例子，假设已经使用了Modernizr库，则可以以一定的准确度判断用户是否在用触摸屏设备，从而据此相应地加载不同的样式表及脚本。

yepnope最常见的用法之一就是加载垫片脚本以弥补老式浏览器缺失的功能。

```javascript
yepnope({
    test: window.json,
    nope: ['json2.js'],
    complete: function () {}
});
```

页面使用了yepnope之后应该变成下面这种漂亮的标记结构：

```html
<html>
  <head>
    <!-- metadata and stylesheets go here -->
     <script src="headScripts.js"></scripts>
     <script src="deferredScripts.js" defer></script>
  </head>
  <body>
    <!-- content goes here -->
  </body>
</html>
```

### Require.js/AMD的智能加载

开发人员想通过脚本加载器让混乱不堪的富脚本应用变得更规整有序一些，而Require.js就是这样一种选择。Require.js这个强大的工具包能够自动和AMD技术一起捋顺哪怕最复杂的脚本依赖图。

```javascript
require(['moment'], function(moment) {
  console.log(moment().format('dddd'));
});
```

require函数接受一个由模块名称构成的数组，然后并行地加载所有这些脚本模块。与yepnope不同，Require.js不会保证按顺序运行目标脚本，只是保证它们的运行次序能满足各自的依赖性要求，但前提是这些脚本的定义遵守了AMD（AsynchronousModule Definition，异步模块定义）规范。

AMD规范的宗旨是替浏览器做CommonJS标准已经替服务器做过的那些事。（Node.js模块即基于CommonJS标准。）AMD推行一个由Require.js负责提供的名叫define的全局函数，该函数有3个参数：一个模块名称，一个模块依赖性列表，以及一个在那些依赖性模块加载结束时触发的回调。

```javascript
define('myApplication' ['jquery'], function ($) {
    $('<body>').append('<p>Hello, async world!</p>');
});
```

注意这里传递给回调的是jQuery对象$。实际上，define接受的这个回调参数一直对应着依赖性列表中的各个模块依赖项。大家也许奇怪：define怎样知道捕获jQuery对象呢？答案是jQuery自己的AMD定义通过其define回调返回了jQuery对象，借此声明“这是我的导出对象”。

```javascript
define( "jquery", [], function () { return jQuery; } );
```

AMD的奥妙不止于此，但这是精华所在。如果应用的每个脚本都添加了AMD定义，则意味着我们只要调用require就能保证其回调不被调用，除非既满足了应用对脚本的直接依赖性，又满足了脚本的依赖性和脚本所依赖的那些脚本的依赖性，并且所有脚本均按最大的并行性进行加载，而运行次序也和依赖图一致。