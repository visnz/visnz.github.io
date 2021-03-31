---
title: "课程设计Linux Docker构建区块链2：基础设施搭建"
date: 2019-01-08
type: ["应用"]
weight: 6
tags: ["区块链","计算机","服务器","Linux"]
categories: ["运维","服务器","计算机"]
description: "在Linux平台上使用Docker创建节点，使用geth基础工具与web3.js基本框架实现区块链的基础设施搭建"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/icon.jpg"
---
## 写在前面
由于一些工作原因，没法使用到老师的一键部署工具。课程设计给了一个机会接触分布式技术以太坊的基本部署
### TL;DR
> 本文简述了在Linux平台上使用Docker构建以太坊工具geth的基础环境，后记录了创建私有链的过程，包括创世区块、节点部署、链接节点、产生交易、编写部署合约，以及最后web3组件对geth进行更多的扩展开发尝试。


## 初步构建与测试
一开始先进入容器熟悉部署环境，最后再使用Dockerfile整理。
### 1. 系统安装
这次选了``base/archlinux``的镜像因为community里直接就有打包好的``go-ethereum``（[官方安装文档](https://github.com/ethereum/go-ethereum/wiki/Building-Ethereum)有arch支持）。

debian系貌似gpgkey有点小问题可以手动解决。

构建镜像并进入``docker run -dit base/archlinux`` 

```sh
pacman -Sy python2 npm geth
# 同步并安装必要包
npm install -g solc
```

可以``docker commit``成快照，相当于dockerfile的一个阶段构建

### 2. 节点启动与用户创建

调用``geth``启动节点。``--testnet``进入全网的测试网。
测试网的数据会同步在``~/.ethash/testnet``下

不带testnet的话连入公网，当前公网数据大约需要下载2天（老师原话，未告知网速）
```sh
geth [--testnet]
```

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/01.png)

geth创建账户测试与数据结构：``geth  --testnet account new``
默认是全网，记得加``--testnet``

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/02.png)

账户的唯一认证是密码（没有账户名）
创建后会产生一个地址值（如图）就是账户的唯一识别

在``dir=~/.ethash/testnet/keystore/``下会保存刚刚的账户信息，包括这个地址值、创建时间等信息。注意保护好。

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/03.png)

查询所在账户：
``geth --testnet account list``

## 创建私有链

创建一个私有链的基本文件夹，在里面创建一个``创世区块``

``mkdir ~/blockchain/private-chain/node1 -p``

创世区块的基本配置
```json
//genesis.json
{
  "config": {
        "chainId": 10,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
  "alloc"      : {},
  "coinbase"   : "0x0000000000000000000000000000000000000000",
  //矿工的账号，创世区块可随意填写
  "difficulty" : "0x20000",
  //设置当前区块的难度，越大越容易
  "extraData"  : "",
  "gasLimit"   : "0x2fefd8",
  //该值设置对GAS的消耗总量限制，用来限制区块能包含的交易信息总和。
  "nonce"      : "0x0000000000000042",
  //nonce就是一个64位随机数，用于挖矿
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  //与nonce配合用于挖矿，由上一个区块的一部分生成的hash
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  //上一个区块的hash值，因为是创世块，故0
  "timestamp"  : "0x00"
}
```

配置文件建议使用官方提供的创世区块配置，或者最小配置：
```json
{
    "config":{
        "chainId": 15,
        "homesteadBlock": 0,
        "eip158Block": 0,
        "eip158Block": 0
    },
    "difficulty": "10000",
    "gasLimit": "2100000",
    "alloc":{}
}
```

### 1. 初始化创世区块

```sh
geth init ~/blockchain/private-chain/node1/genesis.json --datadir ~/blockchain/private-chain/node1/
```

其中指定了节点的创世区块与节点的数据存储位置，使用``geth init``完成初始化

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/04.png)

### 2. 部署节点
```sh
geth --identity "node1" --rpc --rpcport 1111 --rpccorsdomain "*" --datadir "/root/blockchain/private-chain/node1/" --port 6666  --ipcpath "geth\geth1.ipc" --networkid 10
```
其中

- identity表示节点的区分标志 
- rpc启动rpc通信，可以进行智能合约的部署和调试 
- rpcport rpc接口的端口号（1111,2222）
- port 指定该节点的端口号（6666,7777）
- ipcpath 指定进程通信文件，会存放在刚刚创建的node文件夹下
- networkid 指定网络号

过程遇到的问题：在容器内无法将rpc访问地址映射到外面来访问

1. 原因：geth默认是将访问端口映射到localhost，docker连接的是docker的虚拟网卡。故将geth的rpc端口指定到docker内的网卡即可。
2. 解决：在 geth开启矿工（指定rpc端口、ipc文件的命令）同时追加``--nat extip:172.17.0.2 --rpcaddr 172.17.0.2``指定rpc映射在docker的网卡上，指定nat模式对外映射。同时别忘了开放容器端口。

### 3. 启动控制台

