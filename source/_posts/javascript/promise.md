---
title: javascript 异步编程-3.Promise对象和Deferred对象
category: javascript 异步编程读书笔记
date: 2017-12-10 00:00:00
---

# Promise 对象和 Deferred 对象

Promise对象的最大优势仍然在于，它可以轻松从现有Promise对象派生出新的Promise对象。我们可以要求代表着并行任务的两个Promise对象合并成一个Promise对象，由后者负责通知前面那些任务都已完成。也可以要求代表着任务系列中首任务的Promise对象派生出一个能代表任务系列中末任务的Promise对象，这样后者就能知道这一系列任务是否均已完成。待会儿我们就会看到，Promise对象天生就适合用来进行这些操作。

## Promise极简史

- 2007 Dojo框架刚从Twisted框架汲取灵感，新增了一个叫做dojo.Deferred的对象。
- 2009 Kris Zyp有感于dojo.Deferred的影响力提出了CommonJS之Promises/A规范
- 同年，Node.js首次亮相。Node早期的几个版本在其非阻塞式API中用到了Promise。
- 2010 Ryan Dahl决定切换至当时为人所熟知的callback(err, result...)格式，因为Promise是一种属于“用户之境”的甚高层构造。
- 2011 jQuery 1.5在2011年1月携$.ajax重量级重写之势，用其Promise实现震惊了无数初次接触Promise对象的开发者。不过，其他的开发者则忧心忡忡，因为jQuery 1.5对Promises/A规范的无视导致了微妙的API差异。

<!--more-->

## 生成Promise对象

假设我们提示用户应敲击Y键或N键。为此要做的第一件事就是生成一个$.Deferred实例以代表用户做出的决定。

```javascript
var promptDeferred = new $.Deferred();
promptDeferred.always(function () {
    console.log('A choice was made:');
});
promptDeferred.done(function () {
    console.log('Starting game...');
});
promptDeferred.fail(function () {
    console.log('No game today.');
});（
//注： always关键字仅适用于jQuery 1.6 + 。）
```

大家可能会奇怪：为什么本节叫做“生成Promise对象”，却要生成一个Deferred（延迟）实例？别担心，Deferred就是Promise！更准确地说，Deferred是Promise的超集，它比Promise多了一项关键特性：可以直接触发。纯Promise实例只允许添加多个调用，而且必须由其他什么东西来触发这些调用。

```javascript
$('#playGame').focus().on('keypress', function (e) {
    var Y = 121,
        N = 110;
    if (e.keyCode === Y) {
        promptDeferred.resolve();
    } else if (e.keyCode === N) {
        promptDeferred.reject();
    } else {
        return false;
    };
});
```

