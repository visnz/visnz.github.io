---
title: "pr素材代理与简单应用"
date: 2018-03-13
type: ["影视"]
weight: 16
tags: ["影视","ADOBE"]
categories: ["影视"]
description: "代理素材使得在低性能机器上可以实现 大素材打包分发剪辑 && 高计算量剪辑"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/oldicon/pr.png"
---



后期剪辑可能会遇到以下问题

- 素材量巨大或有高解析如4K素材，电脑太烂带不动
- 需要把视频交付给别人去剪辑，素材很多很大，上传网速太慢

使用pr自带的自动代理功能可以有效解决

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/00.png)

基本思路是，把巨大的素材转换成一个小的，可以完成基础剪辑、音频校验等工作的视频素材，以降低CPU在解析素材的时间和硬盘使用率，从而提高处理速度。缺点在于对画面细节丢失精度，比如在稳定器的计算等等。

当需要别人参与剪辑的时候，只需要将代理文件打包好，交付给对方即可。

笔者使用pr版本提供两种代理手段：

- **手动** 创建代理或链接已有代理
  ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/01-1.png)

- 收录时 **自动** 创建代理（下文介绍）

# 代理素材实现高计算量剪辑

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/01.png)

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/02.png)

首先自然要把素材归类到一个文件夹中，并在旁边新建一个pr工程

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/03.png)

在收录设置中，选择创建代理。当然目录建议是放在同一个文件夹，我新建了一个“素材代理”

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/04.png)

其中的预设建议使用GoPro的方案，体积相对H.264较大，该编码专门针对视频中间代理文件，较大程度保留精度。若没有 **稳定、变速光流法** 等需要高精度计算的精剪效果的话，建议使用H.264，或者您可以导入其他预设。

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/05.png)

通常到这里应该有三个项目了，这个时候可以将素材文件夹的素材导入pr中，**pr收录会自动启动AME(Adobe Media Encoder)**，并进行转码和代理连接

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/06.png)

如果收录设置有修改见下图

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/07.png)

到这里基本完成了素材的代理生成，现在要让我们的pr工程启用代理

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/08.png)

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/09.png)

在素材查看器(源)和节目预览(节目)中把切换代理按钮拉下来，分别启动素材代理。**可以看到素材被替换为一个左右两边有两条小黑边的代理素材，则为启动了素材代理**。

# 代理素材+工程打包分发

导入+渲染完所有代理素材后，想要把代理素材全部打包交付给(比如手下或外包团队)去做的话

首先解除原素材和代理素材之间的关系

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/10.png)

选中所有原素材，设为脱机（我是直接归类成一个文件夹）

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/11.png)

如图描述，如果是工作分发，自己按理要保留文件，第二项会删除源文件，看自己需要做选择

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/12.png)

这时候就可以准备打包了，如图↑选择项目管理（打开之前请先创建一条空序列，打包需求）

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/13.png)

随便创建了一个空序列，打包是按序列来打的

右边记得勾选打包未使用素材，否则是空包。新建工程没有预览文件，路径自定义，下面的估计磁盘空间请按素材代理的总大小来计算（“素材代理”文件夹大小），请确保磁盘上有足够空间进行打包

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/proxy/14.png)

可以看到打包之后就只有代理素材和工程文件了，打开照样可以进行剪辑工作

# 回链接素材+渲染

1. 代理的素材会在渲染时候默认替换回原来的素材，可以直接渲染

2. 分发返回的剪辑工作，需要重新连接原素材，再读第一点。

---

# 修订补充

据随机测试，Gopro预设把文件压缩到大约**六分之一到三分之一**，H.264大约压缩到**十二分之一**

因为素材多种且码率参数各有不同，数据仅做概率参考

（抠鼻）反正可以手动创建预设