执行上一部分的部署便启动了该节点，调用``geth attach <ipc file>``接入节点，控制台形式

控制台提供一个json对象访问：(提供补全)

行为|指令
---|---
创建账户(与geth功能一致) | personal.newAccount(“密码”)
查看有哪些账户 | eth.accounts
查看账户余额|eth.getBalance(“账户地址”)
账户间转账|  eth.sendTransaction({from:acc0,to:acc1,value: web3.toWei(1)})
查看当前节点的信息|admin.nodeInfo
查看已经连接的节点数|admin.peers
添加其他节点|admin.addPeer()
解锁账户|personal.unlockAccount()
查看账户余额|eth.getBalance(“账户地址”)
开始挖矿|miner.start()
在私有链上转账|eth.sendTransaction(from,to,value)
列出区块总数|eth.blockNumber
获取区块|eth.getBlock(区块高度 或 区块哈希)
查看某个区块下的所有哈希|eth.getBlock(2).transactions
获取交易|eth.getTransaction(交易哈希)

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/05.png)

1. 转帐需要有链上的矿工在工作才允许发生。
2. 因为创建了私有链，再使用geth查询的是早期创建在本地的公网和测试网的账户

### 4. 同步多个节点工作
将多个节点连接起来，可以用在不同的网络构建虚拟节点网（可以基于端口映射）

1. 获取该节点的编码：
    ```sh
    > admin.nodeInfo.enode
    "enode://9c79b5<此处省略120位>70@127.0.0.1:8545"
    ```

2. 在另一个节点调用添加：
    ```sh
    > admin.addPeer("enode://9c79b5<此处省略120位>70@127.0.0.1:8545")
    ```

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/06.png)

可以看到刚刚创建的第二个节点从第40个区块快速同步到了第3200左右区块

3. 调用``admin.peers``查看节点联系状况

4. 创建了两个docker进行互通，将端口映射到host，从docker寻址host的端口即可


### 5. 构建一笔交易

1. 解锁准备交易的账户（会间隔自动上锁）：

    ```sh
    personal.unlockAccount("0x977bb4ad6b55e23762a9188f08e37bfe3747efe5")
    ```
2. 编写交易函数：
    ```js
    eth.sendTransaction({
        from:"0x977bb4ad6b55e23762a9188f08e37bfe3747efe5",
        to:"0xb9f3c4a13c570b1bfd76a5beb6940b83192b036f",
        value:1000000000000000
        //数据大小自行查找以太币单位
    })
    ```
    返回交易记录地址
    ``"0x90c16925146392e91bc0ac76f84ff9e2189e6bb9818836b872d0cdc5accb4bb1"``
    刚刚试验的是在不同节点之间（不同的创世区块）进行交易，在同一NetworkID即可。

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/07.png)

3. 创建合约的部署交易：json字段如下：

    ```json
    { 
        from: "0x977bb4ad6b55e23762a9188f08e37bfe3747efe5", 
        data: "合约二进制代码"
    }
    ```
4. 合约的调用交易：json字段如下：

    ```json
    { 
        from: "0x977bb4ad6b55e23762a9188f08e37bfe3747efe5", 
        to: "合约地址", 
        data: "指定调用的方法和参数的传递"
    }
    ```

### 6. 编写合约(solidity)
一份标准合约的数据结构：

1. 地址：全局唯一的地址
2. 余额：合约上有钱，给执行者奖励。
3. 代码：合约的执行内容
4. 状况标识符：表示当前合约的状态

合约将部署在矿工的节点上的EVM（以太坊虚拟机）代码运行环境，可实现分布式调用。

1. ``pragma solidity ^0.4.25``指定版本
2. 创建一个测试合约：

    ```
    pragma solidity >=0.4.21;
    contract Info{
        string name;
        uint age;
        function setInfo(string memory tname,uint tage) public{
            name=tname;
            age=tage;
            //public 指定公开访问
        }
        function getInfo() public view returns(string memory,uint){
            return (name,age);
            //view 指定返回视图（只读）
            //returns 指定返回类型
        }
    }
    ```

### 7. 编译、部署合约
可以使用solc编译（npm: solc）

0.5.0版本以后有memory相关的语法要求，在上面代码里在参数里补充了memory参数。

调用编译``solcjs con.sol --bin --abi``，产生bin、abi两种文件。abi是数据结构，bin是字节码（长短不一，千到万字节级别），都是可读的文本。

