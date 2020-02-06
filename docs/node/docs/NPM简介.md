### 什么是 NNPM

包管理工具。

你的命令行程序和打包后的项目上传到 NPM 服务器供别人使用，也可以从 NPM 服务器下载别人的包和命令行程序使用。

#### 查看版本号

```shell
npm -v
```

#### 更新 npm

```shell
npm install npm -g
// 使用淘宝镜像代替官方服务器
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

#### 安装模块

```shell
npm install [module_name]
```

*全局安装和本地安装*

```shell
npm install [module_name]      # 本地安装
npm install [module_name] -g   # 全局安装
```

*本地安装*

- 将安装包放在 ./node_modules 下（运行 npm 命令时所在的目录），如果没有 node_modules 目录，会在当前执行 npm 命令的目录下生成 node_modules 目录。
- 可以通过 require() 来引入本地安装的包。

*全局安装*

- 将安装包放在 /usr/local 下或者你 node 的安装目录。
- 可以直接在命令行里使用。

>注意：遇到错误信息 r: connect ECONNREFUSED 127.0.0.1:8087  的解决办法是
>
>```shell
>npm config set proxy null
>```

### 卸载模块

1. 使用 `npm list -g` 查看模块信息，第一行是模块所在目录信息
2. 进入模块所在目录
3. 执行 `npm uninstall [module_name]`

#### 更新模块

```shell
npm update [module_name]
```

#### 搜索模块

```shell
npm search [module_name]
```

### 创建模块

会创建 package.json ,创建过程中会提醒你去设置部分参数，可以直接按`enter` 跳过。

```shell
npm init 
```

#### 查看安装的模块

```shell
npm list -g     # 查看全部模块
npm list grunt  # 查看某个模块的版本号
```

#### 清空 NPM 本地缓存

```shell
npm cache clear
```

#### 撤销自己发布过的本版

```shell
npm unpublish <package>@<version>
```

#### package.json

package.json 位于模块的目录下，用于定义包的属性。

*属性说明*

- **name** - 包名。
- **version** - 包的版本号。
- **description** - 包的描述。
- **homepage** - 包的官网 url 。
- **author** - 包的作者姓名。
- **contributors** - 包的其他贡献者姓名。
- **dependencies** - 依赖包列表。如果依赖包没有安装，npm 会自动将依赖包安装在 node_module 目录下。
- **repository** - 包代码存放的地方的类型，可以是 git 或 svn，git 可在 Github 上。
- **main** - main 字段指定了程序的主入口文件，require('moduleName') 就会加载这个文件。这个字段的默认值是模块根目录下面的 index.js。
- **keywords** - 关键字

