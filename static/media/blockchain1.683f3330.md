---
title: "课程设计Linux Docker构建区块链1：基本概念"
date: 2019-01-04
type: ["应用"]
weight: 8
tags: ["区块链","计算机","服务器","Linux"]
categories: ["运维","服务器","计算机"]
description: "课程设计捕获关于区块链的学习机会，由于篇幅过长按内容分篇发布。此篇阐述基础概念"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/icon.jpg"
---

## 写在前面
电子货币范畴内的术语、彼此之间的关系大概了解可以参考：

1. [狭义与广义上的电子货币](https://www.wikiwand.com/zh-hant/%E9%9B%BB%E5%AD%90%E8%B2%A8%E5%B9%A3)
2. [区块链与比特币的关系](https://www.zhihu.com/question/47034756)
3. [区块链技术是什么？未来可能用于哪些方面？](https://www.zhihu.com/question/27687960)

## 基本概念

### 挖矿

1. 以一定时间为单位（比如10分钟）所有的交易都会被打包到一个区块中。这个区块需要有一种认证方式来保证其正确性，不同的区块链有不同的共识机制。
2. 以比特币为例，这个新产生的区块的区块头，广播给所有节点。节点开始使用自己产生随机数来暴力破解得出这个新产生的区块的随机数（一个数字猜谜游戏）。
3. 当有一个节点猜出了该随机数，就获得了这个区块的拥有权，并记录下区块内容。其他节点验证这个答案的准确性，准确也一并记录。同时，该节点获得比特币奖励。
4. 所有矿工得知，并转向下一个区块

挖矿的本质：通过计算力，抢夺数据的记录权（大部分是交易记录）
发展期望：使用其他认证机制，而不是通过消耗电力的计算力来完成区块扩展。

### 基本数据结构

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/区块结构.jpg)

1. 区块大小信息（每个区块在 1MB 以内）
2. 区块头（包含前一个节点的 hash. 版本. 时间戳. Nonce随机数. Bits难度目标. Merkle Root 哈希树根）
3. 交易内容（交易总数统计. 交易详细内容）

### 算力
指该计算机（CPU、GPU、FPGA、定制化矿机、矿池）每秒运算哈希的次数。

- 1 kH/s = 每秒1,000哈希，类推
- GTX1080算力：400H/s左右
- i7 4710mq(laptop)算力：不到100H/s
- 算池算力（2019年1月7日，全网前五算力23 EB/s，当前最高为6858.00 PH/s）

### 挖矿难度

1. 区块头字段bits，4个字节，前1个字节表示幂+3个字节表示系数，会产生一个数。这个数表示随机数的认可范围，也标明决定了这个区块被破解的难度。（这个数越小说明越难）
2. 通过一套算法计算出新的区块的的答案，只要这个答案小于挖矿难度，即可被认为获得了拥有权。
3. 算法：``SHA256(SHA256(版本+上一区块hash+根merkle+时间戳+目标难度+nonce值)) ``

### 共识机制
几种比较常用的机制

1. PoW 	- Proof of Work - 工作量证明

    > 通过一定的工作量，来获得相应的奖励
    > 它要求矿工进行哈希运算来获取记账权和新币
    > 以耗费大量时间. 资源为担保，确保记账工作的真实有效性
    > 
    > > tips：跟工作量关系不大，就是纯靠暴力运气，工作多少挣多少，好比劳动密集型产业制度

2. PoS	- Proof of Stake - 权益证明

    > 根据你拥有的币龄给你分配相应的权益
    > 币龄是持币数量和时间的乘积
    > 币龄越大，最有可能抢到记账权
    > 
    > >  tips：不用考虑暴力破解，由概率分配。这样其实容易产生财富集中化，好比金融

3. DPoS	- Delegated Proof of Stake - 授权股份证明机制

    > 让所有持币人都有机会选出自己的代表
    > 参与人以自己的代币为权益证明选出代表
    > 这些代表可以轮流进行记账
    > 或者采用PoS算法加权的获得记账权
    > 代表将收到平均水平区块所含交易费的10%作为报酬
    > 
    > > tips：好比人民代表大会制度

### 杂记

1. 因账号只是一串无规律的16进制字符串且不需要验证，可以让你的账号破解需要的成本大于破解账号的收益。
2. 区块链是旧技术的新组合
3. 没有余额的概念（通过之前的交易记录来验证，转入与转出的钱计算做你的余额，不用管理余额有效性）

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/以太坊.jpg)

## 以太坊
可编程区块链，相比比特币的“全球账簿”，以太坊是“全球计算机”。可以在上面部署服务（下称为合约）。

- 基于区块链，将节点的算力集中起来，维护一个EVM（以太坊虚拟机），上面可以使用合约
- 分两种类型的工作：记账+合约工作
- 奖励机制：普通奖励（记账+合约工作）+记录的叔块奖励
- 节点：对外暴露一个``IPAdress:Port``格式的地址。一个节点可以有多个账户（多个旷工）可以通过工作获取奖励。
- **以太坊的矿工工作实质：为EVM提供算力支持，维护一个线上的EVM**

目前以太坊处于四个阶段中（Frontier（前沿），Homestead（家园），Metropolis（大都会），Serenity（宁静））的第三个阶段，前三个阶段使用PoW机制，最后一个阶段使用PoS。

近期<sup>2019.01.08</sup>以太坊创始人 Vitalik Buterin 表示，计划在一两年内，将以太坊的共识机制从挖矿（PoW）改成利益证明（PoS），可以将耗电量降低99%。

### 账户与合约

> 以太坊的账户分成两种，一种是EOA账户（Externally Owned Account，由私钥控制），里面存储了账户的以太币余额信息和由该账户发起的交易数量（nonce），上面创建的都是EOA账户。
> <br/>
> 另一种是合约账户（Contract Account），是由合约部署交易创建的，里面除了存储以太币余额，还存储了合约代码，以及代码的状态（存储）。
> <br/>
> <br/>通过创建并发送由某个EOA账户私钥签名的交易，可以从该账户给其他EOA账户转账，或者向某个合约账户发起一笔交易（调用合约账户里的某个方法）；
> <br/>而合约账户只能被动触发，接受某个EOA账户（或其他被触发的合约账户）发起的交易，根据该交易的指令执行相应的动作，修改内部状态，或向其他账户发出交易。
> <br/>
> <br/>
> 所谓合约，并不是某个要被执行的合同，其实质是一段代码，可以类比为一个Class，其中定义了一些方法、状态、事件。
> <br/>将这段代码编译成以太坊虚拟机（Ethereum Virtual Machine，简称EVM）机器码，就可以作为 sendTransaction() 方法的 data 参数，被部署到以太坊区块链上（会自动分配一个地址），这样就创建了一个合约账户。
> <br/>后续按照EVM ABI约定发送到该合约账户的交易，可以指定触发这段代码的相应方法执行预定的操作。[^2]

EOA账户由一对公密钥决定，密钥为实质的凭证，公钥后20位为账户的地址。

合约账户包含一份合约（需要被执行的代码）、一定的余额（用于贮存执行合约获得的金额，用交易记录表示）和一个地址。每部署一个合约就产生一个合约账户，这个合约账户的地址由EOA账户+交易记录组成。

### 以太坊的交易
以太坊的交易最直观解释：从外部账户发送到区块链上的另一个账户的消息和签名的数据包[^3]

交易类型：
1. EOA之间的转帐（以太币转移）
2. 部署合约（EOA发起的指令）
3. 调用合约（EOA发起的指令）

交易单位：
1. 以太币（EOA账户的交易单位）
2. Gas（用于指示工作量的单位，如一次交易消耗21000，一个合约消耗万到百万级），用Gas Price表明想要支付给矿工的价格。
![gas消费表](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/gas消费.png)[^3]

交易参数：

1. from：发起者地址，不可空
2. to：接受者地址，空的时候表示创建一个合约
3. value：以太币数量
4. data：数据段，存在时表示创建或调用一个合约的交易
5. Gas Limit：表示交易允许消耗的最大数量
6. Gas Price：发送者愿意支付给矿工的Gas价格
7. nonce：用于区别用户发出交易的标识
8. 交易摘要值、签名部分

### 部署的基本步骤
1. 构建区块链：创建节点、创建账户、挖矿产出货币
2. 编写智能合约+编译+发布智能合约
4. 测试合约函数，编写前端代码，使用 web3.js 组件访问合约
5. 通过分配机制分配到矿工处工作，完成合约

### 相关工具
geth，以太坊的核心工具，可以前往[官网](https://geth.ethereum.org/downloads/)或者[github](https://github.com/ethereum/go-ethereum)下载

开发辅助工具（非必须）

1. [web3.js 框架](https://github.com/ethereum/web3.js/)，一个基础框架，封装 js 并提供服务RPC对接(npm: web3)
2. [Remix IDE](https://github.com/ethereum/remix-ide)，可以使用在线的IDE进行编写，~~也可以用记事本~~(npm: remix-ide)
3. 以太坊Solidity编程语言开发框架 [Truffle](https://github.com/trufflesuite/truffle)（npm: truffle）
4. [ganache-cli](https://github.com/trufflesuite/ganache-cli)(npm: ganache-cli)

以太坊客户端（非必须）

1. [Mist 图形化geth客户端](https://github.com/ethereum/mist)，作钱包用，Mist=geth + Ethereum-Wallet + remix
2. [MetaMask 浏览器插件](https://github.com/MetaMask/metamask-extension) 网页钱包管理器，用来提供密码安全管理


---
参考阅读

1. [课程设计之Linux Docker构建区块链2：基础设施搭建](https://visnz.github.io/post/application2/blockchain2/)
2. [课程设计之Linux Docker构建区块链3：环境docker打包与部署](https://visnz.github.io/post/application2/blockchain3/)
3. [[中文] 以太坊白皮书
](https://github.com/ethereum/wiki/wiki/%5B%E4%B8%AD%E6%96%87%5D-%E4%BB%A5%E5%A4%AA%E5%9D%8A%E7%99%BD%E7%9A%AE%E4%B9%A6)

[^2]: [以太坊区块链长什么样？ —— 自建 ethereum 私有链指南](https://gist.github.com/felix021/750031512a0cc2b0ee5f57ba2d927cbd)
[^3]: 资料部分来源：蚁米链播学院