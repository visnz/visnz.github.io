---
title: "Linux上调用ffmpeg组件压制字幕"
date: 2018-12-28
type: ["应用"]
weight: 9
tags: ["Linux","计算机","影视"]
categories: ["计算机","影视"]
description: "小丸顶层mp4box，底层封装包括不限于x264与ffmpeg，在Linux上直接调用ffmpeg完成在字幕压制"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/wan/icon.png"
---

## 背景
N久之前帮人拍字幕送去审查，这都9102年了才送回来反馈。

修改完要渲染了，用惯Linux不想回Windows，压字幕又要用到小丸工具箱，小丸工具箱又没有Linux version。（~~wine~~）

后来想想小丸底层也是用的ffmpeg，不如直接命令行翻译强上。

## 查找小丸底层命令

在Windows系统盘``HOST_WIN10/Program Files (x86)/MarukoToolbox/logs/``搜出来一份小丸的log文件，可在log中找到小丸的底层调用：

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/wan/00.png)

> "C:\Program Files (x86)\MarukoToolbox\tools\ffmpeg.exe" -i H:\201-5.mp4 -vn -sn -c:a copy -y -map 0:a :0 "C:\Program Files (x86)\MarukoToolbox\temp\201-5_atemp.aac" 

第一个命令是用ffmpeg抽取视频的音频（这个是这批素材的音频问题，需要单独抽离出音频）

> "C:\Program Files (x86)\MarukoToolbox\tools\x264_32-8bit.exe" --crf 10 --preset 8  -I 250 -r 4 -b 3 --me umh -i 1 --scenecut 60 -f 1:1 --qcomp 0.5 --psy-rd 0.3:0 --aq-mode 2 --aq-strength 0.8 --vf subtitles --sub "H:\201-5.srt" -o "C:\Program Files (x86)\MarukoToolbox\temp\201-5_vtemp.mp4" "H:\201-5.mp4" 

第二个是调用x264工具进行字幕编码，输出视频，指定像素比、码率、压制方式等等

> "C:\Program Files (x86)\MarukoToolbox\tools\mp4box.exe" -add "C:\Program Files (x86)\MarukoToolbox\temp\201-5_vtemp.mp4#trackID=1:name=" -add "C:\Program Files (x86)\MarukoToolbox\temp\201-5_atemp.aac#trackID=1:name=" -new "C:\Users\Administrator\Desktop\result\201-5_batch.mp4" 

第三个是把刚刚的两步产生的视频音频组合

## ffmpeg

在查阅了一些资料后，发现抽离、渲染、组合只需要用ffmpeg就行：

1. 创建基本文件夹：``temp``、``output``

2. 同第一步，抽取音频，保存到temp中
    ```bash
    ffmpeg -i $1.mp4 -vn -sn -c:a \
        copy -y -map 0:a:0 ./temp/$1_atemp.aac
    # 抽取acc音频
    ```

3. 因为ffmpeg不支持srt格式，需要转成ass
    ```bash
    ffmpeg -i $1.mp3-字幕.srt ./temp/$1.ass
    # 转换字幕格式
    #（带“.mp3-字幕”是抽音轨识别后科大迅飞自动命名的）
    ```

4. 将抽取的音频+字幕+音频丢进去压制
    ```bash
    ffmpeg -i $1.mp4 -i temp/$1_atemp.aac \
        -vcodec libx264 \       # 指定编码器
        -deinterlace \          # 指定逐行扫描
        -crf 10 \               # CRF速率控制方法
                                #   0为无损，18为视觉无损，
                                #   预设为23，上限为51
        -preset 8 \             # 指定渲染预设
                                #   有10个速度与质量互扼的预设
                                #   veryslow（第二慢） 为 16
                                #   通常指定1～6
        -qcomp 0.5 \            # 指定VBR压缩
        -psy-rd 0.3:0 \         # 降低psy算法的工作量
        -aq-mode 2 \            # 弹性量化模式 指定自动变化
        -aq-strength 0.8 \      # 弹性量化强度降低
        -vf "ass=temp/$1.ass"   # 指定过滤器为字幕
        output/$1_rend.mp4      # 指定输出视频位置
    ```

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/wan/01.png)

可以运行，输出也是正确的，十六核满爆输出

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/wan/02.png)


附上[脚本文件](/files/ffmpeg.sh)

感谢灿杰大佬给了我多一篇博客的机会

---
参考资料

1. [h264编码参数](https://www.jianshu.com/p/b46a33dd958d)

2. [FFMPEG使用参数详解](https://zhuanlan.zhihu.com/p/31674583)

3. [wiki FFmpeg](http://wiki.webmproject.org/ffmpeg)

4. [使用FFmpeg将字幕文件集成到视频文件](https://www.yaosansi.com/post/ffmpeg-burn-subtitles-into-video/)