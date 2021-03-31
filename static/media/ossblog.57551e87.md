---
title: "使用Aliyun OSS托管hexo网站"
date: 2019-07-21
type: ["应用"]
weight: 2
tags: ["计算机","hexo"]
categories: ["运维"]
description: "阿里云OSS对象存储支持简单静态页面的托管，可与其SSL、CDN、DNS等业务接洽，价格按流量与存储计费（几乎白嫖）"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/ossblog/icon.png"
---

## 前言
改造前原本的博客流程：

> 
>     hugo -> gitpush -> Wercker -> Github page (images ref github)
>
>                     -> Dockerfile -> Dockerhub -> deploy (images ref github)
>
>     hexo -> gitpush -> Dockerfile -> Dockerhub -> deploy (images ref github)
> 

Aliyun OSS对象存储用来存储可交付对象，同时支持简单静态页面的托管，可与其SSL、CDN、DNS等业务接洽，价格按流量与存储计费（几乎白嫖）。

改造后：

> 
>     hugo -> git push -> Wercker -> Github page (images ref oss)
>
>          -> ossutil -> deploy (images ref oss)
>
>     hexo -> git push 
>
>          -> ossutil -> deploy (images ref oss)
>      

改造后基本解决了在境内无法读取raw.githubusercontent.com网址图片的问题

## 动手
申请开通阿里云OSS服务，创建一个Bucket，作为一个对象库。通常按业务命名，以及访问，如``guediao``的话，将一个``test``文件上传到这里后，要调用的URL前缀就是：``guediao.oss-cn-shenzhen.aliyuncs.com/test``

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/ossblog/01.png)

比如上面这张图，在项目里的位置是``/pics/ossblog/01.png``，github保存的地址是项目的``https://raw.githubusercontent.com/visnz/blog/master/pics/ossblog/01.png``

放到oss引用的时候就填写``https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/ossblog/01.png``

Bucket默认只能内网读写，需要设置公开可读

### 手动上传
官方提供了不同的[ossutil](https://help.aliyun.com/document_detail/44075.html)，包括桌面版、CLI等。选择了CLI并编写了相关脚本。

具体的流程是：

1. 上传OSS需要AccessKey ID和AccessKey Secret来认证，[申请](https://usercenter.console.aliyun.com/#/manage/ak)后切记妥善保管。**使用这对数据可以修改一系列的你Aliyun的内容**。

2. ``chmod +x ./ossutilmac64 && ./ossutilmac64`` 并填写相对应的配置，具体可参考[官方帮助文档](https://help.aliyun.com/document_detail/120075.html)

3. 存放图片并进行同步。hugo我把pics目录单独拿出来外面，即我要实现``/pics/``到oss上``https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/``的同步：
    ```
    ./`ls|grep ossutil` cp -r -f $1 oss://visnonline/$1
    # 递归强制复制
    # do:  ./synctooss.sh pics/
    ```

然后就能看到图片已经到图床了

4. hexo是把整个站托管上去，就直接在博客根目录同步就行。[网站](https://guediao.top)

## 已发现问题
1. 作为静态存放，oss只能访问index.html而没办法通过目录地址打开网页，即``xxx.com/abc``会跳转到``xxx.com/index.html``而不是``xxx.com/abc/index.html``
2. ossutil强制复制是全部重新传输，只能手动选择进行差异传输
3. 以后就靠hexo+oss快速建站了