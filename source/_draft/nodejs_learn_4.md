---
title: 一起学nodejs(Stream)
category: js
tag: nodejs
---

## Stream

- [stream-handbook](https://github.com/jabez128/stream-handbook)
- [中文版Stream介绍](http://nodejs.cn/api/stream.html)
- [Steam 官网介绍](https://nodejs.org/docs/latest/api/stream.html#stream_stream)
- [淘宝FED团队 深入理解 Node.js Stream 内部机制](http://taobaofed.org/blog/2017/08/31/nodejs-stream/)
- [node-js-streams-everything-you-need-to-know](https://medium.freecodecamp.org/node-js-streams-everything-you-need-to-know-c9141306be93)


Node.js 中有四种基本的流类型：

- Readable - 可读的流 (例如 fs.createReadStream()).
- Writable - 可写的流 (例如 fs.createWriteStream()).
- Duplex - 可读写的流 (例如 net.Socket).
- Transform - 在读写过程中可以修改和变换数据的 Duplex 流 (例如 zlib.createDeflate()).

> Writable 和 Readable 流都会将数据存储到内部的缓冲器（buffer）中。这些缓冲器可以 通过相应的 `writable._writableState.getBuffer()` 或 `readable._readableState.buffer` 来获取。

> 当Readble流实现调用`stream.push(chunk)`方法时,数据被放到缓存器中。如果流的消费者没有调用`stream.read()`方法,这些数据会存在于内部队列中，直到被消费.
> 当内部可读的缓存器达到`highWaterMark`指定的阈值时，流会暂停从底层资源读取数据，直到当前缓冲期的数据被消费(也就是说，流会在内部停止调用`readable._read()`来填充缓存器)

> Writable流可通过反复调用`writable.write(chunk)`方法将数据放到缓存器。当内部可写缓存器中的总大小小于`highWaterMark`指定的阈值时，调用writable.write()将返回`true`,一旦内部缓存器的大小达到或超过`highWaterMark`,调用`writable.write(chunk)`将返回`false`.
> stream API 的关键目标， 尤其对于 `stream.pipe()` 方法， 就是限制缓冲器数据大小，以达到可接受的程度。这样，对于读写速度不匹配的源头和目标，就不会超出可用的内存大小。

> `Duplex` 和 `Transform` 都是可读写的。 在内部，它们都维护了 两个 相互独立的缓冲器用于读和写。 在维持了合理高效的数据流的同时，也使得对于读和写可以独立进行而互不影响。 例如， `net.Socket` 就是 `Duplex` 的实例，它的可读端可以消费从套接字（socket）中接收的数据， 可写端则可以将数据写入到套接字。 由于数据写入到套接字中的速度可能比从套接字接收数据的速度快或者慢， 在读写两端使用独立缓冲器，并进行独立操作就显得很重要了。

http 服务器代码实例，如下:

```javascript
const http = require('http');
const server = http.createServer((req, res) => {
  // req 是 http.IncomingMessage 的实例，这是一个 Readable Stream
  // res 是 http.ServerResponse 的实例，这是一个 Writable Stream
  let body = '';
  // 接收数据为 utf8 字符串，
  // 如果没有设置字符编码，将接收到 Buffer 对象。
  req.setEncoding('utf8');
  // 如果监听了 'data' 事件，Readable streams 触发 'data' 事件 
  req.on('data', (chunk) => {
    body += chunk;
  });
  // end 事件表明整个 body 都接收完毕了 
  req.on('end', () => {
    try {
      S
      // 发送一些信息给用户
      for(let i=0;i<100000000;i){
        data+=i;
      }
      res.write(data);
      res.end();
    } catch (er) {
      // json 数据解析失败 
      res.statusCode = 400;
      return res.end(`error: ${er.message}`);
    }
  });
});
server.listen(1337);

// $ curl localhost:1337 -d "{}"
// object
// $ curl localhost:1337 -d "\"foo\""
// string
// $ curl localhost:1337 -d "not json"
// error: Unexpected token o in JSON at position 1
```

### Writable流

Writable 的例子包括：(所有 Writable 流都实现了 stream.Writable 类定义的接口。)

- HTTP requests, on the client
- HTTP responses, on the server
- fs write streams
- zlib streams
- crypto streams
- TCP sockets
- child process stdin
- process.stdout, process.stderr