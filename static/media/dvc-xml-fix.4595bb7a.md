---
title: "修正达芬奇xml剪辑表素材出入点错误"
date: 2018-09-03
type: ["影视"]
weight: 9
tags: ["影视","达芬奇","NODEJS"]
categories: ["影视"]
description: "Davinci 导出剪辑表的时候，存在剪辑表偏差，使用nodejs读取xml修改剪辑表内容"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/dvcxmlfix/logo.png"
---

## 背景
因为一些原因 需要将一段素材进行场景变换侦测（根据镜头切换，切割视频），然后转换成剪辑表跟素材，进行下一步的操作。在输出剪辑表的时候，发生了“剪切时间点正确，但素材出入点错误”的尴尬局面。

### 场景侦测

达芬奇提供**场景侦测自动分切**的功能：

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/dvcxmlfix/03.png)
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/dvcxmlfix/04.png)

图片一目了然，根据画面变化率进行分辨对视频进行逆向分析（所以一些渐变效果需要自己手动调整右边的时间切段）

下面的紫色的线条可以上下拖动调整变化率敏感度

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/dvcxmlfix/05.png)

将分切好的素材全选生成一个对应序列，在剪辑板块直接导出XML文件（因为需要跟pr接驳，直接用了标准XML）

### 问题
通常来说输出的剪辑表应该是这样的：

AB,C,DEF,GH,I

结果在PR中读取是这样的：

AB,A,ABC,AB,A

即：剪辑点是对的，但素材的起始点不对，全部回到素材的起始点了

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/dvcxmlfix/06.png)

### 分析

1. 直接一个一个改治标不治本

2. 直接用达芬奇渲染分段素材，又担心二次渲染对素材造成影响，以及无缘无故浪费机器性能

3. 直接拆剪辑表，把素材的出入点进行修改

### 实操

剪辑表的格式是普通的xml格式，用一些在线阅读器可以更明朗地观看：

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/dvcxmlfix/02.png)

而其中出现问题的就在每一个素材剪辑段的标签内：

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/dvcxmlfix/01.png)

要做的就是**筛选出指定标签里的剪辑点同步到素材出入点上**

nodejs代码：
```javascript
var fs = require("fs");
var xml2js = require('xml2js');
var parser = new xml2js.Parser();//用于解析xml为json对象
var builder = new xml2js.Builder();
fs.readFile(process.argv.slice(2)[0], "utf-8", function(err,data){
    if (err) {console.log("读取文件失败"); throw err;}
    parser.parseString(data, function(err,res){
        if(err) throw err;
        var medialist=res.xmeml.sequence[0].media[0]//获取media列表
        var vlist=medialist.video[0].track[0].clipitem
        var alist=medialist.audio[0].track[0].clipitem
        for(clip of vlist){
          clip.in[0]=clip.start[0]
          clip.out[0]=clip.end[0]
        }
        for(clip of alist){
          clip.in[0]=clip.start[0]
          clip.out[0]=clip.end[0]
        }
        var outxml = builder.buildObject(res);
        fs.writeFile("output01.xml", outxml.toString(), err1 =>{
            if (err1) {throw err1;}
            console.log("成功写入文件");
        })
    })
})
```

感谢大佬的代码。[引用来源](https://www.jianshu.com/p/a7a2b693d003)
