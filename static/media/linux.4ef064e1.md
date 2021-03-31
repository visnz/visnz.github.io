---
title: "Linux 筆記大整理 & 工具備忘錄"
date: 2018-12-07
type: ["笔记"]
weight: 8
tags: ["计算机","Linux"]
categories: ["笔记","最近"]
description: "找了个机会把文件系统里从入门到入坑的大部分可能还能用得上的Linux知识给整理了一下"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/static/pics/linux/icon.png"
---
## 新的学习笔记<sup>19.01.15</sup>
1. ``pacman -Ss fcitx|grep "^[^ ]"|awk -F"/" '{print $2}'``查询软件包并隐藏描述与库归属
2. ``find ./ ! -name "*.md" -size -1K|xargs cat -n``递归寻找并展示出当前目录下的非md、小于1K的文件内容
3. ``:> test.txt``清空文件内容
4. ``sort unsort.md|uniq -c`` 排序、统计次数
5. ``cat README.md|tr '[:lower:]' '[:upper:]'``把文章小写字母全换成大写
6. ``du -sh `ls` ``查看当前文件夹各文件与文件夹大小
7. ``lsof``：list open file,``-c``查看指定进程、``-i:80``查看连接情况
8. ``ldd``：List Dynamic Dependencies，用于查询程序的依赖哭
## 基本工具

1. ``df``、``fdisk``、``fdisk``等基本硬盤查看工具

2. ls：基本是自帶``--color=auto``這個地燈設置。
 
    - ``-al``列出隱藏與詳細信息
    
    - ``-Csh``着色、大小顯示
 
    - ``-R``超詳細目錄信息

