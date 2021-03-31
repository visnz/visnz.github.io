---
title: "MAC系统手动挂载读写NTFS文件系统"
date: 2018-09-02
type: ["日常"]
weight: 12
tags: ["计算机","MAC","NTFS"]
categories: ["日常"]
description: "解决MAC系统上自动挂载的NTFS外接磁盘无法写入文件操作的问题"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/mac-ntfs/logo.jpg"
---

## 背景

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/mac-ntfs/01.png)

由于一些历史原因，Mac操作系统上插入NTFS（微软开发）的磁盘时，只能读取而不能写入。

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/mac-ntfs/02.png)

无法新建文件夹、删除等操作

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/mac-ntfs/03.png)

Paragon等磁盘工具的收费版有提供写入的功能。而实际操作系统并没有这个限制，可以手动开启。

## 代码

```sh
# 获取传参磁盘的设备文件
devs=`mount|grep $1|awk '{print $1}'`
echo "selected hard disk"+${devs}

# 指定在卷的挂载点
newdevs="/Volumes/可写磁盘"
echo "new mount point"+${newdevs}

# 卸载现有磁盘的挂载点，创建新挂载点
sudo umount /Volumes/$1
sudo mkdir $newdevs

# 挂载设备到新挂载点，指明读写权限
sudo mount -t ntfs -o rw,auto,nobrowse $devs $newdevs

# 链接到桌面
sudo ln -s $newdevs ~/Desktop/WriteableNTFSDisk
```

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/mac-ntfs/04.png)

在桌面就出现重新挂载过的硬盘啦
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/mac-ntfs/05.png)
可以读写
