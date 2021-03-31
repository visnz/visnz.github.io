---
title: "fcitx词库初打包 sunpinyin词库扩充"
date: 2019-01-15
type: ["日常"]
weight: 7
tags: ["LINUX","词库","fcitx","sqlite"]
categories: ["日常"]
description: "fcitx自带那么多工具，就没料到孙拼音用了sqlite把我整得跟孙子似的"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/sosogou/icon.png"
---
# 基本单元

词库的基本单元：编码、符号、词频

> 如：gou'li'guo'jia'sheng'si'yi 苟利国家生死矣 233

次要结构如编码的分隔符（上例是``'``），换行符[^1]、单元间隔等等。

## 安装官方內容

```bash
sudo pacman -S fcitx-table-other fcitx-ui-light
```
一个拓展词库跟一个轻量级的UI

## 不同格式

格式|形式|描述
---|---|---
scel|经修改带偏移的编码方式的文本|搜狗词库|
bin|二进制|搜狗用户词库
org|文本（编码(``'``) 符号）|拼音通用文本形式|
mb|二进制或文本|fcitx使用该格式作为词库|
db|数据库|sunpinyin使用这个



fcitx词库位于``/usr/share/fcitx/pinyin/``或``~/.config/fcitx/pinyin/``
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/sosogou/00.png)

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/sosogou/01.png)

在arch下fcitx-tools在fcitx，包含了createPYMB, readPYBase, readPYMB, mb2org, scel2org多个格式转换工具

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/sosogou/05.png)


# 导入细胞词库scel

```bash
scel2org sogou官方词库.scel > sogou官方.org
```

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/sosogou/02.png)


org是文本词库，基本格式如下：

> gou'li'guo'jia'sheng'si'yi 苟利国家生死矣

