### 什么是 stream 流

流（stream）是 Node.js 中处理流式数据的抽象接口。 `stream` 模块用于构建实现了流接口的对象。

Node.js 提供了多种流对象。 例如，[HTTP 服务器的请求](http://nodejs.cn/s/2RqpEw)和 [`process.stdout`](http://nodejs.cn/s/tQWUzG) 都是流的实例。

流可以是可读的、可写的、或者可读可写的。 所有的流都是 [`EventEmitter`](http://nodejs.cn/s/pGAddE) 的实例。

使用方法如下：

```js
const stream = require('stream');
```

尽管理解流的工作方式很重要，但是 `stream` 模块主要用于开发者创建新类型的流实例。 对于以消费流对象为主的开发者，极少需要直接使用 `stream` 模块。

## 流的类型[#](http://nodejs.cn/api/stream.html#stream_types_of_streams)

[中英对照](http://nodejs.cn/api/stream/types_of_streams.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/types_of_streams.md)

Node.js 中有四种基本的流类型：

- [`Writable`](http://nodejs.cn/s/9JUnJ8) - 可写入数据的流（例如 [`fs.createWriteStream()`](http://nodejs.cn/s/VdSJQa)）。
- [`Readable`](http://nodejs.cn/s/YuDKX1) - 可读取数据的流（例如 [`fs.createReadStream()`](http://nodejs.cn/s/wiVPXD)）。
- [`Duplex`](http://nodejs.cn/s/2iRabr) - 可读又可写的流（例如 [`net.Socket`](http://nodejs.cn/s/wsJ1o1)）。
- [`Transform`](http://nodejs.cn/s/fhVJQM) - 在读写过程中可以修改或转换数据的 `Duplex` 流（例如 [`zlib.createDeflate()`](http://nodejs.cn/s/n6ED45)）。

### 对象模式[#](http://nodejs.cn/api/stream.html#stream_object_mode)

[中英对照](http://nodejs.cn/api/stream/object_mode.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/object_mode.md)

Node.js 创建的流都是运作在字符串和 `Buffer`（或 `Uint8Array`）上。 当然，流的实现也可以使用其它类型的 JavaScript 值（除了 `null`）。 这些流会以“对象模式”进行操作。

当创建流时，可以使用 `objectMode` 选项把流实例切换到对象模式。 将已存在的流切换到对象模式是不安全的。

### 缓冲[#](http://nodejs.cn/api/stream.html#stream_buffering)

[中英对照](http://nodejs.cn/api/stream/buffering.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/buffering.md)

[可写流](http://nodejs.cn/s/9JUnJ8)和[可读流](http://nodejs.cn/s/YuDKX1)都会在内部的缓冲器中存储数据，可以分别使用的 `writable.writableBuffer` 或 `readable.readableBuffer` 来获取。

可缓冲的数据大小取决于传入流构造函数的 `highWaterMark` 选项。 对于普通的流， `highWaterMark` 指定了[字节的总数](http://nodejs.cn/s/K499k3)。 对于对象模式的流， `highWaterMark` 指定了对象的总数。

当调用 [`stream.push(chunk)`](http://nodejs.cn/s/8s3paZ) 时，数据会被缓冲在可读流中。 如果流的消费者没有调用 [`stream.read()`](http://nodejs.cn/s/SpgRaa)，则数据会保留在内部队列中直到被消费。

一旦内部的可读缓冲的总大小达到 `highWaterMark` 指定的阈值时，流会暂时停止从底层资源读取数据，直到当前缓冲的数据被消费 （也就是说，流会停止调用内部的用于填充可读缓冲的 `readable._read()`）。

当调用 [`writable.write(chunk)`](http://nodejs.cn/s/doppiK) 时，数据会被缓冲在可写流中。 当内部的可写缓冲的总大小小于 `highWaterMark` 设置的阈值时，调用 `writable.write()` 会返回 `true`。 一旦内部缓冲的大小达到或超过 `highWaterMark` 时，则会返回 `false`。

`stream` API 的主要目标，特别是 [`stream.pipe()`](http://nodejs.cn/s/Ea2ZNW)，是为了限制数据的缓冲到可接受的程度，也就是读写速度不一致的源头与目的地不会压垮内存。

因为 [`Duplex`](http://nodejs.cn/s/2iRabr) 和 [`Transform`](http://nodejs.cn/s/fhVJQM) 都是可读又可写的，所以它们各自维护着两个相互独立的内部缓冲器用于读取和写入， 这使得它们在维护数据流时，读取和写入两边可以各自独立地运作。 例如，[`net.Socket`](http://nodejs.cn/s/wsJ1o1) 实例是 [`Duplex`](http://nodejs.cn/s/2iRabr) 流，它的可读端可以消费从 socket 接收的数据，而可写端则可以将数据写入到 socket。 因为数据写入到 socket 的速度可能比接收数据的速度快或者慢，所以在读写两端独立地进行操作（或缓冲）就显得很重要了。

## 用于消费流的 API[#](http://nodejs.cn/api/stream.html#stream_api_for_stream_consumers)

[中英对照](http://nodejs.cn/api/stream/api_for_stream_consumers.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/api_for_stream_consumers.md)

几乎所有的 Node.js 应用都在某种程度上使用了流。 下面是一个例子，使用流实现了一个 HTTP 服务器：

```js
const http = require('http');

const server = http.createServer((req, res) => {
  // req 是一个 http.IncomingMessage 实例，它是可读流。
  // res 是一个 http.ServerResponse 实例，它是可写流。

  let body = '';
  // 接收数据为 utf8 字符串，
  // 如果没有设置字符编码，则会接收到 Buffer 对象。
  req.setEncoding('utf8');

  // 如果添加了监听器，则可读流会触发 'data' 事件。
  req.on('data', (chunk) => {
    body += chunk;
  });

  // 'end' 事件表明整个请求体已被接收。 
  req.on('end', () => {
    try {
      const data = JSON.parse(body);
      // 响应信息给用户。
      res.write(typeof data);
      res.end();
    } catch (er) {
      // json 解析失败。
      res.statusCode = 400;
      return res.end(`错误: ${er.message}`);
    }
  });
});

server.listen(1337);

// $ curl localhost:1337 -d "{}"
// object
// $ curl localhost:1337 -d "\"foo\""
// string
// $ curl localhost:1337 -d "not json"
// 错误: Unexpected token o in JSON at position 1
```

[可写流](http://nodejs.cn/s/9JUnJ8)（比如例子中的 `res`）会暴露了一些方法，比如 `write()` 和 `end()` 用于写入数据到流。

当数据可以从流读取时，[可读流](http://nodejs.cn/s/YuDKX1)会使用 [`EventEmitter`](http://nodejs.cn/s/pGAddE) API 来通知应用程序。 从流读取数据的方式有很多种。

[可写流](http://nodejs.cn/s/9JUnJ8)和[可读流](http://nodejs.cn/s/YuDKX1)都通过多种方式使用 [`EventEmitter`](http://nodejs.cn/s/pGAddE) API 来通讯流的当前状态。

[`Duplex`](http://nodejs.cn/s/2iRabr) 流和 [`Transform`](http://nodejs.cn/s/fhVJQM) 流都是可写又可读的。

对于只需写入数据到流或从流消费数据的应用程序，并不需要直接实现流的接口，通常也不需要调用 `require('stream')`。

对于需要实现新类型的流的开发者，可以参阅[用于实现流的API](http://nodejs.cn/s/d2EgSi)章节。

### 可写流[#](http://nodejs.cn/api/stream.html#stream_writable_streams)

[中英对照](http://nodejs.cn/api/stream/writable_streams.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_streams.md)

可写流是对数据要被写入的目的地的一种抽象。

可写流的例子包括：

- [客户端的 HTTP 请求](http://nodejs.cn/s/2F5RHd)
- [服务器的 HTTP 响应](http://nodejs.cn/s/rMXoZ1)
- [fs 的写入流](http://nodejs.cn/s/2uZDVA)
- [zlib 流](http://nodejs.cn/s/duYbh2)
- [crypto 流](http://nodejs.cn/s/FuEfsg)
- [TCP socket](http://nodejs.cn/s/wsJ1o1)
- [子进程 stdin](http://nodejs.cn/s/Su8gEr)
- [`process.stdout`](http://nodejs.cn/s/tQWUzG)、[`process.stderr`](http://nodejs.cn/s/wPv5zY)

上面的一些例子事实上是实现了可写流接口的 [`Duplex`](http://nodejs.cn/s/2iRabr) 流。

所有可写流都实现了 [`stream.Writable`](http://nodejs.cn/s/9JUnJ8) 类定义的接口。

尽管可写流的具体实例可能略有差别，但所有的可写流都遵循同一基本的使用模式，如以下例子所示：

```js
const myStream = getWritableStreamSomehow();
myStream.write('一些数据');
myStream.write('更多数据');
myStream.end('完成写入数据');
```

#### stream.Writable 类[#](http://nodejs.cn/api/stream.html#stream_class_stream_writable)

暂无中英对照[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/class_stream_writable.md)

新增于: v0.9.4

##### 'close' 事件[#](http://nodejs.cn/api/stream.html#stream_event_close)

[中英对照](http://nodejs.cn/api/stream/event_close.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/event_close.md)

新增于: v0.9.4

当流或其底层资源（比如文件描述符）被关闭时触发。 表明不会再触发其他事件，也不会再发生操作。

不是所有可写流都会触发 `'close'` 事件。

##### 'drain' 事件[#](http://nodejs.cn/api/stream.html#stream_event_drain)

[中英对照](http://nodejs.cn/api/stream/event_drain.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/event_drain.md)

新增于: v0.9.4

如果调用 [`stream.write(chunk)`](http://nodejs.cn/s/doppiK) 返回 `false`，则当可以继续写入数据到流时会触发 `'drain'` 事件。

```js
// 向可写流中写入数据一百万次。
// 留意背压（back-pressure）。
function writeOneMillionTimes(writer, data, encoding, callback) {
  let i = 1000000;
  write();
  function write() {
    let ok = true;
    do {
      i--;
      if (i === 0) {
        // 最后一次写入。
        writer.write(data, encoding, callback);
      } else {
        // 检查是否可以继续写入。 
        // 不要传入回调，因为写入还没有结束。
        ok = writer.write(data, encoding);
      }
    } while (i > 0 && ok);
    if (i > 0) {
      // 被提前中止。
      // 当触发 'drain' 事件时继续写入。
      writer.once('drain', write);
    }
  }
}
```

##### 'error' 事件[#](http://nodejs.cn/api/stream.html#stream_event_error)

[中英对照](http://nodejs.cn/api/stream/event_error.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/event_error.md)

新增于: v0.9.4

- [](http://nodejs.cn/s/qZ873x)

当写入数据发生错误时触发。

当触发 `'error'` 事件时，流还未被关闭。

##### 'finish' 事件[#](http://nodejs.cn/api/stream.html#stream_event_finish)

[中英对照](http://nodejs.cn/api/stream/event_finish.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/event_finish.md)

新增于: v0.9.4

调用 [`stream.end()`](http://nodejs.cn/s/nvArK4) 且缓冲数据都已传给底层系统之后触发。

```js
const writer = getWritableStreamSomehow();
for (let i = 0; i < 100; i++) {
  writer.write(`写入 #${i}!\n`);
}
writer.end('写入结尾\n');
writer.on('finish', () => {
  console.error('写入已完成');
});
```

##### 'pipe' 事件[#](http://nodejs.cn/api/stream.html#stream_event_pipe)

[中英对照](http://nodejs.cn/api/stream/event_pipe.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/event_pipe.md)

新增于: v0.9.4

- `src` [](http://nodejs.cn/s/YuDKX1) 通过管道流入到可写流的来源流。

当在可读流上调用 [`stream.pipe()`](http://nodejs.cn/s/Ea2ZNW) 时触发。

```js
const writer = getWritableStreamSomehow();
const reader = getReadableStreamSomehow();
writer.on('pipe', (src) => {
  console.error('有数据正通过管道流入写入器');
  assert.equal(src, reader);
});
reader.pipe(writer);
```

##### 'unpipe' 事件[#](http://nodejs.cn/api/stream.html#stream_event_unpipe)

[中英对照](http://nodejs.cn/api/stream/event_unpipe.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/event_unpipe.md)

新增于: v0.9.4

- `src` [](http://nodejs.cn/s/YuDKX1) [被移除可写流管道](http://nodejs.cn/s/35GqHX)的来源流。

当在可读流上调用 [`stream.unpipe()`](http://nodejs.cn/s/35GqHX) 时触发。

当可读流通过管道流向可写流发生错误时，也会触发 `'unpipe'` 事件。

```js
const writer = getWritableStreamSomehow();
const reader = getReadableStreamSomehow();
writer.on('unpipe', (src) => {
  console.error('已移除可写流管道');
  assert.equal(src, reader);
});
reader.pipe(writer);
reader.unpipe(writer);
```

##### writable.cork()[#](http://nodejs.cn/api/stream.html#stream_writable_cork)

[中英对照](http://nodejs.cn/api/stream/writable_cork.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_cork.md)

新增于: v0.11.2

强制把所有写入的数据都缓冲到内存中。 当调用 [`stream.uncork()`](http://nodejs.cn/s/2Bzt8L) 或 [`stream.end()`](http://nodejs.cn/s/nvArK4) 时，缓冲的数据才会被输出。

当写入大量小块数据到流时，内部缓冲可能失效，从而导致性能下降， `writable.cork()` 主要用于避免这种情况。 对于这种情况，实现了 `writable._writev()` 的流可以用更优的方式对写入的数据进行缓冲。

##### writable.destroy([error])[#](http://nodejs.cn/api/stream.html#stream_writable_destroy_error)

[中英对照](http://nodejs.cn/api/stream/writable_destroy_error.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_destroy_error.md)

新增于: v8.0.0

- `error` [](http://nodejs.cn/s/qZ873x)
- 返回: [](http://nodejs.cn/s/v7Fsu2)

销毁流，并触发 `'error'` 事件且传入 `error` 参数。 调用该方法后，可写流就结束了，之后再调用 `write()` 或 `end()` 都会导致 `ERR_STREAM_DESTROYED` 错误。 实现流时不应该重写这个方法，而是重写 [`writable._destroy()`](http://nodejs.cn/s/7DTgUP)。

##### writable.end([chunk][, encoding][, callback])[#](http://nodejs.cn/api/stream.html#stream_writable_end_chunk_encoding_callback)

[中英对照](http://nodejs.cn/api/stream/writable_end_chunk_encoding_callback.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_end_chunk_encoding_callback.md)

版本历史

- `chunk` [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) | [](http://nodejs.cn/s/6sTGdS) 要写入的数据。 对于非对象模式的流， `chunk` 必须是字符串、 `Buffer`、或 `Uint8Array`。 对于对象模式的流， `chunk` 可以是任何 JavaScript 值，除了 `null`。
- `encoding` [](http://nodejs.cn/s/9Tw2bK) 如果 `chunk` 是字符串，则指定字符编码。
- `callback` [](http://nodejs.cn/s/ceTQa6) 当流结束时的回调函数。
- 返回: [](http://nodejs.cn/s/v7Fsu2)

调用 `writable.end()` 表明已没有数据要被写入[可写流](http://nodejs.cn/s/9JUnJ8)。 可选的 `chunk` 和 `encoding` 参数可以在关闭流之前再写入一块数据。 如果传入了 `callback` 函数，则会做为监听器添加到 [`'finish'`](http://nodejs.cn/s/VBJRc8) 事件。

调用 [`stream.end()`](http://nodejs.cn/s/nvArK4) 之后再调用 [`stream.write()`](http://nodejs.cn/s/doppiK) 会导致错误。

```js
// 先写入 'hello, '，结束前再写入 'world!'。
const fs = require('fs');
const file = fs.createWriteStream('例子.txt');
file.write('hello, ');
file.end('world!');
// 后面不允许再写入数据！
```

##### writable.setDefaultEncoding(encoding)[#](http://nodejs.cn/api/stream.html#stream_writable_setdefaultencoding_encoding)

[中英对照](http://nodejs.cn/api/stream/writable_setdefaultencoding_encoding.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_setdefaultencoding_encoding.md)

版本历史

- `encoding` [](http://nodejs.cn/s/9Tw2bK) 默认的字符编码。
- 返回: [](http://nodejs.cn/s/v7Fsu2)

为[可写流](http://nodejs.cn/s/9JUnJ8)设置默认的 `encoding`。

##### writable.uncork()[#](http://nodejs.cn/api/stream.html#stream_writable_uncork)

[中英对照](http://nodejs.cn/api/stream/writable_uncork.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_uncork.md)

新增于: v0.11.2

将调用 [`stream.cork()`](http://nodejs.cn/s/HbaGHW) 后缓冲的所有数据输出到目标。

当使用 [`writable.cork()`](http://nodejs.cn/s/HbaGHW) 和 `writable.uncork()` 来管理流的写入缓冲时，建议使用 `process.nextTick()` 来延迟调用 `writable.uncork()`。 通过这种方式，可以对单个 Node.js 事件循环中调用的所有 `writable.write()` 进行批处理。

```js
stream.cork();
stream.write('一些 ');
stream.write('数据 ');
process.nextTick(() => stream.uncork());
```

如果一个流上多次调用 [`writable.cork()`](http://nodejs.cn/s/HbaGHW)，则必须调用同样次数的 `writable.uncork()` 才能输出缓冲的数据。

```js
stream.cork();
stream.write('一些 ');
stream.cork();
stream.write('数据 ');
process.nextTick(() => {
  stream.uncork();
  // 数据不会被输出，直到第二次调用 uncork()。
  stream.uncork();
});
```

##### writable.writable[#](http://nodejs.cn/api/stream.html#stream_writable_writable)

暂无中英对照[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_writable.md)

新增于: v0.8.0

- [](http://nodejs.cn/s/jFbvuT)

Is `true` if it is safe to call [`writable.write()`][].

##### writable.writableHighWaterMark[#](http://nodejs.cn/api/stream.html#stream_writable_writablehighwatermark)

[中英对照](http://nodejs.cn/api/stream/writable_writablehighwatermark.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_writablehighwatermark.md)

新增于: v9.3.0

- [](http://nodejs.cn/s/SXbo1v)

返回构造可写流时传入的 `highWaterMark` 的值。

##### writable.writableLength[#](http://nodejs.cn/api/stream.html#stream_writable_writablelength)

[中英对照](http://nodejs.cn/api/stream/writable_writablelength.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_writablelength.md)

新增于: v9.4.0

返回队列中准备被写入的字节数（或对象数）。

##### writable.write(chunk[, encoding][, callback])[#](http://nodejs.cn/api/stream.html#stream_writable_write_chunk_encoding_callback)

[中英对照](http://nodejs.cn/api/stream/writable_write_chunk_encoding_callback.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_write_chunk_encoding_callback.md)

版本历史

- `chunk` [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) | [](http://nodejs.cn/s/6sTGdS) 要写入的数据。  对于非对象模式的流， `chunk` 必须是字符串、 `Buffer` 或 `Uint8Array`。 对于对象模式的流， `chunk` 可以是任何 JavaScript 值，除了 `null`。
- `encoding` [](http://nodejs.cn/s/9Tw2bK) 如果 `chunk` 是字符串，则指定字符编码。
- `callback` [](http://nodejs.cn/s/ceTQa6) 当数据块被输出到目标后的回调函数。
- 返回: [](http://nodejs.cn/s/jFbvuT) 如果流需要等待 `'drain'` 事件触发才能继续写入更多数据，则返回 `false`，否则返回 `true`。

`writable.write()` 写入数据到流，并在数据被完全处理之后调用 `callback`。 如果发生错误，则 `callback` 可能被调用也可能不被调用。 为了可靠地检测错误，可以为 `'error'` 事件添加监听器。

在接收了 `chunk` 后，如果内部的缓冲小于创建流时配置的 `highWaterMark`，则返回 `true` 。 如果返回 `false` ，则应该停止向流写入数据，直到 [`'drain'`](http://nodejs.cn/s/gFdjtJ) 事件被触发。

当流还未被排空时，调用 `write()` 会缓冲 `chunk`，并返回 `false`。 一旦所有当前缓冲的数据块都被排空了（被操作系统接收并传输），则触发 `'drain'` 事件。 建议一旦 `write()` 返回 false，则不再写入任何数据块，直到 `'drain'` 事件被触发。 当流还未被排空时，也是可以调用 `write()`，Node.js 会缓冲所有被写入的数据块，直到达到最大内存占用，这时它会无条件中止。 甚至在它中止之前， 高内存占用将会导致垃圾回收器的性能变差和 RSS 变高（即使内存不再需要，通常也不会被释放回系统）。 如果远程的另一端没有读取数据，TCP 的 socket 可能永远也不会排空，所以写入到一个不会排空的 socket 可能会导致远程可利用的漏洞。

对于 [`Transform`](http://nodejs.cn/s/fhVJQM), 写入数据到一个不会排空的流尤其成问题，因为 `Transform` 流默认会被暂停，直到它们被 pipe 或者添加了 `'data'` 或 `'readable'` 事件句柄。

如果要被写入的数据可以根据需要生成或者取得，建议将逻辑封装为一个[可读流](http://nodejs.cn/s/YuDKX1)并且使用 [`stream.pipe()`](http://nodejs.cn/s/Ea2ZNW)。 如果要优先调用 `write()`，则可以使用 [`'drain'`](http://nodejs.cn/s/gFdjtJ) 事件来防止背压与避免内存问题:

```js
function write(data, cb) {
  if (!stream.write(data)) {
    stream.once('drain', cb);
  } else {
    process.nextTick(cb);
  }
}

// 在回调函数被执行后再进行其他的写入。
write('hello', () => {
  console.log('完成写入，可以进行更多的写入');
});
```

### 可读流[#](http://nodejs.cn/api/stream.html#stream_readable_streams)

[中英对照](http://nodejs.cn/api/stream/readable_streams.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_streams.md)

可读流是对提供数据的来源的一种抽象。

可读流的例子包括：

- [客户端的 HTTP 响应](http://nodejs.cn/s/2RqpEw)
- [服务器的 HTTP 请求](http://nodejs.cn/s/2RqpEw)
- [fs 的读取流](http://nodejs.cn/s/C3Eioq)
- [zlib 流](http://nodejs.cn/s/duYbh2)
- [crypto 流](http://nodejs.cn/s/FuEfsg)
- [TCP socket](http://nodejs.cn/s/wsJ1o1)
- [子进程 stdout 与 stderr](http://nodejs.cn/s/F2vs59)
- [`process.stdin`](http://nodejs.cn/s/gagmJq)

所有可读流都实现了 [`stream.Readable`](http://nodejs.cn/s/YuDKX1) 类定义的接口。

#### 两种读取模式[#](http://nodejs.cn/api/stream.html#stream_two_reading_modes)

[中英对照](http://nodejs.cn/api/stream/two_reading_modes.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/two_reading_modes.md)

可读流运作于两种模式之一：流动模式（flowing）或暂停模式（paused）。

- 在流动模式中，数据自动从底层系统读取，并通过 [`EventEmitter`](http://nodejs.cn/s/pGAddE) 接口的事件尽可能快地被提供给应用程序。
- 在暂停模式中，必须显式调用 [`stream.read()`](http://nodejs.cn/s/SpgRaa) 读取数据块。

所有[可读流](http://nodejs.cn/s/YuDKX1)都开始于暂停模式，可以通过以下方式切换到流动模式：

- 添加 [`'data'`](http://nodejs.cn/s/8CCPjN) 事件句柄。
- 调用 [`stream.resume()`](http://nodejs.cn/s/Zhf28N)。
- 调用 [`stream.pipe()`](http://nodejs.cn/s/Ea2ZNW)。

可读流可以通过以下方式切换回暂停模式：

- 如果没有管道目标，则调用 [`stream.pause()`](http://nodejs.cn/s/jqytVw)。
- 如果有管道目标，则移除所有管道目标。调用 [`stream.unpipe()`](http://nodejs.cn/s/35GqHX) 可以移除多个管道目标。

只有提供了消费或忽略数据的机制后，可读流才会产生数据。 如果消费的机制被禁用或移除，则可读流会停止产生数据。

为了向后兼容，移除 [`'data'`](http://nodejs.cn/s/8CCPjN) 事件句柄不会自动地暂停流。 如果有管道目标，一旦目标变为 `drain` 状态并请求接收数据时，则调用 [`stream.pause()`](http://nodejs.cn/s/jqytVw) 也不能保证流会保持暂停模式。

如果可读流切换到流动模式，且没有可用的消费者来处理数据，则数据将会丢失。 例如，当调用 `readable.resume()` 时，没有监听 `'data'` 事件或 `'data'` 事件句柄已移除。

添加 [`'readable'`](http://nodejs.cn/s/J6CZGb) 事件句柄会使流自动停止流动，并通过 [`readable.read()`](http://nodejs.cn/s/SpgRaa) 消费数据。 如果 [`'readable'`](http://nodejs.cn/s/J6CZGb) 事件句柄被移除，且存在 [`'data'`](http://nodejs.cn/s/8CCPjN) 事件句柄，则流会再次开始流动。

#### 三种状态[#](http://nodejs.cn/api/stream.html#stream_three_states)

[中英对照](http://nodejs.cn/api/stream/three_states.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/three_states.md)

可读流的两种模式是对发生在可读流中更加复杂的内部状态管理的一种简化的抽象。

在任意时刻，可读流会处于以下三种状态之一：

- `readable.readableFlowing === null`
- `readable.readableFlowing === false`
- `readable.readableFlowing === true`

当 `readable.readableFlowing` 为 `null` 时，没有提供消费流数据的机制，所以流不会产生数据。 在这个状态下，监听 `'data'` 事件、调用 `readable.pipe()`、或调用 `readable.resume()` 都会使 `readable.readableFlowing` 切换到 `true`，可读流开始主动地产生数据并触发事件。

调用 `readable.pause()`、 `readable.unpipe()`、或接收到背压，则 `readable.readableFlowing` 会被设为 `false`，暂时停止事件流动但不会停止数据的生成。 在这个状态下，为 `'data'` 事件绑定监听器不会使 `readable.readableFlowing` 切换到 `true`。

```js
const { PassThrough, Writable } = require('stream');
const pass = new PassThrough();
const writable = new Writable();

pass.pipe(writable);
pass.unpipe(writable);
// readableFlowing 现在为 false。

pass.on('data', (chunk) => { console.log(chunk.toString()); });
pass.write('ok'); // 不会触发 'data' 事件。
pass.resume(); // 必须调用它才会触发 'data' 事件。
```

当 `readable.readableFlowing` 为 `false` 时，数据可能会堆积在流的内部缓冲中。

#### 选择一种接口风格[#](http://nodejs.cn/api/stream.html#stream_choose_one_api_style)

[中英对照](http://nodejs.cn/api/stream/choose_one_api_style.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/choose_one_api_style.md)

可读流的 API 贯穿了多个 Node.js 版本，且提供了多种方法来消费流数据。 开发者通常应该选择其中一种方法来消费数据，不要在单个流使用多种方法来消费数据。 混合使用 `on('data')`、 `on('readable')`、 `pipe()` 或异步迭代器，会导致不明确的行为。

对于大多数用户，建议使用 `readable.pipe()`，因为它是消费流数据最简单的方式。 如果开发者需要精细地控制数据的传递与产生，可以使用 [`EventEmitter`](http://nodejs.cn/s/pGAddE)、 `readable.on('readable')`/`readable.read()` 或 `readable.pause()`/`readable.resume()`。

#### stream.Readable 类[#](http://nodejs.cn/api/stream.html#stream_class_stream_readable)

暂无中英对照[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/class_stream_readable.md)

新增于: v0.9.4

##### 'close' 事件[#](http://nodejs.cn/api/stream.html#stream_event_close_1)

[中英对照](http://nodejs.cn/api/stream/event_close_1.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/event_close_1.md)

新增于: v0.9.4

当流或其底层资源（比如文件描述符）被关闭时触发。 表明不会再触发其他事件，也不会再发生操作。

不是所有可读流都会触发 `'close'` 事件。

##### 'data' 事件[#](http://nodejs.cn/api/stream.html#stream_event_data)

[中英对照](http://nodejs.cn/api/stream/event_data.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/event_data.md)

新增于: v0.9.4

- `chunk` [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/6sTGdS) 数据块。 对于非对象模式的流， `chunk` 可以是字符串或 `Buffer`。 对于对象模式的流， `chunk` 可以是任何 JavaScript 值，除了 `null`。

当流将数据块传送给消费者后触发。 当调用 `readable.pipe()`， `readable.resume()` 或绑定监听器到 `'data'` 事件时，流会转换到流动模式。 当调用 `readable.read()` 且有数据块返回时，也会触发 `'data'` 事件。

如果使用 `readable.setEncoding()` 为流指定了默认的字符编码，则监听器回调传入的数据为字符串，否则传入的数据为 `Buffer`。

```js
const readable = getReadableStreamSomehow();
readable.on('data', (chunk) => {
  console.log(`接收到 ${chunk.length} 个字节的数据`);
});
```

##### 'end' 事件[#](http://nodejs.cn/api/stream.html#stream_event_end)

[中英对照](http://nodejs.cn/api/stream/event_end.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/event_end.md)

新增于: v0.9.4

当流中没有数据可供消费时触发。

`'end'` 事件只有在数据被完全消费掉后才会触发。 要想触发该事件，可以将流转换到流动模式，或反复调用 [`stream.read()`](http://nodejs.cn/s/SpgRaa) 直到数据被消费完。

```js
const readable = getReadableStreamSomehow();
readable.on('data', (chunk) => {
  console.log(`接收到 ${chunk.length} 个字节的数据`);
});
readable.on('end', () => {
  console.log('已没有数据');
});
```

##### 'error' 事件[#](http://nodejs.cn/api/stream.html#stream_event_error_1)

[中英对照](http://nodejs.cn/api/stream/event_error_1.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/event_error_1.md)

新增于: v0.9.4

- [](http://nodejs.cn/s/qZ873x)

当流因底层内部出错而不能产生数据、或推送无效的数据块时触发。

##### 'readable' 事件[#](http://nodejs.cn/api/stream.html#stream_event_readable)

[中英对照](http://nodejs.cn/api/stream/event_readable.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/event_readable.md)

版本历史

当流中有数据可供读取时触发。

```javascript
const readable = getReadableStreamSomehow();
readable.on('readable', function() {
  // 有数据可读取。
  let data;

  while (data = this.read()) {
    console.log(data);
  }
});
```

当到达流数据的尽头时， `'readable'` 事件也会触发，但是在 `'end'` 事件之前触发。

`'readable'` 事件表明流有新的动态：要么有新的数据，要么到达流的尽头。 对于前者，[`stream.read()`](http://nodejs.cn/s/SpgRaa) 会返回可用的数据。 对于后者，[`stream.read()`](http://nodejs.cn/s/SpgRaa) 会返回 `null`。 例如，下面的例子中， `foo.txt` 是一个空文件：

```js
const fs = require('fs');
const rr = fs.createReadStream('foo.txt');
rr.on('readable', () => {
  console.log(`读取的数据: ${rr.read()}`);
});
rr.on('end', () => {
  console.log('结束');
});
```

运行上面的脚本输出如下：

```txt
$ node test.js
读取的数据: null
结束
```

通常情况下， `readable.pipe()` 和 `'data'` 事件的机制比 `'readable'` 事件更容易理解。 处理 `'readable'` 事件可能造成吞吐量升高。

如果同时使用 `'readable'` 事件和 [`'data'`](http://nodejs.cn/s/8CCPjN) 事件，则 `'readable'` 事件会优先控制流，也就是说，当调用 [`stream.read()`](http://nodejs.cn/s/SpgRaa) 时才会触发 `'data'` 事件。 `readableFlowing` 属性会变成 `false`。 当移除 `'readable'` 事件时，如果存在 `'data'` 事件监听器，则流会开始流动，也就是说，无需调用 `.resume()` 也会触发 `'data'` 事件。

##### readable.destroy([error])[#](http://nodejs.cn/api/stream.html#stream_readable_destroy_error)

[中英对照](http://nodejs.cn/api/stream/readable_destroy_error.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_destroy_error.md)

新增于: v8.0.0

- `error` [](http://nodejs.cn/s/qZ873x) 传给 `'error'` 事件的错误对象。
- 返回: [](http://nodejs.cn/s/v7Fsu2)

销毁流，并触发 `'error'` 事件和 `'close'` 事件。 调用后，可读流将释放所有的内部资源，且忽视后续的 `push()` 调用。 实现流时不应该重写这个方法，而是重写 [`readable._destroy()`](http://nodejs.cn/s/2c9czW)。

##### readable.isPaused()[#](http://nodejs.cn/api/stream.html#stream_readable_ispaused)

[中英对照](http://nodejs.cn/api/stream/readable_ispaused.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_ispaused.md)

新增于: v0.11.14

- 返回： [](http://nodejs.cn/s/jFbvuT)

返回可读流当前的操作状态。 主要用于 `readable.pipe()` 底层的机制。 大多数情况下无需直接使用该方法。

```js
const readable = new stream.Readable();

readable.isPaused(); // === false
readable.pause();
readable.isPaused(); // === true
readable.resume();
readable.isPaused(); // === false
```

##### readable.pause()[#](http://nodejs.cn/api/stream.html#stream_readable_pause)

[中英对照](http://nodejs.cn/api/stream/readable_pause.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_pause.md)

新增于: v0.9.4

- 返回: [](http://nodejs.cn/s/v7Fsu2)

使流动模式的流停止触发 [`'data'`](http://nodejs.cn/s/8CCPjN) 事件，并切换出流动模式。 任何可用的数据都会保留在内部缓存中。

```js
const readable = getReadableStreamSomehow();
readable.on('data', (chunk) => {
  console.log(`接收到 ${chunk.length} 字节的数据`);
  readable.pause();
  console.log('暂停一秒');
  setTimeout(() => {
    console.log('数据重新开始流动');
    readable.resume();
  }, 1000);
});
```

如果存在 `'readable'` 事件监听器，则该方法不起作用。

##### readable.pipe(destination[, options])[#](http://nodejs.cn/api/stream.html#stream_readable_pipe_destination_options)

[中英对照](http://nodejs.cn/api/stream/readable_pipe_destination_options.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_pipe_destination_options.md)

新增于: v0.9.4

- `destination` [](http://nodejs.cn/s/9JUnJ8) 数据写入的目标。
- `options` [](http://nodejs.cn/s/jzn6Ao) 选项。
  - `end` [](http://nodejs.cn/s/jFbvuT) 当读取器结束时终止写入器。默认为 `true`。
- 返回: [](http://nodejs.cn/s/9JUnJ8) 目标可写流，如果是 [`Duplex`](http://nodejs.cn/s/2iRabr) 流或 [`Transform`](http://nodejs.cn/s/fhVJQM) 流则可以形成管道链。

绑定可写流到可读流，将可读流自动切换到流动模式，并将可读流的所有数据推送到绑定的可写流。 数据流会被自动管理，所以即使可读流更快，目标可写流也不会超负荷。

例子，将可读流的所有数据通过管道推送到 `file.txt` 文件：

```js
const readable = getReadableStreamSomehow();
const writable = fs.createWriteStream('file.txt');
// readable 的所有数据都推送到 'file.txt'。
readable.pipe(writable);
```

可以在单个可读流上绑定多个可写流。

`readable.pipe()` 会返回目标流的引用，这样就可以对流进行链式地管道操作：

```js
const fs = require('fs');
const r = fs.createReadStream('file.txt');
const z = zlib.createGzip();
const w = fs.createWriteStream('file.txt.gz');
r.pipe(z).pipe(w);
```

默认情况下，当来源可读流触发 [`'end'`](http://nodejs.cn/s/ZgviqU) 事件时，目标可写流也会调用 [`stream.end()`](http://nodejs.cn/s/nvArK4) 结束写入。 若要禁用这种默认行为， `end` 选项应设为 `false`，这样目标流就会保持打开：

```js
reader.pipe(writer, { end: false });
reader.on('end', () => {
  writer.end('结束');
});
```

如果可读流发生错误，目标可写流不会自动关闭，需要手动关闭所有流以避免内存泄漏。

##### readable.read([size])[#](http://nodejs.cn/api/stream.html#stream_readable_read_size)

[中英对照](http://nodejs.cn/api/stream/readable_read_size.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_read_size.md)

新增于: v0.9.4

- `size` [](http://nodejs.cn/s/SXbo1v) 要读取的数据的字节数。
- 返回: [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/334hvC) | [](http://nodejs.cn/s/6sTGdS)

从内部缓冲拉取并返回数据。 如果没有可读的数据，则返回 `null`。 默认情况下， `readable.read()` 返回的数据是 `Buffer` 对象，除非使用 `readable.setEncoding()` 指定字符编码或流处于对象模式。

如果可读的数据不足 `size` 个字节，则返回内部缓冲剩余的数据，如果流已经结束则返回 `null`。

如果没有指定 `size` 参数，则返回内部缓冲中的所有数据。

`readable.read()` 应该只对处于暂停模式的可读流调用。 在流动模式中， `readable.read()` 会自动调用直到内部缓冲的数据完全耗尽。

```js
const readable = getReadableStreamSomehow();
readable.on('readable', () => {
  let chunk;
  while (null !== (chunk = readable.read())) {
    console.log(`接收到 ${chunk.length} 字节的数据`);
  }
});
```

如果 `readable.read()` 返回一个数据块，则 `'data'`事件也会触发。

[`'end'`](http://nodejs.cn/s/ZgviqU) 事件触发后再调用 [`stream.read([size\])`](http://nodejs.cn/s/SpgRaa) 会返回`null`，不会抛出错误。

##### readable.readable[#](http://nodejs.cn/api/stream.html#stream_readable_readable)

暂无中英对照[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_readable.md)

新增于: v0.8.0

- [](http://nodejs.cn/s/jFbvuT)

Is `true` if it is safe to call [`readable.read()`][].

##### readable.readableHighWaterMark[#](http://nodejs.cn/api/stream.html#stream_readable_readablehighwatermark)

[中英对照](http://nodejs.cn/api/stream/readable_readablehighwatermark.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_readablehighwatermark.md)

新增于: v9.3.0

- 返回: [](http://nodejs.cn/s/SXbo1v)

返回构造可读流时传入的 `highWaterMark` 的值。

##### readable.readableLength[#](http://nodejs.cn/api/stream.html#stream_readable_readablelength)

[中英对照](http://nodejs.cn/api/stream/readable_readablelength.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_readablelength.md)

新增于: v9.4.0

- 返回: [](http://nodejs.cn/s/SXbo1v)

返回队列中准备读取的字节数（或对象数）。

##### readable.resume()[#](http://nodejs.cn/api/stream.html#stream_readable_resume)

[中英对照](http://nodejs.cn/api/stream/readable_resume.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_resume.md)

版本历史

- 返回: [](http://nodejs.cn/s/v7Fsu2)

将被暂停的可读流恢复触发 [`'data'`](http://nodejs.cn/s/8CCPjN) 事件，并将流切换到流动模式。

`readable.resume()` 可以用来充分消耗流中的数据，但无需实际处理任何数据：

```js
getReadableStreamSomehow()
  .resume()
  .on('end', () => {
    console.log('到达流的尽头，但无需读取任何数据');
  });
```

当存在 `'readable'` 事件监听器时， `readable.resume()` 不起作用。

##### readable.setEncoding(encoding)[#](http://nodejs.cn/api/stream.html#stream_readable_setencoding_encoding)

[中英对照](http://nodejs.cn/api/stream/readable_setencoding_encoding.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_setencoding_encoding.md)

新增于: v0.9.4

- `encoding` [](http://nodejs.cn/s/9Tw2bK) 字符编码。
- 返回: [](http://nodejs.cn/s/v7Fsu2)

为从可读流读取的数据设置字符编码。

默认情况下没有设置字符编码，流数据返回的是 `Buffer` 对象。 如果设置了字符编码，则流数据返回指定编码的字符串。 例如，调用 `readable.setEncoding('utf-8')` 会将数据解析为 UTF-8 数据，并返回字符串，调用 `readable.setEncoding('hex')` 则会将数据编码成十六进制字符串。

```js
const readable = getReadableStreamSomehow();
readable.setEncoding('utf8');
readable.on('data', (chunk) => {
  assert.equal(typeof chunk, 'string');
  console.log('读取到 %d 个字符的字符串数据', chunk.length);
});
```

##### readable.unpipe([destination])[#](http://nodejs.cn/api/stream.html#stream_readable_unpipe_destination)

[中英对照](http://nodejs.cn/api/stream/readable_unpipe_destination.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_unpipe_destination.md)

新增于: v0.9.4

- `destination` [](http://nodejs.cn/s/9JUnJ8) 要移除管道的可写流。
- 返回: [](http://nodejs.cn/s/v7Fsu2)

解绑之前使用 [`stream.pipe()`](http://nodejs.cn/s/Ea2ZNW) 绑定的可写流。

如果没有指定 `destination`, 则解绑所有管道.

如果指定了 `destination`, 但它没有建立管道，则不起作用.

```js
const fs = require('fs');
const readable = getReadableStreamSomehow();
const writable = fs.createWriteStream('file.txt');
// 可读流的所有数据开始传输到 'file.txt'，但一秒后停止。
readable.pipe(writable);
setTimeout(() => {
  console.log('停止写入 file.txt');
  readable.unpipe(writable);
  console.log('手动关闭文件流');
  writable.end();
}, 1000);
```

##### readable.unshift(chunk)[#](http://nodejs.cn/api/stream.html#stream_readable_unshift_chunk)

[中英对照](http://nodejs.cn/api/stream/readable_unshift_chunk.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_unshift_chunk.md)

版本历史

- `chunk` [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) | [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/6sTGdS) 要推回可读队列的数据块。 对于非对象模式的流， `chunk` 必须是字符串、 `Buffer` 或 `Uint8Array`。 对于对象模式的流， `chunk` 可以是任何值，除了 `null`。

将数据块推回内部缓冲。 可用于以下情景：正被消费中的流需要将一些已经被拉出的数据重置为未消费状态，以便这些数据可以传给其他方。

触发 [`'end'`](http://nodejs.cn/s/ZgviqU) 事件或抛出运行时错误之后，不能再调用 `stream.unshift()`。

使用 `stream.unshift()` 的开发者可以考虑切换到 [`Transform`](http://nodejs.cn/s/fhVJQM) 流。 详见[用于实现流的API](http://nodejs.cn/s/d2EgSi)。

```js
// Pull off a header delimited by \n\n
// use unshift() if we get too much
// Call the callback with (error, header, stream)
const { StringDecoder } = require('string_decoder');
function parseHeader(stream, callback) {
  stream.on('error', callback);
  stream.on('readable', onReadable);
  const decoder = new StringDecoder('utf8');
  let header = '';
  function onReadable() {
    let chunk;
    while (null !== (chunk = stream.read())) {
      const str = decoder.write(chunk);
      if (str.match(/\n\n/)) {
        // 发现头部边界。
        const split = str.split(/\n\n/);
        header += split.shift();
        const remaining = split.join('\n\n');
        const buf = Buffer.from(remaining, 'utf8');
        stream.removeListener('error', callback);
        // 在调用 unshift() 前移除 'readable' 监听器。
        stream.removeListener('readable', onReadable);
        if (buf.length)
          stream.unshift(buf);
        // 现在可以从流中读取消息的主体。
        callback(null, header, stream);
      } else {
        // 继续读取头部。
        header += str;
      }
    }
  }
}
```

##### readable.wrap(stream)[#](http://nodejs.cn/api/stream.html#stream_readable_wrap_stream)

[中英对照](http://nodejs.cn/api/stream/readable_wrap_stream.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_wrap_stream.md)

新增于: v0.9.4

- `stream` [](http://nodejs.cn/s/t73H94) 老版本的可读流。
- 返回: [](http://nodejs.cn/s/v7Fsu2)

在 Node.js v0.10 之前，流没有实现当前定义的所有的流模块 API。（详见[兼容性](http://nodejs.cn/s/yPPNyE)）

当使用老版本的 Node.js 时，只能触发 [`'data'`](http://nodejs.cn/s/8CCPjN) 事件或调用 [`stream.pause()`](http://nodejs.cn/s/jqytVw) 方法，可以使用 `readable.wrap()` 创建老版本的流作为数据源。

现在几乎无需使用 `readable.wrap()`，该方法主要用于老版本的 Node.js 应用和库。

例子：

```js
const { OldReader } = require('./old-api-module.js');
const { Readable } = require('stream');
const oreader = new OldReader();
const myReader = new Readable().wrap(oreader);

myReader.on('readable', () => {
  myReader.read(); // 各种操作。
});
```

##### readable[Symbol.asyncIterator]()[#](http://nodejs.cn/api/stream.html#stream_readable_symbol_asynciterator)

暂无中英对照

新增于: v10.0.0



[稳定性: 1](http://nodejs.cn/api/documentation.html#documentation_stability_index) - 试验



- Returns: [](http://nodejs.cn/s/HnG4ws) to fully consume the stream.

```js
const fs = require('fs');

async function print(readable) {
  readable.setEncoding('utf8');
  let data = '';
  for await (const k of readable) {
    data += k;
  }
  console.log(data);
}

print(fs.createReadStream('file')).catch(console.log);
```

If the loop terminates with a `break` or a `throw`, the stream will be destroyed. In other terms, iterating over a stream will consume the stream fully. The stream will be read in chunks of size equal to the `highWaterMark` option. In the code example above, data will be in a single chunk if the file has less then 64kb of data because no `highWaterMark` option is provided to [`fs.createReadStream()`](http://nodejs.cn/s/wiVPXD).

### 双工流与转换流[#](http://nodejs.cn/api/stream.html#stream_duplex_and_transform_streams)

#### stream.Duplex 类[#](http://nodejs.cn/api/stream.html#stream_class_stream_duplex)

[中英对照](http://nodejs.cn/api/stream/class_stream_duplex.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/class_stream_duplex.md)

版本历史

双工流（Duplex）是同时实现了 [`Readable`](http://nodejs.cn/s/YuDKX1) 和 [`Writable`](http://nodejs.cn/s/9JUnJ8) 接口的流。

`Duplex` 流的例子包括：

- [TCP socket](http://nodejs.cn/s/wsJ1o1)
- [zlib 流](http://nodejs.cn/s/duYbh2)
- [crypto 流](http://nodejs.cn/s/FuEfsg)

#### stream.Transform 类[#](http://nodejs.cn/api/stream.html#stream_class_stream_transform)

[中英对照](http://nodejs.cn/api/stream/class_stream_transform.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/class_stream_transform.md)

新增于: v0.9.4

转换流（Transform）是一种 [`Duplex`](http://nodejs.cn/s/2iRabr) 流，但它的输出与输入是相关联的。 与 [`Duplex`](http://nodejs.cn/s/2iRabr) 流一样， `Transform` 流也同时实现了 [`Readable`](http://nodejs.cn/s/YuDKX1) 和 [`Writable`](http://nodejs.cn/s/9JUnJ8) 接口。

`Transform` 流的例子包括：

- [zlib 流](http://nodejs.cn/s/duYbh2)
- [crypto 流](http://nodejs.cn/s/FuEfsg)

##### transform.destroy([error])[#](http://nodejs.cn/api/stream.html#stream_transform_destroy_error)

[中英对照](http://nodejs.cn/api/stream/transform_destroy_error.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/transform_destroy_error.md)

新增于: v8.0.0

- `error` [](http://nodejs.cn/s/qZ873x)

销毁流，并触发 `'error'` 事件。 调用该方法后，transform 流会释放全部内部资源。 实现者不要重写该方法，而应该实现 [`readable._destroy()`](http://nodejs.cn/s/2c9czW)。 `Transform` 流的 `_destroy()` 方法的默认实现会触发 `'close'` 事件。

### stream.finished(stream, callback)[#](http://nodejs.cn/api/stream.html#stream_stream_finished_stream_callback)

[中英对照](http://nodejs.cn/api/stream/stream_finished_stream_callback.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/stream_finished_stream_callback.md)

新增于: v10.0.0

- `stream` [](http://nodejs.cn/s/t73H94) 可读流或可写流。
- `callback` [](http://nodejs.cn/s/ceTQa6) 通知回调函数。

当流不再可读、可写、发生错误、或提前关闭时，通过该函数获得通知。

```js
const { finished } = require('stream');

const rs = fs.createReadStream('archive.tar');

finished(rs, (err) => {
  if (err) {
    console.error('流发生错误', err);
  } else {
    console.log('流已读取完');
  }
});

rs.resume(); // 开始读取流。
```

主要用于处理流被提前销毁（如 HTTP 请求被中止）等异常情况，此时流不会触发 `'end'` 或 `'finish'` 事件。

`finished` 接口也可以 Promise 化：

```js
const finished = util.promisify(stream.finished);

const rs = fs.createReadStream('archive.tar');

async function run() {
  await finished(rs);
  console.log('流已读取完');
}

run().catch(console.error);
rs.resume(); // 开始读取流。
```

### stream.pipeline(...streams[, callback])[#](http://nodejs.cn/api/stream.html#stream_stream_pipeline_streams_callback)

[中英对照](http://nodejs.cn/api/stream/stream_pipeline_streams_callback.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/stream_pipeline_streams_callback.md)

新增于: v10.0.0

- `...streams` [](http://nodejs.cn/s/t73H94) 要用管道连接的两个或多个流。
- `callback` [](http://nodejs.cn/s/ceTQa6) 通知回调函数。

使用管道连接多个流，并传递错误与完成清理工作，当管道连接完成时通知回调函数。

```js
const { pipeline } = require('stream');
const fs = require('fs');
const zlib = require('zlib');

// 使用 pipeline 接口连接多个流，并在管道连接完成时获得通知。
// 使用 pipeline 可以高效地压缩一个可能很大的 tar 文件：

pipeline(
  fs.createReadStream('archive.tar'),
  zlib.createGzip(),
  fs.createWriteStream('archive.tar.gz'),
  (err) => {
    if (err) {
      console.error('管道连接失败', err);
    } else {
      console.log('管道连接成功');
    }
  }
);
```

`pipeline` 接口也可以 Promise 化：

```js
const pipeline = util.promisify(stream.pipeline);

async function run() {
  await pipeline(
    fs.createReadStream('archive.tar'),
    zlib.createGzip(),
    fs.createWriteStream('archive.tar.gz')
  );
  console.log('管道连接成功');
}

run().catch(console.error);
```

## 用于实现流的 API[#](http://nodejs.cn/api/stream.html#stream_api_for_stream_implementers)

[中英对照](http://nodejs.cn/api/stream/api_for_stream_implementers.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/api_for_stream_implementers.md)

`stream` 模块 API 的设计是为了更容易使用 JavaScript 的原型继承模式来实现流。

流的开发者可以声明一个新的 JavaScript 类并继承四个基本流类中之一（`stream.Writeable`、 `stream.Readable`、 `stream.Duplex` 或 `stream.Transform`），且确保调用了对应的父类构造器:

```js
const { Writable } = require('stream');

class MyWritable extends Writable {
  constructor(options) {
    super(options);
    // ...
  }
}
```

根据所创建的流类型，新的流类必须实现一个或多个特定的方法，如下图所示:

| 用例                               | 类                                       | 需实现的方法                           |
| :--------------------------------- | :--------------------------------------- | :------------------------------------- |
| 只读流                             | [`Readable`](http://nodejs.cn/s/YuDKX1)  | `_read`                                |
| 只写流                             | [`Writable`](http://nodejs.cn/s/9JUnJ8)  | `_write`, `_writev`, `_final`          |
| 可读可写流                         | [`Duplex`](http://nodejs.cn/s/2iRabr)    | `_read`, `_write`, `_writev`, `_final` |
| 对写入的数据进行操作，然后读取结果 | [`Transform`](http://nodejs.cn/s/fhVJQM) | `_transform`, `_flush`, `_final`       |

实现流的代码中不应该调用流的公共方法，因为这些方法是给消费者使用的（详见[用于消费流的API](http://nodejs.cn/s/6tpdeZ)）。 这样做可能会导致使用流的应用程序代码产生不利的副作用。

### 简单的实现[#](http://nodejs.cn/api/stream.html#stream_simplified_construction)

[中英对照](http://nodejs.cn/api/stream/simplified_construction.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/simplified_construction.md)

新增于: v1.2.0

对于简单的案例，构造流可以不依赖继承。 直接创建 `stream.Writable`、 `stream.Readable`、 `stream.Duplex` 或 `stream.Transform` 的实例，并传入对应的方法作为构造函数选项。

```js
const { Writable } = require('stream');

const myWritable = new Writable({
  write(chunk, encoding, callback) {
    // ...
  }
});
```

### 实现可写流[#](http://nodejs.cn/api/stream.html#stream_implementing_a_writable_stream)

[中英对照](http://nodejs.cn/api/stream/implementing_a_writable_stream.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/implementing_a_writable_stream.md)

`stream.Writable` 类可用于实现可写流。

自定义的可写流必须调用 `new stream.Writable([options])` 构造函数并实现 `writable._write()` 方法， `writable._writev()` 方法是可选的。

#### new stream.Writable([options])[#](http://nodejs.cn/api/stream.html#stream_constructor_new_stream_writable_options)

[中英对照](http://nodejs.cn/api/stream/constructor_new_stream_writable_options.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/constructor_new_stream_writable_options.md)

版本历史

- `options` [](http://nodejs.cn/s/jzn6Ao)
  - `highWaterMark` [](http://nodejs.cn/s/SXbo1v) 当调用 [`stream.write()`](http://nodejs.cn/s/doppiK) 开始返回 `false` 时的缓冲大小。 默认为 `16384` (16kb), 对象模式的流默认为 `16`。
  - `decodeStrings` [](http://nodejs.cn/s/jFbvuT) 是否把传入 [`stream._write()`](http://nodejs.cn/s/8MvCPB) 的字符串编码为 `Buffer`，使用的字符编码为调用 [`stream.write()`](http://nodejs.cn/s/doppiK) 时指定的。 默认为 `true`。
  - `defaultEncoding` [](http://nodejs.cn/s/9Tw2bK) 当 [`stream.write()`](http://nodejs.cn/s/doppiK) 的参数没有指定字符编码时默认的字符编码。 默认为 `'utf8'`。
  - `objectMode` [](http://nodejs.cn/s/jFbvuT) 是否可以调用 [`stream.write(anyObj)`](http://nodejs.cn/s/doppiK)。 一旦设为 `true`，则除了字符串、 `Buffer` 或 `Uint8Array`，还可以写入流实现支持的其他 JavaScript 值。 默认为 `false`。
  - `emitClose` [](http://nodejs.cn/s/jFbvuT) 流被销毁后是否触发 `'close'` 事件。 默认为 `true`。
  - `write` [](http://nodejs.cn/s/ceTQa6) 对 [`stream._write()`](http://nodejs.cn/s/8MvCPB) 方法的实现。
  - `writev` [](http://nodejs.cn/s/ceTQa6) 对 [`stream._writev()`](http://nodejs.cn/s/7qpAhU) 方法的实现。
  - `destroy` [](http://nodejs.cn/s/ceTQa6) 对 [`stream._destroy()`](http://nodejs.cn/s/7DTgUP) 方法的实现。
  - `final` [](http://nodejs.cn/s/ceTQa6) 对 [`stream._final()`](http://nodejs.cn/s/ehUmqz) 方法的实现。

例子：

```js
const { Writable } = require('stream');

class MyWritable extends Writable {
  constructor(options) {
    // 调用 stream.Writable() 构造函数。
    super(options);
    // ...
  }
}
```

使用 ES6 之前的语法：

```js
const { Writable } = require('stream');
const util = require('util');

function MyWritable(options) {
  if (!(this instanceof MyWritable))
    return new MyWritable(options);
  Writable.call(this, options);
}
util.inherits(MyWritable, Writable);
```

使用简化的构造函数：

```js
const { Writable } = require('stream');

const myWritable = new Writable({
  write(chunk, encoding, callback) {
    // ...
  },
  writev(chunks, callback) {
    // ...
  }
});
```

#### writable._write(chunk, encoding, callback)[#](http://nodejs.cn/api/stream.html#stream_writable_write_chunk_encoding_callback_1)

[中英对照](http://nodejs.cn/api/stream/writable_write_chunk_encoding_callback_1.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_write_chunk_encoding_callback_1.md)

- `chunk` [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/6sTGdS) 要写入的数据块。 会一直是 `buffer`，除非 `decodeStrings` 选项设为 `false` 或者流处于对象模式。
- `encoding` [](http://nodejs.cn/s/9Tw2bK) 如果 `chunk` 是字符串，则指定字符编码。 如果 `chunk` 是 `Buffer` 或者流处于对象模式，则无视该选项。
- `callback` [](http://nodejs.cn/s/ceTQa6) 当数据块被处理完成后的回调函数。

所有可写流的实现必须提供 [`writable._write()`](http://nodejs.cn/s/8MvCPB) 方法将数据发送到底层资源。

[`Transform`](http://nodejs.cn/s/fhVJQM) 流会提供自身实现的 [`writable._write()`](http://nodejs.cn/s/8MvCPB)。

该函数不能被应用程序代码直接调用。 它应该由子类实现，且只能被内部的 `Writable` 类的方法调用。

无论是成功完成写入还是写入失败出现错误，都必须调用 `callback`。 如果调用失败，则 `callback` 的第一个参数必须是 `Error` 对象。 如果写入成功，则 `callback` 的第一个参数为 `null`。

在 `writable._write()` 被调用之后且 `callback` 被调用之前，所有对 `writable.write()` 的调用都会把要写入的数据缓冲起来。 当调用 `callback` 时，流将会触发 [`'drain'`](http://nodejs.cn/s/gFdjtJ)事件。 如果流的实现需要同时处理多个数据块，则应该实现 `writable._writev()` 方法。

如果在构造函数选项中设置 `decodeStrings` 属性为 `false`，则 `chunk` 会保持原样传入 `.write()`，它可能是字符串而不是 `Buffer`。 这是为了实现对某些特定字符串数据编码的支持。 当 `decodeStrings` 为 `false` 时，则 `encoding` 参数指定字符串的字符编码。 否则，则 `encoding` 不起作用。 `writable._write()` 方法有下划线前缀，因为它是在定义在类的内部，不应该被用户程序直接调用。

#### writable._writev(chunks, callback)[#](http://nodejs.cn/api/stream.html#stream_writable_writev_chunks_callback)

[中英对照](http://nodejs.cn/api/stream/writable_writev_chunks_callback.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_writev_chunks_callback.md)

- `chunks` [](http://nodejs.cn/s/jzn6Ao) 要写入的多个数据块。 每个数据块的格式为`{ chunk: ..., encoding: ... }`。
- `callback` [](http://nodejs.cn/s/ceTQa6) 当全部数据块被处理完成后的回调函数。

该函数不能被应用程序代码直接调用。 该函数应该由子类实现，且只能被内部的 `Writable` 类的方法调用。

`writable._writev()` 能够同时处理多个数据块。 如果实现了该方法，调用该方法时会传入当前缓冲在写入队列中的所有数据块。

`writable._writev()` 方法有下划线前缀，因为它是在定义在类的内部，不应该被用户程序直接调用。

#### writable._destroy(err, callback)[#](http://nodejs.cn/api/stream.html#stream_writable_destroy_err_callback)

[中英对照](http://nodejs.cn/api/stream/writable_destroy_err_callback.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_destroy_err_callback.md)

新增于: v8.0.0

- `err` [](http://nodejs.cn/s/qZ873x) 可能发生的错误。
- `callback` [](http://nodejs.cn/s/ceTQa6) 回调函数。

`_destroy()` 方法会被 [`writable.destroy()`](http://nodejs.cn/s/tLkQ7J) 调用。 它可以被子类重写，但不能直接调用。

#### writable._final(callback)[#](http://nodejs.cn/api/stream.html#stream_writable_final_callback)

[中英对照](http://nodejs.cn/api/stream/writable_final_callback.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/writable_final_callback.md)

新增于: v8.0.0

- `callback` [](http://nodejs.cn/s/ceTQa6) 当结束写入所有剩余数据时的回调函数。

`_final()` 方法不能直接调用。 它应该由子类实现，且只能通过内部的 `Writable` 类的方法调用。

该方法会在流关闭之前被调用，且在 `callback` 被调用后触发 `'finish'` 事件。 主要用于在流结束之前关闭资源或写入缓冲的数据。

#### 写入时的异常处理[#](http://nodejs.cn/api/stream.html#stream_errors_while_writing)

[中英对照](http://nodejs.cn/api/stream/errors_while_writing.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/errors_while_writing.md)

当 `writable._write()` 和 `writable._writev()` 在执行期间发生的错误时，建议调用回调函数并传入错误对象作为第一个参数。 这样 `Writable` 就会触发 `'error'` 事件。 从 `writable._write()` 中抛出错误可能会导致无法预期的结果。 使用回调可以确保对错误进行一致且可预测的处理。

```js
const { Writable } = require('stream');

const myWritable = new Writable({
  write(chunk, encoding, callback) {
    if (chunk.toString().indexOf('a') >= 0) {
      callback(new Error('无效的数据块'));
    } else {
      callback();
    }
  }
});
```

#### 可写流的例子[#](http://nodejs.cn/api/stream.html#stream_an_example_writable_stream)

[中英对照](http://nodejs.cn/api/stream/an_example_writable_stream.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/an_example_writable_stream.md)

以下举例了一个相当简单（并且有点无意义）的自定义的 `Writable` 流的实现。 虽然这个特定的 `Writable` 流的实例没有任何实际的特殊用途，但该示例说明了一个自定义的 [`Writable`](http://nodejs.cn/s/9JUnJ8) 流实例的每个必需元素：

```js
const { Writable } = require('stream');

class MyWritable extends Writable {
  _write(chunk, encoding, callback) {
    if (chunk.toString().indexOf('a') >= 0) {
      callback(new Error('数据块是无效的'));
    } else {
      callback();
    }
  }
}
```

#### 在可写流中解码 buffer[#](http://nodejs.cn/api/stream.html#stream_decoding_buffers_in_a_writable_stream)

[中英对照](http://nodejs.cn/api/stream/decoding_buffers_in_a_writable_stream.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/decoding_buffers_in_a_writable_stream.md)

解码 buffer 是一个常见的任务，例如使用转换流处理字符串输入。 当使用多字节的字符编码（比如 UTF-8）时，这是一个重要的处理。 下面的例子展示了如何使用 `StringDecoder` 和 [`Writable`](http://nodejs.cn/s/9JUnJ8) 解码多字节的字符串。

```js
const { Writable } = require('stream');
const { StringDecoder } = require('string_decoder');

class StringWritable extends Writable {
  constructor(options) {
    super(options);
    this._decoder = new StringDecoder(options && options.defaultEncoding);
    this.data = '';
  }
  _write(chunk, encoding, callback) {
    if (encoding === 'buffer') {
      chunk = this._decoder.write(chunk);
    }
    this.data += chunk;
    callback();
  }
  _final(callback) {
    this.data += this._decoder.end();
    callback();
  }
}

const euro = [[0xE2, 0x82], [0xAC]].map(Buffer.from);
const w = new StringWritable();

w.write('货币: ');
w.write(euro[0]);
w.end(euro[1]);

console.log(w.data); // 货币: €
```

### 实现可读流[#](http://nodejs.cn/api/stream.html#stream_implementing_a_readable_stream)

[中英对照](http://nodejs.cn/api/stream/implementing_a_readable_stream.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/implementing_a_readable_stream.md)

`stream.Readable` 类可用于实现可读流。

自定义的可读流必须调用 `new stream.Readable([options])` 构造函数并实现 `readable._read()` 方法。

#### new stream.Readable([options])[#](http://nodejs.cn/api/stream.html#stream_new_stream_readable_options)

[中英对照](http://nodejs.cn/api/stream/new_stream_readable_options.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/new_stream_readable_options.md)

- `options` [](http://nodejs.cn/s/jzn6Ao)
  - `highWaterMark` [](http://nodejs.cn/s/SXbo1v) 从底层资源读取数据并存储在内部缓冲区中的最大[字节数](http://nodejs.cn/s/K499k3)。 默认为 `16384` (16kb), 对象模式的流默认为 `16`。
  - `encoding` [](http://nodejs.cn/s/9Tw2bK) 如果指定了，则使用指定的字符编码将 buffer 解码成字符串。 默认为 `null`。
  - `objectMode` [](http://nodejs.cn/s/jFbvuT) 流是否可以是一个对象流。 也就是说 [`stream.read(n)`](http://nodejs.cn/s/SpgRaa) 会返回对象而不是 `Buffer`。 默认为 `false`。
  - `read` [](http://nodejs.cn/s/ceTQa6) 对 [`stream._read()`](http://nodejs.cn/s/5hv4Rd) 方法的实现。
  - `destroy` [](http://nodejs.cn/s/ceTQa6) 对 [`stream._destroy()`](http://nodejs.cn/s/2c9czW) 方法的实现。

```js
const { Readable } = require('stream');

class MyReadable extends Readable {
  constructor(options) {
    // 调用 stream.Readable(options) 构造函数。
    super(options);
    // ...
  }
}
```

使用 ES6 之前的语法：

```js
const { Readable } = require('stream');
const util = require('util');

function MyReadable(options) {
  if (!(this instanceof MyReadable))
    return new MyReadable(options);
  Readable.call(this, options);
}
util.inherits(MyReadable, Readable);
```

使用简化的构造函数：

```js
const { Readable } = require('stream');

const myReadable = new Readable({
  read(size) {
    // ...
  }
});
```

#### readable._read(size)[#](http://nodejs.cn/api/stream.html#stream_readable_read_size_1)

[中英对照](http://nodejs.cn/api/stream/readable_read_size_1.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_read_size_1.md)

版本历史

- `size` [](http://nodejs.cn/s/SXbo1v) 要异步读取的字节数。

该函数不能被应用程序代码直接调用。 它应该由子类实现，且只能被内部的 `Readable` 类的方法调用。

所有可读流的实现必须提供 `readable._read()` 方法从底层资源获取数据。

当 `readable._read()` 被调用时，如果从资源读取到数据，则需要开始使用 [`this.push(dataChunk)`](http://nodejs.cn/s/8s3paZ) 推送数据到读取队列。 `_read()` 应该持续从资源读取数据并推送数据，直到 `readable.push()` 返回 `false`。 若想再次调用 `_read()` 方法，则需要恢复推送数据到队列。

一旦 `readable._read()` 被调用，它不会被再次调用，除非调用了 [`readable.push()`](http://nodejs.cn/s/8s3paZ)。 这是为了确保 `readable._read()` 在同步执行时只会被调用一次。

`size` 是可选的参数。 对于读取是一个单一操作的实现，可以使用 `size` 参数来决定要读取多少数据。 对于其他的实现，可以忽略这个参数，只要有数据就提供数据。 不需要等待指定 `size` 字节的数据在调用 [`stream.push(chunk)`](http://nodejs.cn/s/8s3paZ)。

`readable._read()` 方法有下划线前缀，因为它是在定义在类的内部，不应该被用户程序直接调用。

#### readable._destroy(err, callback)[#](http://nodejs.cn/api/stream.html#stream_readable_destroy_err_callback)

[中英对照](http://nodejs.cn/api/stream/readable_destroy_err_callback.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_destroy_err_callback.md)

新增于: v8.0.0

- `err` [](http://nodejs.cn/s/qZ873x) 可能发生的错误。
- `callback` [](http://nodejs.cn/s/ceTQa6) 回调函数。

`_destroy()` 方法会被 [`readable.destroy()`](http://nodejs.cn/s/6VkV74) 调用。 它可以被子类重写，但不能直接调用。

#### readable.push(chunk[, encoding])[#](http://nodejs.cn/api/stream.html#stream_readable_push_chunk_encoding)

[中英对照](http://nodejs.cn/api/stream/readable_push_chunk_encoding.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_push_chunk_encoding.md)

版本历史

- `chunk` [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/ZbDkpm) | [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/334hvC) | [](http://nodejs.cn/s/6sTGdS) 要推入读取队列的数据块。  对于非对象模式的流， `chunk` 必须是字符串、 `Buffer` 或 `Uint8Array`。  对于对象模式的流， `chunk` 可以是任何 JavaScript 值。
- `encoding` [](http://nodejs.cn/s/9Tw2bK) 字符串数据块的字符编码。 必须是有效的 `Buffer` 字符编码，例如 `'utf8'` 或 `'ascii'`。
- 返回: [](http://nodejs.cn/s/jFbvuT) 如果还有数据块可以继续推入，则返回 `true`，否则返回 `false`。

当 `chunk` 是 `Buffer`、 `Uint8Array` 或字符串时， `chunk` 的数据会被添加到内部队列中供流消费。 在没有数据可写入后，给 `chunk` 传入 `null` 表示流的结束（EOF）。

当可读流处在暂停模式时，使用 `readable.push()` 添加的数据可以在触发 [`'readable'`](http://nodejs.cn/s/J6CZGb) 事件时通过调用 [`readable.read()`](http://nodejs.cn/s/SpgRaa) 读取。

当可读流处于流动模式时，使用 `readable.push()` 添加的数据可以通过触发 `'data'` 事件读取。

`readable.push()` 方法被设计得尽可能的灵活。 例如，当需要封装一个带有'暂停/继续'机制与数据回调的底层数据源时，该底层数据源可以使用自定义的可读流实例封装：

```js
// `source` 是一个有 `readStop()` 和 `readStart()` 方法的对象，
// 当有数据时会调用 `ondata` 方法，
// 当数据结束时会调用 `onend` 方法。

class SourceWrapper extends Readable {
  constructor(options) {
    super(options);

    this._source = getLowlevelSourceObject();

    // 每当有数据时，将其推入内部缓冲。
    this._source.ondata = (chunk) => {
      // 如果 push() 返回 `false`，则停止读取。
      if (!this.push(chunk))
        this._source.readStop();
    };

    // 当读取到尽头时，推入 `null` 表示流的结束。
    this._source.onend = () => {
      this.push(null);
    };
  }
  // 当流想推送更多数据时， `_read` 会被调用。
  _read(size) {
    this._source.readStart();
  }
}
```

`readable.push()` 只能被可读流的实现调用，且只能在 `readable._read()` 方法中调用。

对于非对象模式的流，如果 `readable.push()` 的 `chunk` 参数为 `undefined`，则它会被当成空字符串或 buffer。 详见 [`readable.push('')`](http://nodejs.cn/s/yZyoMF)。

#### 读取时的异常处理[#](http://nodejs.cn/api/stream.html#stream_errors_while_reading)

[中英对照](http://nodejs.cn/api/stream/errors_while_reading.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/errors_while_reading.md)

当 `readable._read()` 在执行期间发生的错误时，建议触发 `'error'` 事件而不是抛出错误。 从 `readable._read()` 中抛出错误可能会导致无法预期的结果。 使用`'error'` 事件可以确保对错误进行一致且可预测的处理。

```js
const { Readable } = require('stream');

const myReadable = new Readable({
  read(size) {
    if (checkSomeErrorCondition()) {
      process.nextTick(() => this.emit('error', err));
      return;
    }
    // 各种处理。
  }
});
```

#### 可读流的例子[#](http://nodejs.cn/api/stream.html#stream_an_example_counting_stream)

[中英对照](http://nodejs.cn/api/stream/an_example_counting_stream.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/an_example_counting_stream.md)

下面是一个可读流的简单例子，依次触发读取 1 到 1,000,000：

```js
const { Readable } = require('stream');

class Counter extends Readable {
  constructor(opt) {
    super(opt);
    this._max = 1000000;
    this._index = 1;
  }

  _read() {
    const i = this._index++;
    if (i > this._max)
      this.push(null);
    else {
      const str = String(i);
      const buf = Buffer.from(str, 'ascii');
      this.push(buf);
    }
  }
}
```

### 实现双工流[#](http://nodejs.cn/api/stream.html#stream_implementing_a_duplex_stream)

[中英对照](http://nodejs.cn/api/stream/implementing_a_duplex_stream.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/implementing_a_duplex_stream.md)

[双工流](http://nodejs.cn/s/2iRabr)同时实现了[可读流](http://nodejs.cn/s/YuDKX1)和[可写流](http://nodejs.cn/s/9JUnJ8)，例如 TCP socket 连接。 因为 JavaScript 不支持多重继承，所以使用 `stream.Duplex` 类实现[双工流](http://nodejs.cn/s/2iRabr)（而不是使用 `stream.Readable` 类和 `stream.Writable` 类）。

`stream.Duplex` 类的原型继承自 `stream.Readable` 和寄生自 `stream.Writable`，但是 `instanceof` 对这两个基础类都可用，因为重写了 `stream.Writable` 的 [`Symbol.hasInstance`](http://nodejs.cn/s/D1EDvM)。

自定义的双工流必须调用 `new stream.Duplex([options])` 构造函数并实现 `readable._read()` 和 `writable._write()` 方法。

#### new stream.Duplex(options)[#](http://nodejs.cn/api/stream.html#stream_new_stream_duplex_options)

[中英对照](http://nodejs.cn/api/stream/new_stream_duplex_options.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/new_stream_duplex_options.md)

版本历史

- `options` [](http://nodejs.cn/s/jzn6Ao) 同时传给 `Writable` 和 `Readable` 的构造函数。  * `allowHalfOpen` [](http://nodejs.cn/s/jFbvuT) 如果设为 `false`，则当可读端结束时，可写端也会自动结束。 默认为 `true`。
  - `readableObjectMode` [](http://nodejs.cn/s/jFbvuT) 设置流的可读端为 `objectMode`。 如果 `objectMode` 为 `true`，则不起作用。 默认为 `false`。
  - `writableObjectMode` [](http://nodejs.cn/s/jFbvuT) 设置流的可写端为 `objectMode`。 如果 `objectMode` 为 `true`，则不起作用。 默认为 `false`。
  - `readableHighWaterMark` [](http://nodejs.cn/s/SXbo1v) 设置流的可读端的 `highWaterMark`。 如果已经设置了 `highWaterMark`，则不起作用。
  - `writableHighWaterMark` [](http://nodejs.cn/s/SXbo1v) 设置流的可写端的 `highWaterMark`。 如果已经设置了 `highWaterMark`，则不起作用。

```js
const { Duplex } = require('stream');

class MyDuplex extends Duplex {
  constructor(options) {
    super(options);
    // ...
  }
}
```

使用 ES6 之前的语法：

```js
const { Duplex } = require('stream');
const util = require('util');

function MyDuplex(options) {
  if (!(this instanceof MyDuplex))
    return new MyDuplex(options);
  Duplex.call(this, options);
}
util.inherits(MyDuplex, Duplex);
```

使用简化的构造函数：

```js
const { Duplex } = require('stream');

const myDuplex = new Duplex({
  read(size) {
    // ...
  },
  write(chunk, encoding, callback) {
    // ...
  }
});
```

#### 双工流的例子[#](http://nodejs.cn/api/stream.html#stream_an_example_duplex_stream)

[中英对照](http://nodejs.cn/api/stream/an_example_duplex_stream.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/an_example_duplex_stream.md)

下面是一个双工流的例子，封装了一个可读可写的底层资源对象。

```js
const { Duplex } = require('stream');
const kSource = Symbol('source');

class MyDuplex extends Duplex {
  constructor(source, options) {
    super(options);
    this[kSource] = source;
  }

  _write(chunk, encoding, callback) {
    // 底层资源只处理字符串。
    if (Buffer.isBuffer(chunk))
      chunk = chunk.toString();
    this[kSource].writeSomeData(chunk);
    callback();
  }

  _read(size) {
    this[kSource].fetchSomeData(size, (data, encoding) => {
      this.push(Buffer.from(data, encoding));
    });
  }
}
```

双工流最重要的方面是，可读端和可写端相互独立于彼此地共存在同一个对象实例中。

#### 对象模式的双工流[#](http://nodejs.cn/api/stream.html#stream_object_mode_duplex_streams)

[中英对照](http://nodejs.cn/api/stream/object_mode_duplex_streams.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/object_mode_duplex_streams.md)

对双工流来说，可以使用 `readableObjectMode` 和 `writableObjectMode` 选项来分别设置可读端和可写端的 `objectMode`。

在下面的例子中，创建了一个变换流（双工流的一种），对象模式的可写端接收 JavaScript 数值，并在可读端转换为十六进制字符串。

```js
const { Transform } = require('stream');

// 转换流也是双工流。
const myTransform = new Transform({
  writableObjectMode: true,

  transform(chunk, encoding, callback) {
    // 强制把 chunk 转换成数值。
    chunk |= 0;

    // 将 chunk 转换成十六进制。
    const data = chunk.toString(16);

    // 推送数据到可读队列。
    callback(null, '0'.repeat(data.length % 2) + data);
  }
});

myTransform.setEncoding('ascii');
myTransform.on('data', (chunk) => console.log(chunk));

myTransform.write(1);
// 打印: 01
myTransform.write(10);
// 打印: 0a
myTransform.write(100);
// 打印: 64
```

### 实现转换流[#](http://nodejs.cn/api/stream.html#stream_implementing_a_transform_stream)

[中英对照](http://nodejs.cn/api/stream/implementing_a_transform_stream.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/implementing_a_transform_stream.md)

[转换流](http://nodejs.cn/s/fhVJQM)是一种[双工流](http://nodejs.cn/s/2iRabr)，它会对输入做些计算然后输出。 例如 [zlib](http://nodejs.cn/s/duYbh2) 流和 [crypto](http://nodejs.cn/s/FuEfsg) 流会压缩、加密或解密数据。

输出流的大小、数据块的数量都不一定会和输入流的一致。 例如， `Hash` 流在输入结束时只会输出一个数据块，而 `zlib` 流的输出可能比输入大很多或小很多。`stream.Transform` 类可用于实现了一个[转换流](http://nodejs.cn/s/fhVJQM)。

`stream.Transform` 类继承自 `stream.Duplex`，并且实现了自有的 `writable._write()` 和 `readable._read()` 方法。 自定义的转换流必须实现 [`transform._transform()`](http://nodejs.cn/s/N8nFbP) 方法，[`transform._flush()`](http://nodejs.cn/s/mErApk) 方法是可选的。

当使用转换流时，如果可读端的输出没有被消费，则写入流的数据可能会导致可写端被暂停。

#### new stream.Transform([options])[#](http://nodejs.cn/api/stream.html#stream_new_stream_transform_options)

[中英对照](http://nodejs.cn/api/stream/new_stream_transform_options.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/new_stream_transform_options.md)

- `options` [](http://nodejs.cn/s/jzn6Ao) 同时传给 `Writable` 和 `Readable` 的构造函数。
  - `transform` [](http://nodejs.cn/s/ceTQa6) 对 [`stream._transform()`](http://nodejs.cn/s/N8nFbP) 的实现。
  - `flush` [](http://nodejs.cn/s/ceTQa6) 对 [`stream._flush()`](http://nodejs.cn/s/mErApk) 的实现。

```js
const { Transform } = require('stream');

class MyTransform extends Transform {
  constructor(options) {
    super(options);
    // ...
  }
}
```

使用 ES6 之前的语法：

```js
const { Transform } = require('stream');
const util = require('util');

function MyTransform(options) {
  if (!(this instanceof MyTransform))
    return new MyTransform(options);
  Transform.call(this, options);
}
util.inherits(MyTransform, Transform);
```

使用简化的构造函数：

```js
const { Transform } = require('stream');

const myTransform = new Transform({
  transform(chunk, encoding, callback) {
    // ...
  }
});
```

#### 'finish' 与 'end' 事件[#](http://nodejs.cn/api/stream.html#stream_events_finish_and_end)

[中英对照](http://nodejs.cn/api/stream/events_finish_and_end.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/events_finish_and_end.md)

[`'finish'`](http://nodejs.cn/s/VBJRc8) 事件来自 `stream.Writable` 类，[`'end'`](http://nodejs.cn/s/ZgviqU) 事件来自 `stream.Readable` 类。 当调用了 [`stream.end()`](http://nodejs.cn/s/nvArK4) 并且 [`stream._transform()`](http://nodejs.cn/s/N8nFbP) 处理完全部数据块之后，触发 `'finish'` 事件。 当调用了 [`transform._flush()`](http://nodejs.cn/s/mErApk) 中的回调函数并且所有数据已经输出之后，触发 `'end'` 事件。

#### transform._flush(callback)[#](http://nodejs.cn/api/stream.html#stream_transform_flush_callback)

[中英对照](http://nodejs.cn/api/stream/transform_flush_callback.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/transform_flush_callback.md)

- `callback` [](http://nodejs.cn/s/ceTQa6) 当剩余的数据被 flush 后的回调函数。

该函数不能被应用程序代码直接调用。 它应该由子类实现，且只能被内部的 `Readable` 类的方法调用。

某些情况下，转换操作可能需要在流的末尾发送一些额外的数据。 例如， `zlib` 压缩流时会储存一些用于优化输出的内部状态。 当流结束时，这些额外的数据需要被 flush 才算完成压缩。

自定义的[转换流](http://nodejs.cn/s/fhVJQM)的 `transform._flush()` 方法是可选的。 当没有更多数据要被消费时，就会调用这个方法，但如果是在 [`'end'`](http://nodejs.cn/s/ZgviqU) 事件被触发之前调用则会发出可读流结束的信号。

在 `transform._flush()` 的实现中， `readable.push()` 可能会被调用零次或多次。 当 flush 操作完成时，必须调用 `callback` 函数。

`transform._flush()` 方法有下划线前缀，因为它是在定义在类的内部，不应该被用户程序直接调用。

#### transform._transform(chunk, encoding, callback)[#](http://nodejs.cn/api/stream.html#stream_transform_transform_chunk_encoding_callback)

[中英对照](http://nodejs.cn/api/stream/transform_transform_chunk_encoding_callback.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/transform_transform_chunk_encoding_callback.md)

- `chunk` [](http://nodejs.cn/s/6x1hD3) | [](http://nodejs.cn/s/9Tw2bK) | [](http://nodejs.cn/s/6sTGdS) 要转换的数据块。 `chunk` 总会是一个 buffer，除非 `decodeStrings` 选项设为 `false` 或者流处在对象模式。
- `encoding` [](http://nodejs.cn/s/9Tw2bK) 如果 `chunk` 是字符串，则 `encoding` 是字符串的字符编码。 如果 `chunk` 是 buffer，则 `encoding` 的值为 'buffer'。
- `callback` [](http://nodejs.cn/s/ceTQa6) 当 `chunk` 处理完成时的回调函数。

该函数不能被应用程序代码直接调用。 它应该由子类实现，且只能被内部的 `Readable` 类的方法调用。

所有转换流的实现都必须提供 `_transform()` 方法来接收输入并生产输出。 `transform._transform()` 的实现会处理写入的字节，进行一些计算操作，然后使用 `readable.push()` 输出到可读流。

`transform.push()` 可能会被调用零次或多次用来从每次输入的数据块产生输出，调用的次数取决需要多少数据来产生输出的结果。

输入的数据块有可能不会产生任何输出。

当前数据被完全消费之后，必须调用 `callback` 函数。 当处理输入的过程中发生出错时， `callback` 的第一个参数传入 `Error` 对象，否则传入 `null`。 如果 `callback` 传入了第二个参数，则它会被转发到 `readable.push()`。 就像下面的例子：

```js
transform.prototype._transform = function(data, encoding, callback) {
  this.push(data);
  callback();
};

transform.prototype._transform = function(data, encoding, callback) {
  callback(null, data);
};
```

`transform._transform()` 方法有下划线前缀，因为它是在定义在类的内部，不应该被用户程序直接调用。

`transform._transform()` 不能并行调用。 流使用了队列机制，无论同步或异步的情况下，都必须先调用 `callback` 之后才能接收下一个数据块。

#### stream.PassThrough 类[#](http://nodejs.cn/api/stream.html#stream_class_stream_passthrough)

[中英对照](http://nodejs.cn/api/stream/class_stream_passthrough.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/class_stream_passthrough.md)

`stream.PassThrough` 类是一个无关紧要的[转换流](http://nodejs.cn/s/fhVJQM)，只是单纯地把输入的字节原封不动地输出。 它主要用于示例或测试，但有时也会用于某些新颖的流的基本组成部分。

## 其他注意事项[#](http://nodejs.cn/api/stream.html#stream_additional_notes)

### 兼容旧版本的 Node.js[#](http://nodejs.cn/api/stream.html#stream_compatibility_with_older_node_js_versions)

暂无中英对照[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/compatibility_with_older_node_js_versions.md)

Prior to Node.js 0.10, the `Readable` stream interface was simpler, but also less powerful and less useful.

- Rather than waiting for calls to the [`stream.read()`](http://nodejs.cn/s/SpgRaa) method, [`'data'`](http://nodejs.cn/s/8CCPjN) events would begin emitting immediately. Applications that would need to perform some amount of work to decide how to handle data were required to store read data into buffers so the data would not be lost.
- The [`stream.pause()`](http://nodejs.cn/s/jqytVw) method was advisory, rather than guaranteed. This meant that it was still necessary to be prepared to receive [`'data'`](http://nodejs.cn/s/8CCPjN) events *even when the stream was in a paused state*.

In Node.js 0.10, the [`Readable`](http://nodejs.cn/s/YuDKX1) class was added. For backward compatibility with older Node.js programs, `Readable` streams switch into "flowing mode" when a [`'data'`](http://nodejs.cn/s/8CCPjN) event handler is added, or when the [`stream.resume()`](http://nodejs.cn/s/Zhf28N) method is called. The effect is that, even when not using the new [`stream.read()`](http://nodejs.cn/s/SpgRaa) method and[`'readable'`](http://nodejs.cn/s/J6CZGb) event, it is no longer necessary to worry about losing [`'data'`](http://nodejs.cn/s/8CCPjN) chunks.

While most applications will continue to function normally, this introduces an edge case in the following conditions:

- No [`'data'`](http://nodejs.cn/s/8CCPjN) event listener is added.
- The [`stream.resume()`](http://nodejs.cn/s/Zhf28N) method is never called.
- The stream is not piped to any writable destination.

For example, consider the following code:

```js
// WARNING!  BROKEN!
net.createServer((socket) => {

  // we add an 'end' listener, but never consume the data
  socket.on('end', () => {
    // It will never get here.
    socket.end('The message was received but was not processed.\n');
  });

}).listen(1337);
```

Prior to Node.js 0.10, the incoming message data would be simply discarded. However, in Node.js 0.10 and beyond, the socket remains paused forever.

The workaround in this situation is to call the

```js
// Workaround
net.createServer((socket) => {
  socket.on('end', () => {
    socket.end('The message was received but was not processed.\n');
  });

  // start the flow of data, discarding it.
  socket.resume();
}).listen(1337);
```

In addition to new `Readable` streams switching into flowing mode, pre-0.10 style streams can be wrapped in a `Readable` class using the [`readable.wrap()`](http://nodejs.cn/s/8NH3ye) method.

### readable.read(0)[#](http://nodejs.cn/api/stream.html#stream_readable_read_0)

[中英对照](http://nodejs.cn/api/stream/readable_read_0.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_read_0.md)

在某些情况下，需要触发底层可读流的刷新，但实际并不消费任何数据。 在这种情况下，可以调用 `readable.read(0)`，返回 `null`。

如果内部读取缓冲小于 `highWaterMark`，且流还未被读取，则调用 `stream.read(0)` 会触发调用底层的 [`stream._read()`](http://nodejs.cn/s/5hv4Rd)。

虽然大多数应用程序几乎不需要这样做，但 Node.js 中会出现这种情况，尤其是在可读流类的内部。

### readable.push('')[#](http://nodejs.cn/api/stream.html#stream_readable_push)

[中英对照](http://nodejs.cn/api/stream/readable_push.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/readable_push.md)

不推荐使用 `readable.push('')`。

向一个非对象模式的流推入一个零字节的字符串、 `Buffer` 或 `Uint8Array` 会产生副作用。 因为调用了 [`readable.push()`](http://nodejs.cn/s/8s3paZ)，该调用会结束读取进程。 然而，因为参数是一个空字符串，没有数据被添加到可读缓冲, 所以也就没有数据可供用户消费。

### 调用 `readable.setEncoding()` 之后 `highWaterMark` 的差异[#](http://nodejs.cn/api/stream.html#stream_highwatermark_discrepancy_after_calling_readable_setencoding)

[中英对照](http://nodejs.cn/api/stream/highwatermark_discrepancy_after_calling_readable_setencoding.html)[提交修改](https://github.com/nodejscn/node-api-cn/edit/master/stream/highwatermark_discrepancy_after_calling_readable_setencoding.md)

使用 `readable.setEncoding()` 会改变 `highWaterMark` 属性在非对象模式中的作用。

一般而言，当前缓冲的大小是以字节为单位跟 `highWaterMark` 比较的。 但是调用 `setEncoding()` 之后，会开始以字符为单位进行比较。

大多数情况下，使用 `latin1` 或 `ascii` 时是没有问题的。 但在处理含有多字节字符的字符串时，需要小心。



- [查看官方文档API](http://nodejs.cn/api/stream.html)