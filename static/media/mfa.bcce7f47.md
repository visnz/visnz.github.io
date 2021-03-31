---
title: "Authy进行网络账户安全认证"
date: 2018-12-25
type: ["笔记"]
weight: 7
tags: ["计算机","MFA","笔记"]
categories: ["笔记","最近"]
description: "不要绑定手机不要实名！使用基于otpauth的身份验证器启用验证码登陆"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/mfa/icon.jpg"
---
# 简述

MFA是种老生常谈的认证方式：

> [MFA](https://www.wikiwand.com/zh-cn/%E5%A4%9A%E9%87%8D%E8%A6%81%E7%B4%A0%E9%A9%97%E8%AD%89)：多重要素验证（英语：Multi-factor authentication，缩写为 MFA）是一种计算机访问控制的方法，用户要通过两种以上的认证机制之后，才能得到授权，使用(计算机)资源。

通常认证步骤越多，相对会越安全。

> 1) something they know, 2) something they have, or 3) something they are.例如，用户要输入PIN码，插入银行卡，最后再经指纹比对，通过这三种认证方式，才能获得授权。这种认证方式可以提高安全性。

其他的比如安全令牌（通过设备ID+时间生成一次性密码）、U盾（U盘认证工具，接入计算机后完成认证）

2FA是MFA的一种类型，即使用上述三种中的两种进行认证，在繁琐与安全的一个中点

两步认证(Two-step verification or two-step authentication)是对2FA的具体实现的称呼

# 谷歌提供的MFA功能

在[你自己的谷歌帐号设置-安全性](https://myaccount.google.com/security)页面在``登陆 Google``版块可以选择打开``两步认证``（需要在已经登陆的设备上先认证、配置备用方式），现在默认拥有几种认证方式

1. 已登陆设备批准（如在手机、iPad上有Google软件）

2. 刚刚的备用认证方式（手机电话、短信）

下面提供额外几种认证方式：

1. 备用验证码（与时间相关地生成临时的单次有效的验证码，可以记下来以备不时之需）

2. 实体安全密钥（安全令牌，插入计算机后可以完成登记）

3. **身份验证器**，在设备上下载一个身份验证器（应用软件，遵循一套算法生成密码进行认证）。会出现一个二维码，可以用身份验证器扫描添加。

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/mfa/02.png)

身份验证器的种类不少，这里使用[Authy](https://authy.com/)，在里面添加新的账户

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/mfa/03.png)

二维码的内容为：``otpauth://totp/Google%3Avisn0518%40gmail.com?secret=icm5pgwxi4mzf00000000000000000&issuer=Google``

采用传统otp认证方式。其中secret是开通时候生成的单次有效的32位的钥匙，扫码的时候在Authy里会记录下这个secret，与时间运算并生成**验证码**（所以关掉2FA再开的时候需要更新这个key）

再次登陆的时候就有要求使用身份验证器的认证啦

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/mfa/01.png)

其实同理，不少网站现在也慢慢支持了MFA。我现在自己Authy里使用的有：

1. Github

2. Microsoft Mail

3. Twitter

4. Aliyun

5. Google 

6. Vultr

7. Bitbucket

8. Cloudflare

除此之外还有不少网站支持MFA：

1. AWS

2. Amazon.com

3. Digital Ocean

4. Discord

5. Dropbox

6. EA

7. Facebook

8. HeroKu

9. Slack

10. Tumblr

11. [WordPress](https://github.com/WordPress/WordPress)

12. [JumpServer](https://github.com/jumpserver/jumpserver)

腾讯云、淘宝、Bilibili都没有提供该认证方式<sup>18.12.25</sup>

# 关于使用的原因
1. 国外大多服务账户，没有要求实名认证、手机绑定，最多需要你提供邮箱救援支持。相对于国内绑定隐私美其名曰安全，这种对等匿名的验证方式会更注重自己的隐私

2. 基于一套与时间相关的算法约定，可以节省不少用于认证的资源（短信验证每年的资源消耗量、造成的手机垃圾短信污染问题不容小觑），而且可以离线认证，这对于我这种一回宿舍就没网没信号的真是太棒了

3. ~~但是手机丢了就惨了~~当然服务账户允许通过注册帐号的邮箱重置密码，实在不放心可以在申请的时候把secret给记下来。

# 关于算法

在申请二步认证的时候，生成一个32位的secret钥匙，服务器一份，你验证器一份。

每隔一定间隔（比如30秒），验证器跟服务器都在这个时间间隔内，以相同的时间基点生成一个验证码。即``验证码 = f(时间基点,secret)``

在这个间隔期间填写验证码并与后台做比较，完成验证。所以只要时间是准确的（范围内），就可以实现离线认证。

算法是公开的，这些都不是什么新东西啦。用于开发的话，otp的库也到处都有啦，像[npm上的otpauth库](https://www.npmjs.com/package/otpauth)

---
参考资料

1. [資訊安全「Authy」兩階段驗證軟體](https://www.vedfolnir.com/authy-app-information-security-2-step-verification-support-12565.html)

2. [谷歌验证 (Google Authenticator) 的实现原理是什么？](https://www.zhihu.com/question/20462696)

3. [阮一峰的网络日志 - 双因素认证（2FA）教程](http://www.ruanyifeng.com/blog/2017/11/2fa-tutorial.html)