---
title: 一起学nodejs(Buffer)
category: js
tag: nodejs
---
## Buffer 对象

官网是这么说的:
> 在 ECMAScript 2015 (ES6) 引入 TypedArray 之前，JavaScript 语言没有读取或操作二进制数据流的机制。 Buffer 类被引入作为 Node.js API 的一部分，使其可以在 TCP 流或文件系统操作等场景中处理二进制数据流。
> TypedArray 现已被添加进 ES6 中，Buffer 类以一种更优化、更适合 Node.js 用例的方式实现了 Uint8Array API。
> Buffer 类的实例类似于**整数数组**，但 Buffer 的**大小是固定**的、且在 V8**堆外分配物理内存**。 Buffer 的大小在被创建时确定，且**无法调整大小**。

<!--more-->
在 Node.js 中，Buffer 类是随 Node 内核一起发布的核心库。Buffer 库为 Node.js 带来了一种存储原始数据的方法，可以让 Node.js 处理二进制数据，每当需要在 Node.js 中处理I/O操作中移动的数据时，就有可能使用 Buffer 库。原始数据存储在 Buffer 类的实例中。一个 Buffer 类似于一个整数数组，但它对应于 V8 堆内存之外的一块**原始内存**。

说明：

1. 在 **Node.js v6** 之前的版本中，Buffer 实例是通过 Buffer 构造函数创建的，它根据提供的参数返回不同的 Buffer
1. 为了使 Buffer 实例的创建更可靠、更不容易出错，各种 new Buffer() 构造函数已被 废弃，并由 **Buffer.from()**、**Buffer.alloc()**、和 **Buffer.allocUnsafe()** 方法替代。

