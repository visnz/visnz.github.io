---
title: "整合 MTProxy VMess ss 协议服务到同一单元服务"
date: 2019-03-12
type: ["应用"]
weight: 3
originweight: 7
hidetime: 2019-05-12
tags: ["DOCKER","服务器"]
categories: ["服务器","最近"]
description: "三个服务合一是三倍的方便与快乐"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/v2ray/icon.png"
---
# 问题
本篇用词隐晦，懂的懂，不懂别问了。

出于一些人为或非人为因素，需要做一系列的服务迁移：

1. 使用[MTProxy 一键搭建管理脚本](https://github.com/FunctionClub/MTProxy-Bash)构建的MTProxy。
2. 使用脚本部署的 direct/go.sh 部署的执行VMess协议的 [Project V](https://www.v2ray.com/) 组件
3. 以及使用不知哪里来的神仙ssserver部署的协议

多个软件以及多个配置文件。



# 具体实现
Project V 提供多个``inbound``，也提供多个``inboundDetour``。原本的使用是inbound接受vmessi协议，并协商下一个随机端口，随机的配置就写在inboundDetour。

> 以下配置均无法直接投入使用
> 有需要的朋友请查阅手册自行修改

```json
// 单个入口
"inbound": {
    "port": 0,
    "protocol": "vmess",
    "settings": {
        "clients": [
            {
                "id": "00000000-0000-0000-0000-000000000000",
                "level": 20,
                "alterId": 20000
            }
        ]
    }
}
// 单个分流入口
"inboundDetour": [
    {
        "protocol": "vmess",
        "port": "1000-2000",
        "tag": "randomrandom",
        "allocate": {
            "strategy": "random",
            "concurrency": 10000,
            "refresh": 1
        },
    }
]

```

同时，inboundDetour与inbound也允许接受来自多个接入（片段）：
```json
// 在分流入口添加 SS：
"inboundDetour": [
    {
        "protocol": "shadwosokcs",
        "port": 0,
        "settings": {
            "method": "",
            "password": "",
            "level": 100
        }
    }
]
// 在分流入口添加 MTProxy：
"inboundDetour": [
    {
        "tag": "in",
        "port": 0,
        "protocol": "mtproto",
        "settings": {
            "users": [{"secret": "00000000000000000000000000000000"}]
        }
    }
]
```

MTProxy是一个单独的转发，需要在outboundDetour与routing内写入转发规则：
```json
"outboundDetour": [{
        "tag": "out",
        "protocol": "mtproto"
    }
]
 "routing": {
        "strategy": "rules",
        "settings": {
            "rules": [{
                "type": "field",
                "inboundTag": ["in"],
                "outboundTag": "out"
              }]
        }
}
```

# 后记

这样就完成了三个服务统一使用 Project V 提供的工具进行整合

 使用[Project V 官方提供的docker](https://hub.docker.com/r/v2ray/official/)服务更加，只需要一个``config.json``+一个``docker run``

 