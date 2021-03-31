---
title: "flv/mkv文件提取pr可导入格式"
date: 2018-08-09
type: ["影视"]
weight: 16
tags: ["影视"]
categories: ["影视"]
description: "屡次被问flv跟mkv格式为什么不能导入pr，以及如何解决"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/flv/shuoshu.png"
---

### FLV
早期视频导入flash后导出swf体积庞大，flv（Flash Video）格式早期产生用于解决这个问题。flv是**flash文件作为外壳保护（封装）视频**，并转换成串流格式。

同时：

1. 外壳可以保护原视频内容、版权等

2. 串流媒体传输支持在网络上进行点播（视频点播网站常用，现在逐渐被H5取代）

3. 视频多用h.264编码，音频多用acc或mp3，但是有一层外壳，可以直接提取

4. pr并不是没办法导入flv，只要有相应编解码器就可以导入（早期pr能解H.263，可以导入早期flv视频）pr用的编解码器体系参见AME，导入不了大多是flv的封装格式AME非能解

解决办法：

1. 直接对flv文件进行去外壳+重新封装（**小丸工具箱-封装，flv进去mp4出来最暴力**）

2. 抽出视频（含外壳）+抽出音频，再手动封装

![小丸工具箱](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/flv/02.png)

（支持批量）

![对比图](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/flv/01.png)

### MKV

mkv（Matroska格式的一种）提供容器格式，用于存放多条视频、多条音频、多份字幕等，同时也支持串流媒体传输，支持选单（像DVD一样）

不能拉入pr的原因是包含多个黏在一起的视频、音频、字幕等，而无法作为一个单独的素材导入，故需要提取

小丸工具箱自带抽取工具（以及第三方抽取工具）
![小丸工具箱](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/flv/03.png)

同时在封装版面也提供了mkv封装

### 硕鼠flv下载器

硕鼠也提供了bilibili专门的下载器（能突破限速，这个还是蛮实用的）

同时可以安装上“硕鼠转换”，可以直接转换flv到mp4（暂无mov？），不过没有参数可调，而且压缩得很惨

![硕鼠](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/flv/05.png)

(最后89M被压剩14M)


另外在bilibili大视频会分片，以及番剧没办法在“kanbilibli”直接找到（可以去kanbilibili上搜索，依然可以找到），硕鼠可以直接根据番剧地址下载

![硕鼠](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/flv/06.png)
同时提供了“硕鼠合并”，可以直接只能拼合（不会删除原分片）
