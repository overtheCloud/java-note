## Hello World

创建一个最简单的服务器 server.js

```javascript
// 获取 http 模块
var http = require('http');
// 创建服务
http.createServer(function (request, response) {
    // 设置响应头
    response.writeHead(200, {'Content-Type': 'text/plain'});
    // 输出响应信息
    response.end('Hello World');
}).listen(8088); // 指定端口
```

通过命令执行

```shell
node server.js
```

在浏览器访问 `http://localhost:8088` 看到页面显示 `Hello World` 表示成功。

