### http（HTTP）

要使用 HTTP 服务器和客户端，必须 `require('http')`。

Node.js 中的 HTTP 接口旨在支持传统上难以使用的协议的许多特性。 特别是，大块的、可能块编码的消息。 接口永远不会缓冲整个请求或响应，用户能够流式传输数据。

HTTP 消息头由如下对象表示：

```js
{ 'content-length': '123',
  'content-type': 'text/plain',
  'connection': 'keep-alive',
  'host': 'mysite.com',
  'accept': '*/*' }
```

键是小写的。值不能被修改。

为了支持所有可能的 HTTP 应用程序，Node.js 的 HTTP API 都非常底层。 它仅进行流处理和消息解析。 它将消息解析为消息头和消息主体，但不会解析具体的消息头或消息主体。

有关如何处理重复消息头的详细信息，参阅 [`message.headers`](http://nodejs.cn/s/9aiRnT)。

接收到的原始消息头保存在 `rawHeaders` 属性中，该属性是 `[key, value, key2, value2, ...]` 的数组。 例如，上面的消息头对象可能具有的 `rawHeaders` 列表如下所示：

```js
[ 'ConTent-Length', '123456',
  'content-LENGTH', '123',
  'content-type', 'text/plain',
  'CONNECTION', 'keep-alive',
  'Host', 'mysite.com',
  'accepT', '*/*' ]
```

- [官方文档API](http://nodejs.cn/api/http.html)

## http.Agent 类

新增于: v0.3.4

`Agent` 负责管理 HTTP 客户端的连接持久性和重用。 它为给定的主机和端口维护一个待处理请求队列，为每个请求重用单独的套接字连接，直到队列为空，此时套接字被销毁或放入连接池，以便再次用于请求到同一个主机和端口。 销毁还是放入连接池取决于 `keepAlive` [选项](http://nodejs.cn/api/http.html#http_new_agent_options)。

连接池中的连接已启用 TCP Keep-Alive，但服务器仍可能关闭空闲连接，在这种情况下，它们将从连接池中删除，并且当为该主机和端口发出新的 HTTP 请求时将建立新连接。 服务器也可以拒绝通过同一连接允许多个请求，在这种情况下，必须为每个请求重新建立连接，并且不能放入连接池。 `Agent` 仍将向该服务器发出请求，但每个请求都将通过新连接发生。

当客户端或服务器关闭连接时，它将从连接池中删除。 连接池中任何未使用的套接字都将被销毁，以便当没有未完成的请求时不用保持 Node.js 进程运行。 （参阅 [`socket.unref()`](http://nodejs.cn/s/DAQ9Qm)）。

当不再使用时最好 [`destroy()`](http://nodejs.cn/s/qZQu4q) `Agent` 实例，因为未使用的套接字会消耗操作系统资源。

当套接字触发 `'close'` 事件或 `'agentRemove'` 事件时，则套接字将从代理中删除。 当打算长时间保持一个 HTTP 请求打开而不将其保留在代理中时，可以执行以下操作：

```js
http.get(options, (res) => {
  // 做些事情。
}).on('socket', (socket) => {
  socket.emit('agentRemove');
});
```

代理也可以用于单个请求。 通过提供 `{agent: false}` 作为 `http.get()` 或 `http.request()` 函数的选项，则将使用一次性的具有默认选项的 `Agent` 用于客户端连接。

`agent:false` 的示例：

```js
http.get({
  hostname: 'localhost',
  port: 80,
  path: '/',
  agent: false  // 仅为此一个请求创建一个新代理。
}, (res) => {
  // 用响应做些事情。
});
```

### new Agent([options])

新增于: v0.3.4

- `options` <Object>  要在代理上设置的可配置选项集。可以包含以下字段：
  - `keepAlive` <boolean> 即使没有未完成的请求，也要保持套接字，这样它们就可以被用于将来的请求而无需重新建立 TCP 连接。**默认值:** `false`。
  - `keepAliveMsecs` <number> 当使用 `keepAlive` 选项时，指定用于 TCP Keep-Alive 数据包的[初始延迟](http://nodejs.cn/s/qqbgUD)。当 `keepAlive` 选项为 `false` 或 `undefined` 时则忽略。**默认值:** `1000`。
  - `maxSockets` <number> 每个主机允许的套接字的最大数量。**默认值:** `Infinity`。
  - `maxFreeSockets` <number> 在空闲状态下保持打开的套接字的最大数量。仅当 `keepAlive` 被设置为 `true` 时才相关。**默认值:** `256`。
  - `timeout` <number> 套接字的超时时间，以毫秒为单位。这会在套接字被连接之后设置超时时间。

[`socket.connect()`](http://nodejs.cn/s/787XLt) 中的 `options` 也受支持。

被 [`http.request()`](http://nodejs.cn/s/d1myoL) 使用的默认的 [`http.globalAgent`](http://nodejs.cn/s/g7BhW2) 具有所有这些值且被设置为各自的默认值。

要配置其中任何一个，则必须创建自定义的 [`http.Agent`](http://nodejs.cn/s/HRCnER) 实例。

```js
const http = require('http');
const keepAliveAgent = new http.Agent({ keepAlive: true });
options.agent = keepAliveAgent;
http.request(options, onResponseCallback);
```

### agent.createConnection(options[, callback])

新增于: v0.11.4

- `options` <Object>  包含连接详细信息的选项。 查看 [`net.createConnection()`](http://nodejs.cn/s/xu7F69) 以获取选项的格式。
- `callback` <Function> 接收创建的套接字的回调函数。
- 返回: <net.socket>

生成用于 HTTP 请求的套接字或流。

默认情况下，此函数与 [`net.createConnection()`](http://nodejs.cn/s/xu7F69) 相同。 但是，如果需要更大的灵活性，自定义代理可能会覆盖此方法。

可以通过以下两种方式之一提供套接字或流：通过从此函数返回套接字或流，或者通过将套接字或流传给 `callback`。

`callback` 的参数是 `(err, stream)`。

### agent.keepSocketAlive(socket)

新增于: v8.1.0

- `socket` <net.socket>

当 `socket` 与请求分离并且可以由 `Agent` 保留时调用。 默认行为是：

```js
socket.setKeepAlive(true, this.keepAliveMsecs);
socket.unref();
return true;
```

此方法可以由特定的 `Agent` 子类重写。 如果此方法返回一个假值，则将销毁套接字而不是将其保留以用于下一个请求。

### agent.reuseSocket(socket, request)

新增于: v8.1.0

- `socket` <net.socket>
- `request` <http.ClientRequest>

由于 keep-alive 选项而在持久化后将 `socket` 附加到 `request` 时调用。 默认行为是：

```js
socket.ref();
```

此方法可以由特定的 `Agent` 子类重写。

### agent.destroy()

新增于: v0.11.4

销毁代理当前使用的所有套接字。

通常没有必要这样做。 但是，如果使用启用了 `keepAlive` 的代理，则最好在代理不再使用时显式关闭代理。 否则，在服务器终止套接字之前，套接字可能会挂起很长时间。

### agent.freeSockets

新增于: v0.11.4

- <Object>

一个对象，其中包含当启用 `keepAlive` 时代理正在等待使用的套接字数组。 不要修改。

### agent.getName(options)

新增于: v0.11.4

- `options` <Object>  一组选项，为生成名称提供信息。
  - `host` <string> 请求发送至的服务器的域名或 IP 地址。
  - `port` <number> 远程服务器的端口。
  - `localAddress` <string> 为网络连接绑定的本地接口。
  - `family` <number> 如果不等于 `undefined`，则必须为 4 或 6。
- 返回: <string>

获取一组请求选项的唯一名称，以判定一个连接是否可以被重用。 对于 HTTP 代理，这返回 `host:port:localAddress` 或 `host:port:localAddress:family`。 对于 HTTPS 代理，该名称包括 CA、证书、密码、以及其他可判定套接字可重用性的 HTTPS/TLS 特有的选项。

### agent.maxFreeSockets

新增于: v0.11.7

- <number>

默认设置为 256。 对于启用了 `keepAlive` 的代理，这将设置在空闲状态下保持打开的最大套接字数。

### agent.maxSockets

新增于: v0.3.6

- <number>

默认情况下设置为 `Infinity`。 决定代理可以为每个来源打开多少并发套接字。 来源是 [`agent.getName()`](http://nodejs.cn/s/F2jZJb) 的返回值。

### agent.requests

新增于: v0.5.9

- <Object> 

一个对象，包含尚未分配给套接字的请求队列。 不要修改。

### agent.sockets

新增于: v0.3.6

- <Object> 

一个对象，包含当前代理正在使用的套接字数组。 不要修改。

## http.ClientRequest 类

新增于: v0.1.17

此对象由 [`http.request()`](http://nodejs.cn/s/d1myoL) 内部创建并返回。 它表示正在进行的请求，且其请求头已进入队列。 请求头仍然可以使用 [`setHeader(name, value)`](http://nodejs.cn/s/ZBShBB)、[`getHeader(name)`](http://nodejs.cn/s/noVJNv) 或 [`removeHeader(name)`](http://nodejs.cn/s/vzuErq) 改变。 实际的请求头将与第一个数据块一起发送，或者当调用 [`request.end()`](http://nodejs.cn/s/GAdV2u) 时发送。

要获得响应，则为请求对象添加 [`'response'`](http://nodejs.cn/s/qwaiK8) 事件监听器。 当接收到响应头时，将、则会从请求对象触发 [`'response'`](http://nodejs.cn/s/qwaiK8) 事件。 [`'response'`](http://nodejs.cn/s/qwaiK8) 事件执行时有一个参数，该参数是 [`http.IncomingMessage`]<http.IncomingMessage> 的实例。

在 [`'response'`](http://nodejs.cn/s/qwaiK8) 事件期间，可以添加监听器到响应对象，比如监听 `'data'` 事件。

如果没有添加 [`'response'`](http://nodejs.cn/s/qwaiK8) 事件处理函数，则响应将被完全丢弃。 如果添加了 [`'response'`](http://nodejs.cn/s/qwaiK8) 事件处理函数，则必须消费完响应对象中的数据，每当有 `'readable'` 事件时调用 `response.read()`、或添加 `'data'` 事件处理函数、或通过调用 `.resume()` 方法。 在消费完数据之前，不会触发 `'end'` 事件。 此外，在读取数据之前，它将占用内存，最终可能导致进程内存不足的错误。

Node.js 不检查 Content-Length 和已传输的主体的长度是否相等。

请求继承自[流](http://nodejs.cn/s/kUvpNm)，且额外实现以下内容：

### 'abort' 事件

新增于: v1.4.1

当请求被客户端中止时触发。 此事件仅在第一次调用 `abort()` 时触发。

### 'connect' 事件

新增于: v0.7.0

- `response` <http.IncomingMessage>
- `socket` <net.socket>
- `head` <Buffer>

每次服务器使用 `CONNECT` 方法响应请求时都会触发。 如果未监听此事件，则接收 `CONNECT` 方法的客户端将关闭其连接。

客户端和服务器对演示了如何监听 `'connect'` 事件：

```js
const http = require('http');
const net = require('net');
const url = require('url');

// 创建 HTTP 隧道代理。
const proxy = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('响应内容');
});
proxy.on('connect', (req, cltSocket, head) => {
  // 连接到原始服务器。
  const srvUrl = url.parse(`http://${req.url}`);
  const srvSocket = net.connect(srvUrl.port, srvUrl.hostname, () => {
    cltSocket.write('HTTP/1.1 200 Connection Established\r\n' +
                    'Proxy-agent: Node.js-Proxy\r\n' +
                    '\r\n');
    srvSocket.write(head);
    srvSocket.pipe(cltSocket);
    cltSocket.pipe(srvSocket);
  });
});

// 代理正在运行。
proxy.listen(1337, '127.0.0.1', () => {

  // 向隧道代理发出请求。
  const options = {
    port: 1337,
    host: '127.0.0.1',
    method: 'CONNECT',
    path: 'nodejs.cn:80'
  };

  const req = http.request(options);
  req.end();

  req.on('connect', (res, socket, head) => {
    console.log('已连接');

    // 通过 HTTP 隧道发出请求。
    socket.write('GET / HTTP/1.1\r\n' +
                 'Host: nodejs.cn:80\r\n' +
                 'Connection: close\r\n' +
                 '\r\n');
    socket.on('data', (chunk) => {
      console.log(chunk.toString());
    });
    socket.on('end', () => {
      proxy.close();
    });
  });
});
```

### 'continue' 事件

新增于: v0.3.2

当服务器发送 `100 Continue` HTTP 响应时触发，通常是因为请求包含 `Expect: 100-continue`。 这是客户端应发送请求主体的指令。

### 'information' 事件

新增于: v10.0.0

服务器发送 `1xx` 响应时触发（不包括 `101 Upgrade`）。 使用包含具有状态码的对象的回调触发此事件。

```js
const http = require('http');

const options = {
  host: '127.0.0.1',
  port: 8080,
  path: '/length_request'
};

// 发出请求。
const req = http.request(options);
req.end();

req.on('information', (res) => {
  console.log(`获得主响应之前的信息: ${res.statusCode}`);
});
```

`101 Upgrade` 状态不会触发此事件，因为它们与传统的 HTTP 请求/响应链断开，例如 Web 套接字、现场 TLS 升级、或 HTTP 2.0。 要收到 `101 Upgrade` 的通知，请改为监听 [`'upgrade'`](http://nodejs.cn/s/xjUtig) 事件。

### 'response' 事件

新增于: v0.1.0

- `response` <http.IncomingMessage>

当收到此请求的响应时触发。 此事件仅触发一次。

### 'socket' 事件

新增于: v0.5.3

- `socket` <net.socket>

将套接字分配给此请求后触发。

### 'timeout' 事件

新增于: v0.7.8

当底层套接字因不活动而超时时触发。 这只会通知套接字已空闲。 必须手动中止请求。

另请参见：[`request.setTimeout()`](http://nodejs.cn/s/qX7N7E)。

### 'upgrade' 事件

新增于: v0.1.94

- `response` <http.IncomingMessage>
- `socket` <net.socket>
- `head` <Buffer>

每次服务器响应升级请求时发出。 如果未监听此事件且响应状态码为 `101 Switching Protocols`，则接收升级响应头的客户端将关闭其连接。

客户端服务器对，演示如何监听 `'upgrade'` 事件。

```js
const http = require('http');

// 创建 HTTP 服务器。
const srv = http.createServer( (req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('响应内容');
});
srv.on('upgrade', (req, socket, head) => {
  socket.write('HTTP/1.1 101 Web Socket Protocol Handshake\r\n' +
               'Upgrade: WebSocket\r\n' +
               'Connection: Upgrade\r\n' +
               '\r\n');

  socket.pipe(socket);
});

// 服务器正在运行。
srv.listen(1337, '127.0.0.1', () => {

  // 发送请求。
  const options = {
    port: 1337,
    host: '127.0.0.1',
    headers: {
      'Connection': 'Upgrade',
      'Upgrade': 'websocket'
    }
  };

  const req = http.request(options);
  req.end();

  req.on('upgrade', (res, socket, upgradeHead) => {
    console.log('接收到响应');
    socket.end();
    process.exit(0);
  });
});
```

### request.abort()

新增于: v0.3.8

将请求标记为中止。 调用此方法将导致响应中剩余的数据被丢弃并且套接字被销毁。

### request.aborted

新增于: v0.11.14

如果请求已中止，则此值是请求中止的时间，自 1970 年 1 月 1 日 00:00:00 UTC 以来的毫秒数。

### request.connection

新增于: v0.3.0

- <net.socket>

请参阅 [`request.socket`](http://nodejs.cn/s/FUP815)。

### request.end([data[, encoding]][, callback])

版本历史

- `data` <string> | <Buffer>
- `encoding` <string>
- `callback` <Function>
- 返回: <this>

完成发送请求。 如果部分请求主体还未发送，则将它们刷新到流中。 如果请求被分块，则发送终止符 `'0\r\n\r\n'`。

如果指定了 `data`，则相当于调用 [`request.write(data, encoding)`](http://nodejs.cn/s/6ATcHB) 之后再调用 `request.end(callback)`。

如果指定了 `callback`，则当请求流完成时将调用它。

### request.finished

新增于: v0.0.1

- <boolean>

如果调用了 [`request.end()`](http://nodejs.cn/s/GAdV2u)，则 `request.finished` 属性将为 `true`。 如果请求是通过 [`http.get()`](http://nodejs.cn/s/MV8Bd1) 发起的，则会自动调用 `request.end()`。

### request.flushHeaders()

新增于: v1.6.0

刷新请求头。

出于效率原因，Node.js 通常会缓冲请求头，直到调用 `request.end()` 或写入第一个请求数据块。 然后，它尝试将请求头和数据打包到单个 TCP 数据包中。

这通常是期望的（它节省了 TCP 往返），但是可能很晚才发送第一个数据。 `request.flushHeaders()` 绕过优化并启动请求。

### request.getHeader(name)

新增于: v1.6.0

- `name` <string>
- 返回: <any>

读取请求中的一个请求头。 注意，该名称不区分大小写。 返回值的类型取决于提供给 [`request.setHeader()`](http://nodejs.cn/s/ZBShBB) 的参数。

```js
request.setHeader('content-type', 'text/html');
request.setHeader('Content-Length', Buffer.byteLength(body));
request.setHeader('Cookie', ['type=ninja', 'language=javascript']);
const contentType = request.getHeader('Content-Type');
// contentType 是 'text/html'。
const contentLength = request.getHeader('Content-Length');
// contentLength 的类型为数值。
const cookie = request.getHeader('Cookie');
// cookie 的类型为字符串数组。
```

### request.maxHeadersCount

- <number> **默认值:** `2000`。

限制最大响应头数。 如果设置为 `0`，则不会应用任何限制。

### request.removeHeader(name)

新增于: v1.6.0

- `name` <string>

移除已定义到请求头对象中的请求头。

```js
request.removeHeader('Content-Type');
```

### request.setHeader(name, value)

新增于: v1.6.0

- `name` <string>
- `value` <any>

为请求头对象设置单个请求头的值。 如果此请求头已存在于待发送的请求头中，则其值将被替换。 这里可以使用字符串数组来发送具有相同名称的多个请求头。 非字符串值将被原样保存。 因此 [`request.getHeader()`](http://nodejs.cn/s/noVJNv) 可能会返回非字符串值。 但是非字符串值将转换为字符串以进行网络传输。

```js
request.setHeader('Content-Type', 'application/json');
```

或：

```js
request.setHeader('Cookie', ['type=ninja', 'language=javascript']);
```

### request.setNoDelay([noDelay]

新增于: v0.5.9

- `noDelay` <boolean>

一旦将套接字分配给此请求并且连接了套接字，就会调用 [`socket.setNoDelay()`](http://nodejs.cn/s/q9UswY)。

### request.setSocketKeepAlive([enable][, initialdelay])

新增于: v0.5.9

- `enable` <boolean>
- `initialDelay` <number>

一旦将套接字分配给此请求并连接了套接字，就会调用 [`socket.setKeepAlive()`](http://nodejs.cn/s/qqbgUD)。

### request.setTimeout(timeout[, callback])

新增于: v0.5.9

- `timeout` <number> 请求超时前的毫秒数。
- `callback` <Function> 发生超时时要调用的可选函数。相当于绑定到 `'timeout'` 事件。
- 返回: <http.ClientRequest>

一旦将套接字分配给此请求并且连接了套接字，就会调用 [`socket.setTimeout()`](http://nodejs.cn/s/XC4Yyj)。

### request.socket

新增于: v0.3.0

- <net.socket>

指向底层套接字。 通常用户无需访问此属性。 特别是，由于协议解析器附加到套接字的方式，套接字将不会触发 `'readable'` 事件。 也可以通过 `request.connection` 访问 `socket`。

```js
const http = require('http');
const options = {
  host: 'nodejs.cn',
};
const req = http.get(options);
req.end();
req.once('response', (res) => {
  const ip = req.socket.localAddress;
  const port = req.socket.localPort;
  console.log(`您的 IP 地址是 ${ip}，源端口是 ${port}`);
  // 使用响应对象。
});
```

### request.write(chunk[, encoding][, callback])

新增于: v0.1.29

- `chunk` <string> | <Buffer>
- `encoding` <string>
- `callback` <Function>
- 返回: <boolean>

发送一个请求主体的数据块。 通过多次调用此方法，可以将请求主体发送到服务器，在这种情况下，建议在创建请求时使用 `['Transfer-Encoding', 'chunked']` 请求头行。

`encoding` 参数是可选的，仅当 `chunk` 是字符串时才适用。 默认为 `'utf8'`。

`callback` 参数是可选的，当刷新此数据块时调用，但仅当数据块非空时才会调用。

如果将整个数据成功刷新到内核缓冲区，则返回 `true`。 如果全部或部分数据在用户内存中排队，则返回 `false`。 当缓冲区再次空闲时，则触发 `'drain'` 事件。

当使用空字符串或 buffer 调用 `write` 函数时，则什么也不做且等待更多输入。

## http.Server 类

新增于: v0.1.17

此类继承自 [`net.Server`](http://nodejs.cn/s/gBYjux) 并具有以下额外的事件：

### 'checkContinue' 事件

新增于: v0.3.0

- `request` <http.IncomingMessage>
- `response` <http.ServerResponse>

每次收到 HTTP `Expect: 100-continue` 的请求时都会触发。 如果未监听此事件，服务器将自动响应 `100 Continue`。

处理此事件时，如果客户端应继续发送请求主体，则调用 [`response.writeContinue()`](http://nodejs.cn/s/vzUamu)，如果客户端不应继续发送请求主体，则生成适当的 HTTP 响应（例如 `400 Bad Request`）。

请注意，在触发和处理此事件时，不会触发 [`'request'`](http://nodejs.cn/s/2qCn57) 事件。

### 'checkExpectation' 事件

新增于: v5.5.0

- `request` <http.IncomingMessage>
- `response` <http.ServerResponse>

每次收到带有 HTTP `Expect` 请求头的请求时触发，其中值不是 `100-continue`。 如果未监听此事件，则服务器将根据需要自动响应 `417 Expectation Failed`。

请注意，在触发和处理此事件时，不会触发 [`'request'`](http://nodejs.cn/s/2qCn57) 事件。

### 'clientError' 事件

版本历史

- `exception` <error>
- `socket` <net.socket>

如果客户端连接触发 `'error'` 事件，则会在此处转发。 此事件的监听器负责关闭或销毁底层套接字。 例如，用户可能希望使用自定义 HTTP 响应更优雅地关闭套接字，而不是突然切断连接。

默认行为是尽可能使用 HTTP `400 Bad Request` 响应关闭套接字，否则立即销毁套接字。

`socket` 是发生错误的 [`net.Socket`](http://nodejs.cn/s/wsJ1o1) 对象。

```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.end();
});
server.on('clientError', (err, socket) => {
  socket.end('HTTP/1.1 400 Bad Request\r\n\r\n');
});
server.listen(8000);
```

当 `'clientError'` 事件发生时，没有 `request` 或 `response` 对象，因此必须将发送的任何 HTTP 响应（包括响应头和有效负载）直接写入 `socket` 对象。 必须注意确保响应是格式正确的 HTTP 响应消息。

`err` 是 `Error` 实例，有以下两个额外的部分：

- `bytesParsed`: Node.js 可能正确解析的请求包的字节数。
- `rawPacket`: 当前请求的原始数据包。

### 'close' 事件

新增于: v0.1.4

当服务器关闭时触发。

### 'connect' 事件

新增于: v0.7.0

- `request` <http.IncomingMessage> HTTP 请求的参数，与 [`'request'`](http://nodejs.cn/s/2qCn57) 事件中的一样。
- `socket` <net.socket> 服务器和客户端之间的网络套接字。
- `head` <Buffer> 隧道流的第一个数据包（可能为空）。

每次客户端请求 HTTP `CONNECT` 方法时触发。 如果未监听此事件，则请求 `CONNECT` 方法的客户端将关闭其连接。

触发此事件后，请求的套接字将没有 `'data'` 事件监听器，这意味着它需要绑定才能处理发送到该套接字上的服务器的数据。

### 'connection' 事件

新增于: v0.1.0

- `socket` <net.socket>

建立新的 TCP 流时会触发此事件。 `socket` 通常是 [`net.Socket`](http://nodejs.cn/s/wsJ1o1) 类型的对象。 通常用户无需访问此事件。 特别是，由于协议解析器附加到套接字的方式，套接字将不会触发 `'readable'` 事件。 也可以通过 `request.connection` 访问 `socket`。

用户也可以显式触发此事件，以将连接注入 HTTP 服务器。 在这种情况下，可以传入任何 [`Duplex`](http://nodejs.cn/s/2iRabr) 流。

### 'request' 事件

新增于: v0.1.0

- `request` <http.IncomingMessage>
- `response` <http.ServerResponse>

每次有请求时都会触发。 注意，每个连接可能有多个请求（在 HTTP Keep-Alive 连接的情况下）。

### 'upgrade' 事件

版本历史

- `request` <http.IncomingMessage> HTTP 请求的参数，与 [`'request'`](http://nodejs.cn/s/2qCn57) 事件中的一样。
- `socket` <net.socket> 服务器与客户端之间的网络套接字。
- `head` <Buffer> 升级后的流的第一个数据包（可能为空）。

每次客户端请求 HTTP 升级时发出。 监听此事件是可选的，客户端无法坚持更改协议。

触发此事件后，请求的套接字将没有 `'data'` 事件监听器，这意味着它需要绑定才能处理发送到该套接字上的服务器的数据。

### server.close([callback])

新增于: v0.1.90

- `callback` <Function>

停止服务器接受新连接。 请参见 [`net.Server.close()`](http://nodejs.cn/s/zZ874N)。

### server.listen()

启动 HTTP 服务器监听连接。 此方法与 [`net.Server`](http://nodejs.cn/s/gBYjux) 中的 [`server.listen()`](http://nodejs.cn/s/xGksiu) 相同。

### server.listening

新增于: v5.7.0

- <boolean> 表明服务器是否正在监听连接。

### server.maxHeadersCount

新增于: v0.7.0

- <number> **默认值:** `2000`。

限制最大传入请求头数。 如果设置为 0，则不会应用任何限制。

### server.headersTimeout

新增于: v10.14.0

- <number> **默认值:** `40000`。

限制解析器等待接收完整 HTTP 请求头的时间。

如果不活动，则适用 [`server.timeout`](http://nodejs.cn/s/yJbd7V) 中定义的规则。 但是，如果请求头发送速度非常慢（默认情况下，每 2 分钟最多一个字节），那么基于不活动的超时仍然允许连接保持打开状态。 为了防止这种情况，每当请求头数据到达时，进行额外的检查，自建立连接以来，没有超过 `server.headersTimeout` 毫秒。 如果检查失败，则在服务器对象上触发 `'timeout'` 事件，并且（默认情况下）套接字被销毁。 有关如何自定义超时行为的详细信息，请参阅 [`server.timeout`](http://nodejs.cn/s/yJbd7V)。

### server.setTimeout([msecs][, callback])

新增于: v0.9.12

- `msecs` <number> **默认值:** `120000`（2 分钟）。
- `callback` <Function>
- 返回: <http.Server>

设置套接字的超时值，并在服务器对象上触发 `'timeout'` 事件，如果发生超时，则将套接字作为参数传入。

如果服务器对象上有 `'timeout'` 事件监听器，则将使用超时的套接字作为参数调用它。

默认情况下，服务器的超时值为 2 分钟，如果超时，套接字会自动销毁。 但是，如果将回调分配给服务器的 `'timeout'` 事件，则必须显式处理超时。

### server.timeout

新增于: v0.9.12

- <number> 超时时间（以毫秒为单位）。**默认值:** `120000`（2 分钟）。

认定套接字超时的不活动毫秒数。

值为 `0` 将禁用传入连接的超时行为。

套接字超时逻辑在连接时设置，因此更改此值仅影响到服务器的新连接，而不影响任何现有连接。

### server.keepAliveTimeout

新增于: v8.0.0

- <number> 超时时间（以毫秒为单位）。**默认值:** `5000`（5 秒）。

服务器在完成写入最后一个响应之后，在销毁套接字之前需要等待其他传入数据的非活动毫秒数。 如果服务器在保持活动超时被触发之前接收到新数据，它将重置常规非活动超时，即 [`server.timeout`](http://nodejs.cn/s/yJbd7V)。

值为 `0` 将禁用传入连接上的保持活动超时行为。 值为 `0` 使得 http 服务器的行为与 8.0.0 之前的 Node.js 版本类似，后者没有保持活动超时。

套接字超时逻辑在连接时设置，因此更改此值仅影响到服务器的新连接，而不影响任何现有连接。

## http.ServerResponse 类

新增于: v0.1.17

此对象由 HTTP 服务器在内部创建，而不是由用户创建。 它作为第二个参数传给 [`'request'`](http://nodejs.cn/s/2qCn57) 事件。

响应继承自[流](http://nodejs.cn/s/kUvpNm)，并额外实现以下内容：

### 'close' 事件

新增于: v0.6.7

表明在调用 [`response.end()`](http://nodejs.cn/s/sqAtet) 或能够刷新之前终止了底层连接。

### 'finish' 事件

新增于: v0.3.6

响应发送后触发。 更具体地说，当响应头和主体的最后一段已经切换到操作系统以通过网络传输时，触发该事件。 这并不意味着客户端已收到任何信息。

### response.addTrailers(headers)

新增于: v0.3.0

- `headers` <Object> 

此方法将 HTTP 尾部响应头（一种在消息末尾的响应头）添加到响应中。

只有在使用分块编码进行响应时才会发出尾部响应头; 如果不是（例如，如果请求是 HTTP/1.0），它们将被静默丢弃。

请注意，HTTP 需要发送 `Trailer` 响应头才能发出尾部响应头，并在其值中包含响应头字段列表。 例如：

```js
response.writeHead(200, { 'Content-Type': 'text/plain',
                          'Trailer': 'Content-MD5' });
response.write(fileData);
response.addTrailers({ 'Content-MD5': '7895bf4b8828b55ceaf47747b4bca667' });
response.end();
```

尝试设置包含无效字符的响应头字段名称或值将导致抛出 [`TypeError`](http://nodejs.cn/s/Z7Lqyj)。

### response.connection

新增于: v0.3.0

- <net.socket>

参阅 [`response.socket`](http://nodejs.cn/s/oiu4ph)。

### response.end([data][, encoding][, callback])

版本历史

- `data` <string> | <Buffer>
- `encoding` <string>
- `callback` <Function>
- 返回: <this>

此方法向服务器发出信号，表明已发送所有响应头和主体，该服务器应该视为此消息已完成。 必须在每个响应上调用此 `response.end()` 方法。

如果指定了 `data`，则相当于调用 [`response.write(data, encoding)`](http://nodejs.cn/s/gvWo3m) 之后再调用 `response.end(callback)`。

如果指定了 `callback`，则当响应流完成时将调用它。

### response.finished

新增于: v0.0.2

- <boolean>

布尔值，表明响应是否已完成。 默认为 `false`。 在 [`response.end()`](http://nodejs.cn/s/sqAtet) 执行之后，该值将为 `true`。

### response.getHeader(name)

新增于: v0.4.0

- `name` <string>
- 返回: <any>

读出已排队但未发送到客户端的响应头。 请注意，该名称不区分大小写。 返回值的类型取决于提供给 [`response.setHeader()`](http://nodejs.cn/s/rqeM3J) 的参数。

```js
response.setHeader('Content-Type', 'text/html');
response.setHeader('Content-Length', Buffer.byteLength(body));
response.setHeader('Set-Cookie', ['type=ninja', 'language=javascript']);
const contentType = response.getHeader('content-type');
// contentType 是 'text/html'。
const contentLength = response.getHeader('Content-Length');
// contentLength 的类型为数值。
const setCookie = response.getHeader('set-cookie');
// setCookie 的类型为字符串数组。
```

### response.getHeaderNames()

新增于: v7.7.0

- 返回: <string>

返回一个数组，其中包含当前传出的响应头的唯一名称。 所有响应头名称都是小写的。

```js
response.setHeader('Foo', 'bar');
response.setHeader('Set-Cookie', ['foo=bar', 'bar=baz']);

const headerNames = response.getHeaderNames();
// headerNames === ['foo', 'set-cookie']
```

### response.getHeaders()

新增于: v7.7.0

- 返回: <Object> 

返回当前传出的响应头的浅拷贝。 由于使用浅拷贝，因此可以更改数组的值而无需额外调用各种与响应头相关的 http 模块方法。 返回对象的键是响应头名称，值是各自的响应头值。 所有响应头名称都是小写的。

`response.getHeaders()` 方法返回的对象不是从 JavaScript `Object` 原型继承的。 这意味着典型的 `Object` 方法，如 `obj.toString()`、 `obj.hasOwnProperty()` 等都没有定义并且不起作用。

```js
response.setHeader('Foo', 'bar');
response.setHeader('Set-Cookie', ['foo=bar', 'bar=baz']);

const headers = response.getHeaders();
// headers === { foo: 'bar', 'set-cookie': ['foo=bar', 'bar=baz'] }
```

### response.hasHeader(name)

新增于: v7.7.0

- `name` <string>
- 返回: <boolean>

如果当前在传出的响应头中设置了由 `name` 标识的响应头，则返回 `true`。 请注意，响应头名称匹配不区分大小写。

```js
const hasContentType = response.hasHeader('content-type');
```

### response.headersSent

新增于: v0.9.3

- <boolean>

布尔值（只读）。 如果已发送响应头，则为 `true`，否则为 `false`。

### response.removeHeader(name)

新增于: v0.4.0

- `name` <string>

移除排队等待中的隐式发送的响应头。

```js
response.removeHeader('Content-Encoding');
```

### response.sendDate

新增于: v0.7.5

- <boolean>

如果为 `true`，则 Date 响应头将自动生成并在响应中发送（如果响应头中尚不存在）。 默认为 `true`。

这应该仅在测试时才禁用，HTTP 响应需要 Date 响应头。

### response.setHeader(name, value)

新增于: v0.4.0

- `name` <string>
- `value` <any>

为隐式响应头设置单个响应头的值。 如果此响应头已存在于待发送的响应头中，则其值将被替换。 在这里可以使用字符串数组来发送具有相同名称的多个响应头。 非字符串值将被原样保存。 因此 [`response.getHeader()`](http://nodejs.cn/s/pbE7VN) 可能返回非字符串值。 但是非字符串值将转换为字符串以进行网络传输。

```js
response.setHeader('Content-Type', 'text/html');
```

或：

```js
response.setHeader('Set-Cookie', ['type=ninja', 'language=javascript']);
```

尝试设置包含无效字符的响应头字段名称或值将导致抛出 [`TypeError`](http://nodejs.cn/s/Z7Lqyj)。

当使用 [`response.setHeader()`](http://nodejs.cn/s/rqeM3J) 设置响应头时，它们将与传给 [`response.writeHead()`](http://nodejs.cn/s/fnj9oM) 的任何响应头合并，其中 [`response.writeHead()`](http://nodejs.cn/s/fnj9oM) 的响应头优先。

```js
// 返回 content-type = text/plain
const server = http.createServer((req, res) => {
  res.setHeader('Content-Type', 'text/html');
  res.setHeader('X-Foo', 'bar');
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('ok');
});
```

如果调用了 [`response.writeHead()`](http://nodejs.cn/s/fnj9oM) 方法并且尚未调用此方法，则它将直接将提供的响应头值写入网络通道而不在内部进行缓存，并且响应头上的 [`response.getHeader()`](http://nodejs.cn/s/pbE7VN) 将不会产生预期的结果。 如果需要渐进的响应头填充以及将来可能的检索和修改，则使用 [`response.setHeader()`](http://nodejs.cn/s/rqeM3J) 而不是 [`response.writeHead()`](http://nodejs.cn/s/fnj9oM)。

### response.setTimeout(msecs[, callback])

新增于: v0.9.12

- `msecs` <number>
- `callback` <Function>
- 返回: <http.ServerResponse>

将套接字的超时值设置为 `msecs`。 如果提供了回调，则会将其作为监听器添加到响应对象上的 `'timeout'` 事件中。

如果没有 `'timeout'` 监听器添加到请求、响应、或服务器，则套接字在超时时将被销毁。 如果有回调处理函数分配给请求、响应、或服务器的 `'timeout'` 事件，则必须显式处理超时的套接字。

### response.socket

新增于: v0.3.0

- <net.socket>

指向底层的套接字。 通常用户不需要访问此属性。 特别是，由于协议解析器附加到套接字的方式，套接字将不会触发 `'readable'` 事件。 在调用 `response.end()` 之后，此属性将为空。 也可以通过 `response.connection` 访问 `socket`。

```js
const http = require('http');
const server = http.createServer((req, res) => {
  const ip = res.socket.remoteAddress;
  const port = res.socket.remotePort;
  res.end(`您的 IP 地址是 ${ip}，您的源端口是 ${port}`);
}).listen(3000);
```

### response.statusCode

新增于: v0.4.0

- <number>

当使用隐式的响应头时（没有显式地调用 [`response.writeHead()`](http://nodejs.cn/s/fnj9oM)），此属性控制在刷新响应头时将发送到客户端的状态码。

```js
response.statusCode = 404;
```

响应头发送到客户端后，此属性表示已发送的状态码。

### response.statusMessage

新增于: v0.11.8

- <string>

当使用隐式的响应头时（没有显式地调用 [`response.writeHead()`](http://nodejs.cn/s/fnj9oM)），此属性控制在刷新响应头时将发送到客户端的状态消息。 如果保留为 `undefined`，则将使用状态码的标准消息。

```js
response.statusMessage = 'Not found';
```

响应头发送到客户端后，此属性表示已发送的状态消息。

### response.write(chunk[, encoding][, callback])

新增于: v0.1.29

- `chunk` <string> | <Buffer>
- `encoding` <string> **默认值:** `'utf8'`。
- `callback` <Function>
- 返回: <boolean>

如果调用此方法并且尚未调用 [`response.writeHead()`](http://nodejs.cn/s/fnj9oM)，则将切换到隐式响应头模式并刷新隐式响应头。

这会发送一块响应主体。 可以多次调用该方法以提供连续的响应主体片段。

注意，在 `http` 模块中，当请求是 HEAD 请求时，则省略响应主体。 同样地， `204` 和 `304` 响应不得包含消息主体。

`chunk` 可以是字符串或 buffer。 如果 `chunk` 是一个字符串，则第二个参数指定如何将其编码为字节流。 当刷新此数据块时将调用 `callback`。

这是原始的 HTTP 主体，与可能使用的更高级别的多部分主体编码无关。

第一次调用 [`response.write()`](http://nodejs.cn/s/gvWo3m) 时，它会将缓冲的响应头信息和主体的第一个数据块发送给客户端。 第二次调用 [`response.write()`](http://nodejs.cn/s/gvWo3m) 时，Node.js 假定数据将被流式传输，并分别发送新数据。 也就是说，响应被缓冲到主体的第一个数据块。

如果将整个数据成功刷新到内核缓冲区，则返回 `true`。 如果全部或部分数据在用户内存中排队，则返回 `false`。 当缓冲区再次空闲时，则触发 `'drain'` 事件。

### response.writeContinue()

新增于: v0.3.0

向客户端发送 `HTTP/1.1 100 Continue` 消息，表示应发送请求主体。 请参阅 `Server` 上的 [`'checkContinue'`](http://nodejs.cn/s/ixa4zt) 事件。

### response.writeHead(statusCode[, statusMessage][, headers])

版本历史

- `statusCode` <number>
- `statusMessage` <string>
- `headers` <Object> 

向请求发送响应头。 状态码是一个 3 位的 HTTP 状态码，如 `404`。 最后一个参数 `headers` 是响应头。 可以可选地将用户可读的 `statusMessage` 作为第二个参数。

```js
const body = 'hello world';
response.writeHead(200, {
  'Content-Length': Buffer.byteLength(body),
  'Content-Type': 'text/plain' });
```

此方法只能在消息上调用一次，并且必须在调用 [`response.end()`](http://nodejs.cn/s/sqAtet) 之前调用。

如果在调用此方法之前调用了 [`response.write()`](http://nodejs.cn/s/gvWo3m) 或 [`response.end()`](http://nodejs.cn/s/sqAtet)，则将计算隐式或可变的响应头并调用此函数。

当使用 [`response.setHeader()`](http://nodejs.cn/s/rqeM3J) 设置响应头时，则与传给 [`response.writeHead()`](http://nodejs.cn/s/fnj9oM) 的任何响应头合并，且 [`response.writeHead()`](http://nodejs.cn/s/fnj9oM) 的优先。

如果调用此方法并且尚未调用 [`response.setHeader()`](http://nodejs.cn/s/rqeM3J)，则直接将提供的响应头值写入网络通道而不在内部进行缓存，响应头上的 [`response.getHeader()`](http://nodejs.cn/s/pbE7VN) 将不会产生预期的结果。 如果需要渐进的响应头填充以及将来可能的检索和修改，则改用 [`response.setHeader()`](http://nodejs.cn/s/rqeM3J)。

```js
// 返回 content-type = text/plain
const server = http.createServer((req, res) => {
  res.setHeader('Content-Type', 'text/html');
  res.setHeader('X-Foo', 'bar');
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('ok');
});
```

注意，Content-Length 以字节而非字符为单位。 上面的示例可行是因为字符串 `'hello world'` 仅包含单字节字符。 如果主体包含更高编码的字符，则应使用 `Buffer.byteLength()` 来判断给定编码中的字节数。 Node.js 不检查 Content-Length 和已传输的主体的长度是否相等。

尝试设置包含无效字符的响应头字段名称或值将导致抛出 [`TypeError`](http://nodejs.cn/s/Z7Lqyj)。

### response.writeProcessing()

新增于: v10.0.0

向客户端发送 `HTTP/1.1 102` 处理消息，表明可以发送请求主体。

## http.IncomingMessage 类

新增于: v0.1.17

`IncomingMessage` 对象由 [`http.Server`](http://nodejs.cn/s/jLiRTh) 或 [`http.ClientRequest`](http://nodejs.cn/s/2F5RHd) 创建，并分别作为第一个参数传给 [`'request'`](http://nodejs.cn/s/2qCn57) 和 [`'response'`](http://nodejs.cn/s/qwaiK8) 事件。 它可用于访问响应状态、消息头、以及数据。

它实现了[可读流](http://nodejs.cn/s/YuDKX1)接口，还有以下额外的事件、方法、以及属性。

### 'aborted' 事件

新增于: v0.3.8

当请求中止时触发。

### 'close' 事件

新增于: v0.4.2

表明底层连接已关闭。 与 `'end'` 事件一样，每个响应只触发一次此事件。

### message.aborted

新增于: v10.1.0

- <boolean>

如果请求已中止，则 `message.aborted` 属性为 `true`。

### message.complete

新增于: v0.3.0

- <boolean>

如果已收到并成功解析完整的 HTTP 消息，则 `message.complete` 属性将为 `true`。

此属性可用于判断客户端或服务器在连接终止之前是否完全传输消息：

```js
const req = http.request({
  host: '127.0.0.1',
  port: 8080,
  method: 'POST'
}, (res) => {
  res.resume();
  res.on('end', () => {
    if (!res.complete)
      console.error(
        '消息仍在发送时终止了连接');
  });
});
```

### message.destroy([error])

新增于: v0.3.0

- `error` <error>

在接收 `IncomingMessage` 的套接字上调用 `destroy()`。 如果提供了 `error`，则会触发 `'error'` 事件，并将 `error` 作为参数传递给事件上的任何监听器。

### message.headers

新增于: v0.1.5

- <Object> 

请求或响应的消息头对象。

消息头的名称和值的键值对。 消息头的名称都是小写的。

```js
// 打印类似以下：
//
// { 'user-agent': 'curl/7.22.0',
//   host: '127.0.0.1:8000',
//   accept: '*/*' }
console.log(request.headers);
```

原始消息头中的重复项会按以下方式处理，具体取决于消息头的名称：

- 重复的 `age`、 `authorization`、 `content-length`、 `content-type`、 `etag`、 `expires`、 `from`、 `host`、 `if-modified-since`、 `if-unmodified-since`、 `last-modified`、 `location`、 `max-forwards`、 `proxy-authorization`、 `referer`、 `retry-after` 或 `user-agent` 会被丢弃。
- `set-cookie` 始终是一个数组。重复项都会添加到数组中。
- 对于重复的 `cookie` 消息头，其值会使用与 '; ' 连接到一起。
- 对于所有其他消息头，其值会使用 ', ' 连接到一起。

### message.httpVersion

新增于: v0.1.1

- <string>

在服务器请求的情况下，表示客户端发送的 HTTP 版本。 在客户端响应的情况下，表示连接到的服务器的 HTTP 版本。 可能是 `'1.1'` 或 `'1.0'`。

另外， `message.httpVersionMajor` 是第一个整数， `message.httpVersionMinor` 是第二个整数。

### message.method

新增于: v0.1.1

- <string>

仅对从 [`http.Server`](http://nodejs.cn/s/jLiRTh) 获取的请求有效。

请求方法为字符串。 只读。 示例：`'GET'`、 `'DELETE'`。

### message.rawHeaders

新增于: v0.11.6

- <string>

原始请求头/响应头的列表，与接收到的完全一致。

请注意，键和值位于同一列表中。 它不是元组列表。 因此，偶数偏移是键值，奇数偏移是关联的值。

消息头名称不是小写的，并且不会合并重复项。

```js
// 打印类似于：
//
// [ 'user-agent',
//   '这是无效的，因为只能有一个值',
//   'User-Agent',
//   'curl/7.22.0',
//   'Host',
//   '127.0.0.1:8000',
//   'ACCEPT',
//   '*/*' ]
console.log(request.rawHeaders);
```

### message.rawTrailers

新增于: v0.11.6

- <string>

原始的请求/响应的尾部消息头的键和值，与接收到的完全一致。 仅在 `'end'` 事件中填充。

### message.setTimeout(msecs, callback)

新增于: v0.5.9

- `msecs` <number>
- `callback` <Function>
- 返回: <http.IncomingMessage>

调用 `message.connection.setTimeout(msecs, callback)`。

### message.socket

新增于: v0.3.0

- <net.socket>

与连接关联的 [`net.Socket`](http://nodejs.cn/s/wsJ1o1) 对象。

通过 HTTPS 的支持，使用 [`request.socket.getPeerCertificate()`](http://nodejs.cn/s/zQ1LhW) 获取客户端的身份验证详细信息。

### message.statusCode

新增于: v0.1.1

- <number>

仅对从 [`http.ClientRequest`](http://nodejs.cn/s/2F5RHd) 获取的响应有效。

3 位 HTTP 响应状态码。 例如 `404`。

### message.statusMessage

新增于: v0.11.10

- <string>

仅对从 [`http.ClientRequest`](http://nodejs.cn/s/2F5RHd) 获取的响应有效。

HTTP 响应状态消息（原因短语）。 例如 `OK` 或 `Internal Server Error`。

### message.trailers

新增于: v0.3.0

- <Object> 

请求/响应的尾部消息头对象。 仅在 `'end'` 事件中填充。

### message.url

新增于: v0.1.90

- <string>

仅对从 [`http.Server`](http://nodejs.cn/s/jLiRTh) 获取的请求有效。

请求的 URL 字符串。 它仅包含实际 HTTP 请求中存在的 URL。 如果请求是：

```txt
GET /status?name=ryan HTTP/1.1\r\n
Accept: text/plain\r\n
\r\n
```

则 `request.url` 将是：

```js
'/status?name=ryan'
```

要将 url 解析为其各个部分，可以使用 `require('url').parse(request.url)`：

```txt
$ node
> require('url').parse('/status?name=ryan')
Url {
  protocol: null,
  slashes: null,
  auth: null,
  host: null,
  port: null,
  hostname: null,
  hash: null,
  search: '?name=ryan',
  query: 'name=ryan',
  pathname: '/status',
  path: '/status?name=ryan',
  href: '/status?name=ryan' }
```

要从查询字符串中提取参数，可以使用 `require('querystring').parse` 函数，或者可以将 `true` 作为第二个参数传递给 `require('url').parse`：

```txt
$ node
> require('url').parse('/status?name=ryan', true)
Url {
  protocol: null,
  slashes: null,
  auth: null,
  host: null,
  port: null,
  hostname: null,
  hash: null,
  search: '?name=ryan',
  query: { name: 'ryan' },
  pathname: '/status',
  path: '/status?name=ryan',
  href: '/status?name=ryan' }
```

## http.METHODS

新增于: v0.11.8

- <string>

解析器支持的 HTTP 方法列表。

## http.STATUS_CODES

新增于: v0.1.22

- <Object> 

所有标准 HTTP 响应状态码的集合，以及每个状态码的简短描述。 例如， `http.STATUS_CODES[404] === 'Not Found'`。

## http.createServer([options][, requestlistener])

版本历史

- `options` <Object> 
  - `IncomingMessage` <http.IncomingMessage> 指定要使用的 `IncomingMessage` 类。用于扩展原始的 `IncomingMessage`。**默认值:** `IncomingMessage`。
  - `ServerResponse` <http.ServerResponse> 指定要使用的 `ServerResponse` 类。用于扩展原始 `ServerResponse`。**默认值:** `ServerResponse`。
- `requestListener` <Function>
- 返回: <http.Server>

返回新建的 [`http.Server`](http://nodejs.cn/s/jLiRTh) 实例。

`requestListener` 是一个自动添加到 [`'request'`](http://nodejs.cn/s/2qCn57) 事件的函数。

## http.get(options[, callback])

## http.get(url[, options][, callback])

版本历史

- `url` <string> | <URL>
- `options` <Object>  接受与 [`http.request()`](http://nodejs.cn/s/d1myoL) 相同的 `options`，且 `method` 始终设置为 `GET`。从原型继承的属性将被忽略。
- `callback` <Function>
- 返回: <http.ClientRequest>

由于大多数请求都是没有主体的 GET 请求，因此 Node.js 提供了这个便捷的方法。 这个方法与 [`http.request()`](http://nodejs.cn/s/d1myoL) 的唯一区别是它将方法设置为 GET 并自动调用 `req.end()`。 注意，由于 [`http.ClientRequest`](http://nodejs.cn/s/2F5RHd) 章节中所述的原因，回调必须注意消费响应数据。

`callback` 调用时只有一个参数，该参数是 [`http.IncomingMessage`]<http.IncomingMessage> 的实例。

获取 JSON 的示例：

```js
http.get('http://nodejs.cn/index.json', (res) => {
  const { statusCode } = res;
  const contentType = res.headers['content-type'];

  let error;
  if (statusCode !== 200) {
    error = new Error('请求失败\n' +
                      `状态码: ${statusCode}`);
  } else if (!/^application\/json/.test(contentType)) {
    error = new Error('无效的 content-type.\n' +
                      `期望的是 application/json 但接收到的是 ${contentType}`);
  }
  if (error) {
    console.error(error.message);
    // 消费响应数据来释放内存。
    res.resume();
    return;
  }

  res.setEncoding('utf8');
  let rawData = '';
  res.on('data', (chunk) => { rawData += chunk; });
  res.on('end', () => {
    try {
      const parsedData = JSON.parse(rawData);
      console.log(parsedData);
    } catch (e) {
      console.error(e.message);
    }
  });
}).on('error', (e) => {
  console.error(`出现错误: ${e.message}`);
});
```

## http.globalAgent

新增于: v0.5.9

- <http.Agent>

`Agent` 的全局实例，作为所有 HTTP 客户端请求的默认值。

## http.maxHeaderSize

新增于: v10.15.0

- <number>

只读属性，指定 HTTP 消息头的最大允许大小（以字节为单位）。 默认为 8KB。 可使用 [`--max-http-header-size`](http://nodejs.cn/s/HfsyuU) 命令行选项进行配置。

## http.request(options[, callback])

## http.request(url[, options][, callback])

版本历史

- `url` <string> | <URL>
- `options` <Object> 
  - `protocol` <string> 使用的协议。**默认值:** `'http:'`。
  - `host` <string> 请求发送至的服务器的域名或 IP 地址。**默认值:** `'localhost'`。
  - `hostname` <string> `host` 的别名。为了支持 [`url.parse()`](http://nodejs.cn/s/b28B2A)，如果同时指定 `host` 和 `hostname`，则使用 `hostname`。
  - `family` <number> 当解析 `host` 或 `hostname` 时使用的 IP 地址族。有效值为 `4` 或 `6`。如果没有指定，则同时使用 IP v4 和 v6。
  - `port` <number> 远程服务器的端口。**默认值:** `80`。
  - `localAddress` <string> 为网络连接绑定的本地接口。
  - `socketPath` <string> Unix 域套接字。如果指定了 `host` 或 `port` 之一（它们指定了 TCP 套接字），则不能使用此选项。
  - `method` <string> 一个字符串，指定 HTTP 请求的方法。**默认值:** `'GET'`。
  - `path` <string> 请求的路径。应包括查询字符串（如果有）。例如 `'/index.html?page=12'`。当请求的路径包含非法的字符时，则抛出异常。目前只有空格被拒绝，但未来可能会有所变化。**默认值:** `'/'`。
  - `headers` <Object>  包含请求头的对象。
  - `auth` <string> 基本的身份验证，即 `'user:password'`，用于计算授权请求头。
  - `agent` <http.Agent> | <boolean> 控制 [`Agent`](http://nodejs.cn/s/HRCnER) 的行为。可能的值有：
    - `undefined` (默认): 对此主机和端口使用 [`http.globalAgent`](http://nodejs.cn/s/g7BhW2)。
    - `Agent` 对象: 显式地使用传入的 `Agent`。
    - `false`: 使用新建的具有默认值的 `Agent`。
  - `createConnection` <Function> 当 `agent` 选项未被使用时，用来为请求生成套接字或流的函数。这可用于避免创建自定义的 `Agent` 类以覆盖默认的 `createConnection` 函数。详见 [`agent.createConnection()`](http://nodejs.cn/s/nH3X12)。任何[双工流](http://nodejs.cn/s/2iRabr)都是有效的返回值。
  - `timeout` <number>: 指定套接字超时的数值，以毫秒为单位。这会在套接字被连接之前设置超时。
  - `setHost` <boolean>: 指定是否自动添加 `Host` 请求头。**默认值:** `true`。
- `callback` <Function>
- 返回: <http.ClientRequest>

Node.js 为每个服务器维护多个连接以发出 HTTP 请求。 此函数允许显式地发出请求。

`url` 可以是字符串或 [`URL`](http://nodejs.cn/s/5dwq7G) 对象。 如果 `url` 是一个字符串，则会自动使用 [`url.parse()`](http://nodejs.cn/s/b28B2A) 解析它。 如果它是一个 [`URL`](http://nodejs.cn/s/5dwq7G) 对象，则会自动转换为普通的 `options` 对象。

如果同时指定了 `url` 和 `options`，则对象会被合并，其中 `options` 属性优先。

可选的 `callback` 参数会作为单次监听器被添加到 [`'response'`](http://nodejs.cn/s/qwaiK8) 事件。

`http.request()` 返回 [`http.ClientRequest`](http://nodejs.cn/s/2F5RHd) 类的实例。 `ClientRequest` 实例是一个可写流。 如果需要使用 POST 请求上传文件，则写入到 `ClientRequest` 对象。

```js
const postData = querystring.stringify({
  'msg': '你好世界'
});

const options = {
  hostname: 'nodejs.cn',
  port: 80,
  path: '/upload',
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': Buffer.byteLength(postData)
  }
};

const req = http.request(options, (res) => {
  console.log(`状态码: ${res.statusCode}`);
  console.log(`响应头: ${JSON.stringify(res.headers)}`);
  res.setEncoding('utf8');
  res.on('data', (chunk) => {
    console.log(`响应主体: ${chunk}`);
  });
  res.on('end', () => {
    console.log('响应中已无数据');
  });
});

req.on('error', (e) => {
  console.error(`请求遇到问题: ${e.message}`);
});

// 将数据写入请求主体。
req.write(postData);
req.end();
```

注意，在示例中调用了 `req.end()`。 使用 `http.request()` 时，必须始终调用 `req.end()` 来表示请求的结束，即使没有数据被写入请求主体。

如果在请求期间遇到任何错误（DNS 解析错误、TCP 层的错误、或实际的 HTTP 解析错误），则会在返回的请求对象上触发 `'error'` 事件。 与所有 `'error'` 事件一样，如果没有注册监听器，则会抛出错误。

以下是需要注意的一些特殊的请求头。

- 发送 `'Connection: keep-alive'` 会通知 Node.js 与服务器的连接应该持续到下一个请求。
- 发送 `'Content-Length'` 请求头会禁用默认的分块编码。
- 发送 `'Expect'` 请求头会立即发送请求头。通常情况下，当发送 `'Expect: 100-continue'` 时，应设置超时时间和 `'continue'` 事件的监听器。详见 RFC2616 的第 8.2.3 节。
- 发送授权请求头会使用 `auth` 选项覆盖以计算基本的身份验证。

使用 [`URL`](http://nodejs.cn/s/5dwq7G) 作为 `options` 的示例：

```js
const options = new URL('http://abc:xyz@nodejs.cn');

const req = http.request(options, (res) => {
  // ...
});
```

在成功的请求中，会按以下顺序触发以下事件：

- `'socket'` 事件
- `'response'` 事件
  - `res` 对象上任意次数的 `'data'` 事件（如果响应主体为空，则根本不会触发 `'data'` 事件，例如在大多数重定向中）
  - `res` 对象上的 `'end'` 事件
- `'close'` 事件

如果出现连接错误，则触发以下事件：

- `'socket'` 事件
- `'error'` 事件
- `'close'` 事件

如果在连接成功之前调用 `req.abort()`，则按以下顺序触发以下事件：

- `'socket'` 事件
- (在这里调用 `req.abort()`)
- `'abort'` 事件
- `'error'` 事件并带上错误信息 `'Error: socket hang up'` 和错误码 `'ECONNRESET'`
- `'close'` 事件

如果在响应被接收之后调用 `req.abort()`，则按以下顺序触发以下事件：

- `'socket'` 事件
- `'response'` 事件
  - `res` 对象上任意次数的 `'data'` 事件
- (在这里调用 `req.abort()`)
- `'abort'` 事件
- `res` 对象上的 `'aborted'` 事件
- `'close'` 事件
- `res` 对象上的 `'end'` 事件
- `res` 对象上的 `'close'` 事件

注意，设置 `timeout` 选项或使用 `setTimeout()` 函数不会中止请求或执行除添加 `'timeout'` 事件之外的任何操作。