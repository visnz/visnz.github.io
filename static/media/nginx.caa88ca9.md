---
title: "nginx配置反向代理公开文件系统"
date: 2018-12-23
type: ["应用"]
weight: 7
tags:  ["计算机","服务器","NGINX"]
categories: ["服务器","最近"]
description: "应对业务需求上手nginx，将机器自身文件系统映射到公开网络"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/nginx/icon.jpg"
---

# 笔记

一个老生常谈的服务器引擎工具，拖了这么多年终于给安排上了学习一下。

一篇简单的[简介文章](http://blog.51cto.com/zhangfengzhe/2064524)可以看一下：Nginx是一款**轻量级的Web服务器**、**反向代理**服务器，由于它的**内存占用少**，启动极快，**高并发能力强**，在互联网项目中广泛应用。

早期nginx手动启动，现在使用systemd挂在后台服务，以及现在也有docker官方打包支持。

nginx本身自带虚拟服务器与systemd部署属于系统单例，并且配合一系列的扩展模块来完成服务，可以使用docker多实例部署（为什么要这样呢）。

在配置文件中可以完成对多台虚拟服务器的配置，完成流量分发等等。

配置文件位置：``/etc/nginx/nginx.conf``。[详细配置参考](https://www.w3cschool.cn/nginx/nginx-d1aw28wa.html)

```bash
# /etc/nginx/nginx.conf
error_log logs/error.log info;  # 指定错误日志的地址 info级别
worker_processes 2;             # 指定用于响应的进程

events {                        # 事件模块
    use epoll;                  # 指定使用epoll模型响应
    worker_connections 2048;    # 指定每个进程允许的最大连接数，
                                # 理论上每台服务器（虚拟）最大连接数为65535
    keepalive_timeout 60;       # 设置链接超时时间
}

# 设定http服务器，利用它的反向代理功能提供负载均衡支持
http{
    # 构建流量负载规则
    upstream guediao.cn{
        server IPAddressA:80 weight=3;
        server IPAddressB:80 weight=1;
        # 具体的流量负载规则还比较多，可参考文档
    }
        server{                     # 构建一个服务器
        listen *:80 default_server; # 指定为默认服务器
        server_name mainserver.org; # 命名
        root /home/visn/Desktop/;   # 指定服务器的根目录
        access_log logs/acc.log;    # 指定访问日志
        index index.html;           # 指定访问的主页
        location / {
            return 200 index2.html;  # 指定返回的内容
        }
        location /root {            # 如果地址是/root
            rewrite ^ /;            # 重定向到/
        }
        location /error {           # 如果地址是/error
            return 503;             # 抛出503页面
        }
        error_page   500 502 503 504  /50x.html;
        # 指定错误页面的地址
        location = /50x.html {
            root /home/visn/Desktop/;
        }
    }
    server{
        listen *:81;
        server_name routeserver;
        return 200 "helloworld";
    }
}
```
## 映射hugo作为测试

尽管hugo本身支持调用``hugo server``映射到本地做测试，一开始的想法来源于此那就也做一下试试

```bash
user visn;
events{}
http{
    server {                         # 构建一个服务器
        listen 80;   
        root /home/visn/git/blog/public;
    }
}
```

非常简单的配置就能将本地文件系统映射为公开地址

tips可要记得把``user``填上。因为这个忽略饶了不少弯路

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/nginx/01.png)

修正补充<sup>18.12.25</sup>

npm提供live-server工具，支持快速实现本地文件系统对外映射

不过就是没有``hugo server``方便，不是每按一次^S就自动渲染的

tips是记得把``config.toml``里的baseURL改成``/``

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/nginx/04.png)

live-server也集成了类似``live-reload``浏览器自动重载功能，本地的网页等内容修改，保存可以及时同步到浏览器做调试。（只是遇上了hugo是要md渲染web的嗯）

## 配置对外文件服务

主要应用场景比如用作服务器的时候，离线下载的任务丢上去，过段时间就可以打开网页直接下载

添加了一些实际的显示功能
```bash
    autoindex on;
    autoindex_exact_size off;
    autoindex_localtime on;
```

添加密码访问
```bash
    auth_basic "Enter your name and password";
    auth_basic_user_file /var/www/html/.htpasswd;
```
有一个经典用来生成这类访问密码的工具``htpasswd``，被放在了``archlinuxcn/apache-tools``（苦笑.jpg)

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/nginx/02.png)

生成一个与用户名对应的哈希，输入时候检查

htpasswd的一些使用：
```bash
htpasswd -c <dir> <username>          #创建用户
htpasswd -b <dir> <username> <passwd> #修改密码用户
htpasswd -D <dir> <username>          #删除用户
```

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/nginx/03.png)

基本完成

# 一些补充记录<sup>18.12.25</sup>

功能与对应配置：

1. 访问转发

    ```bash
    location / {
        proxy_pass https://www.google.com/;
        # 转发请求到别的网站去
        # 请求头转发参考proxy_set_header
    }
    ```

2. 提供一个地址监控nginx的状态

    ```bash
    location /NginxStatus {
        stub_status on;
        access_log on;
    }
    ```


# 写在最后
nginx的功能相当强大，目前也只是根据自己需要订制了一些简单的服务实现，以后还会有更大的用场。

nginx：我也只是做了一点微小的工作

---
参考资料：

1. [十分钟-Nginx入门到上线](https://juejin.im/post/58846fceb123db7389d2b70e)

2. [w3cschool Nginx 入门指南](https://www.w3cschool.cn/nginx/)

3. [【小哥哥, 跨域要不要了解下】NGINX 反向代理](https://juejin.im/post/5c0e6d606fb9a049f66bf246)