这里说到了[ES6 TypeArray](http://es6.ruanyifeng.com/#docs/arraybuffer)

```javascript
const testType = Buffer.alloc(1,255);
const testType2 = Buffer.alloc(1,256);
console.log(testType)
console.log(testType2)
// <Buffer ff>
// <Buffer 00> ，用256 填充的时候溢出了
//说明Buffer使用的确实是不带符号整数`Uint8`视图类型的TypedArray
```

### 关于 UnSafe 官网的说明

> 当调用 Buffer.allocUnsafe() 和 Buffer.allocUnsafeSlow() 时，被分配的内存段是未初始化的（没有用 0 填充）。 虽然这样的设计使得内存的分配非常快，但已分配的内存段可能包含潜在的敏感旧数据。 使用通过 Buffer.allocUnsafe() 创建的没有被完全重写内存的 Buffer ，在 Buffer 内存可读的情况下，可能泄露它的旧数据。
Node.js 可以在一开始就使用 **--zero-fill-buffers** 命令行启动选项强制所有创建时自动用 0 填充。

### 类方法关键API

#### Buffer.alloc() 方法

Buffer.alloc(size[, fill[, encoding]])

- size `integer` 新建的 Buffer 期望的长度
- fill `string` | `Buffer` | `integer` 用来预填充新建的 Buffer 的值。 默认: 0
- encoding `string` 如果 fill 是字符串，则该值是它的字符编码。 默认: 'utf8'

```javascript
// 不指定,默认用0填充
const buf = Buffer.alloc(5);
// 输出: <Buffer 00 00 00 00 00>
console.log(buf);

//初始化一个Buffer 用某个字符填充
const buf1 = Buffer.alloc(5, 'a');
// 输出: <Buffer 61 61 61 61 61>
console.log(buf1);

//初始化一个Buffer 用另一个Buffer填充,默认只取填充buffer的第一个字节
const buf3 = Buffer.alloc(5, Buffer.alloc(1,255));
// 输出 <Buffer ff ff ff ff ff>
console.log(buf3);
// 输入 5 
console.log(buf3.length);
// 输入 5 
console.log(Buffer.byteLength(buf3));

// 'Hello World' base64 后的字符串表示,第二个参数是编码后的字符串，
const buf2 = Buffer.alloc(11, 'aGVsbG8gd29ybGQ=', 'base64');
// 输出: <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
console.log(buf2);
```

#### Buffer.from(string[, encoding] 与 Buffer.from(buffer)

- string `string` 要编码的字符串
- encoding `string` string 的字符编码。 默认: 'utf8'
- buffer `Buffer` 一个要拷贝数据的已存在的 Buffer

```javascript
// 从一个Buffer 拷贝一个Buffer
const bufFrom1 = Buffer.from('buffer');
const bufFrom2 = Buffer.from(bufFrom1);
bufFrom1[0] = 0x61; //'a'
// 输出: auffer
console.log(bufFrom1.toString());
// 输出: buffer
console.log(bufFrom2.toString());

const bufFrom3= Buffer.from('this is a tést');
// 输出: this is a test
console.log(bufFrom3.toString());
// 输出: this is a tC)st
console.log(bufFrom3.toString('ascii'));

const bufFrom4= Buffer.from('aGVsbG8gd29ybGQ=', 'base64');
// 输出: Hello world
console.log(bufFrom4.toString());
```

#### Buffer.allocUnsafe() 

```javascript
const str = 'Node.js';
const bufUnsafe = Buffer.allocUnsafe(str.length);
// 输出: 不确定
console.log(bufUnsafe.toString('ascii'));
for (let i = 0; i < str.length; i++) {
  bufUnsafe[i] = str.charCodeAt(i);
}
// 输出: Node.js
console.log(bufUnsafe.toString('ascii'));
```

> 注意，Buffer 模块会预分配一个大小为 Buffer.poolSize 的内部 Buffer 实例作为快速分配池， 用于使用 Buffer.allocUnsafe() 新创建的 Buffer 实例，
仅限于当 size 小于或等于 Buffer.poolSize 除以2后的最大整数值.
> 对这个预分配的内部内存池的使用，是调用 Buffer.alloc(size, fill) 和 Buffer.allocUnsafe(size).fill(fill) 的关键区别。 具体地说，Buffer.alloc(size, fill) 永远不会使用这个内部的 Buffer 池，但如果 size 小于或等于 Buffer.poolSize 的一半， Buffer.allocUnsafe(size).fill(fill) 会使用这个内部的 Buffer 池。

实例池的大小默认为: 8192(8k),可以修改,值为Buffer类的一个属性

```javascript
Buffer.poolSize = 9*1024;
```

### 其他的类方法

```javascript
/* 输出: ½ + ¼ = ¾: 9 个字符, 12 个字节 */
const bufOther3 = '\u00bd + \u00bc = \u00be';
console.log(Buffer.byteLength(bufOther3));//11
console.log(Buffer.byteLength(bufOther3,'utf-8'));//11

const bufTo = Buffer.concat([bufC1, bufC2, bufC3]); //buffer拼接
const bufTo1 = Buffer.concat([bufC1, bufC2, bufC3], totalLength);
const bufTo2 = Buffer.concat([bufC1, bufC2, bufC3],41);
console.log(bufTo);
console.log(bufTo1);//42
console.log(bufTo2);//42
//<Buffer 61 61 61 61 61 61 61 61 61 61 ff ff ff ff ff ff ff ff ff ff ff ff ff ff 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00>
//<Buffer 61 61 61 61 61 61 61 61 61 61 ff ff ff ff ff ff ff ff ff ff ff ff ff ff 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00>
//<Buffer 61 61 61 61 61 61 61 61 61 61 ff ff ff ff ff ff ff ff ff ff ff ff ff ff 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00>

```

### 实例方法

- buf.copy

```javascript
const bufCopy = Buffer.allocUnsafe(26);
const bufCopy2 = Buffer.allocUnsafe(26).fill('!');
for (let i = 0; i < 26; i++) {
  // 97 是 'a' 的十进制 ASCII 值
  bufCopy[i] = i + 97;
}
//拷贝目标，目标开始位，源buffer的开始位，源buffer的结束位
bufCopy.copy(bufCopy2, 8, 16, 20);

// abcdefghijklmnopqrstuvwxy
console.log(bufCopy.toString('ascii', 0, 25));
// 输出: !!!!!!!!qrst!!!!!!!!!!!!!
console.log(bufCopy2.toString('ascii', 0, 25));
```

- buf.entries() //创建并返回一个[index,byte] 迭代器

```javascript
const buf = Buffer.from('buffer');
// 输出:
//   [0, 98]
//   [1, 117]
//   [2, 102]
//   [3, 102]
//   [4, 101]
//   [5, 114]
for (const pair of buf.entries()) {
  console.log(pair);
}
```

- buf.keys() // 创建并返回一个包含 buf 键名（索引）的迭代器。

```javascript
const buf = Buffer.from('buffer');

// 输出:
//   0
//   1
//   2
//   3
//   4
//   5
for (const key of buf.keys()) {
  console.log(key);
}
```

- buf.values()

```javascript

const buf = Buffer.from('buffer');
// 输出:
//   98
//   117
//   102
//   102
//   101
//   114
for (const value of buf.values()) {
  console.log(value);
}
```

- buffer.transcode(source, fromEnc, toEnc)

```javascript
const buffer = require('buffer');
const newBuf = buffer.transcode(Buffer.from('€'), 'utf8', 'ascii');
console.log(newBuf.toString('ascii'));
// 输出: '?'
```

- buf.includes(value[, byteOffset][, encoding])
- buf.indexOf(value[, byteOffset][, encoding])
- buf.lastIndexOf(value[, byteOffset][, encoding])

### Buffer 目前支持的字符编码包括：

ascii - 仅支持 7 位 ASCII 数据。如果设置去掉高位的话，这种编码是非常快的。
utf8 - 多字节编码的 Unicode 字符。许多网页和其他文档格式都使用 UTF-8 。
utf16le - 2 或 4 个字节，小字节序编码的 Unicode 字符。支持代理对（U+10000 至 U+10FFFF）。
ucs2 - utf16le 的别名。
base64 - Base64 编码。
latin1 - 一种把 Buffer 编码成一字节编码的字符串的方式。
binary - latin1 的别名。
hex - 将每个字节编码为两个十六进制字符。
