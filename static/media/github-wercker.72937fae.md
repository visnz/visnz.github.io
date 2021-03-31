---
title: "Github图床瘦身记+Wercker多地挖坑初体验"
date: 2019-04-05
type: ["应用"]
weight: 2
tags: ["计算机","GITHUB","Wercker","CICD"]
categories: ["运维"]
description: "Github当图床，图片字体整顿瘦身，Wercker自动部署双源站"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/github-wercker/logo.png"
--- 
# 缘起
许早之前参照[大佬 Wercker+Hugo+Github Pages 搭建並自動部署博客](https://axionl.me/2017/12/25/wercker-%E8%A9%A6%E6%B0%B4/)部署的博客，关于wercker CICD方面一直没有时间搞搞，现因毕业设计等大坑的缘故便开始学习起来。

## 问题
1. 弄清具体CICD各路过程以及撰写新的部署流程
2. 博客图片过多需要外迁到图床

# TL;DR
简单描述CICD部署结构：

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/github-wercker/01.png)

瘦身方面将博客内的图片、css中的font链接都换成了Blog项目里的``raw.githubusercontent.com``地址，由于经``hugo-build``不携带巨量图片、字体文件，渲染生成的静态博客文件从130M直降到5.2M。故图片、字体存在Blog项目中，博客文件引用自Blog项目。

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/github-wercker/02.png)

# 关于瘦身
尽管找了一些公用免费图床（国内又拍、七牛，大佬自建sm.ms），以及开源自建图床[Lychee](https://github.com/electerious/Lychee)，想想还是感觉不放心的样子。毕竟想到万一哪里崩溃了也得迁移麻烦，而且跟项目分开了整理起来也麻烦。

所以索性丢到blog项目里吧不管了（你们墙内看不到就看不到吧哼）

把pics从static里移出来+所有md把相对路径``/pics/``改成github提供用户存储的``https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/``

# CI/CD
关于CICD的解释网文已经有很多，简单来说就是使用一套自动化系统，定义一系列的执行流程，以达到从代码提交实现环境建设、测试、渲染、部署、报告的功能。如上一节插图所示，不少CICD系统支持通过github使用webhook触发，根据项目目录下的自动化操作配置文件，进行操作。

在这里因为早期遗留问题使用wercker。推荐[Travis CI](https://travis-ci.org/)。用于生产上尽管可以自建但如果不是量产级+低延迟需要，还是推荐使用第三方。

## wercker.yaml文件
最后写成的双路部署文件在[这里](https://github.com/visnz/blog/blob/master/wercker.yml)
结构是这样的
``` yaml
box: debian
# box指定使用的基础镜像
build-guediao:
# 创建一个流程，可自己命名，流程无关顺序
    steps:
    # 定义步骤
        - arjen/hugo-build: 
        # 在wercker有 steps store 里面有大部分需要用到的步骤
            config: config.guediao.toml
            # 步骤配置参数，可查
deploy-gitea:
    steps:
        - install-packages:
        # 此步骤包含对软件安装的封装，debian调用apt，诸如此类
            packages: openssh-client rsync
        - add-to-known_hosts:
        # rsync需要使用ssh链接，添加信任主机
            hostname: $HOSTNAME
            # 环境变量，在wercker新建后可定义
            # 或者在wercker中创建步骤后可定义单独的变量
        - mktemp:
            envvar: PRIVATEKEY_FILE
            # 创建临时文件，此处没有 $ 
        - create-file:
            # 创建一个临时文件存储密钥，rsync调用需要
            name: write key
            filename: $PRIVATEKEY_FILE
            content: $Deploy_to_Guediao_PRIVATE
            overwrite: true

#        - script:
#            # 此处亦可使用scp，则前面无需安装rsync
#            name: transfer application
#            code: |
#                scp -r -i $PRIVATEKEY_FILE public/* $DEPLOY_USER@$HOSTNAME:$DEPLOY_PATH/

        - sh4pe/rsync-deploy:
                destination: $DEPLOY_PATH   # 在环境变量中指定 最终要同步到服务器上的文件地址
                source: ./public/           # 此处指定只复制hugo渲染出来的public目录
                sshkey: $PRIVATEKEY_FILE    # 在环境变量中指定 一个临时的密钥存放地址
                user: $DEPLOY_USER          # 在环境变量中指定 登陆用户
                host: $HOSTNAME             # 在环境变量中指定 主机地址
```
尽量不要在配置文件中硬编码，在wercker的环境变量（或分步骤的单独变量）中，可以进行灵活调整而不需要修改配置文件。以上所引用到的变量配置如下：

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/github-wercker/03.png)

前两行是通过右下角``Generate SSH Keys``生成，记得将``PUBLIC``公钥添加到 $DEPLOY_USER 下``~/.ssh/authorized_keys``之中，rsync才可以登陆访问

其余的都是字面意思。

因为博客baseurl问题，配置了两个config.toml以及两个工作流来完成多路部署：

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/github-wercker/04.png)

一次push之后即可自动完成从``hugo-build``到``rsync``、``hugo-build``到``gh-page``两个工作流

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/github-wercker/05.png)

剩下的配置访问就自己搞搞啦

# 最后
emmm 说出来可能很蠢，不少次都是直接在master修改``wercker.yml``commit然后push（tg的gitbot一直响个不停），以至于两天28个commit而其实啥事都没做……现在考虑一下把项目内迁到gitea再做实验呜

