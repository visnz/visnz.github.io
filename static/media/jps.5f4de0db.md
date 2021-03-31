---
title: "Ubuntu搭建Jumpserver 脚本整理与问题解决"
date: 2019-03-05
type: ["应用"]
weight: 3
originweight: 7
hidetime: 2019-05-05
tags: ["计算机","JUMPSERVER","LINUX"]
categories: ["运维"]
description: "整理了自己搭建过程中，根据官方做的脚本整合，与问题解决记录"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps/logo.png"
---
# 写在前面
官方没有提供一键脚本，原本以为按照官方文档能够快速安装，实践过程中发生了诸多问题得手动解决，在这里做一些整合与简单记录

- 时间点：2019年3月4日
- 机房：Aliyun华南 Vultr东京
- OS：Ubuntu 18.04
- 安装目录：/opt/

# 脚本
## 声明变量与密码暂存

```sh
#!/bin/bash
# this script was installation of jumpserver in ubuntu 18.04
# ref: http://docs.jumpserver.org/zh/docs/setup_by_ubuntu.html
Server_IP='' 
# fill the Server_IP with IPv4
jumpserver_path='/opt/jumpserver'

file='./backup'
# this file as tempfile to save the passwd which generating by random:
if [ -f $file ]; then 
    DB_PASSWORD=`cat $file|grep DB_PASSWORD|cut -d ' ' -f 2`
    SECRET_KEY=`cat $file|grep SECRET_KEY|cut -d ' ' -f 2`
    BOOTSTRAP_TOKEN=`cat $file|grep BOOTSTRAP_TOKEN|cut -d ' ' -f 2`
    # 如果已经生成了几个密码，直接读取即可
else
    DB_PASSWORD=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 24`      # 生成随机数据库密码      
    SECRET_KEY=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 50`       # 生成随机SECRET_KEY       
    BOOTSTRAP_TOKEN=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 16`  # 生成随机BOOTSTRAP_TOKEN 
    echo "DB_PASSWORD $DB_PASSWORD" > $file
    echo "SECRET_KEY $SECRET_KEY" >> $file
    echo "BOOTSTRAP_TOKEN $BOOTSTRAP_TOKEN" >> $file
fi
```
## 基本组件更新与基础环境

```sh
apt update && apt -y upgrade 
apt -y install wget gcc libffi-dev git python-pip language-pack-zh-hans  mysql-server libmysqlclient-dev python3-pip python3-dev python3-venv  redis-server
```
redis设置的时候可能会出错
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps/01.png)
因为尝试对IPv6的地址进行监听，如果服务器不提供的话，会一直卡住。

```sh
## redis 修正
sed -i "s/bind 127.0.0.1 ::1/bind 127.0.0.1/g" /etc/redis/redis.conf 
sudo systemctl restart redis-server.service 
```

设置语言环境与数据库
```sh
echo 'LANG="zh_CN.utf8"' > /etc/default/locale
export LC_ALL="zh_CN.utf-8"
echo '基本软件安装完成'
mysql -u root -p -e "create database jumpserver default charset 'utf8'; grant all on jumpserver.* to 'jumpserver'@'127.0.0.1' identified by '$DB_PASSWORD'; flush privileges;"
```
## 进入虚拟环境安装
创建进入虚拟环境
```sh
python3 -m venv /opt/py3
source /opt/py3/bin/activate
```
jumpserver下载与依赖安装
```sh
mkdir $jumpserver_path
git clone https://github.com/jumpserver/jumpserver.git $jumpserver_path
# 此处用于获取依赖列表，列表随git更新
apt-get -y install `cat $jumpserver_path/requirements/deb_requirements.txt`
```
py组建安装
```sh
pip3 install --upgrade setuptools 
pip3 install wheel
# 组件修正。使用此处时候发生wheel相关缺失
pip3 install -r $jumpserver_path/requirements/requirements.txt 
```
jumpserver配置修改
```sh
cp $jumpserver_path/config_example.yml $jumpserver_path/config.yml
sed -i "s/SECRET_KEY:/SECRET_KEY: $SECRET_KEY/g" $jumpserver_path/config.yml
sed -i "s/BOOTSTRAP_TOKEN:/BOOTSTRAP_TOKEN: $BOOTSTRAP_TOKEN/g" $jumpserver_path/config.yml
sed -i "s/# DEBUG: true/DEBUG: false/g" $jumpserver_path/config.yml
sed -i "s/# LOG_LEVEL: DEBUG/LOG_LEVEL: ERROR/g" $jumpserver_path/config.yml
sed -i "s/# SESSION_EXPIRE_AT_BROWSER_CLOSE: false/SESSION_EXPIRE_AT_BROWSER_CLOSE: true/g" $jumpserver_path/config.yml
sed -i "s/DB_PASSWORD: /DB_PASSWORD: $DB_PASSWORD/g" $jumpserver_path/config.yml
```
启动服务器
```sh
$jumpserver_path/jms start all -d 
# 此处使用stop、restart等完成控制
```
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps/02.png)
## 对外接口
```sh
## docker安装
apt-get -y install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
apt-get -y update
apt-get -y install docker-ce
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
systemctl restart docker.service

