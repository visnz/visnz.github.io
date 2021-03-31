---
title: "[ 后续 ] Jumpserver 套SSL认证+CDN反代 "
date: 2019-03-09
type: ["应用"]
weight: 3
originweight: 7
hidetime: 2019-05-09
tags: ["计算机","JUMPSERVER","LINUX","CDN","SSL"]
categories: ["运维","服务器"]
description: "把堡垒机所在服务器丢到CDN反代保护，写了个脚本实现SSL注册+重新激活"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps/logo.png"
---

前文：[Ubuntu搭建Jumpserver 脚本整理与问题解决](https://visnz.github.io/post/application2/jps/)。在构建了一台管理资产的堡垒机，本身的安全很重要。在域名上使用SSL验证域名与服务器反代来加强安全。

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps2/1.png)

# SSL
有一些网站是提供免费证书试用，通常的商用证书年费也是千元级以上，这里用[Let’s Encrypt](https://letsencrypt.org/)的免费证书。网上关于如何申请Let’s Encrypt证书的教程太多了（移步谷歌），出于``定时激活``跟``多台服务器多个域名``，就整理成一个脚本。

基本原理：

1. 本地生成一个 Let's Encrypt账户私钥、一个域名钥匙。

2. 由域名钥匙生成一个域名证书请求文件(CSR)，验证机构将访问这个域名下的某个位置获取验证文件，以生成一个CRT证书

3. 从官方letsencrypt.org获取一个pem钥，与CRT证书合并产生一个``chained.pem``文件

4. 使用域名钥匙、``chained.pem``文件作为一个认证对，服务器可以实现https访问

```sh
# 脚本片段
YOUR_DOMAIN=""
DIR="/var/www/httpssl/acme-tiny"
NGINX_CONF='/etc/nginx/nginx.conf'

apt update -y;apt install  -y mlocate openssl git nginx
echo '更新本机软件完毕'
mkdir -p $DIR && git clone https://github.com/diafygi/acme-tiny.git $DIR/
echo '拉取acme-tiny'
## 创建CSR
# 创建一个 Let's Encrypt账户私钥，以便让其识别你的身份
openssl genrsa 4096 > $DIR/account.key
# 创建域名证书请求文件(CSR)
openssl genrsa 4096 > $DIR/domain.key
openssl req -new -sha256 -key $DIR/domain.key -subj "/" -reqexts SAN -config <(cat `updatedb ;locate openssl.cnf|grep usr` <(printf "[SAN]\nsubjectAltName=DNS:$YOUR_DOMAIN")) > $DIR/domain.csr

# 临时替换 nginx 配置，开放访问
mv $NGINX_CONF $NGINX_CONF.bak
echo "events{}http{server{listen 80; server_name $YOUR_DOMAIN;location /.well-known/acme-challenge/ {
        alias $DIR/;
    }}}">$NGINX_CONF
systemctl reload nginx;
nginx -t && echo "nginx 临时配置完成"

python $DIR/acme_tiny.py --account-key $DIR/account.key --csr $DIR/domain.csr --acme-dir $DIR/ > $DIR/signed.crt
## 此时得到证书 signed.crt
mv $NGINX_CONF.bak $NGINX_CONF
systemctl reload nginx;
nginx -t && echo "nginx 配置恢复完成"

wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > $DIR/intermediate.pem 
cat $DIR/signed.crt $DIR/intermediate.pem > $DIR/chained.pem

echo '已生成 $DIR/chained.pem 与 $DIR/domain.key'

```
## 注册renew服务

重新验证大概如下：
```sh
# 检测 $DIR/chained.pem 文件存在与否，不存在则实行：
## 构建临时nginx
mv $NGINX_CONF $NGINX_CONF.bak
echo "events{}http{server{listen 80; server_name $YOUR_DOMAIN;location /.well-known/acme-challenge/ {
        alias $DIR/;
    }}}">$NGINX_CONF
nginx -t && echo "nginx 临时配置完成"
systemctl reload nginx;

## 获取生成新证书
python $DIR/acme_tiny.py --account-key $DIR/account.key --csr $DIR/domain.csr --acme-dir $DIR/ > /tmp/signed.crt || exit
wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > $DIR/intermediate.pem
cat /tmp/signed.crt $DIR/intermediate.pem > $DIR/chained.pem

## 恢复nginx
mv $NGINX_CONF.bak $NGINX_CONF
nginx -t && echo "nginx 配置恢复完成"
systemctl reload nginx;
```
p.s. 可以执行脚本手动刷新

## 注册到cron
```sh
# 注册事件
chmod +x `pwd`/acme-tiny.sh
echo  '0 0 1 * * '`pwd`'/acme-tiny.sh' >./temp && crontab -u root temp && rm temp
systemctl restart cron
# 此处不同系统使用的 cron 会有所不同 比如 cronie 等
echo '已注册自动事件，请勿删除脚本'
```


脚本执行完基本是这样
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps2/01.jpg)

# nginx.conf

主要修改有两个，一个是指定服务通过443端口访问且开启ssl验证。另一个是将到80的流量重定向URL到443。
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps2/02.png)
左为新文件，右为原文件
```sh
# 重启
systemctl reload nginx;
```
# CDN加速
CDN加速原理约等于机房缓存，按一定淘汰算法淘汰（一般来说访问量越多，网站越快）。国外知名Cloudflare在国内表现不佳，就直接使用了自家阿里云的CDN全站加速了

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps2/03.jpg)


域名原本解析到IP地址，由CDN接管后，域名使用CNAME直接解析到CDN为你分配的节点即可：
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps2/04.png)
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps2/05.png)

# 最后

1. ssl套上之后或许要重启或重新部署docker（启动配置对于自身的解析）

2. 嗯在搞了免费证书之后阿里云客服打电话问我需不需要免费证书……他们那里有提供

3. 国内CDN用阿里云还不错，国外套CF吧

4. [脚本](/files/jps/acme-tiny.sh)