3. ``tty``查看当前 tty 终端=>``/dev/pts/*``，可以用于重定向输出

4. ``init *`` runlevel

5. ``&``丢后台

6. 后台任务检查``jobs``任务管理器``top/htop``、``ps -ax``

7. 内存检查``free``

8. 端口占用检查``netstat -anp|grep [port]``、**``lsof -i``**

9. 变量修改：``export LANG=zh_CN.UTF-8``变量追加：``export LANG=$LANG:en_US.UTF-8``

10. sudo组：``echo "%sudoer ALL=(ALL:ALL) ALL">> /etc/sudoers``

11. ``dd if=* of=* bs=1M count=200``

12. 查看查找：``file``、``type``、``whereis``

13. ``locate``：查找，使用 mlocate 包，使用 updatedb 更新数据库

14. ``chattr``設置文件安全屬性（``lsattr``現實）

    - A atime訪問時間不會因爲訪問而修改（隱藏讀取痕跡）

    - S 數據立即寫入，不參與緩存（適合重要數據寫入，防止斷電丟失）

    - a append追加寫入權限（適用於log、passwd文件）

    - c 壓縮加密文件，以時間換空間（長時間一次的大文件讀寫存儲等）

    - i 文件不能刪改、不能鏈接

    - s 文件徹底刪除（適合重要數據）

    - u 文件刪除的話，只隱藏文件訪問，可以找回

15. ``chsh -s /bin/bash username``切換默認shell

16. ``bc``基本計算器

17. ``time``測試軟件運行時間

### 目录结构

- 通常/作为根目录，/usr（unix software resource 用户软件库）,/home（用户家目录）,/var（服务目录）会单独规划。

    - /usr里会有单独的bin sbin lib lib64 share local 

    - /usr/share/doc 程序文档，man 将访问

    - /usr/local //软件源码编译安装的附加软件库

    - /usr/bin   实现系统 扩展功能 的可执行文件（应用启动，与开机和内核操作无关）

    - /usr/share 结构独立的数据/文件储存器，DE的desktop文件在這裏

    - /usr/include（c/c++中的include文件，以tarball安装时候，会用到不少）

    - /usr/lib[64] （应用程序的库文件）

    - /usr/local（本机管理员自己下载的程序）

    - /var/log 里有 log 文件

    - /var/cache 网络访问或本地程序访问产生的暂存文件

    - /var/lib 动态数据相关操作会使用到的库

- /boot   #系统启动需要的文件（内核）,grub信息在這裏

- /swap

    - 内存<4G，设置二倍。

    - 内存>4G，swap分区大小等于物理内存。

- /bin 系统基本命令 /sbin 系统相关命令 /lib[64] 库（内核模块在这） /media  挂载媒体

- /opt 第三方软件

- /etc 配置文件

    - [/etc/resolv.conf](https://wiki.archlinux.org/index.php/Resolv.conf_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)) 域名服務相關

    - ~~/etc/init.d/ 服务启动/停止的脚本~~Selling my underpants to systemd

    - /etc/X11/ XWindow相关配置

- /dev pts 终端 null 空 random 随机

- /proc  #伪文件系统，内核参数的映射文件

    - /proc/version 內核版本號

    - /proc/net/dev 查看網卡詳細信息

- /sys   #伪文件系统，系统硬件参数的映射文件

### 文件操作

1. 文件时间：使用touch更新文件三个时间，可以用于获取新的sshkey

    - mtime：内容修改时间

    - ctime：权限修改时间

    - atime：文件访问时间

2. 命令位置查詢：``whereis``（比find快，比which多）可用locate（mlocate）

3. 文件類型：``file``

4. find：全盘查找工具``sudo find / -name "hello.*"``

    - -name [regex]

    - -size +50k

    - -type d目录、p管道等

    - -perm 模式

5. linux文件系統：ext3 ext4 xfs

### shell腳本

1. 判斷文件夾是否存在，若存在並進入``[ $1 ] && cd $1``

2. 判斷文件是否存在，不存在則創建``if [ ! -f $1 ];then `touch $1` fi``

2. 羣組變量：``sites1=("163.com" "ustc.edu.cn" "aliyun.com")``聚合羣組變量：``all=(${sites1[@]} ${sites2[@]})``

3. 遍歷./文件：``for var in `ls -1` do ... done``

### 旁門

- ``echo {a,b,c}{d,e,f}`` => ``ad ae af bd be bf cd ce cf``

- 图片藏种技术： ``cat file1 file2>file3``


## 雜七雜八工具使用

1. 樹狀圖展示，僅展開1層``tree -L 2``

    - ``-sh``附帶大小

    - ``-p``附帶權限
 
    - ``-P [regex]``只顯示適配文件（會包含文件夾）

    - ``-C``色彩表示

2. 备份工具：``dump``备份 ``restore``恢复

3. 打包工具：``tar/gzip/zcat/bzip2/compress/7z``

4. C家工具：``gcc/g++``编译器``gdb``调试器

5. git：``git config --global user.`` 設置用戶信息

6. 虛擬機：``qemu``、``xen``、``kvm``

### 桌面工具

- ``pitivi``、``shotcut``：影片编辑软件

- ``shotwell``：图片管理器

- ``gimp``、``Pixlr``：Linux下的图片编辑工具（PS）

- ``Scribus``：對標Adobe InDesign

- ``Inkscape``：對標Adobe Illustrator

- ``darktable``：對標Adobe Lightroom

- ``Black Magic Fusion``：Nuke 節點式後期效果合成，現已與davinci合併

- ``blender``：3D建模（python腳本）

- ``gitkarken``：开源且免费的 git 可视化管理工具（官方推荐）

- ``LibreOffice``：office

- ``openvisualtraceroute``<sup>aur</sup>：跨平臺的traceroute視覺化展示（基於java）

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/linux/01.png)

## 運維相關

1. 用戶登錄信息：``last``

2. 查看-修改時區：``date -R; ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime``

3. 用戶運行狀態：``w``、``uptime``

4. 實時滾動的log：``tail -f [file]`` 

5. 魂歸垃圾桶：`` >/dev/null 2>&1``

6. 网卡信息：``、etc/sysconfig/network-scripts/``

7. ``route -n``   #查看路由表

8. ``/etc/resolv.conf``  #dns的全局配置文件

9. 例行工作：``crontab``（/etc/crontab）

    - 分 时 日 月 周（周和月、日调用有冲突）

    - ``* * * * * reboot``每分钟重启一次

    - ``*/2 * * * * reboot``每两分钟重启一次

    - ``20,50 9-20/3 * * * reboot``每天9到20点每三小时的20分，50分各执行一次

    - ``0 0 * * * pacman -Syu``每天自动滚包一次