## 部署 Coco
docker pull jumpserver/jms_coco
docker run --name jms_coco -d -p 2222:2222 -p 5000:5000 -e CORE_HOST=http://$Server_IP:8080 -e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN jumpserver/jms_coco
# BOOTSTRAP_TOKEN 为 Jumpserver/config.yml 里面的 BOOTSTRAP_TOKEN

## Guacamole
docker pull jumpserver/jms_guacamole
docker run --name jms_guacamole -d -p 8081:8081 -e JUMPSERVER_SERVER=http://$Server_IP:8080 -e BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN jumpserver/jms_guacamole
# 以上两个镜像官方文档使用了1.4.8稳定版，换成了latest

##  Luna
wget https://github.com/jumpserver/luna/releases/download/1.4.8/luna.tar.gz
mv luna.tar.gz /opt/luna.tar.gz
tar -zxvf /opt/luna.tar.gz
chown -R root:root /opt/luna

## Nginx
apt-get -y install curl gnupg2 ca-certificates lsb-release
add-apt-repository "deb http://nginx.org/packages/ubuntu/ $(lsb_release -cs) nginx"
curl -fsSL http://nginx.org/keys/nginx_signing.key | sudo apt-key add -
apt-get -y install nginx

rm -rf /etc/nginx/conf.d/default.conf
mv nginx.conf /etc/nginx/nginx.conf 
# 这里的nginx.conf要求跟这个脚本在同一个目录下

nginx -t  # 如果没有报错请继续
systemctl restart nginx

echo -e "\033[31m 你的jumpserver数据库密码是 $DB_PASSWORD \033[0m"
echo -e "\033[31m 你的SECRET_KEY是 $SECRET_KEY \033[0m"
echo -e "\033[31m 你的BOOTSTRAP_TOKEN是 $BOOTSTRAP_TOKEN \033[0m"
echo -e "\033[31m 请记得检查 $jumpserver_path/config.yml 文件\033[0m"
echo -e "\033[31m 你的服务器IP是 $Server_IP \033[0m"
```
# 问题记录
除了上面redis与IPv6、pip3的wheel，还有一个就是nginx反代开启之后的css丢失：
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps/03.png)
网页下载的时候是css文件被识别为``text/plain``
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps/04.png)
curl获取的header也是显示nginx对于css文件也采用``text/plain``
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps/05.png)

查找一些文档之后，了解到nginx里有一个文件``/etc/nginx/mime.types``，通过后缀把请求打上MIME类型标签
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps/06.png)

故整合官方给的nginx配置+导入这个文件后配置如下：
```
user root;
events{}
http{
    include /etc/nginx/mime.types; # 就是这行字（微笑
    server {
        listen 80;
        server_name mainserver;     # 你的域名
        client_max_body_size 100m;  # 录像及文件上传大小限制
        location /luna/ {
            try_files $uri / /index.html;
            alias /opt/luna/;
        }
        location /media/ {
            add_header Content-Encoding gzip;
            root /opt/jumpserver/data/;
        }
        location /static/ {
            root /opt/jumpserver/data/;
        }
        location /socket.io/ {
            proxy_pass       http://localhost:5000/socket.io/;
            proxy_buffering off;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            access_log off;
        }
        location /coco/ {
            proxy_pass       http://localhost:5000/coco/;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            access_log off;
        }
        location /guacamole/ {
            proxy_pass       http://localhost:8081/;
            proxy_buffering off;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $http_connection;
            access_log off;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location / {
            proxy_pass http://localhost:8080;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/jps/07.png)
尽情享用
# 宕机重启
```sh
# python环境
source /opt/py3/bin/activate
# 启动容器
docker container ls -a|grep jms_guacamole|cut -c 1-15|xargs docker start
docker container ls -a|grep jms_coco|cut -c 1-15|xargs docker start
# 启动主服务
# $jumpserver_path/jms start all -d 
/opt/jumpserver/jms start all -d 
```

# 写在最后
1. 虽然是按着文档安装，一路上还是遇到了很多不得不自己分析学习新内容，并进行调整的错误。
2. 尽管是写了脚本，在安装的时候还是记得实时监督，出现问题及时调试。密码在脚本一开始就生成，多次执行脚本不会覆写。三个密码保存在同目录下的backup，注意保管。
3. 有些IaaS厂商也提供了堡垒机安全管理的业务，怕麻烦可直接购买。
4. 附件：[脚本](/files/jps/jumpserver-ubt.sh) [nginx配置](/files/jps/nginx.conf)

# 相关链接

- [Ubuntu 18.04 安装文档](http://docs.jumpserver.org/zh/docs/setup_by_ubuntu18.html)

- [Jumpserver Github](https://github.com/jumpserver/jumpserver)