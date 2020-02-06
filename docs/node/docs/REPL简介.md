### 什么是 REPL

REPL 是 Node.js 自带的一个命令行交互式解释器，类似 Windows 上的 CMD 和 Linux 的终端和 Shell。

#### 打开REPL

```shell
$ node
>       # 已经打开了 REPL ，键入指令然后 enter 执行 
```

### 示例

```shell  
$ node
> 1 + 1   # 执行计算
2
> 2       # 直接输出
2
> var a = 1 # 定义变量并赋值
undefined
> console.log(a) # 执行打印操作
1
undefined
> do {     # 执行多行指令，输入 { 后回车会自动
... a++;
... console.log(a);
... }while(a < 3);
2
3
undefined
> 2
2
undefined
> console.log(_)  # 使用 _ 获取上一个表达式的结果
2
undefined
```