10. 權限相關

    - useradd 用戶新增 -m創建家目錄

    - usermod 用戶修改

    - groupadd 組增加

    - groupmod 組修改

    - chmod 改變文件權限

    - chown 改變文件所有者

    - chgrp 改變文件組所有

11. 隨機密碼產生器：``cat /dev/urandom``cat隨機數``od -x``轉換爲十六進制``head -n 20``獲取頭20行``tr -d ' '``去掉空格``md5sum``獲取驗證碼``awk '{print $1}'``獲取校驗和

12. 腳本設密碼：``echo "user:pswd" > tmp && cat tmp | chpasswd``

13. ``echo 'PermitRootLogin without-password' >> /etc/ssh/sshd_config``root免密登陸。通常是``echo 'PermitRootLogin no' >> /etc/ssh/sshd_config``來禁止以root登陸


## 摘录

### 后台工作与服务

> - 两种daemon的工作模式
>     - super daemon：一个特殊的daemon，用于管理daemon监听和请求，接受到新请求，会向指定的daemon发起激活。通常用于响应大量的通用服务。
>         - 多线程响应：一个服务对于多个服务进程，同时响应多个服务对象
>         - 单线程响应：服务模型类似一个多路复用器。
>     - stand alone：单独的一个持续运行的daemon，通常用于响应特殊服务。
>         - 信号响应：一旦有信号就立即响应
>         - 间隔响应：每隔一段时间响应。
> - stand alone
>     - ``/etc/services``记录着对大部分接口的服务监听。
>     - ``/etc/init.d/``目录保存daemon启动脚本，stand alone启动
>     - ``/etc/systemd/``各种服务的初始化环境配置文件
>     - ``/etc/``各种服务的配置文件
>     - ``/var/lib/``各服务产生数据的记录目录
>     - service：对服务进行控制，替代对/etc/init.d/的控制
> - super daemon
>   - 该服务由xinetd管理，配置文件位于``/etc/xinetd.conf``（里面也记录面对同一个服务最多提供多少链接、面对同一个来源用户提供多少链接等信息）

### 进程管理
- ``fork()``创建子进程，0表示当前的子进程，大于0时为父进程PID，-1为创建失败

- 函数调用一次单产生两个返回值，一个返回给子进程一个返回给父进程

```c
pid=fork();
if(!pid)printf("child process\n");
else if(pid>0)printf("parent process\n");
else printf("fork fail\n", );
```
 
 ## 设计原则

 Linux设计原则：

1. 单一目的的小工具组成

2. 一切皆文件

3. 尽量避免与用户交互

4. 所有的配置文件都保存为纯文本格式

## 网站资源

常用的国外Linux资源：
    
- [来自Linux和开放源代码界的新闻](lwn.net)

- [最齐全的Linux/UNIX软件库](www.freshmeat.net)

- [信息最全的Linux学习网站](www.justlinux.com )

- [内核的官方网站](www.kernel.org Linux )

- [提供全方位的Linux信息](www.linux.com )

- [提供内核信息和补丁的汇总](www.linuxhq.com )

- [非常完整的Linux新闻站点](www.linuxtoday.com)

国内的Linux的资源：
    
- [国内最大的Linux/UNIX技术社区网站](www.chinaunix.net)

- [Linux伊甸园，最大的中文开源资讯门户网站](www.linuxeden.com)

- [中国Linux公社，拥有自己的Linux发行版本](www.linuxfans.org)

- [提供各种Linux资源、包括资讯、软件、手册等](www.linuxsir.org)


[一分大大佬早期的学习笔记](/files/拼客笔记.md)
