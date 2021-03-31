---
title: "猫国建设者 长篇自动化脚本(外挂?)"
date: 2019-08-16
type: ["日常"]
weight: 2
tags: ["脚本","游戏","jquery"]
categories: ["日常"]
description: "继上一篇文章使用jq+devtool之后继续开发成游戏自动脚本"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/moe/icon2.png"
---
<link rel="stylesheet" type="text/css" href="/css/tag.css"> 

## 前言

”编程学习是从写外挂开始的“这句话还是有些道理的。自从前几天用jquery设置定时事件后，再也无法阻挡懒癌发作的脚步。

功能已经集成了封面图的内容，还有许多微小的工作

## 原理与结构

1. 常量层：这一层记录从游戏页面抓取的信息与脚本的常量信息，包含部分按钮位置信息、脚本默认间隔时间等。

        const defaultDelay = 60000
        const Jobs = {"伐木工": 0,"农民": 1,"学者": 2,"猎人": 3,"矿工": 4,"牧师": 5,"地质学家": 6,"工程师": 7}

2. 接口层：把游戏操作经过常量层的信息，抽象出可以调用的接口，比如定位到资源换算按钮、职业分配加减的按钮等操作。
2. 脚本逻辑层：把接口组合成脚本逻辑，如简单的换算所有木材、分配职业、自动调整最低农民<tag>TBD</tag>、保存游戏、获取当前木材数量、换算脚手架、进入某个面板，同时设定面板展示信息与执行间隔（使用``set/clearInterval()``函数），如管理工作方法、定义面板：

        管理Jobs = () => {
            _记录当前面板Index()
            _进入管理面板()
            _管理Jobs()
            _返回记录面板Index()
        }
        AllOperators = [{
            panel: "基本",
            list: [
                { DisplayName: "抬头看天空", timer: fastDelay },
                { DisplayName: "自动打猎" },
                { DisplayName: "赞美太阳" },
                { DisplayName: "管理Jobs", timer: verySlowDelay },
                { DisplayName: "小猫升职器", timer: verySlowDelay },
        ]}

3. UI逻辑层：按照脚本逻辑提供的面板对象进行渲染、注册事件

具体可以直接查看[源码](https://github.com/visnz/Allend-practice/blob/master/js/moejq.js)

可以直接在浏览器的``console``直接C/V/Enter[这部分内容](https://github.com/visnz/Allend-practice/blob/master/js/dist/_moejq.js)

### 杂记

1. 使用了``"use strict";``严格模式。
2. 建立了简单的MVC模型，由M生成V并注册C，只注册了简单事件通过C修改V（毕竟没有强交互逻辑）
3. 左边的合成面板跟职业面板是固定的，可以直接通过索引获取。但是建筑物不是固定的。
4. for循环连点采集猫薄荷的<tag>卡机刨~~木~~坟法</tag>，狂吃内存刷猫薄荷。中速（每秒10000次，可以出10K猫薄荷），定期保存。
5. UI层使用了多层map/reduce来形成统一渲染，注册每一个事件都使用了``setInterval()``的回调
6. 写了一个div专门作用于log，使用logThis()进行屏幕输出。
7. 特殊的样式直接写死在编码中，其余使用了网页原本的css样式
8. compliers：写了一个``.vscode/task.json``设置，用于优化js：

        "java -jar /usr/bin/closure-compiler-v20190729.jar --js ${file} --js_output_file  ${fileDirname}/dist/_${fileBasename}"

## TBD

- [ ] 内存定期清除
- [ ] 配置保存
- [ ] 添加到tampermonkey
- [ ] 可以自定义脚本策略
- [ ] 可以调整时间

### 策略版块

1. 最低人员工作安排：通过判断增减情况来调整人员情况，以确保每个资源有最少的人员需求
2. 全速冲资源：安排最少人员确保基础资源，空出最多人数，进而指定某一个资源进行全速增进。
3. 监听增长某一种建筑物、某种科学。（自动合成+点击增长）

etc...