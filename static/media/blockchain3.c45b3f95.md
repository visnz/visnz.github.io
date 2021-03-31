---
title: "课程设计Linux Docker构建区块链3：环境docker打包与部署"
date: 2019-01-10
type: ["应用"]
weight: 6
tags: ["区块链","计算机","服务器","Linux"]
categories: ["运维","服务器","计算机"]
description: "简单记录镜像制作到打包到部署，以及一个小功能的实现"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/icon.jpg"
---

## 准备镜像
为了使得部署更简便，可以在之前的基础上打包一个安装了基本geth环境的Docker镜像便于多点部署。

### 1. docker构建镜像并安装

使用上一篇的archlinux构建，这一篇记录了再docker里操作熟悉环境的过程，Dockerfile在结尾

```bash
➜  ~ sudo docker pull base/archlinux

➜  ~ sudo docker run -dit base/archlinux
576db1cf56fb4fd9425adfa4f3e7c00d7e3c538a7fb402376bf70e943aa93462

➜  ~ sudo docker exec -it 576 bash
```

```bash
# 安装基础系统
pacman -Sy python2 npm geth net-tools nano
npm i -g solc

# 构建创世区块
mkdir ~/blockchain/private-chain/node -p
echo '{'                             >~/blockchain/private-chain/node/genesis.json
echo '    "config":{'               >>~/blockchain/private-chain/node/genesis.json
echo '        "chainId": 15,'       >>~/blockchain/private-chain/node/genesis.json
echo '        "homesteadBlock": 0,' >>~/blockchain/private-chain/node/genesis.json
echo '        "eip158Block": 0,'    >>~/blockchain/private-chain/node/genesis.json
echo '        "eip158Block": 0'     >>~/blockchain/private-chain/node/genesis.json
echo '    },'                       >>~/blockchain/private-chain/node/genesis.json
echo '    "difficulty": "100",'     >>~/blockchain/private-chain/node/genesis.json
echo '    "gasLimit": "2100000",'   >>~/blockchain/private-chain/node/genesis.json
echo '    "alloc":{}'               >>~/blockchain/private-chain/node/genesis.json
echo '}'                            >>~/blockchain/private-chain/node/genesis.json

# 完成初始化
geth init ~/blockchain/private-chain/node/genesis.json --datadir ~/blockchain/private-chain/node/
```

编写执行脚本
```bash
# ~/geth.init
ip=`ifconfig eth0 |grep inet|awk '{print $2}'`
echo 'ip acquire:'$ip
# 获取对外网卡的地址

# 启动挖矿的脚本
# 第一个参数指定RPC端口
# 第二个参数指定节点的端口
geth --identity "node" --rpc --rpcport $1 --rpccorsdomain "*" --datadir "~/blockchain/private-chain/node/" --port $2  --ipcpath "gethipc.ipc" --networkid 10 --nat extip:$ip --rpcaddr $ip

# example: geth.init 1111 6666
```
```bash
# ~/geth.attach
geth attach ~/blockchain/private-chain/node/gethipc.ipc
```

收尾工作
```bash
chmod +x ~/geth.*
history -c
```


### 2. 推送节点镜像

1. 使用dockerhub
    ```bash
    sudo docker login
    ```
    登陆的话认真信息保存在``/root/.docker/config.json``，妥善保管
2. 创建repo，repo会产生一个``<username>/<reponame>``格式的镜像名，将作为整个dockerhub唯一的识别符

3. 打包镜像、打tag
    ```bash
    sudo docker commit -a "visnz" -m "geth env basic build up" 576 archlinux:geth-basic
    # 最后指定为新的镜像名+tag
    sudo docker tag archlinux:geth-basic visnz/archlinux-geth-basic:geth-basic
    # 将新的镜像+tag绑定
    sudo docker push visnz/archlinux-geth-basic:geth-basic
    # 推送到dockerhub
    ```

docker层结构，只推送新增的层即可：

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/09.png)

tips: 建议在境外机器上完成镜像构建，国内push速度

### 3. 部署节点镜像

1. 回到服务器上直接部署：
    ```bash
    # 拉取镜像
    sudo docker pull visnz/archlinux-geth-basic:geth-basic
    # 开放rpc端口为14010
    sudo docker run -dit -p 14010:14010 visnz/archlinux-geth-basic:geth-basic
    ```
    ```bash
    # 用写好的两个脚本直接运行
    cd && ./geth.init
    ```
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/10.png)

## 写一个啥功能好呢

虽然我也想在课程设计用sol构建更复杂功能，不过sol语言本身的不足（缺乏语法糖、应用的库、与其他语言的扩展联通），以及不同版本之间的一些语法规则不同引发的兼容性问题（早期合约不一定能在新的版本编译器上通过），遂决定还是做个简单一点的能顺利部署的就好了。

```
pragma solidity ^0.5.0;

contract Repeat{
    string sentence="今天我一定来";
    uint repeatCount=0;
    function changeAllSentence(string memory sen) public{
        sentence = sen;
    }
    function getCount() public view returns (uint count){
        return repeatCount;
    }
    function repeat() public {
        repeatCount += 1;
    }
    function clear() public {
        repeatCount = 0;
    }
}
```
（得有60%的时间在语法修正上）

html网页代码、js动态改写不赘述，相关的调用在上一篇也都讲到了。最后的效果如下：

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/blockchain/11.png)

---
参考资料

1. [课程设计之Linux Docker构建区块链1：基本概念](https://visnz.github.io/post/application2/blockchain1/)
2. [课程设计之Linux Docker构建区块链2：基础设施搭建](https://visnz.github.io/post/application2/blockchain2/)
3. [Solidity中文文档](https://solidity-cn.readthedocs.io/zh/develop/)
4. [打包了的镜像地址](https://cloud.docker.com/repository/docker/visnz/archlinux-geth-basic)