---
title: "Systemd 扎記"
date: 2018-12-15
type: ["笔记"]
weight: 8
tags: ["LINUX","SYSTEMD","笔记"]
categories: ["笔记","最近"]
description: "System的一些基础概念、使用与备忘"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/systemd/icon.png"
---

## 簡述

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/systemd/01.jpg)

\# sell my underpant to systemd \#

- systemd(System Daemon, 系統守護進程)：是一個Linux的系統與服務管理器，由``systemctl``進行控制，支援系統快照與回復系統狀態，支持自動掛載等。已完美取代``sysvinit``

- 系統啓動過程：

    1. BIOS/UEFI啓動
    2. GRUB引導啓動
    3. 尋找/boot(GSP分區)並載入內核
    4. CPUIDLE與內核初始化(PID=0)
    5. /sbin/init（systemd啓動，PID=1）
    6. fork啓動systemd註冊的基層服務

- 不同進程工具檢出PID=1的進程不一樣，``systemd``（``pstree -Apn``檢出結果）或是``/sbin/init``（``ps -aux``檢出結果）
```bash
$ ls -al /sbin/init
lrwxrwxrwx 1 root root 22 Nov 26 21:35 /sbin/init -> ../lib/systemd/systemd
```

## 基本使用

- ``systemctl``：進入主界面

- ``systemctl --failed``查看启动失败的单元

- ``systemctl start/enable/disable/stop/restart/mask/unmask <Unit>``：相應地對一個unit進行操作

- ``systemctl daemon-reload``：掃描新的有變動的單元，通常更新配置後需要使用

- 圖形化管理界面：``systemd-ui``

- 遠程管理：``systemctl -H root@example``

- ``systemctl list-unit-files --type=service``：列出可用單元及其情況（指定service類型）

- ``systemd-*``系列也提供了許多方便實用的工具

    - ``systemd-analyze blame``：分析啓動時花費的時間

    - ``systemd-analyze critical-chain httpd.service``：追蹤服務的啓動鏈

    - ``systemctl list-dependencies httpd.service``：依賴鏈

## Service Unit 編寫

- systemd管理的最小單元是一個 Unit，包括系統服務（.service）、掛載點（.mount）、sockets（.sockets） 、系統設備（.device）、交換分割區（.swap）、檔案路徑（.path）、啟動目標（.target）

- 所有可用的單元檔案存放在目錄中，有優先級之分：（從低到高）

    - ``/usr/lib/systemd/system/ ``軟件包

    - ``/etc/systemd/system/`` 系統管理員安裝

- 帶``@``的單元：屬於範本單元，如v2ray@.service，v2ray@1.service則是其的一個實例，可用於在一臺機器上啓動多個服務。[相關鏈接](http://www.bkjia.com/LINUXxt/1025409.html)

### [Unit]段

指定Unit的描述``Description=xxx``、依賴關係：

- 必須依賴：``Requires=network.target``，在前者啓動後再啓動

- 可選依賴：``Wants=network.target``，前者不啓動也可以啓動

- 同步依賴：``After=network.target``，若寫上，將不會與指定單元併發（異步）啓動

### [Service]段

- 描述類型（``Type``）：

    - simple：立即啓動，不fork

    - forking：父程序fork後退出，常用於自定守護進程。提供PIDFile便於追蹤

    - oneshot：一次運行

- 提供``PIDFile``：通常在``/var/run/``下，可自己指定PID文件，以便systemd和自己追蹤

- ``Environment``：指定環境變量 

- ``ExecStartPre``：指定程序預啓動的命令

- ``ExecStart``：指定啓動程序的命令

- ``ExecStartPost``：指定程序啓動之後的命令

- ``ExecReload``：指定程序重啓命令

- ``ExecStop``：指定停止程序的命令

- ``ExecStopPost``：指定程序停止之後的命令

- ``Restart``：指定重啓場合（always、on-abort、on-failure等）

- ``RestartSec``：指定重啓間隔時間

- ``CPUShares``：指定服務的CPU分配額，一個CPU爲1024,兩個爲2048

- ``KillMode``：指定systemd如何停止服務，指定爲``process``表示只停止主進程，默認爲``control-group``，會將整個進程殺死

### Exam1：啓動時自定義運行一個腳本
``/etc/systemd/system/myscript.service``
```ini
[Unit]
Description=My script

[Service]
ExecStart=/usr/bin/my-script

[Install]
WantedBy=multi-user.target 
```

### Exam2：設置自定義http服務
```ini
[Unit]
Description=My http service
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/var/run/myhttp.pid
ExecStart=/usr/sbin/myhttp start
ExecStop=/usr/sbin/myhttp stop
ExecReload=/usr/sbin/myhttp restart
Restart=always

[Install]
WantedBy=multi-user.target
```

## Timer使用

- 可用於臨時接管cron的systemd，用於跟蹤服務，要放在服務同目錄（如 foo.timer 和 foo.service）

- ``systemctl list-timers``查看已有的timer

- 與Service Unit不同的是，Service版塊被換用爲Timer：

    - ``OnCalendar=weekly``指定每週執行，可換daily等

    - ``OnBootSec=15min``指定開機後一段時間執行

    - ``OnUnitActiveSec=1w``指定在該事件執行後，間隔一段時間再執行（示例爲一週）

    - ``Persistent=true``因爲關機沒有執行的任務，在服務啓動時候立即執行

## journald
一个接管``syslog``的系统日志管理进程，使用``journalctl``来管理

常用参数：

- ``-b 2`` 查看上上次开机的日志

- ``-f`` 实时更新

- ``-p 3`` 查看0,1,2（紧急、中断、危险级别）[级别](https://wiki.archlinux.org/index.php/systemd_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E6%97%A5%E5%BF%97)日志

- ``--since=today --until="10 min ago"``、``--since="2018-12-21 18:30:20"``界定i时间

- ``_PID=1`` 指定PID

- ``-k`` 只输出内核信息

- ``-u v2ray.service`` 查看指定单元的log

- ``--vacuum-time=2days`` 清除日志2天前的日志

日志的文件位置``/var/log/journal``，配置文件在``/etc/systemd/journald.conf``，服务为``systemd-journald.service``



## 其他

- [阮老師systemd使用教程](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-part-two.html)

- [Systemd in Arch Wiki](https://wiki.archlinux.org/index.php/Systemd_(%E6%AD%A3%E9%AB%94%E4%B8%AD%E6%96%87))

- [詳細使用範本 from linux中國](https://linux.cn/article-5926-1.html)

- [Timer使用參考](https://wiki.archlinux.org/index.php/Systemd/Timers_(简体中文))