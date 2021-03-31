---
title: "Tg自用RSSBot+私人RSSHub服务"
date: 2019-02-01
type: ["应用"]
weight: 3
originweight: 7
hidetime: 2019-03-01
tags: ["LINUX","计算机","RSS","RSSHub","服务器","Docker"]
categories: ["服务器","最近"]
description: "官方RSSHubBot匿了 自己搭一个"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/tgbot-rsshub/icon.png"
---

## 背景

因为老是白嫖[官方Bot](https://telegram.me/rsshubbot)，没支持了寻思找一下其他的Feedr，找了几个都退休了，还是自己搭一个吧毕竟也得部署RSSHub。

~~好久没用rss有点跟不上时代的感觉了~~

结构上：

- TG Bot（接通到Bot后端的接口）

- Bot后端（维护订阅列表+查询）

- RSSHub Docker（RSSHub的引擎，维护内容查询）


## Bot后端

Bot后台使用了大佬iovxw的[rssbot](https://github.com/iovxw/rssbot)，直接Release下载即可

丢到systemd里，rssfile用于储存一个订阅列表，后接上token
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/tgbot-rsshub/02.png)

## TG Bot
按规矩找 [@BotFather](https://telegram.me/BotFather) 申请bot并保留Token
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/tgbot-rsshub/01.png)

按官方文档``/setcommands``设置命令

```
rss       - 显示当前订阅的 RSS 列表，加 raw 参数显示链接
sub       - 订阅一个 RSS: /sub http://example.com/feed.xml
unsub     - 退订一个 RSS: /unsub http://example.com/feed.xml
unsubthis - 使用此命令回复想要退订的 RSS 消息即可退订, 不支持 Channel
export    - 导出为 OPML
```

## RSSHub 部署
根据官方文档直接部署，把请求的端口映射到docker网卡（而不是公网地址）

```bash
sudo docker pull diygod/rsshub
sudo docker run -dit -p 172.17.0.1:1200:1200 diygod/rsshub
```


## 订阅

直接按内网地址，通过RSSHub订阅即可

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/tgbot-rsshub/03.png)

- [订阅参考文档](https://docs.rsshub.app/)