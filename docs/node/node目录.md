### 什么是 Node.js

Node.js 就是运行在服务端的 JavaScript。Node.js是一个事件驱动I/O服务端JavaScript环境，基于Google的V8引擎，V8引擎执行Javascript的速度非常快，性能非常好。非常适合 IO 密集型应用。使用事件驱动模型。

### 安装 Node.js

官网下载然后一直下一步到结束即可，没什么难的。安装结束后打开 CMD，键入 `node -v` 和 `npm -v` 查看版本。

```shell
>node -v
v10.16.1
>npm -v
6.9.0
```

##### 参考： [配置 Sublime Text3 运行 Node.js](./docs/SublimeText3配置Nodejs插件)

### HelloWorld

[如何启动和关闭一个 Node 程序](./docs/如何启动和关闭Node程序.md)

### Node.js 有哪些内容

- [NPM简介](./docs/NPM简介.md) - 包管理工具

- [REPL简介](./docs/REPL简介.md) - Node 自带的交互式解释器

- [模块module](./docs/模块module.md) - 每个文件都被视为一个独立的模块

- [事件触发器](./docs/事件触发器events.md) - events

- [缓冲器](./docs/缓冲器Buffer.md) - Buffer

- [流](./docs/stream流.md) - Stream

- [文件系统](./docs/文件系统fs.md) - fs

- 网络处理

  > [http](./docs/http.md) - HTTP 服务
  >
  > [http2](http://nodejs.cn/api/http2.html) - HTTP2
  >
  > [https](http://nodejs.cn/api/https.html) - TTTPS
  >
  > [url](./docs/url.md) - 处理 URL
  >
  > [querystring](./docs/querystring.md) - 处理 GET 请求中的参数
  >
  > [处理GET和POST请求](./docs/处理GET和POST请求.md) 

- 其他

  > [dns](http://nodejs.cn/api/dns.html) - 域名服务器
  >
  > [path](http://nodejs.cn/api/path.html) - 处理文件路径和目录路径
  >
  > [net](http://nodejs.cn/api/net.html) - 创建基于流的 TCP 或 [IPC](http://nodejs.cn/s/rAdYjf) 的服务器与客户端
  >
  > [dgram](http://nodejs.cn/api/dgram.html) - UPD 数据包 socket 的实现
  >
  > [domain]() - 简化异步代码的异常处理，可以捕捉处理try catch无法捕捉的
  >
  > [os](http://nodejs.cn/api/os.html) - 基本的系统操作
  >
  > [error](http://nodejs.cn/api/errors.html) - 异常
  >
  > [readline](http://nodejs.cn/api/readline.html) - 逐行读取可读流
  >
  > [string_decoder](http://nodejs.cn/api/string_decoder.html) - 字符串解码器
  >
  > [timer](http://nodejs.cn/api/timers.html) - 定时器
  >
  > [util](http://nodejs.cn/api/util.html) - 工具
  >
  > [cluster](http://nodejs.cn/api/cluster.html) - 集群
  >
  > [assert](http://nodejs.cn/api/assert.html) - 断言

- 数据库

  > MySQL
  >
  > MongoDB
  >
  > 

- expressjs - web 框架

### [基于 Node 的 15 个应用场景](./docs/基于Node的15个应用场景.md)

