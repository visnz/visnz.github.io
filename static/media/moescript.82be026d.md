---
title: "猫国建设者 DevTool+jquery脚本 制作游戏自动脚本"
date: 2019-08-13
type: ["日常"]
weight: 2
tags: ["脚本","游戏","jquery"]
categories: ["日常"]
description: "使用浏览器调试功能撰写自动执行脚本"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/moe/icon.jpg"
---
<link rel="stylesheet" type="text/css" href="/css/tag.css"> 

## 前言

沉迷猫国建设者却不经常上，放到了PC上挂机。放到了PC还是没办法做到三分钟看一次处理一次，索性用js写个脚本

## 思路

通过浏览器的开发工具，定位页面按钮，设置定时回调完成脚本动作

## 过程简述

1. 存档导出粘贴进来即可，在页面右键调出调试工具（DevTool）进入控制台
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/moe/01.png)
2. <tag>此步骤可能引入漏洞产生风险</tag>。控制台下方输入``allow pasting``开启脚本粘贴。
3. 使用CSS选择器获取元素：需要点击或获取信息的部件，通过查看元素获取，剪贴板会获得一个字符串，如：``div.res-row:nth-child(6) > div:nth-child(6) > div:nth-child(1)``
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/moe/02.png)
    ![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/moe/03.png)
4. 函数调用：
     - 使用``$("#ELEMENT_ID").text()``获取内容，返回字符串，可以用于判断
     - 使用``$("#ELEMENT_ID").click()``模拟点击
     - 更多功能参考[jquery手册](https://www.w3school.com.cn/jquery/jquery_reference.asp)
5. 使用``async``回调来设置异步事件(原理层面的解释)：
    ```js
    // 设置一个0延迟的异步回调，相当于一个新的并行线程
    setTimeout(async()=>{
        while(true){
            // 设置一个轮询的回调事件：阻塞一段时间执行某一部分内容
            await new Promise(resolve=>setTimeout(resolve,MS))
            // TODO
        }
    },0)
    ```

## Example

空余全派遣为猎人，设置定间隔点击``派出猎人(N次)``按钮

```js
// 设置一个0延迟的异步回调，相当于一个新的并行线程
setTimeout(async()=>{
    while(true){
        await new Promise(resolve=>setTimeout(resolve,100000))
        // 100秒点击一次猎人
        $("#fastHuntContainerCount").click()
    }
},0)
```

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/moe/04.png)

或者换用更简便的``setInterval(callback,ms)``,上文可以改写为:

```js
// 设置一个0延迟的异步回调，相当于一个新的并行线程
setInterval(()=>{$("#fastHuntContainerCount").click()},100000)
```

在控制台里粘贴执行即可

## 后记

官方有提供机器人[小猫科学家](https://likexia.gitee.io/cat-zh/wiki/?file=004-%E7%AC%AC%E4%B8%89%E6%96%B9%E5%B7%A5%E5%85%B7/02-%E7%8C%AB%E5%92%AA%E7%A7%91%E5%AD%A6%E5%AE%B6)，喜欢自己动手可以自己写一写