---
title: 一起学nodejs(Buffer与Stream)
category: js
tag: nodejs
---

## Buffer 对象

官网是这么说的:
> 在 ECMAScript 2015 (ES6) 引入 TypedArray 之前，JavaScript 语言没有读取或操作二进制数据流的机制。 Buffer 类被引入作为 Node.js API 的一部分，使其可以在 TCP 流或文件系统操作等场景中处理二进制数据流。
> TypedArray 现已被添加进 ES6 中，Buffer 类以一种更优化、更适合 Node.js 用例的方式实现了 Uint8Array API。
> Buffer 类的实例类似于**整数数组**，但 Buffer 的**大小是固定**的、且在 V8**堆外分配物理内存**。 Buffer 的大小在被创建时确定，且**无法调整大小**。


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

官网的例子:

```javascript
// 创建一个长度为 10、且用 0 填充的 Buffer。
const buf1 = Buffer.alloc(10);

// 创建一个长度为 10、且用 0x1 填充的 Buffer。
const buf2 = Buffer.alloc(10, 1);


// 创建一个长度为 10、且未初始化的 Buffer。
// 这个方法比调用 Buffer.alloc() 更快，
// 但返回的 Buffer 实例可能包含旧数据，
// 因此需要使用 fill() 或 write() 重写。
const buf3 = Buffer.allocUnsafe(10);

// 创建一个包含 [0x1, 0x2, 0x3] 的 Buffer。
const buf4 = Buffer.from([1, 2, 3]);

// 创建一个包含 UTF-8 字节 [0x74, 0xc3, 0xa9, 0x73, 0x74] 的 Buffer。
const buf5 = Buffer.from('test');

// 创建一个包含 Latin-1 字节 [0x74, 0xe9, 0x73, 0x74] 的 Buffer。
const buf6 = Buffer.from('test', 'latin1');

const buf7 = Buffer.alloc(11, 'aGVsbG8gd29ybGQ=', 'base64');

```

### 关于 UnSafe 官网的说明

> 当调用 Buffer.allocUnsafe() 和 Buffer.allocUnsafeSlow() 时，被分配的内存段是未初始化的（没有用 0 填充）。 虽然这样的设计使得内存的分配非常快，但已分配的内存段可能包含潜在的敏感旧数据。 使用通过 Buffer.allocUnsafe() 创建的没有被完全重写内存的 Buffer ，在 Buffer 内存可读的情况下，可能泄露它的旧数据。
Node.js 可以在一开始就使用 **--zero-fill-buffers** 命令行启动选项强制所有创建时自动用 0 填充。

### Buffer 目前支持的字符编码包括：

ascii - 仅支持 7 位 ASCII 数据。如果设置去掉高位的话，这种编码是非常快的。
utf8 - 多字节编码的 Unicode 字符。许多网页和其他文档格式都使用 UTF-8 。
utf16le - 2 或 4 个字节，小字节序编码的 Unicode 字符。支持代理对（U+10000 至 U+10FFFF）。
ucs2 - utf16le 的别名。
base64 - Base64 编码。
latin1 - 一种把 Buffer 编码成一字节编码的字符串的方式。
binary - latin1 的别名。
hex - 将每个字节编码为两个十六进制字符。

### Buffer 初始化方法k

//http://www.runoob.com/nodejs/nodejs-buffer.html

Buffer.alloc(size[, fill[, encoding]])： 返回一个指定大小的 Buffer 实例，如果没有设置 fill，则默认填满 0
Buffer.allocUnsafe(size)： 返回一个指定大小的 Buffer 实例，但是它不会被初始化，所以它可能包含敏感的数据
Buffer.allocUnsafeSlow(size)
Buffer.from(array)： 返回一个被 array 的值初始化的新的 Buffer 实例（传入的 array 的元素只能是数字，不然就会自动被 0 覆盖）
Buffer.from(arrayBuffer[, byteOffset[, length]])： 返回一个新建的与给定的 ArrayBuffer 共享同一内存的 Buffer。
Buffer.from(buffer)： 复制传入的 Buffer 实例的数据，并返回一个新的 Buffer 实例
Buffer.from(string[, encoding])： 返回一个被 string 的值初始化的新的 Buffer 实例