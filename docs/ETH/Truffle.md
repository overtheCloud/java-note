## Truffle

一个简化的智能合约开发、测试环境。

>官网链接 https://www.trufflesuite.com/docs/truffle/quickstart

####  安装

```shell
npm install -g truffle
```

#### 初始化

Truffle 环境可以打成包，并被其他人下载引用。

```
# 通过下载 Box 初始化环境，与下面的命令二选一执行
truffle unbox metacoin
# 创建一个最基础的环境
truffle init
```

初始化完成后，会在当前目录下生成以下文件

- contract/* 合约文件
- migration/ 部署脚本
- test/ 测试文件
- truffle.js 配置文件

#### 编译

```shell
truffle complile
```

编译能部署到合约中的文件。

##### 部署

Truffle 有两种部署的方式，一种是部署到 Truffle 自带的私有链。一种是部署到 Ganache 搭建的私有链。

##### 部署到自带的私有链

```shell
# 执行后会打开下面的交互终端
truffle develop
# 执行部署， 会打印合约调用信息
truffle(development)>migrate
```

##### 部署到 Ganache

- 修改配置文件 truffle.js

```js
module.exports = {
  networks: {
    development: {
      host: "127.0.0.1",
      port: 7545,
      network_id: "*"
    }
  }
};
```

- 打开 Ganache
- 执行部署

```shell
# 直接部署
truffle migrate
```

#### 调用合约

```shell
# 打开交互终端
truffle console
# 
truffle(development)>let instance = await MetaCoin.deployed()
truffle(development)>let accounts = await web3.eth.getAccounts()
truffle(development)>let balance = await instance.getBalance(accounts[0])
truffle(development)>balance.toNumber()
truffle(development)>let ether = await instance.getBalanceInEth(accounts[0])
truffle(development)>ether.toNumber()
truffle(development)>instance.sendCoin(accounts[1], 500)
truffle(development)>let received = await instance.getBalance(accounts[1])
truffle(development)>received.toNumber()
truffle(development)>let newBalance = await instance.getBalance(accounts[0])
truffle(development)>newBalance.toNumber()
```



## Ganache

一个简化的私有链。

>官网链接  https://www.trufflesuite.com/ganache

