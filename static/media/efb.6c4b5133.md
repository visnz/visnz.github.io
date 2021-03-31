---
title: "Docker-Compose部署微信机器人到TG"
date: 2019-04-12
type: ["应用"]
weight: 2
tags: ["计算机","WechatBot","Telegram","Docker-Compose"]
categories: ["服务器"]
description: "在telegram可以直连微信，注册TgBot获取token，在服务器上用docker部署WechatBot"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/efb/logo.png"
--- 
# 简介
在tg上申请一个bot，该bot连接到服务器后台的efb（微信web托管程序），bot负责转发你的指令（比如chat、link等）到后台的efb，efb将信息转发到微信。
# 具体操作
## 申请bot
输入``@botfather``点击访问或 [@botfather](https://telegram.me/BotFather) ``/start``开始使用。

``/newbot`` 申请新bot，并输入你的``bot名``。然后他继续要求你输入bot的``ID识别名``（必须以bot结尾）。因为是私人用，我两个都起了一样的名（无所谓）

完成后，他会给一个``HTTP API``的token，保存之（[efb-config](#efb-config)中需要用到）。

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/efb/01.png)

``/setprivacy``来指定私有性，disable的话，可以添加某个人（使用link）到某个tg群组，发到群组里的消息也会发到那个人那里。

``/setjoingroups``来指定能否加组，disable的话，不能被添加到群组。（与上一条不冲突）

``/setcommands``来设置指令，根据efb本身已有的功能添加下面内容：

```
link - 将会话绑定到 Telegram 群组
chat - 生成会话头
recog - 回复语音消息以进行识别
extra - 获取更多功能
```

## 获取userid
[efb-config](#efb-config)中需要用到你自己的userID（telegramID），通过[这里](https://t.me/get_id_bot)点击start获取你的ID

## Docker-compose
```
version: '3'
services:
    efb-wechatbot:
        container_name: wechatbot
        image: royx/docker-efb
        restart: always
        volumes:
            - ${EFB_CONFIG}:/opt/ehForwarderBot/config.py
            - ${EFB_DB}:/opt/ehForwarderBot/plugins/eh_telegram_master/tgdata.db
        env_file: .env
```
## .env
```sh
# efb
EFB_CONFIG=./efb-config
## 此处指定配置文件
EFB_DB=./efb-tgdata.db
## 此处指定暂存信息的db，大约5到10M左右
```
## efb-config
替换下面的``token``、``admins``两项
```py
# 一个py格式的配置文件
# ./efb-config
master_channel = 'plugins.eh_telegram_master', 'TelegramChannel'
slave_channels = [('plugins.eh_wechat_slave', 'WeChatChannel')]
eh_telegram_master = {
    "token": "123456789:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA",
    "admins": [123456789],
    "bing_speech_api": ["xxx", "xxx"],
    "baidu_speech_api": {
        "app_id": 0,
        "api_key": "xxx",
        "secret_key": "xxx"
    }
}
```
# 部署
```sh
touch ./efb-tgdata.db
docker-compose up
```
或者后台模式：
```sh
touch ./efb-tgdata.db
docker-compose up -d
docker logs wechatbot
```
可以看到二维码，扫码登陆


# 后话
1. efb是接管wechat的web在后台的程序上（基于python），意味着你登陆windows/Mac平台如果登陆微信客户端，efb会掉线，需要你到tg（有时得去后台重新扫码登陆）
2. 基本上有tg代理wechat，就能够取代wechat在电脑平台上的工作地位了，传文件、传消息什么的也都挺快
3. efb有时候会出一些小问题，比如因为文件过多积累使得queue无法发送（在tg无法发消息到微信，或微信的消息传不过来），可以通过重启解决，但程序只抛出异常而不是终结，就导致docker无法因为其异常而自动重启。（不过出现的几率不大，可能一个月一次，需手动重置重启）
4. tips：通过该程序可以有效转换tg里珍藏多年的gif表情包（而不会转为mp4）

# 参考
1. 附图：[EFB 简明安装教程：用 Telegram 收发微信 [基于 Docker]](https://www.appinn.com/efb-tutorial-with-docker/)
2. [efb镜像 in dockerhub](https://hub.docker.com/r/royx/docker-efb/)
3. [efb原项目](https://github.com/blueset/ehForwarderBot)