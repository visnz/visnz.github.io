---
title: "Arch Linux 札记"
date: 2019-01-27
#date: 2018-01-20
type: ["笔记"]
weight: 6
tags: ["LINUX","计算机"]
categories: ["笔记","最近"]
description: "一系列arch使用过程的小笔记"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/oldicon/arch.png"
---

记录下平时可能会用到的（查阅用）

---

## [艾老师](https://axionl.me/)打的两个包<sup>19.03.01</sup>

- baidupcs([baidupcs-go-bin](https://aur.archlinux.org/packages/baidupcs-go-bin/))：[linux下的百度云终端版](https://github.com/iikira/BaiduPCS-Go)（go版本）
- musicbox([netease-musicbox ](https://aur.archlinux.org/packages/netease-musicbox/))：[linux下的网易云终端版](https://github.com/darknessomi/musicbox)

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/arch/04.png)

## Notable<sup>19.02.27</sup> <sup>AUR</sup>  

一款简单美观的markdown笔记管理器，目前支持打tag、附件（文件系统里会在运行目录下，创建打上tag的副本），有简单的导出功能，但还不支持**图片**与**注脚**。

貌似仅识别markdown文件。

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/arch/03.jpg)


## [x2go](https://wiki.archlinux.org/index.php/X2Go_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))<sup>19.01.27</sup> 
1. 为云端vps提供安全桌面访问（by ssh）
2. 云端安装x2goserver和[与其兼容的桌面系统](https://wiki.x2go.org/doku.php/doc:de-compat)
3. 本地安装x2goclient

参考文章：[如何在 Linux 上使用 x2go 设置远程桌面](https://linux.cn/article-5708-1.html)

## Mac上Arch<sup>19.01.19</sup> 
1. 如果全舍弃macos而不是双系统，要更容易装一些（via. [archwiki](https://wiki.archlinux.org/index.php/Mac_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))）。安装跟平时PC等安装步骤一致。
2. 硬件可能有些小麻烦，无线网卡不一定arch带驱动，有无线网卡或USB/雷电转RJ45会方便很多
3. linux-macbook<sup>archlinuxcn</sup>一些内核组件
4. 盖上本本休眠的功能，手动设置一下ACPI相关内容

## time<sup>19.01.15</sup> 
> 1. atime(access time)是在读取文件或执行文件时更改，也可以认为是文件最后一次被读取的时间。
> 2. ctime（change time)是在写入文件，随更改所有者、权限时而更改，也就是文件状态最后一次被改变的时间。（索引节点改变）
> 3. mtime(modify time)：写入文件时随文件的内容更改而更改，可以理解为是文件内容最后一次被修改的时间。[^1]

[^1]: [Linux中ctime mtime atime文件时间的区别](http://www.voidcn.com/article/p-kbngkkhc-uv.html)

1. cat会改变atime（pick Linux中国某篇一发即删的文章）
    > relatime属性
    >   从kernel2.6.29开始，默认集成了一个 relatime的属性。使用这个特性来挂装文件系统后，只有当mtime比atime更新的时候，才会更新atime。
    > 
    > 使用场景：
    > 在文件读操作很频繁的系统中，atime更新所带来的开销很大，所以在挂装文件系统的时候使用noatime属性来停止更新atime。但是有些程序需要根据atime进行一些判断和操作，这个时候relatime特性就派上用场了。其实在事实上，这个时候atime和mtime已经是同一个time，所以可以理解这个选项就是为了实现对atime的兼容才推出的，并不是一个新的时间属性。[^1]

2. ctime是文件在文件系统层面的改变（权限、写入等），mtime是文件自身内容的修改

## 思维导图工具<sup>19.01.07</sup> 
<link rel="stylesheet" type="text/css" href="/css/tag.css"> 

轻量思维导图工具 

- freemind<tag>awtwarn</tag> 
- [mindmapp](https://mindmapp.org/)<sup>aur</sup><tag>npmwarn</tag> 

## youtube-dl & you-get <sup>18.12.28</sup>
youtube-dl命令行下载油管视频工具（[github](https://github.com/rg3/youtube-dl)）

提供了列表抓取、代理、regex表达式、filter过滤器、重定向、视频多格式、安全认证等功能

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/arch/02.jpg)

[you-get](https://github.com/soimort/you-get) 在pip3库中，是一个开源的用于抓取各种网站视频图片资源的工具，包括不限于油管、vimeo、推特、Ins、niconico、bilibili、163music、acfun、新浪、音悦Tai、youku、腾讯、芒果。

## flameshot<sup>18.12.27</sup>
提供接近[snipaste](https://www.snipaste.com/)功能的截图工具（[github](https://github.com/lupoDharkael/flameshot)）

![](https://github.com/lupoDharkael/flameshot/raw/master/img/preview/animatedUsage.gif
)

KDE桌面``alt + space``调用延时截图``sleep 1 && flameshot gui``
## DNSMasq<sup>18.12.22</sup>

[DNSMasq](https://wiki.archlinux.org/index.php/Dnsmasq_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))服务使用本地做DNS缓存：``echo "listen-address=127.0.0.1" >> /etc/dnsmasq.conf``

添加本地解析：``/etc/resolv.conf``在最开头添加本地解析为最先

防止篡改：``sudo chattr +i /etc/resolv.conf``

配合create_ap组件给子网提供dns服务，在``/etc/dnsmasq.conf里``的``listen-address``中添加子网中的网关地址``127.0.0.1,10.0.0.1`` 

默认dnsmasq关闭了DHCP功能：<sup>18.12.26</sup>
```bash
# /etc/dnsmasq.conf
interface=<LAN-NIC>
bind-interfaces
dhcp-range=10.0.0.2,10.0.0.50,12h
dhcp-host=aa:bb:cc:dd:ee:ff,10.0.0.1
```

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/arch/01.png)


## dig``dnsutils``<sup>18.12.20</sup>

## steam 冬季大促销<sup>18.12.19</sup>

泼皮买了个linux上能跑的，~~可爱，想玩~~

包名：steam steam-native-runtime(multilib)

报了找不到``~/.local/share/Steam/ubuntu12_32/steam-runtime/run.sh``的错

補了包：fontconfig lib32-fontconfig  nvidia-dkms nvidia-utils lib32-nvidia-utils

（nvidia与nvidia-dkms冲突，慎）

---

## libvirt虚拟机<sup>18.12.17</sup>

基于[KVM](https://wiki.archlinux.org/index.php/KVM_(%E6%AD%A3%E9%AB%94%E4%B8%AD%E6%96%87))，[libvirt](https://wiki.archlinux.org/index.php/Libvirt_(%E6%AD%A3%E9%AB%94%E4%B8%AD%E6%96%87))提供一系列虚拟机服务的集合（包括命令控制工具virsh、守护进程libvirtd）

virt-manager图形化界面，qemu-kvm基础包<sup>18.12.29</sup>

依赖firewalld、ipset、ebtables、dnsmasq，安装后手动启动libvirtd、firewalld守护进程开始使用

libvirt没有载入default网络，位置在/etc/libvirt/qemu/networks/default.xml
``sudo virsh net-define /etc/libvirt/qemu/networks/default.xml``载入``libvirtd``服务并重启守护进程。

``virsh net-autostart default``标记自动启动


## git 指定密钥登陆<sup>18.12.15</sup>

``./.ssh/config``

```bash
Host github.com
    HostName github.com
    IdentityFile ~/.ssh/$priviteKey
    User git

Host new.visn.online
    HostName new.visn.online
    IdentityFile ~/.ssh/archlinux
    Port 20069
    User visn

```

其中$priviteKey为登记在github setting中的公密钥对的密钥

## sshd端口修改<sup>18.12.13</sup>

指定端口：``echo 'Port=20069' >> /etc/ssh/sshd_config``

``systemctl daemon-reload && sudo systemctl restart sshd``

---

## 免密登陸<sup>18.11.31</sup>

1. 生成key pair: ``ssh-keygen -t rsa``默認會存儲在``~/.ssh/``下包含一個``id_rsa``/``id_rsa.pub``

2. 服務器獲取pubkey:

```bash
mkdir ~/.ssh ; touch ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && chmod 700 ~/.ssh
echo [key_pub] >> ./authorized_keys
```

3. 同時可以用於github（需要以ssh方式clone）、各種遠程服務器實例的免密登陸

```bash
#!/bin/bash
useradd -m $1 && echo $1":"$1"8875" |chpasswd $1
mkdir /home/$1/.ssh ; touch /home/$1/.ssh/authorized_keys && chmod 600 /home/$1/.ssh/authorized_keys && chmod 700 /home/$1/.ssh
chown $1:$1 /home/$1/.ssh/authorized_keys
chown $1:$1 /home/$1/.ssh
echo $2 >> /home/$1/.ssh/authorized_keys
```

## uget

[瀏覽器插件](https://chrome.google.com/webstore/detail/uget-integration/efjgjleilhflffpbnkaofpmdnajdpepi)

``sudo pacman -S uget uget-integrator-chrome uget-integrator-chromium``

## 雜

- archlinux pacman db : ``/var/lib/pacman/db.lck``

- [archlinux 啓動未引導而進入grub解決方法](https://www.openfoundry.org/tw/foss-programs/9267-linux-grub2-fixing)，寫入：`` sudo grub-mkconfig -o /boot/grub/grub.cfg``

- arch 使用 cronie(systemd) 管理計劃任務（disable），可以使用``crontab -e``編輯文件。三連擊``daemon-reload enable restart``


## fcitx
```bash
# ~/.xprofile
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
```
補上安裝：fcitx-im (fcitx-qt4 fcitx-qt5 fcitx-gtk3 fcitx-gtk2)
KDE图形配置程序：kcm-fcitx fcitx-configtool

chromium vscode裏可以使用


---
# 18年11月28日及以前

## hostap+ruijie

软件包：create_ap(community)

查詢網卡 ifconfig ：net-tools(core)

開關網卡 ``ip link set enp40s0 up``

``sudo create_ap <无线网卡>[<有线网卡>][<SSID>[<passwd>]]``
or

config file: /etc/create_ap.conf
``sudo systemctl enable/start create_ap``

銳捷每一分鐘心跳檢查一次多網卡，``crontab -e``不支持秒級，用了個蠢辦法

```bash
* * * * * /home/visn/rjsupplicant/rjsupplicant.sh -d 1 -u 1500000000 -p 123456 -n enp4s0 >> /var/log/rjs.log
* * * * * sleep 10; /home/visn/rjsupplicant/rjsupplicant.sh -d 1 -u 1500000000 -p 123456 -n enp4s0 >> /var/log/rjs.log
* * * * * sleep 20; /home/visn/rjsupplicant/rjsupplicant.sh -d 1 -u 1500000000 -p 123456 -n enp4s0 >> /var/log/rjs.log
* * * * * sleep 30; /home/visn/rjsupplicant/rjsupplicant.sh -d 1 -u 1500000000 -p 123456 -n enp4s0 >> /var/log/rjs.log
* * * * * sleep 40; /home/visn/rjsupplicant/rjsupplicant.sh -d 1 -u 1500000000 -p 123456 -n enp4s0 >> /var/log/rjs.log
* * * * * sleep 50; /home/visn/rjsupplicant/rjsupplicant.sh -d 1 -u 1500000000 -p 123456 -n enp4s0 >> /var/log/rjs.log
```

## NTFS只读系统

1. 软件包：ntfs-3g（``sudo ntfs-3g /dev/sdb3 /mnt/test``）然后卸载+挂载 / logout

2. 因为在windows那边读取的时候存在一些失误操作导致无法写入，使用``ntfsfixboot``（aur）

## 藍牙音響

1. 安裝基本藍牙包：bluez 和 bluez-utils

``sudo systemctl enable/start bluetooth``

2. 協議無法識別 出現： a2dp-sink profile connect failed  Protocol not available 安裝 pulseaudio-bluetooth

3. 重啓PulseAudio server & Bluetooth生效
```bash
killall pulseaudio
pulseaudio --start
systemctl restart bluetooth
```


