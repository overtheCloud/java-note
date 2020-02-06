1. **Web开发：Express + EJS + Mongoose/MySQL**

[express](http://expressjs.com/) 是轻量灵活的Nodejs Web应用框架，它可以快速地搭建网站。Express框架建立在Nodejs内置的Http模块上，并对Http模块再包装，从而实际Web请求处理的功能。

[ejs](https://github.com/visionmedia/ejs)是一个嵌入的Javascript模板引擎，通过编译生成HTML的代码。

[mongoose](http://mongoosejs.com/) 是MongoDB的对象模型工具，通过Mongoose框架，可以进行访问MongoDB的操作。

[mysql](https://github.com/felixge/node-mysql) 是连接MySQL数据库的通信API，可以进行访问MySQL的操作。

2. **REST开发：Restify**

[restify](http://mcavage.me/node-restify/) 是一个基于Nodejs的REST应用框架，支持服务器端和客户端。restify比起express更专注于REST服务，去掉了express中的template, render等功能，同时强化了REST协议使用，版本化支持，HTTP的异常处理。

3. **Web聊天室(IM)：Express + Socket.io**

[socket.io](http://socket.io/)一个是基于Nodejs架构体系的，支持websocket的协议用于时时通信的一个软件包。socket.io 给跨浏览器构建实时应用提供了完整的封装，socket.io完全由javascript实现。

4. **Web爬虫：Cheerio/Request**

[cheerio](http://matthewmueller.github.io/cheerio/) 是一个为服务器特别定制的，快速、灵活、封装jQuery核心功能工具包。Cheerio包括了 jQuery核心的子集，从jQuery库中去除了所有DOM不一致性和浏览器不兼容的部分，揭示了它真正优雅的API。Cheerio工作在一个非常简单，一致的DOM模型之上，解析、操作、渲染都变得难以置信的高效。基础的端到端的基准测试显示Cheerio大约比JSDOM快八倍(8x)。Cheerio封装了@FB55兼容的htmlparser，几乎能够解析任何的 HTML 和 XML document。

5. **Web博客：Hexo**

[Hexo](http://hexo.io/) 是一个简单地、轻量地、基于Node的一个静态博客框架。通过Hexo我们可以快速创建自己的博客，仅需要几条命令就可以完成。

发布时，Hexo可以部署在自己的Node服务器上面，也可以部署github上面。对于个人用户来说，部署在github上好处颇多，不仅可以省去服务器的成本，还可以减少各种系统运维的麻烦事(系统管理、备份、网络)。所以，基于github的个人站点，正在开始流行起来….

6.  **Web论坛: nodeclub**

[Node Club](https://github.com/cnodejs/nodeclub/) 是用 Node.js 和 MongoDB 开发的新型社区软件，界面优雅，功能丰富，小巧迅速， 已在Node.js 中文技术社区 [CNode](http://cnodejs.org/) 得到应用，但你完全可以用它搭建自己的社区。

7. **Web幻灯片：Cleaver**

[Cleaver](http://jdan.github.io/cleaver/) 可以生成基于Markdown的演示文稿。如果你已经有了一个Markdown的文档，30秒就可以制作成幻灯片。Cleaver是为Hacker准备的工具。

8. **前端包管理平台: bower.js**

[Bower](http://bower.io/) 是 twitter 推出的一款包管理工具，基于nodejs的模块化思想，把功能分散到各个模块中，让模块和模块之间存在联系，通过 Bower 来管理模块间的这种联系。

9. **OAuth认证：Passport**

[Passport](http://passportjs.org/)项目是一个基于Nodejs的认证中间件。Passport目的只是为了“登陆认证”，因此，代码干净，易维护，可以方便地集成到其他的应用中。Web应用一般有2种登陆认证的形式：用户名和密码认证登陆,OAuth认证登陆。Passport可以根据应用程序的特点，配置不同的认证机制。本文将介绍，用户名和密码的认证登陆。

10. **定时任务工具: later**

[Later](http://bunkat.github.io/later/) 是一个基于Nodejs的工具库，用最简单的方式执行定时任务。Later可以运行在Node和浏览器中。

11. **浏览器环境工具: browserify**

[Browserify](http://browserify.org/) 的出现可以让Nodejs模块跑在浏览器中，用require()的语法格式来组织前端的代码，加载npm的模块。在浏览器中，调用browserify编译后的代码，同样写在<script>标签中。

用 Browserify 的操作，分为3个步骤。1. 写node程序或者模块, 2. 用Browserify 预编译成 bundle.js, 3. 在HTML页面中加载bundle.js。

12. **命令行编程工具：Commander**

[commander](http://visionmedia.github.io/commander.js/) 是一个轻巧的nodejs模块，提供了用户命令行输入和参数解析强大功能。commander源自一个同名的Ruby项目。commander的特性：自记录代码,自动生成帮助,合并短参数（“ABC”==“-A-B-C”）,默认选项,强制选项,命令解析,提示符。

13. **Web控制台工具: tty.js**

[tty.js](https://github.com/chjj/tty.js/) 是一个支持在浏览器中运行的命令行窗口，基于node.js平台，依赖socket.io库，通过websocket与Linux系统通信。特性：支持多tab窗口模型; 支持vim,mc,irssi,vifm语法; 支持xterm鼠标事件; 支持265色显示; 支持session。

14. **客户端应用工具: node-webkit**

[Node-Webkit](https://github.com/rogerwang/node-webkit) 是NodeJS与WebKit技术的融合，提供一个跨Windows、Linux平台的客户端应用开发的底层框架，利用流行的Web技术（Node.JS，JavaScript，HTML5）来编写应用程序的平台。应用程序开发人员可以轻松的利用Web技术来实现各种应用程序。Node-Webkit性能和特色已经让它成为当今世界领先的Web技术应用程序平台。

15. **操作系统: node-os**

[NodeOS](http://node-os.com/) 是采用NodeJS开发的一款友好的操作系统，该操作系统是完全建立在Linux内核之上的，并且采用shell和NPM进行包管理，采用NodeJS不仅可以很好地进行包管理，还可以很好的管理脚本、接口等。目前，Docker和Vagrant都是采用NodeOS的首个版本进行构建的。