[例子](http://jsfiddle.net/TrevorBurnham/PJ6Bf/)

Promise只能执行或拒绝一次，之后就失效了。我们断言，Promise对象会一直保持挂起状态，
直到被执行或拒绝。对Promise对象调用state（状态）方法，可以查看其状态是"pending"、"resolved"，还是"rejected"。（到jQuery 1.7才添加了state方法，此前的版本使用的是isResolved和isRe-jected。）
如果正在进行的一次性异步操作的结果可以笼统地分成两种（如成功/失败，或接受/拒绝），则生成Deferred对象就能直观地表达这次任务。

### 生成纯的promise对象

我们刚刚了解到Deferred对象也是Promise对象，那么，如何得到一个不是Deferred对象的Promise对象呢？很简单，对Deferred对象调用promise方法即可。

```javascript
var promptPromise = promptDeferred.promise();
```
promptPromise只是promptDeferred对象的一个没有re-solve/reject方法的副本。我们把回调绑定至Deferred或其下辖的Promise并无不同，因为这两个对象本质上分享着同样的回调。它们也分享着同样的state（返回的状态值为"pending"、"re-solved"或"rejected"）。这意味着，对同一个Deferred对象生成多个Promise对象是毫无意义的。事实上，jQuery给出的只不过是同一个对象。

```javascript
var promise1 = promptDeferred.promise();
var promise2 = promptDeferred.promise();
console.log(promise1 === promise2); // true
```

每个Deferred对象都含有一个Promise对象，而每个Promise对象都代表着一个Deferred对象。有了De-ferred对象，就可以控制其状态，而有了纯Promise对象，只能读取其状态及附加回调。

### jQuery API中的Promise对象

开头列举了jQuery的Ajax函数（$.ajax、$.get及$.post）可返回的几个Promise对象。
Ajax是演示Promise的绝佳用例：每次对远程服务器的调用都或成功或失败，而我们希望以不同的方式来处理这两种情况。不过，Promise也同样适用于本地的一些异步操作，譬如动画。

```javascript
$('.error').fadeIn(afterErrorShown);

var errorPromise = $('.error').fadeIn().promise();
errorPromise.done(afterErrorShown);

var $flash = $('.flash');
var showPromise = $flash.show().promise();
var hidePromise = $flash.hide().promise();
```

相当简单，对不对？在jQuery 1.6及jQuery 1.7中，jQuery对象的promise方法只是一种权宜之计。如果使用Deferred对象的resolve方法作为动画的回调，即可自行轻松生成一个行为完全相同的动画版Promise对象

```javascript
var slideUpDeferred = new $.Deferred();
$('.menu').slideUp(slideUpDeferred.resolve);
var slideUpPromise = slideUpDeferred.promise();
```

jQuery 1.8又向jQuery大家庭中新添了一种Promise资源：$.ready. promise()也能生成一个Promise对象，
并且当文档就绪时即执行该对象。这意味着以下3行代码现在是等效的。

```javascript
$(onReady); 
$(document).ready(onReady); 
$.ready.promise().done(onReady);
```

## 向回调传递数据

```javascript
var aDreamDeferred = new $.Deferred();
aDreamDeferred.done(function(subject) {
  console.log('I had the most wonderful dream about', subject);
});
aDreamDeferred.resolve('the JS event model');
```

执行或拒绝Deferred对象时，提供的任何参数都会转发至相应的回调。
还有一些特殊的方法能实现在特定上下文中运行回调（即将this设置为特定的值）：resolveWith和rejectWith。
此时只需传递上下文环境作为第一个参数，同时以数组的形式传递所有其他

```javascript
var slashdotter = {
    comment: function (editor) {
        console.log('Obviously', editor, 'is the best text editor.');
    }
};
var grammarDeferred = new $.Deferred();
grammarDeferred.done(function (verb, object) {
    this[verb](object);
});
grammarDeferred.resolveWith(slashdotter, ['comment', 'Emacs']);
```

然而，将参数打包成数组是很痛苦的。所以还有一个小窍门：不再使用resolveWith/rejectWith，而是直接在目标上下文中调用resolve/ reject方法，这是因为resolve/reject可以直接将其上下文环境传递至自己所触发的回调。因此，对于前面那个例子，使用以下代码亦可得到同样的结果。

```javascript
grammarDeferred.resolve.call(slashdotter, 'comment', 'Emacs');
```

### 进度通知

jQuery团队意识到了进度的重要并遵守Promises/A规范，
于是在jQuery 1.7中为Promise对象新添了一种可以调用无数次的回调。
这个回调叫做progress（进度）。举个例子，假设有人正在奋力达成美国全国小说写作月（National Novel WritingMonth，简写为NaNoWriMo）项目设定的日均码字目标，而我们希望更新一个指示器以反映他距离实现这个目标还有多远。

```javascript
var nanowrimoing = $.Deferred();
var wordGoal = 5000;
nanowrimoing.progress(function (wordCount) {
    var percentComplete = Math.floor(wordCount / wordGoal * 100);
    $('#indicator').text(percentComplete + '% complete');
});
nanowrimoing.done(function () {
    $('#indicator').text('Good job!');
});

$('#document').on('keypress', function () {
    var wordCount = $(this).val().split(/\s+/).length;
    if (wordCount >= wordGoal) {
        nanowrimoing.resolve();
    };
    nanowrimoing.notify(wordCount);
});
```

Deferred对象的notify（通知）调用会调用我们设定的progress回调。就像resolve和reject一样，notify也能接受任意参数。
请注意，一旦执行了nanowrimoing对象，则再作nanowrimoing.notify调用将不会有任何反应，这就像任何额外的resolve调用及reject调用也会被直接无视一样。

Promise对象接受3种回调形式：done、fail和progress。执行Promise对象时，运行的是done回调；拒绝Promise对象时，运行的是fail回调；对处于挂起状态的Deferred对象调用notify时，运行的是progress回调。

## Promise 对象的合并

Promise对象的逻辑合并技术有一个最常见的用例：判定一组异步任务何时完成。假设我们正在播放一段演示视频，同时又在加载服务器上的一个游戏。我们希望这两件事一旦结束（对次序没有要求），就马上启动游戏。

```javascript
var gameReadying = $.when(tutorialPromise, gameLoadedPromise);
gameReadying.done(startGame);
```

when相当于Promise执行情况的逻辑与运算符（AND）。
一旦给定的所有Promise均已执行，就立即执行when方法产生的Promise对象；或者，一旦给定的任意一个Promise被拒绝，就立即拒绝when产生的Promise。

```javascript
$.when($.post('/1', data1), $.post('/2', data2)).then(onPosted, onFailure);
```

调用成功时，when可以访问下辖的各个成员Promise对象的回调参数，不过这么做很复杂。这些回调参数会当作参数列表进行传递，传递的次序和成员Promise对象传递给when方法时一样。如果某个成员Promise对象提供多个回调参数，则这些参数会先转换成数组
虽然有可能，但如果不是绝对必要，我们不应该自行解析when回调的参数，
相反应该直接向那些传递至when方法的成员Promise对象附加回调来收集相应的结果。

```javascript
var serverData = {};
var getting1 = $.get('/1').done(function (result) {
    serverData['1'] = result;
});
var getting2 = $.get('/2').done(function (result) {
    serverData['2'] = result;
});
$.when(getting1, getting2).done(function () { // 获得的信息现在都已位于serverData……
});
```

### 函数的promise用法

$.when及其他能取用Promise对象的jQuery方法均支持传入非Promise对象作为参数。
这些非Promise参数会被当成因相应参数位置已赋值而执行的Promise对象来处理。

```javascript
$.when('foo')
```

会生成一个因赋值'foo'而立即执行的Promise对象。

```javascript
var promise = $.Deferred().resolve('manchu');
$.when('foo', promise)
```

会生成一个因赋值'foo'和'manchu'而立即执行的Promise对象。

```javascript
var promise = $.Deferred().resolve(1, 2, 3);
$.when('test', promise)
```

会生成一个因赋值'test'和数组[1,2,3]而立即执行的Promise对象。（请记住，Deferred对象传递多个参数给resolve方法时，$.when会把这些参数转换成一个数组。）

$.when如何知道参数是不是Promise对象呢？答案是：jQuery负责检查$.when的各个参数是否带有promise方法，如果有就使用该方法返回的值。Promise对象的promise方法会直接返回自身。

jQuery对象也可以有promise方法，这意味着$.when方法强行将那些带promise方法的jQuery对象转换成了jQuery动画版Promise对象。因此，如果想生成一个在抓取某些数据且已完成#loading动画之后执行的Promise对象，只需写下下面这样的代码：

```javascript
var fetching = $.get('/myData');
$.when(fetching, $('#loading'));
```

只是请记住，必须要在动画开始之后再执行$.when生成的那个Promise对象。
如果#loading的动画队列为空，则立即执行相应的Promise对象。

## 管道连接未来

在JavaScript中常常无法便捷地执行一系列异步任务，一个主要原因是无法在第一个任务结束之前就向第二个任务附加处理器。举个例子，假设我们要从一个URL抓取数据（GET），接着又将这些数据发送给另一个URL（POST）。

```javascript
var getPromise = $.get('/query');
getPromise.done(function (data) {
    var postPromise = $.post('/search', data);
});
```

看到这里的问题了吗？在GET操作成功之前我们无法对postPromise对象绑定回调，因为这时postPromise对象还不存在！除非我们已经得到因$.get调用而异步抓取的数据，否则甚至无法进行那个负责生成postPromise对象的$.post调用！

这正是jQuery 1.6为Promise对象新增pipe（管道）方法的原因。pipe好像在说：“请针对这个Promise对象给我一个回调，我会归还一个Promise对象以表示回调运行的结果。”

```javascript
var getPromise = $.get('/query');
var postPromise = getPromise.pipe(function(data) {
    return $.post('/search', data);
}
```

看起来就像黑魔法，对吧？下面是详情大揭秘：pipe最多能接受3个参数，它们对应着Promise对象的3种回调类型：done、fail和progress。
也就是说，我们在上述例子中只提供了执行getPromise时应运行的那个回调。当这个回调返回的Promise对象已经执行/拒绝时，pipe方法返回的那个新Promise对象也就可以执行/拒绝。

我们也可以通过修改pipe回调参数来“滤清”Promise对象。如果pipe方法的回调返回值不是Promise/Deferred对象，它就会变成回调参数。举例来说，假设有个Promise对象发出的进度通知表示成0与1之间的某个数，则可以使用pipe方法生成一个完全相同的Promise对象，但它发出的进度通知却转变成可读性更高的字符串。

- 如果pipe回调返回的是Promise对象，则pipe生成的那个Promise对象会模仿这个Promise对象。
- 如果pipe回调返回的是非Promise对象（值或空白），则pipe生成的那个Promise对象会立即因该赋值而执行、拒绝或得到通知，具体取决于调用pipe的那个初始Promise对象刚刚发生了什么。

pipe判定参数是否为Promise对象的方法和$.when完全一样：如果pipe的参数带有promise方法，则该方法的返回值会被当作Promise对象以代表调用pipe的那个初始Promise对象。再重申一次，promise. promise() === promise。

### 管道级联技术

pipe方法并不要求提供所有的可能回调。事实上，我们通常只想写成这样：

```javascript
var pipedPromise = originalPromise.pipe(successCallback);
var pipedPromise = originalPromise.pipe(null, failCallback);
```

```javascript
var getPromise = $.get('/query');
getPromise.done(function (data) {
    var postPromise = $.post('/search', data);
});

var step1 = $.post('/step1', data1);
var step2 = step1.pipe(function () {
    return $.post('/step2', data2);
});
var lastStep = step2.pipe(function () {
    return $.post('/step3', data3);
});
```

这里的lastStep对象当且仅当所有这3个Ajax调用都成功完成时才执行，其中任意一个Ajax调用未能成功完成，lastStep均被拒绝。如果只在乎整体进程，则可以省略掉前面的变量声明。

```javascript
var posting = $.post('/step1', data1).pipe(function () {
    return $.post('/step2', data2).pipe(function () {
        return $.post('/step3', data3);
    });
});
```

当然，这会重现金字塔厄运。大家应该了解这种书写风格，不过请尽量逐一声明pipe生成的那些Promise对象。也许并不需要这些变量名称，但它们能让代码更加自文档化
接下来简单介绍一下其主要替代方案：CommonJS的Promises/A规范及其旗舰版实现Q.js。

### jQuery与Promises/A的对比

jQuery的Promise与CommonJS的Promises/A几乎完全一样。
Q.js库是最流行的Promises/A实现，其提供的方法甚至能与jQuery的Promise和谐共存。这两者的区别只是形式上的，即用相同的词语表示不同的含义。
在jQuery 1.8问世之前，jQuery的then方法只是一种可以同时调用done、fail和progress这3种回调的速写法，
而Promises/A的then在行为上更像是jQuery的pipe。jQuery 1.8订正了这个问题，使得then成为pipe的同义词。不过，由于向后兼容的问题，jQuery的Promise再如何对Promises/A示好也不太会招人待见。
当然还有其他一些细微的差别。例如，在Promises/A规范中，由then方法生成的Promise对象是已执行还是已拒绝，取决于由then方法调用的那个回调是返回值还是抛出错误。（在jQuery的Promise对象的回调中抛出错误是个糟糕的主意，因为错误不会被捕获。）

基于上述原因，应该尽量避免在同一个项目中与多个Promise实现“打情骂俏”。如果因jQuery方法而得到Promise对象，请使用jQuery的Promise。如果因使用其他库而得到CommonJS Promise对象，则请遵守Promises/A规范。而Q.js可以轻松“消化”jQuery的Promise对象。

```javascript
var qPromise = Q.when(jqPromise);
```

### 用Promise对象代替回调函数

理想情况下，开始执行异步任务的任何函数都应该返回Promise对象。遗憾的是，大多数JavaScript API（包括所有浏览器及Node.js均使用的那些原生函数）都基于回调函数，而不是基于Promise对象。在本节中，我们将看到如何在基于回调函数的API中使用Promise对象。

```javascript
var timing = new $.Deferred();
setTimeout(timing.resolve, 500);

var fileReading = new $.Deferred();
fs.readFile(filename, 'utf8', function (err) {
    if (err) {
        fileReading.reject(err);
    } else {
        ileReading.resolve(Array.prototype.slice.call(arguments, 1));
    };
});
```

总这么写很麻烦，所以何不写一个工具函数以根据任何给定Deferred对象来生成Node风格的回调呢？

```javascript
deferredCallback = function (deferred) {
    return function (err) {
        if (err) {
            deferred.reject(err);
        } else {
            deferred.resolve(Array.prototype.slice.call(arguments, 1));
        };
    };
}
```

有了这个工具函数，前面那个例子就可以写成这样：

```javascript
var fileReading = new $.Deferred();
fs.readFile(filename, 'utf8', deferredCallback(fileReading));
```

Q.js的Deferred对象为此提供了一个现成的node方法：

```javascript
var fileReading = Q.defer();
fs.readFile(filename, 'utf8', fileReading.node());
```

随着Promise越来越流行，会有越来越多的JavaScript库循着jQuery的脚步，要求其异步函数必须返回Promise对象。到那时，只需要几行代码就能将想用的任何异步函数转变成Promise对象的生成函数。