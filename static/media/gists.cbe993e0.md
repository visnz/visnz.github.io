---
title: "VSCode記 Gists同步部署生產環境(Atom&Vim)"
date: 2018-12-05
type: ["应用"]
weight: 7
tags: ["计算机","GITHUB","VSCODE"]
categories: ["运维"]
description: "使用gists将生产工具配置进行同步与开发"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/vscode/vscode.png"
---
## 問題描述

- 擁有**不止一個生產環境**，而且這些生產環境也**不一定持久**的情況下（隨時有可能一鍵重裝、部署新機器），不同生產環境要求有相同的配置（同步需求）

- 有隨時隨地**看到好 extensions 就想 install 的壞習慣**

- 作為生產力工具，需要**快速、自動化、可管理的**部署以減少不必要損失（包括不限於擴展配置）

## 解決

使用較為小型的私人代碼在線託管服務（大型網路里專門用於管理配置的配置服務器），將配置代碼上傳託管，按需獲取。有安全需求可自建機器與認證。

### VSCode

VSCode 中有一個提供 Setting Sync Anywhere 的擴展，使用 Github 提供的 [Gists](http://gohom.win/2015/11/26/gist/) 服務，短代碼在線託管（允許版本管理）

[Setting Sync](https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync) 基本步驟如下：

1. github里生成gist token（用於[身份分離認證](https://juejin.im/post/5a6c60166fb9a01caf37a5e5) 複製保存好）

2. vscode中使用``setting sync``擴展進行同步：upload，產生一個公開的以Username、gistID為依據的網頁（每次更新配置都會產生新的gistID，注意保存）

3. 獲取gist ID（可以在github gists 服務里找到URL最後一個便是），同步的時候使用gistID、token獲得

[詳細操作步驟參考](https://medium.com/@mvpdw06/%E5%A6%82%E4%BD%95%E5%9C%A8%E4%B8%8D%E5%90%8C%E7%9A%84%E9%9B%BB%E8%85%A6%E4%B8%8A%E5%90%8C%E6%AD%A5-vs-code-%E7%9A%84%E8%A8%AD%E5%AE%9A-82e7cd818ea7)

### Atom

Atom裏也有支持設置同步的``sync-settings``

``apm install sync-setting``

也需要提供gist token 不過需要自己手動在gists上創建一個gist

將gistID在設置中設置

### vim

vim方面寫了個js腳本來完成下載 上傳就懶得寫了直接webpage edit一下得了（懶

```js
const https=require("https")
const fs = require('fs');
const gistID="------------------------------------"
const remoteFilename="--------------"
const filepath="------------------------"

download(gistID,remoteFilename,filepath)

//提供指定gistID，遠程文件名（嗯目前只支持了一個），以及保存的地方
function download(gistID,remoteFilename,filepath){
    //調用github api，獲取指定gist的相關信息
    //需要附上headers
    https.get("https://api.github.com/gists/"+gistID,{headers:{'User-Agent':'gister'}},function(res){
        var jsondata="";
        res.on('data', function(data){
            jsondata+=data
            //數據持續讀取，獲取指定gist的動態地址
        });
        res.on('end',function(){
            // 等待全部下載完，獲取gist內容
            url=JSON.parse(jsondata)['files'][remoteFilename]['raw_url'];
            https.get(url,{headers:{'User-Agent':'gister'}},function(res){
                var gistdata="";
                res.on('data', function(data){
                    gistdata+=data
                    //獲取文本
                });
                res.on('end',function(){
                    fs.stat(filepath,function(err,stat) {
                        //若文件不存在則創建文件
                        if(!(stat&&stat.isFile())) {fs.open(filepath,'w',(e,fd)=>{console.log(e);})}
                    });
                    fs.writeFileSync(filepath,gistdata);
                    console.log("download finished");
                });
            })
        });
    })
}
```

[Github API支援](https://segmentfault.com/a/1190000015144126#articleHeader4)

[Github API v3 手冊](https://developer.github.com/v3/gists/)

## 寫在最後

[Gist Tutorial](https://www.labnol.org/internet/github-gist-tutorial/28499/)

### 其他擴展

[Chinese (Simplified) Language Pack for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=MS-CEINTL.vscode-language-pack-zh-hans)：中文語言包

[Markdown Preview Enhanced](https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced)：媽蛋擴展

[Todo Tree](https://marketplace.visualstudio.com/items?itemName=Gruntfuggly.todo-tree)：會識別媽蛋文件里的todo內容集成管理（挖坑

[Code Spell Checker](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker)：英語學習機之拼寫檢查

[AutoFileName](https://marketplace.visualstudio.com/items?itemName=spywhere.guides)：文件導入名稱補全

[Guides](https://marketplace.visualstudio.com/items?itemName=spywhere.guides)：代碼基線校準（賞心悅目

[vscode-faker](https://marketplace.visualstudio.com/items?itemName=deerawan.vscode-faker)：生產faker實例數據用於測試

[Vscode Google Translate](https://marketplace.visualstudio.com/items?itemName=funkyremi.vscode-google-translate)：快速翻譯工具

[vscode-icons](https://marketplace.visualstudio.com/items?itemName=robertohuertasm.vscode-icons)：美化工具

### js~~邪教~~擴展

[ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)：基本法

[JavaScript (ES6) code snippets](https://marketplace.visualstudio.com/items?itemName=xabikos.JavaScriptSnippets)：語法

[npm](https://marketplace.visualstudio.com/items?itemName=eg2.vscode-npm-script)：黑洞中心

[Document This](https://marketplace.visualstudio.com/items?itemName=joelday.docthis)：傳教說明書生成器
