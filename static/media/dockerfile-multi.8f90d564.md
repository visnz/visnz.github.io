---
title: "Dockerfile多阶构建带nginx的Hexo站点镜像（？"
date: 2019-04-11
type: ["应用"]
weight: 7
tags: ["Dockerfile","nginx","hexo","多阶构建","FRP"]
categories: ["运维"]
description: "Dockerfile多阶构建，节点免node依赖部署hexo站点"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/dockerfile-multi/logo.png"
---


# 起因
因为gitea部署在阿里云，wercker每次的构建部署机器不是固定的，Webhook到指定ipaddress不是每次都可以发得出去。

所以每一次push不一定能等到 build & rsync 。就索性写个Dockerfile吧团队的人也能拉去部署

（有考虑过nginx转发webhook，可行性暂未知）
（gitea的webhook能通过在阿里云上配置v2ray转发与否，可行性暂未知）

~~小声比比：反正也是用``hexo g``＋nginx一个反代就够了 干啥要搞那么麻烦（划掉~~

# 思路
用node镜像hexo渲染，把public部分放到有nginx的镜像中，暴露端口，直接访问

# [Dockerfile](http://hub.guediao.top/visnz/hexo-guediao/src/branch/master/Dockerfile)
```
FROM node
ENV BLOG_ADDRESS=http://hub.guediao.top/visnz/ \
    BLOG_NAME=hexo-guediao
# 引入node并添加易变变量

RUN git clone $BLOG_ADDRESS$BLOG_NAME  
WORKDIR /$BLOG_NAME/

RUN npm install -g hexo-cli \
    && npm update \ 
    && hexo generate 
# update步骤不要忽略，单独安装hexo－cli后会提示找不到hexo等错误

FROM nginx 
ENV BLOG_NAME=hexo-guediao

WORKDIR /$BLOG_NAME/
COPY --from=0 /$BLOG_NAME/public /$BLOG_NAME
# 从上一步骤中复制文件到新镜像中
RUN echo "user root;events{}http{include /etc/nginx/mime.types;server{listen 80;location / {root /$BLOG_NAME/;}}}" > /etc/nginx/nginx.conf
EXPOSE 80
# 重写nginx配置
```

# 构建运行
```sh
# $pwd = /which/git/clone/to/hexo-guediao
docker build . -g hexo-guediao
docker run -it --rm -p $yourPort:80 hexo-guediao
```
# 后话
一开始是朋友说想拿去自己构建看看，问帮忙写个Dockerfile
后来写完才想想…… 

你用``hexo server``不就好了吗……

算是找了个机会接触docker多阶构建吧