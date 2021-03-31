---
title: "使用git简单合并pr多版本，剪辑与调色同期进行"
date: 2019-01-14
type: ["影视"]
weight: 9
tags: ["影视","ADOBE"]
categories: ["影视"]
description: ""
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/icon.jpg"
---

## 缘起

因为朋友问到``怎么才能剪辑与调色同时进行，最后合并``，第一个反应是用git来管理。如果使用pr内建的方法的话，新的原声pr都用lumetri调色（或者你们自用插件），方法基本就是导入导出预设而已。

勾起了想在pr外面套一层git管理的欲望。

## TL;DR
> pr工程文件是gz打包的xml文件，但pr操作对对象的记录却随时间顺序不同而不同，所以使用git合并会导致素材引用混乱引起错误。
> 
> 1. 原问题解决方案：还是用预设导入导出吧
> 2. git合并解决方案：在导入素材后先在工程内构建引用对象再签出
> 3. 结论：pr因为以时序编号对象，不适合使用版本管理器
> 4. 建议：直接拉到最下面看结论

## 开始

1. prporj实质上是使用gz打包压缩的xml文件，创建文件后保存解压即可看到
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/01.jpg)
    左边为prproj文件，gz压缩格式的二进制，右边是解压后的工程文件
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/02.jpg)

2. 在工程文件夹构建git版本管理器、提交初始版本
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/03.jpg)
    可以看到第一次提交的有prproj以及解压出来的文件这两个
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/04.jpg)

3. 创建并签出分支，开始导入素材，保存并解压获得一个拥有素材的xml文件
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/05.jpg)
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/06.jpg)
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/07.jpg)

4. 添加完素材开始多人同步工作：签出剪辑、调色两个分支

5. 简单对素材进行调色，并保存
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/08.jpg)
    git发现prproj有变化，将其解压出xml。
    解压出来的xml文件会直接覆盖原文件，在版本管理工具里可以看到修改的内容：其中多了几行对素材文件的追踪与历史记录的内容
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/09.jpg)
    提交即可看到新的提交
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/10.jpg)

6. 签出到剪辑分支，做一个简单剪辑
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/11.jpg)
    解压后也可以看到新增了对于剪辑的描述
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/12.jpg)
    提交确认后可以看到从素材分支叉开了两个分支：
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/13.jpg)
    工程结构上这两个结构是解耦的，但是pr……
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/14.jpg)
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/15.jpg)

7. 喵的pr你的底层引用结构我服
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/16.jpg)
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/17.jpg)


## 写在最后
原理上进行合并都是OK的，pr内部对obj对象的编号机制不是采用散列，而是使用随时间增长的正序数。

这就直接导致剪辑里对同一段素材的引用是57、58，在调色的素材里因为添加了lumetri，而变成了154、155

建议如果是开始尝试使用git管理pr工程的话，可以开始考虑在一开始导入素材的时候，签出调色前再添加一个把所有素材都添上效果器

## pr这么不耐用怎么还有这么多人用？

~~FCP、Davinci不知道比你们高到哪里去了，我跟他(们)谈笑风生~~

吐槽针对``原生pr``一些不是必须有、但没有很不爽的功能

- 效果器的节点式统一管理，ae的引用和表达式做得比较方便
- easy一点的剪辑工具（以最少正交基的工具集，参考Davinci）
- 没有自建的素材代理算法加持（参考达芬奇）
- 没有内建版本管理，签出基本靠复制工程对象，造成空间时间浪费
- 没有提供脚本编写器应对批量素材处理等工具

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/prgit/end.jpg)

pr作为一款入门到准专业级都适用的``剪辑集成工具``，已经是相当优秀的了（相比其他同等定位），带我走进了一行走出了一行。

他是开始但不是结束，新的需求总会有新的工具出现，行业也因此欣欣向荣吧。

---
了解更多

- [DaVinci/Fusion 节点式效果软件初体验:Tracker跟踪效果](https://visnz.github.io/post/media10/davinci-track/)
- [修正达芬奇xml剪辑表素材出入点错误](https://visnz.github.io/post/media10/dvc-xml-fix/)
- [Linux上调用ffmpeg组件压制字幕](https://visnz.github.io/post/application2/wan/)