部署合约：
```js
// 此代码在geth控制台执行

> var abi= abi数据结构
//抛出undefined就是正确
> var code= '0x<bin字节码>'
//记得在最前面加上0x表示十六进制 
> personal.unlockAccount("0xb9f3c4a13c570b1bfd76a5beb6940b83192b036f")
//解锁账户
> var contract = eth.contract(abi).new(1000000, 'Test Token', 'TEST', 2, {
    from: "0xb9f3c4a13c570b1bfd76a5beb6940b83192b036f", 
    data: code, 
    gas: 1000000
})
> contract.transactionHash
//获取该合约的交易哈希，用于查看合约创建的情况
//"0xc9397f596e93eec03350ee5f6706436081d08d192444c123d907eb2c61b77fac"
> contract.address
//获取哈希，这里获得的就是合约的地址了，调用需要用到这个
//"0xb1c5b6e81b256e9e20d884f68dbc200a37bebdbc"
//不同时间产生的合约哈希都各不相同（与时间相关）
//进而确保合约的唯一性
//如果地址返回undefined，说明还没被打包进区块，记得开启矿工工作
```

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/08.png)

### 8. geth控制台调用合约

1. 构建合约对象
    ```js
    > var abi= abi数据结构
    > var cont=eth.contract(abi).at("0xb1c5b6e81b256e9e20d884f68dbc200a37bebdbc")
    //contract.address合约的地址
    ```

2. 设置该节点的默认消费账户，记得解锁
    ```js
    eth.defaultAccount="0xb9f3c4a13c570b1bfd76a5beb6940b83192b036f"

    cont.getInfo()
    // 调用免费接口-> ["", 0]

    cont.setInfo("22",22)
    // 调用收费接口-> 交易hash，因为有费用产生，故有交易记录：
    // "0xbe7de0f8332f60368c182d9267989681bc59aa60b2a230be19e9bc0e22909562"

    cont.getInfo()
    // 调用查询-> ["22", 22]
    ```

### 9. 在网页上通过web3组件调用

容器内两个挖矿节点互联，可以在外面编写网页连接到里面去，并实现对合约的调用。主要思路如下：

1. 在网页里引用 web3.js 框架（github有说明）
2. 在网页中创建 web3 对象、指定节点、生成合约对象，调用方法
```html
    <!-- 依照官方指引从cdn导入web3.js -->
    <script src="https://cdn.jsdelivr.net/gh/ethereum/web3.js@1.0.0-beta.36/dist/web3.min.js" integrity="sha256-nWBTbvxhJgjslRyuAKJHK+XcZPlCnmIAAMixz6EefVk=" crossorigin="anonymous"></script>

    <!-- 创建一组name age对象与h苏阿信按钮 -->
    <label for="">name</label>
    <input type="text"  id="name">
    <label for="">age</label>
    <input type="text"  id="age">
    <button id="reflushBtn">update</button>
```

3. 连接节点
```js
if (typeof web3 !== 'undefined') {
    node = new Web3(web3.currentProvider);
    // 如果已经存在就使用缓存
} else {
    // 连接私有节点
    node = new Web3(new Web3.providers.HttpProvider("http://127.0.0.1:1111"));
    // 刚刚把容器里的1111作为RPC Port
    console.log(node)
    // 检查连接状态
}
```
4. 指定合约
```js
var abi = [{"constant":true,"inputs":[],"name":"getInfo","outputs":[{"name":"","type":"string"},{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"tname","type":"string"},{"name":"tage","type":"uint256"}],"name":"setInfo","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}]
// 这个数据结构是合约的abi，指定一个接口
var myContract = new node.eth.Contract(abi,"0xb1c5b6e81b256e9e20d884f68dbc200a37bebdbc");
// 创建合约对象，后面是合约的地址，部署成功的话可以在交易记录里查得到（contractAddress）
```
5. 封装合约底层方法
```js
function getInfo(){
    // 调用合约的getInfo方法
    infoContract.methods.getInfo().call({
            from:'0xb9f3c4a13c570b1bfd76a5beb6940b83192b036f'
            // 发起者地址是用户的，用户是要在私有链上的
        },function(error,result){
            if(error)console.log(error);
            if(result)console.log(result);
            // 其中result的[0],[1]分别指向了name和age，在合约的方法中有指定
    });
}
function setInfo(_name,_age){
        infoContract.methods.setInfo(_name, _age).send({
                from:'0xb9f3c4a13c570b1bfd76a5beb6940b83192b036f'
            // 发起者地址是用户的，用户是要在私有链上的
            },function(error,result){
                if(error)console.log(error);
                if(result)console.log(result);
                // 合约里没有指定返回值，故result为null
            });
        // set方法可以设置有一定的花费金额
}
```

## 写在最后
本篇简单记录了从基本构建私有链、创世区块、部署节点、控制节点，到交易的实现、合约的编写编译部署，到最后实现合约的调用。

基于此我们得到了一个基础的框架，我们可以封装好这些底层节点并在此之上可以编写合约完成更多的功能

本篇部分资料来自蚁米链播学院，引用声明

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/teacher.jpg)


---

参考阅读

1. [课程设计之Linux Docker构建区块链1：基本概念](https://visnz.github.io/post/application2/blockchain1/)
2. [课程设计之Linux Docker构建区块链3：环境docker打包与部署](https://visnz.github.io/post/application2/blockchain3/)
3. [geth搭建以太坊私有链](https://github.com/TTY-00/Blog/issues/6)