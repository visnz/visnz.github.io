---
title: "DaVinci/Fusion 节点式效果软件初体验:Tracker跟踪效果"
date: 2018-06-03
type: ["影视"]
weight: 9
tags: ["影视","达芬奇"]
categories: ["影视"]
description: "节点式影视效果软件尝试，实现了一个最基础的跟踪效果"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/fusion.png"
---

## 背景
（Adobe用户习惯剥离计划进行中）

因为最新的Davinci15把BlackMagic自家的Fusion内置了，现在基本集素材整理、剪辑、特效、调色、调音和渲染于一身，效果在达芬奇里面做也是方便点的。借这个机会尝试一下节点式效果工具

![结果图](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/01.png)
达芬奇15中的Fusion页面

## 素材与需求
（其实是过段时间bdf制作希望高点逼格）素材是多人出入场的舞蹈，将人物的名字附加到舞蹈者的运动趋势上

于是随机找了一段电脑里**带有运动趋势的素材**和**一个白条**做尝试

[视频素材](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/原片.mov)

图片素材
![素材](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/02.png)

结果视频：
[BDF2018宣传片 after 1:20](https://www.bilibili.com/video/av22787517)

## 操作

导入两部分素材，在时间线上部署底层素材
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/03.png)

直接切换到fusion工作台
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/04.png)

节点式主要是以清晰明朗的效果器结构与关系来展示复杂的效果逻辑，这一点比ae在层级控制上有更多的灵活性
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/05.png)
图中为节点面板，左边是媒体素材，右边是媒体的输出，要做的就是在这两个节点之间添加效果器

为输入素材添加跟踪器，并添加节点进行跟踪，跟踪……素材里的女孩的蝴蝶结
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/06.png)
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/07.png)
下面是添加中间节点的效果器 中间是跟踪结果 右上角向右跟踪

添加人名条并将其运动轨迹指定为跟踪器的分析结果
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/08.png)
左下角将新素材连接到跟踪器 右边切换到跟踪模式，切换到匹配移动(match move)模式，则结果会出现人名条已经被附加到跟踪器的轨迹上

修改人名条的相对位置匹配到指定点上
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/09.png)

简单的节点效果制作就完成了，切换回剪辑模式即可继续剪辑。因为效果是基于素材的，在剪辑里没有额外增加新的素材，整个工程更加简洁明了（pr和ae你们自己反思反思）
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/10.png)

## 结尾

- [视频效果](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/效果.mov)

- 提供了更灵活的调节工具（当然需要你的硬件）

- 提供了更清晰明朗的工程结构

- **最最重要的是，它可以在交付里重新生成片段xml，自定义帧余量渲染套底，整个回批的素材包只包括必须素材的必须片段，会小很多很多！**

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/fusion/11.png)
前排抱走~~诸君的~~老婆
