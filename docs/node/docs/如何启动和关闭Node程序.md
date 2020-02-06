### 如何启动 Node 程序

```shell
node [fileName]
```

### 如何停止 Node 程序

#### Windows 

1. 打开 CMD 或者 PowerShell

2. 通过端口查找进程 ID

   ```shell
   > netstat -ano|findstr PORT
     TCP    0.0.0.0:3000           0.0.0.0:0              LISTENING       7160
     TCP    [::]:3000              [::]:0                 LISTENING       7160
     TCP    [::1]:3000             [::1]:60242            TIME_WAIT       0
     TCP    [::1]:3000             [::1]:60269            TIME_WAIT       0
     TCP    [::1]:60243            [::1]:3000             TIME_WAIT       0
     TCP    [::1]:60270            [::1]:3000             TIME_WAIT       0
   ```

3. 通过进程 ID 查看进程名称

   ```SHELL
   >tasklist |findstr 7160
   node.exe                      7160 Console                    2     20,924 K
   ```

4. 根据进程名称关闭 Node 进程

   ```shell
   >taskkill /f /t /im node.exe
   成功: 已终止 PID 7160 (属于 PID 7336 子进程)的进程。
   ```

#### Linux

1. 找到进程 ID

   ```shell
   >ps -ef|grep node
   ```

2. 关闭进程

   ```shell
   kill PID
   ```

   



