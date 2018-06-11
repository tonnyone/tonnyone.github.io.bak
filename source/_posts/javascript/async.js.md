---
title: javascript 异步编程-4.async.js工作流控制
category: javascript 异步编程读书笔记
date: 2017-12-15 00:00:00
---

# Async 工作流控制

假设需要执行一组I/O操作（或者并行执行，或者串行执行），该怎么做呢？这个问题在Node中非常常见，以至于有了个专有名称：工作流控制（也称作控制工作流）。就像Underscore.js可以大幅度简化同步代码中的迭代一样，优秀的工作流控制库也可以消解异步代码中的套话。

目前最流行的工作流控制库当属Caolan McMahon开发的强大的[Async.js](https://github.com/caolan/async)。
事实上，在我写作本书的时候，Async.js是npm登记在案的请求第三多的库，它正与Un-derscore.js、Express这样的超级巨星一起共沐荣光。

<!--more-->

## 异步工作流的次序问题

假设想先按字母顺序读取recipes（菜谱）目录中的所有文件，接着把读取出的这些内容连接成一个字符串并显示出来。使用同步方法很容易做到这一点。

```javascript
var fs = require('fs');
process.chdir('recipes');
var concatenation = '';
fs.readdirSync('.').filter(function (filename) {
    return fs.statSync(filename).isFile();
}).forEach(function (filename) {
    concatenation += fs.readFileSync(filename, 'utf8')
});
console.log(concatenation);
```

不过，所有这种I/O阻塞的效率都极其低下，尤其是当应用程序还能同时做点其他事情的时候。

```javascript
concatenation += fs.readFileSync(filename, 'utf8');
```

换成异步代码

```javascript
fs.readFile(filename, 'utf8', function(err, contents) {
   if (err) throw err;  concatenation += contents;}
);
```

因为这么做根本无法保证按照做出readFile调用的次序来触发readFile调用的回调。
readFile仅仅负责告诉操作系统开始读取某个文件。
对操作系统而言，读取短文件通常比读取长文件更快一些。因此，菜谱内容添加到concatenation字符串的次序是不可预知的。

## 异步的数据收集方法

```javascript
var fs = require('fs'); process.chdir('recipes');
concatenation = '';
fs.readdir('.', function (err, filenames) {
    if (err) throw err;

    function readFileAt(i) {
        var filename = filenames[i];
        fs.stat(filename, function (err, stats) {
            if (err) throw err;
            if (!stats.isFile()) return readFileAt(i + 1);
            fs.readFile(filename, 'utf8', function (err, text) {
                if (err) throw err;
                concatenation += text;
                if (i + 1 === filenames.length) {
                    return console.log(concatenation);
                }
                readFileAt(i + 1);
            });
        });
    }
    readFileAt(0);
});
```

上面的代码使用递归解决这个问题，存在一下两个问题

- 回调中抛出异常,异常无法追踪
- 错误处理的逻辑，重复书写三次

### async.js 的函数式写法

我们想把同步迭代器所使用的filter和forEach方法替换成相应的异步方法。Async.js给了我们两个选择

- async.filter和async.forEach，它们会并行处理给定的数组。
- async.filterSeries和async.forEachSeries，它们会顺序处理给定的数组。

并行运行这些异步操作应该会更快，那为什么还要使用序列式方法呢？原因有两个。

- 前面提到的工作流次序不可预知的问题。我们确实可以先把结果存储成数组，然后再joining（联接）数组来解决这个问题，但这毕竟多了一个步骤。
- Node及其他任何应用进程能够同时读取的文件数量有一个上限。如果超过这个上限，操作系统就会报错。如果能顺序读取文件，则无需担心这一限制。

```javascript
var async = require('async');
var fs = require('fs');
process.chdir('recipes');
var concatenation = '';
var dirContents = fs.readdirSync('.');
async.filter(dirContents, isFilename, function (filenames) {
    async.forEachSeries(filenames, readAndConcat, onComplete);
});

function isFilename(filename, callback) {
    fs.stat(filename, function (err, stats) {
        if (err) throw err;
        callback(stats.isFile());
    });
}

function readAndConcat(filename, callback) {
    fs.readFile(filename, 'utf8', function (err, fileContents) {
        if (err) return callback(err);
        concatenation += fileContents;
        callback();
    });
}

function onComplete(err) {
    if (err) throw err;
    console.log(concatenation);
}
```

filter和forEach并不是仅有的与标准函数式迭代方法相对应的Async.js工具函数。Async.js还提供了以下方法：

- reject/rejectSeries，与filter刚好相反；
- map/mapSeries，1:1变换
- reduce/reduceRight，值的逐步变换
- detect/detectSeries，找到筛选器匹配的值
- sortBy，产生一个有序副本
- some，测试是否至少有一个值符合给定标准
- every，测试是否所有值均符合给定标准。

### 　Async.js的错误处理技术

初始版本的异步代码实现一共有3条throw语句。
到了Async.js版本只用了2条throw，不过所有错误仍然会被抛出。
Async.js是怎么做到的呢？为什么不能只用1条throw呢？

简单来说，Async.js遵守Node的约定。这意味着所有的I/O回调都形如callback(err, results...)，唯一的例外是结果为布尔型的回调。布尔型回调的写法就是callback(result)，所以上一代码示例中的isFilename迭代器需要自己亲自处理错误。

```javascript
function isFilename(filename, callback) {
  fs.stat(filename, function(err, stats) {
    if (err) throw err;
    callback(stats.isFile());
  });
}
```

要怪就怪Node的fs.exists首开这一先河吧！而这也意味着使用了Async.js数据收集方法（filter/filterSeries、reject/reject-Series、detect/detectSeries、some、every等）的迭代器均无法报告错误。

所以，如果callback(err)确实是在readAndConcat中被调用的，则这个err会传递给完工回调（即onComplete）。Async.js只负责保证onComplete只被调用一次，而不管是因首次出错而调用，还是因成功完成所有操作而调用。

```javascript
function onComplete(err) {
  if (err) throw err;
  console.log(concatenation);
}
```

Node的错误处理约定对Async.js数据收集方法而言也许并不理想，但对于Async.js的6所有其他方法而言，
遵守这些约定可以让错误干净利落地从各个任务流向完工回调。下一节会看到更多这样的例子。

## Async.js的任务组织技术

Async.js的数据收集方法解决了一个异步函数如何运用于一个数据集的问题。但如果是一个函数集而不是一个数据集，又该怎么办呢？本节将探讨Async.js中一些可以派发异步函数并收集其结果的强大工具。

### 异步函数序列的运行

假设我们希望某一组异步函数能依次运行。在不使用工具函数的情况下，可能会编写出类似这样的代码：

```javascript
funcs[0](function () {
    funcs[1](function () {
        funcs[2](onComplete);
    })
});
```

- async.series
- async.waterfall


这两个方法均接受一组函数(任务列表),并按照顺序运行。
二者给任务列表中的每个函数均传递一个Node风格的回调。

前者提供给各个任务的只有回调，而后者还会提供任务列表中前一任务的结果。（所谓“结果”，指的是各个任务传递给其回调的非错误的值。）

```javascript
var async = require('async'); 
var start = new Date;
async.series([function (callback) {
        setTimeout(callback, 100);
    },
    function (callback) {
        setTimeout(callback, 300);
    },
    function (callback) {
        setTimeout(callback, 200);
    }
], function (err, results) {
    console.log('Completed in ' + (new Date - start) + 'ms');
});
```

（将async.series替换为async.waterfall不会对这个例子造成任何影响，因为这里各个任务的回调在运行时均不带参数。）

因为任务列表中的各个任务会按顺序完成，所以会在600毫秒之后（实际上比600毫秒稍长一些）运行完工回调（即因完成整个工作流事件而调用的回调，又称完工事件处理器）。Async.js传递给任务列表中每个函数的回调好像在说：“出错了吗（回调的首参数是否为错误）？如果没出错，我就要收集结果（回调的次参数）并运行下一个任务了。

### 异步函数的并行运行

Async.js提供了async.series的并行版本，即async.paral-lel。就像async.series一样，async.parallel也接受一组形为function(callback) {...}的函数作为参数，但会再加上一个（可选的）在触发最末回调后运行的完工事件处理器。

```javascript
var async = require('async'); 
var start = new Date;
async.parallel([function (callback) {
    setTimeout(callback, 100);
}, function (callback) {
    setTimeout(callback, 300);
}, function (callback) {
    setTimeout(callback, 200);
}], function (err, results) {
    console.log('Completed in ' + (new Date - start) + 'ms');
});
```

**注意:** async.series完成工作流需要用掉3次延时的总和（约600毫秒），而async.parallel的用时只是最长的那次延时（约300毫秒）。更为便利的是，**Async.js按照任务列表的次序向完工事件处理器传递结果，而不是按照生成这些结果的次序**。这样，我们既
拥有了并行机制的性能优势，又没有失去结果的可预知性。

## 异步的工作流排队技术

大多数情况下，前两节介绍的那些简单方法足以解决我们的异步窘境，但async.series和async.parallel均存在各自的局限性。

- 任务列表是静态的。一旦调用了async.series或async.par-allel，就再也不能增减任务了。   
- 不可能问：“已经完成多少任务了？”任务处于黑箱状态，除非我们自行从任务内部派发更新信息。
- 只有两个选择，要么是完全没有并发性，要么是不受限制的并发性。这对文件I/O任务可是个大问题。如果要操作上千个文件，当然不想因按顺序操作而效率低下，但如果试着并行执行所有操作，又很可能会激怒操作系统。

Async.js提供了一种可以解决上述所有问题的全能方法：**async.queue**:

async.queue的底层基本理念令人想起DMV（DynamicManagement View，动态管理视图）。它可以同时应对很多人（最多时等于在岗办事员的数目），但并不是每位办事员前面各排一个队，而是维持着一个排号队列。人到了就排队，并取得一个排队号码。任何一个办事员空闲时，就会叫下一个排队号码。

async.queue的接口比async.series和async.parallel稍微复杂一些。async.queue接受的参数有两个：一个是worker（办事员）函数，而不是一个函数列表；一个是代表着concurrency（并发度）的值，代表了办事员最多可同时处理的任务数。async.queue的返回值是一个队列，我们可以向这个队列推入任意的任务数据及可选的回调。

```javascript
var async = require('async');

function worker(data, callback) {
    console.log(data);
    callback();
}
var concurrency = 2;
var queue = async.queue(worker, concurrency);
queue.push(1)
queue.push(2)
queue.push(3)
```

不过内在还是有点小区别：并发度为2时，需要两轮才能遍历事件队列；如果并发度为1，则需要3轮才能遍历，每轮输出一行代码；如果并发度为3或更大的值，则只需要1轮即可遍历。并发度为0的队列不会做任何事情。如果想要最大的并发度，请直接使用Infinity关键字。

### 任务的入列
虽然queue.push与[].push同名，但二者存在两个很关键的差别。

```javascript
queue.push([1, 2, 3]);
```

等价于

```javascript
queue.push(1);
queue.push(2);
queue.push(3);
```

这意味着不能直接使用数组作为任务的数据。不过可以使用其他任何东西（甚至函数）作为任务的数据。事实上，如果想让async.queue像async.series/async.parallel那样也使用一组函数作为任务列表，只需定义一个其次参数会直接传递给其首参数的worker函数。

```javascript
function worker(task, callback) { 
  task(callback);
}
var concurrency = 2;
var queue = async.queue(worker, concurrency);
queue.push(tasks);
```

第二个差别。async.queue中的每次push调用可附带提供一个回调函数。如果提供了，该回调函数会直接送给worker函数作为其回调参数。因此，（假设worker函数确实运行了其回调，即它未因抛出错误而直接关停）下面这个例子将会触发3次输出事件，即输出3次'Task complete!'。

```javascript
queue.push([1, 2, 3], function(err, result) {            console.log('Task complete!');
});
```

对async.queue而言，push方法的回调函数非常重要，因为async.queue不像async.series/async.parallel那样可以在内部存储每次任务的结果。如果想要这些结果，就必须自行去捕获。

### 完工事件的处理

和async.series及其类似方法一样，我们也可以给async.queue指定一个完工事件处理器。不过，这时并不是传递完工事件处理器作为async.queue方法的参数，而是要附加它作为async.queue对象的drain（排空）属性。

```javascript
var async = require('async');
function worker(data, callback) {
    setTimeout(callback, data);
}
var concurrency = 2;
var queue = async.queue(worker, concurrency);
var start = new Date;
queue.drain = function () {
    console.log('Completed in ' + (new Date - start) + 'ms');
};
queue.push([100, 300, 200]);
```

回想一下：async.series完成工作流需要大约600毫秒的时间（3次延时的总和），而async.parallel只用掉约300毫秒,。这里的并发度为2，所以工作流一开始就会并行运行前两次延时。不过结束运行那次100毫秒的延时之后，队列里的下一个任务（即200毫秒的延时）将会立即开始运行。因此，在这种情况下，async.queue和async.parallel差不多在同一时刻结束运行。这里工作流的次序起到了关键作用：如果第3次入列的是那个300毫秒的延时任务，则整个队列需用时约400毫秒才能完成。

遗憾的是，这意味着async.queue不能像async.waterfall那样提供清晰排序的结果。如果想收集那些入列的任务的结果数据，就只能靠自己了。

### 队列的高级回调方法

尽管drain常常是我们唯一要用到的事件处理器，但async.queue还是提供了其他一些事件及其处理器。

- 队末任务开始运行时，会调用队列的empty方法。（队末任务运行结束时，会调用队列的drain方法。）
- 达到并发度的上限时，会调用队列的saturated方法。
- 如果提供了一个函数作为push方法的次参数，则在结束运行给定任务时会调用该函数，或在给定任务列表中的每个任务结束运行时均调用一次该函数。

## 极简主义者Step 工作流控制

[Step](https://github.com/creationix/step)