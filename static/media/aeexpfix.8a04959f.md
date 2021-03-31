---
title: "修复ae中英版本差异导致js表达式错误"
date: 2018-11-02
type: ["影视"]
weight: 9
tags: ["ADOBE","影视","计算机"]
categories: ["影视"]
description: "After Effect顶层使用javascript做脚本，中英模板之前差异常出现引用丢失的问题"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/aeexpfix/ae.png"
---

# 背景

下载一些模板打开之后发现预览窗口下警告“表达式错误”，导致效果无法显示

原因：通常是因为原来的模板，在使用系统组件的表达式中引用了英文，在中文的ae中打开后，系统组件无法被正确引用。

# 解决思路

## 手动修bug法：在图层表达式中修复引用

把``英文名引用``换用``中文名``，**量小的时候建议修复**，量多的话参考第二个解决思路

中英的关系如下：

1. 滑块 = Slider

2. 角度 = Angle

3. 复选框 = Checkbox

4. 颜色 = Color

5. 点 = Point

6. 图层 = Layer

7. 3D 点 = 3DPoint

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/aeexpfix/01.png)

通过提示直接查找每一个错误的引用，将引用（在图层里，通常是红色代表出错）的表达式里的英文，转成对应的中文

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/aeexpfix/02.png)

发现错误，找到表达式如下：

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/aeexpfix/03.png)

打开“Your Text”图层，效果如下

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/aeexpfix/04.png)

将“Fast Blur”改为“快速模糊（旧版）” 提示仍有错误，根据提示将“Blurriness”改为“模糊度”，成功改正错误

## 一劳永逸法：修改底层中英引用

软件里的系统组件的语言文本，将中文引用改为英文引用，其他不变，使其适配英文表达式。**出现大量错误时候推荐，一劳永逸**

在Support Files-zdictionaries（win）或Contents/Dictionaries/zh_CN（mac）文件夹（语言文件相关）下，找到after_effects_zh-Hans.dat

改动常见的语言错误如下图。（中英对应解决方案1中的中英映射）

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/aeexpfix/05.jpg)

之后 reload AE 即可

# 总结

你看这个ae英文版它又大又圆，真香。

# 参考资料

- [贴吧吧友解决方案](http://tieba.baidu.com/f?kz=5626448845&red_tag=m3200829573)

- [表达式修复工具](http://www.lookae.com/universalizer3/)

- [详细修复及原理博客](https://blog.part3.me/ae-fix-expression-error/)
