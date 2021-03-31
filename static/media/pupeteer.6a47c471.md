---
title: "Puppeteer初体验 & 爬网课视频"
date: 2018-05-09
type: ["应用"]
weight: 7
tags: ["计算机","爬虫","NODEJS","PUPPETTEER"]
categories: ["运维"]
description: "nodejs使用headless浏览器爬虫对网络课程资源进行爬取"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/pup/title.png"
---
## 背景
手痒点了Coursera课程试听，忘了这回事直到PayPal提示我一笔60美元的支出（心如刀割）。想想认真听课要不把它录下来，以后可以看。一个一个抓太麻烦…索性写个爬虫把视频都爬下来好了…

### 分析
日常万能抓的``Video Downloader professional``抓不了
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/pup/01.png)

网页源码里却如此露骨，还是静态的
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/pup/02.png)

未注册访问Coursera是可以访问到（我所购买课程）的所有课程，视频所在网址如``https://www.coursera.org/learn/daoyan-siwei-weiyingren/lecture/0W13U/fen-jing-tou-jiao-ben``，中间有一个五位``/[A-z0-9]/``防爬的文本，禁止爬虫直接根据名字爬到视频页面。

**通过Puppeteer模拟登陆，内部抓取所有视频的网址。再用Puppeteer自动打开网页抓取视频链接，再用下载工具批量下载重命名**

（虽然后来发现模拟登陆后更麻烦…索性一个一个自己复制了…）

## 工具
- [Nodejs](https://nodejs.org/)

- [Puppeteer](https://github.com/GoogleChrome/puppeteer)：无图形化运行chrome的API

## 编码
``npm install --save puppeteer``

不少教程给了[截图示例](https://blog.fundebug.com/2017/11/01/guide-to-automating-scraping-the-web-with-js/)，代码内容也一目了然
```js
async function getPic() {
  const browser = await require('puppeteer').launch();
  const page = await browser.newPage();
  await page.goto('https://google.com');
  await page.screenshot({path: './google.png'});
  await browser.close();
}
```

### 模拟点击
因为视频页面需要点击播放，视频区域才会被替换成视频链接，所以使用puppeteer本身浏览器模拟点击的API
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/pup/04.png)
通过css选择器找到点击的区域，调用``page.click("选择器的复制内容")``即可

### 最终
给定一系列键值对（排序编号为键，有视频的网页地址为值）
```js
var page = await browser.newPage();
//打开新页面
await page.goto(addr["value"],{ timeout: 50000});
//跳转网页，重订超时时间
await page.waitFor(5000);
//等待网页打开
console.log("in "+addr["value"]);
await page.click("#rendered-content > div > div > section > div.rc-VLPLoggedOutPage > div.rc-VLPVideoPlayer > div > div.video-overlay.video-js.vjs-big-play-centered > button");
// 模拟点击网页
await page.waitFor(5000);
// 等待网页响应
result = await page.evaluate(()=>{
  // 对网页元素进行抓取
  var title=document.querySelector("#rendered-content > div > div > section > div.rc-VLPLoggedOutPage > div.rc-VLPVideoPlayer > h1").innerText;
  // 抓取视频题目
  var ad=document.querySelector("#vjs_video_3_html5_api").getAttribute("src");
  // 抓取视频地址
  return {title,ad};
});
require("child_process").exec(`axel -n 10 \"`+result["ad"]+`\" -o ./`+addr["key"]+result["title"]+".webm\n",(e,so,se)=>{});
//直接调用本地axel下载已经抓取到的链接+重新命名
```

运行结果
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/pup/03.png)
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/pup/05.png)

### 遇到的问题

- 网站的反爬虫机制

- 开启chrome需要的资源有点…

- 底层promise rejections的报错处理

### 额外的想法

- 刷点击量？刷票？

- 刷网课！？（净是些不正当的想法）

- 新Amazon反反爬虫？
