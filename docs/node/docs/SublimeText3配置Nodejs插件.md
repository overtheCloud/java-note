## SublimeText3 配置 NodeJs 插件

1. 在 Sublime Text3 中安装 Node.js 插件

`ctrl` + `shift` + `p` 后输入 `ins` 找到 `Package Control:Install Package` ，然后回车确定。

首选项 -> Package Control -> 搜索 Nodejs -> 回车确定。等待安装插件成功。

2. 修改插件的配置文件

首选项 -> 浏览插件目录 -> 找到 Nodejs 文件夹，修改下面两个文件

Nodejs.sublime-settings

```json
{
  "save_first": true,
  // ！！！ 此处修改为 Nodejs 安装目录中的 node.exe 的绝对路径
  "node_command": "C:\Program Files\nodejs\node.exe",
  // ！！！ 此处修改为 Nodejs 安装目录中的 npm.cmd ，很多教程中此处是 npm.exe ，但是我安装的版本中没有这个运行程序
  "npm_command": "C:\Program Files\nodejs\npm.cmd",
  "node_path": false,
  "expert_mode": false,
  "output_to_new_tab": false
}
```

Nodejs.sublime-build

```json
{
  "cmd": ["node", "$file"],
  "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)",
  "selector": "source.js",
  "shell": true,
   // 修改编码
  "encoding": "utf8",
  "windows":
    {
        // ！！！ 此处修改
        "cmd": ["taskkill", "/F", "/IM", "node.exe", "&", "node", "$file"]
    },
    "linux":
    {
        "shell_cmd": "killall node; /usr/bin/env node $file"
    },
    "osx":
    {
        "shell_cmd": "killall node; /usr/bin/env node $file"
    }
}
```

3. 测试

在 Sublime Text3 新建一个文件 demo.js ：

```javascript
console.log('hello world');
```

保存后 `ctrl + b`运行 demo.js ,控制台打印:

```shell
hello world
```

#### 关闭 node

工具 -> 取消编译，快捷键 `ctrl + break`。