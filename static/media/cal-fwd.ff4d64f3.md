---
title: "Google Calendar nodejs作cal轉發服務器（nginx后）"
date: 2018-12-07
type: ["应用"]
weight: 7
tags: ["NODEJS","计算机","服务器"]
categories: ["服务器","最近"]
description: "将行程表缓存并映射提供大陆网路访问，同步谷歌行程表"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/cal-fwd/icon2.png"
---

## 問題

- 心繫家庭，將自己的行程表分享給家人，不失爲加強聯絡的一個方法。

- 世界上最遠的距離不是6881公里，而是分享 Google Calendar 日程表到媽媽的手機裏

## TL;DR
- 用境外服务器定时缓存，或者转发获取
- nodejs我的蠢办法请继续阅读
- nginx聪明方法拉到最后

## 解決

- 因爲設置了cal的公開地址，寫個轉發服務器即可。

- 因爲不是谷歌服務的體系，沒有實施更新與推送，只能自己做心跳同步。

- ~~想借機搭建一下https服務器的看了大廠ca價格就軟了~~，[Let's Encrypt](https://letsencrypt.org/)開源證書玩一下下 -> [開源證書安裝教程](http://foofish.net/https-free-for-lets-encrypt.html)

    - [SSL提供安全套接層](https://www.jianshu.com/p/41f7ae43e37b)，在HTTP傳輸數據時提供非對稱加密。通常需要申請第三方認證機構獲取CA證書，年費多分佈在四位數到五位數，按域名算加成，需要定期續費。[加密原理簡介](https://www.netadmin.com.tw/article_content.aspx?sn=1106140008)

    - Let's Encrypt進行域名認證，要求在生成認證的時候在服務器上指定用於訪問獲取的指定隨機文件，用於驗證。

##  ssl方面

```bash
# server

openssl genrsa 4096 > account.key
openssl genrsa 4096 > domain.key
# 創建用戶key 域名key

openssl req -new -sha256 -key domain.key -subj "/CN=example.com" > domain.csr
# 驗證域名

mkdir /var/www/html/.well-known/
mkdir /var/www/html/.well-known/acme-challenge/
# 用了無腦爆炸的apache2驗證方式，創建等會驗證時候訪問的目錄
# 嗯僅作嘗試，正規驗證千萬別這樣

wget https://raw.githubusercontent.com/diafygi/acme-tiny/master/acme_tiny.py
python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir /var/www/html/.well-known/acme-challenge/ > ./signed.crt
# 獲取腳本，由剛剛用戶key與域名csr申請得到驗證，signed.crt爲最終驗證文件

```
用於持續驗證該ssl的[腳本](https://github.com/diafygi/acme-tiny)

在nodejs上載入
```js
const https = require('https');
const fs = require('fs');
const options = {
    key: fs.readFileSync('/home/----/domain.key'),
    cert: fs.readFileSync('/home/----/signed.crt')
};
https.createServer(options, (req, res) => {
  res.writeHead(200);
  res.end('hello world\n');
}).listen(8080);
```

關於配置過程發生[瀏覽器認定證書爲不安全](https://www.zhihu.com/question/40718588)的問題解決參考

## 定期獲取新的ics文件

- 因爲沒有推送跟回調，需要手動拉取最新的ics。使用``cron``設置服務器定時事務

```js
const https=require("https")
const fs = require('fs');
var CronJob = require('cron').CronJob;

const url="https://calendar.google.com/calendar/ical/user@example.com/public/basic.ics"
const caltmp="/home/----/cal-temp.ics"

//指定事務，每分鐘更新一次
new CronJob('0 *  * * * *', function() {
    if(sync(url,caltmp))
  console.log('sync completed');
}, null, true, 'Asia/Shanghai');

//同步函數，寫入文件
function sync(url,filepath){
    var caldata="";
    https.get(url,(res)=>{
        res.on('data',(data)=>{
            caldata+=data;
        })
        res.on('end',()=>{
            fs.stat(filepath,function(err,stat) {
                if(!(stat&&stat.isFile())) {
                    fs.open(filepath,'w',(e,fd)=>{if(e)console.log(e);})
                }
                fs.writeFileSync(filepath,caldata);
                console.log("download finished");
                return true;
            });
        });
    })
}
```

## 部署交付與後臺服務
- 部署交付服務：

```js
https.createServer(options,(req,res)=>{
    res.writeHead(200,{"Content-Type":"text/calendar;charset=UTF-8"})
    res.write(fs.readFileSync(caltmp))
    res.end()
}).listen(8080);
```

部署systemd服務：
``sudo nano /etc/systemd/system/cfsd.service``
```bash
[Unit]
Description=CalenderForwardServer Service
After=network.target
Wants=network.target

[Service]
Type=simple
PIDFile=/var/run/cfsd.pid
ExecStart=/usr/bin/nodejs /home/----/cfs/index.js
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

## 快點告訴媽媽

- iPhone 手機 ``設置 - 密碼與用戶 - 添加賬戶 - 其他 - 添加已訂閱日曆``，在其中輸入剛剛的交付地址。

- 靜候幾分鐘後即可同步訂閱日曆到媽媽手機


## nginx一行反代版
2019年3月13日修订

使用nginx一行反代即可完成以上工作：
```sh
# /etc/nginx/nginx.conf
user root;
events{}
http{
    include /etc/nginx/mime.types; 
    server {
        listen 13334;
        location / {
            proxy_pass https://calendar.google.com/calendar/ical/------%40gmail.com/public/basic.ics;
        }
    }
}
```
一行命令即可
```sh
docker run -dit -p 13334:13334 -v /etc/nginx/nginx-cfs.conf:/etc/nginx/nginx.conf nginx
```