有scel直接转mb格式的工具：sg2fcitx [教程](http://www.mintos.org/skill/fcitx-sougou.html)

## 与原有词库融合

将原有词库文件解析出来``mb2org -f /usr/share/fcitx/pinyin/pyphrase.mb -s >./fcitx.org``

把``fcitx.org``跟自己的词库融合起来：
```bash
cat fcitx.org sogou.org > ./mix.org
sort mix.org > mix.sort.org 
```

## 打包
用来打包词库的工具是``createPYMB``，[man文档](https://www.systutorials.com/docs/linux/man/1-createPYMB/)：

> createPYMB \<PinyinFile\> \<PhraseFile\>
> 
> - Pinyin File
>   - Pinyin File is a file with pinyin and one character per line, separated with space. One available file is in the source of fcitx, named gbkpy.org.
> 
> - Phrase File
>   - Phrase File is a file with full pinyin separated with ' and the corresponding phrase. The default phrase file of fcitx can be downloaded at http://fcitx.googlecode.com/files/pinyin.tar.gz.

第一个参数是提供一个拼音映射表，``gbkpy.org``是``编码 符号``格式的通用文本词库，可以参考[这个](https://github.com/pkg-ime/fcitx/blob/master/data/gbkpy.org)

第二个参数提供以编码的分隔符为``'``的``编码 符号``的org文件
```bash
createPYMB gbkpy.org mix.sort.org  
```
生成四个文件：字库、转换错误、词库、已转换成功的词条
``pybase.mb  pyERROR  pyphrase.mb  pyPhrase.ok``

复制导入
```bash
sudo mv /usr/share/fcitx/pinyin/pyphrase.mb /usr/share/fcitx/pinyin/pyphrase.mb.bak
sudo mv /usr/share/fcitx/pinyin/pybase.mb /usr/share/fcitx/pinyin/pybase.mb.bak
sudo cp ./pyphrase.mb /usr/share/fcitx/pinyin/
sudo cp ./pybase.mb /usr/share/fcitx/pinyin/
```

## 导入个人词库bin

sogou输入法7.1版以后个人词库只道出bin格式
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/sosogou/04.jpg)

bin格式转换为其他格式：[深蓝词库转换（Windows）](https://github.com/studyzy/imewlconverter)(>=2.4)，将其转换为org格式。(支持格式自定义)
![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/sosogou/03.jpg)

# sunpinyin词库

sunpinyin使用SQLite format 3的db作为词库，位于``~/.sunpinyin/userdict``，可以直接启动sqlite3查询：
```sql
sqlite> select * from dict limit 15;
1|2|15|5|0|0|0|0|13|21|0|0|0|0|0|0|0|0|0|0|指定
2|2|12|14|0|0|0|0|21|20|0|0|0|0|0|0|0|0|0|0|镜像
3|2|1|17|0|0|0|0|22|10|0|0|0|0|0|0|0|0|0|0|补上
4|2|1|12|0|0|0|0|8|14|0|0|0|0|0|0|0|0|0|0|搬家
5|2|17|5|0|0|0|0|7|7|0|0|0|0|0|0|0|0|0|0|手逗
6|2|22|14|0|0|0|0|13|14|0|0|0|0|0|0|0|0|0|0|以下
7|2|13|22|0|0|0|0|18|13|0|0|0|0|0|0|0|0|0|0|迁移
8|2|5|13|0|0|0|0|21|13|0|0|0|0|0|0|0|0|0|0|定期
9|2|16|15|0|0|0|0|30|29|0|0|0|0|0|0|0|0|0|0|重装
10|2|17|15|0|0|0|0|13|30|0|0|0|0|0|0|0|0|0|0|时钟
11|4|22|12|16|15|0|0|13|18|30|29|0|0|0|0|0|0|0|0|一件重装
12|2|9|12|0|0|0|0|11|14|0|0|0|0|0|0|0|0|0|0|更佳
13|2|1|4|0|0|0|0|5|9|0|0|0|0|0|0|0|0|0|0|备份
14|3|22|1|21|0|0|0|13|1|24|0|0|0|0|0|0|0|0|0|一把梭
15|2|17|12|0|0|0|0|7|18|0|0|0|0|0|0|0|0|0|0|手贱
sqlite> .schema
CREATE TABLE dict(          id INTEGER PRIMARY KEY, len INTEGER,          i0 INTEGER, i1 INTEGER, i2 INTEGER, i3 INTEGER, i4 INTEGER, i5 INTEGER,          f0 INTEGER, f1 INTEGER, f2 INTEGER, f3 INTEGER, f4 INTEGER, f5 INTEGER,          t0 INTEGER, t1 INTEGER, t2 INTEGER, t3 INTEGER, t4 INTEGER, t5 INTEGER,          utf8str TEXT, UNIQUE (i0, i1, i2, i3, i4, i5, utf8str));
CREATE INDEX index_0 ON dict (len, i0, i1, i2, i3, i4, i5);
```
数据表结构如下：
第一列是index，第二列是符号的长度，根据不同的数据表定义，支持不同的最长符号定义。往后是六位的声母（映射），接着六位的韵母（映射），再后六位还未清楚是什么内容，最后是符号


```sql
select * from dict where i0==0 limit 10;
```
通过筛选得知映射表如下：

声母|数字|韵母|数字|韵母|数字
---|---|---|---|---|---
a|0||0|uo|24
b|1|a|1|uai|25
p|2|o|2|ui|26
m|3|e|3|uan|27
f|4|ai|4|un|28
d|5|ei|5|uang|29
t|6|ao|6|ong|30
n|7|ou|7|v|31
l|8|an|8|ve|32
g|9|en|9|iong|33
k|10|ang|10
h|11|eng|11
j|12|er|12
q|13|i|13
x|14|ia|14
zh|15|ie|15
ch|16|iao|16
sh|17|iu|17
r|18|ian|18
z|19|in|19
c|20|iang|20
s|21|ing|21
y|22|u|22
w|23|ua|23

普通话声母为23个，韵母为32个。带有一个``INTEGER PRIMARY KEY``所以合并表会比较麻烦。

可以参照一下[google code](https://code.google.com/archive/p/hslinuxextra/downloads)上的一些词库，直接下载导入，以及[这篇文章](http://www.mintos.org/skill/fcitx-sougou.html)

## 写在最后

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/sosogou/end.jpg)

---
参考资料

1. [fcitx拼音输入法的搜狗细胞词库转换和导入教程](https://www.librehat.com/fcitx-sogou-pinyin-cell-database-convert-import-guide/)

2. [ibus-libpinyin可用32万txt词库](http://blog.dreamlikes.cn/archives/952)

[^1]: ``\r``是回车，``\n``是换行。具体区别追溯到打字机的基本结构。windows下每行结尾是``\n\r``，Unix是``\n``，Mac下是``\r``
[^2]: 瞄了一眼这个db格式不是很直观，有兴趣的旁有可以深入了解下
