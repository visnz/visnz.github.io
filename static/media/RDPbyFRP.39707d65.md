---
title: "使用FRP实现公网映射工作站RDP远程服务"
date: 2018-05-02
type: ["应用"]
weight: 7
tags: ["WINDOWS","计算机","服务器","RDP","FRP"]
categories: ["运维"]
description: "Windows10工作站远程桌面连接：frp内网穿透映射RDP服务"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/oldicon/RDP.png"
---

## 背景
### 问题
- 小作坊影视后期，素材量大，传输不方便。

- 有一台核心工作站，协同工作者不集中，其各自设备不给力

- 想办法让协同工作者能够远程参与到工作站中

### 构思

开放工作站多用户协作，远程连接端口，允许其他网络协同工作者连接进来

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/RDP/RDP.png)

优点：协同工作者彼此之间不需要复制多份大文件素材，不用过多考虑**素材传输** 问题、**设备负载** 问题。整理素材、粗剪、精剪等工作可以同时进行。

缺点：需要有**流畅网速保障**（公网服务器、自身分发速度、协同工作者的下载速度），剪辑预览的时候更需要有网速保障（通常本地工作站不需要考虑这些）。


## 所需工具
- Windows10桌面开放连接：[RDP Wrapper](https://github.com/stascorp/rdpwrap)

- 端口映射工具：[frp](https://github.com/fatedier/frp)

- 本地工作站，能负载多用户登录

- 一台中继服务器，国内外皆可，需要保证其上下行速度流畅（记得打开你阿里的port ban列表）

## 配置
### 多用户登录
先创建新用户和密码，把用户放入``Remote Desktop Users``组
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/RDP/RDP00.0.png)

允许你的电脑接受远程连接
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/RDP/RDP00.1.png)

Windows登录某个用户就会挤下线另一个用户，首先允许你的工作站实现多用户同时登录：RDPWrap->``install.bat``（记得关闭你的360、某安全管家）

后可使用``RDPConf.exe``配置具体内容
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/RDP/RDP00.png)

无法``[fully supported]``的可以尝试``update.bat``更新

使用``RDPCheck.exe``对本地多用户登录进行测试
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/RDP/RDP01.png)

### 多会话冲突
测试过程出现过多会话访问，工作站拒绝连接。原因是Windows系统防范过多会话连接。可在cmd->``gpedit.msc``->管理模板- Windows组件->远程桌面服务->远程桌面主机会话中，会话时间限制、连接等进行限制。

自定义最高连接数、设置断开连接后的会话生存时间等等。详细一些操作可自行搜索
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/RDP/RDP02.png)

### 配置服务器：frps
服务器上直接下载[RDP Wrapper](https://github.com/stascorp/rdpwrap)对应版本，修改``frps.ini``。可以简单按照[RDP文档的示例](https://github.com/fatedier/frp/blob/master/README_zh.md)配置一个监听接口，安全一点可以引入``stcp``等。

``./frps -c ./frps.ini``启动

至于需要配置``systemd service``和``crontab``的请自行配置

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/RDP/RDP03.png)

### 配置工作站：frpc
下载[RDP Wrapper](https://github.com/stascorp/rdpwrap)对应版本修改``frpc.ini``。

简单配置做测试
```ini
[common]
server_addr = 你服务器地址
server_port = 7000  服务器的穿透端口（frps.ini中指定的端口）

[RDP]
type = tcp
local_ip = 0.0.0.0  监听本地
local_port = 3389   RDP的端口，RDPConf.exe中指定
remote_port = 6000  在服务器上提供连接到local_port的服务端口
```

- [ ] 过后补充相对安全配置文件

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/RDP/RDP04.png)

``frpc -c ./frpc.ini``启动，可看到**服务器显示有连接接入**

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/RDP/RDP05.png)

### 连接测试

运行``mstsc``，连接到``服务器地址:6000（务器的服务端口）``

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/RDP/RDP06.png)

在保持网速上下行的情况下，可流畅执行工作站任务（测试两机分别位于不同校园网，2M网速已可以完成基本操作）

RDP根据设置可以重定向音频，不过会有折损。色彩只需要考虑屏幕显色差异。相对来说会比较适合**需要大计算量**和**信息同步需要**的协同工作，比如3d建模的分散作业、调色、筛选素材、渲染等。

## 结束

- 记得时刻留意你的任务管理器（性能、用户面板）

- Linux协同工作不需要这么麻烦的，您可以考虑一下Linux+Lightwork+Davinci+Blender+Nuke

- 直接暴露RDP端口到公网也是不安全的，配置stcp或者做好安全监控措施

- 可以随时随地登录内网工作站，像极了IronMan维罗妮卡？

- Mac貌似还无法连接？
