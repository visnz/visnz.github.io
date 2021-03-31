---
title: "python调用tcpdump监管流量去向"
date: 2018-12-22
type: ["应用"]
weight: 7
tags: ["LINUX","计算机","PYTHON","TCPDUMP","服务器"]
categories: ["运维","最近"]
description: "python与tcpdump配合shell基础文本工具，完成对流量去向的简单监控"
featuredImage: "https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/tcpdump-py/icon.jpg"
---

冬至到了先给大家拜个晚年（说完就开起了风扇）

\# 请广东尊重一下冬天

## 问题

偶尔会有访问到[alphabet系列网站](https://www.wikiwand.com/zh/Alphabet)的时候被[recaptcha](https://www.wikiwand.com/zh-hant/ReCAPTCHA)识别为危险链接

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/tcpdump-py/01.png)

纳了闷儿了平时也没D谷歌啊莫不是流量被什么~~邪恶势力~~拦截了。想借个练手的机会试一下python脚本套用tcpdump做一些简单的运维工作。

## 分析

### tcpdump读取流量

有可能被识别为危险流量大概是过于频繁发送流量。由于使用的流量代理，几个朋友也会同时用这台服务器代理过去访问，而且还有几个是油管通，产生大量流量也正常。

先使用tcpdump筛选出出口网卡的发包：
``tcpdump -i ens3 -t -nn 'src <server-public-ip-address>'``

其中``-i``指示网卡，可以通过``ifconfig``或者``cat /proc/net/dev``查询。``-t``去除时间，``-nn``不显示域名直接显示ip地址

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/tcpdump-py/02.png)

tcpdump已经给了基础的格式化，使用awk截断可以获取目标地址，并丢到文件里做记录：``|awk '{print $4}' 1> /tmp/test.log``

### 多线程、log截断、计数、保存记录

多线程：因为tcpdump调用是持续占用stdout的，而且包流量也比较大，在python里使用双线程一路调用``tcpdump``，另一路专门做定时的截断、统计、记录。

log截断：将文件移动到新的名字（指定）造成记录截断，就可以不用暂停tcpdump的输出结果持续统计无漏

计数：每间隔一段时间截断刚刚由``tcpdump``产生的文件，使用collections的Counter对文件里的地址进行统计，以地址为key计数。

保存记录：毕竟总不能一直盯着terminal看，就保存到文件里以后再统一分析吧，构建了专门用于record的方法，以便统一指定输出方式

### 从ip地址到地理地址

``geoip-database``是一个IP到国家的关系映射数据库，``geoip``是一套用于查询的库与工具。arch上包为``geoip``，debian系为``geoip-bin``

使用``geoiplookup <ip-address>``对ip地址进行地理寻址（愣

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/tcpdump-py/03.png)

## python代码

第一部分获取网卡、地址并生成用于并发线程调用的命令
```python
netdev=os.popen("cat /proc/net/dev|grep en|awk -F ':' '{print $1}'").read().split('\n')[0]
# 指定网卡，可以使用/proc/net/dev与ifconfig查看

src_ban=" src "+os.popen("ifconfig "+netdev+" |grep inet|grep -v inet6|awk '{print $2}'").read().split('\n')[0]
# 获取该网卡使用的地址，在tcpdump中只检查该源地址发出的包

# 函数提供一个对目标地址的过滤参数接口，可以不对某些终点地址进行过滤
def tcpdumppy(*dst_ban):
    cmd="sudo tcpdump -i "+netdev+" -t -nn ' "+src_ban      
    for i in dst_ban:
        cmd+=" and ! dst "+i        
        # 排除掉需要ban的目标地址
    cmd+="'|awk '{print $4}' 1> /tmp/test.log"
    # 生成tcp执行命令
    print(cmd)
    os.system(cmd)
    # 该函数最终将执行tcpdump进行抓包的命令
```

第二部分指定计数方式
```python
# 函数载入一个counter，截断并分析从tcpdump抓取的流量目的地址
# 进行统计并间隔输出
def count(sec):
    second=int(sec)
    while 1:
        print("start to sleep")
        time.sleep(second)
        print("sleep for "+sec+"s")
        os.system("mv /tmp/test.log /tmp/test.log.tmp")     
        # 记录截断，移到新文件，保持不会占用太大空间
        dt=os.popen("date").read()
        # 记录截断的时间
        c=Counter()                                         
        # 创建计数器
        with open("/tmp/test.log.tmp") as f:
            while 1:
                line = f.readline()
                # 按行读取刚刚截断的文件
                if not line:
                    break
                ip=".".join(line.split('.')[0:-1])          
                # 将末尾的服务协议去除，再还原之前的地址
                c[ip]= c[ip] + 1              
                # 以ip地址为key累加
            # 文件读取完毕，累计完毕，翻译地址并记录：
            record("=== fetch info in "+sec+"s ===")
            for i in c:
                location=os.popen("geoiplookup "+i).read() 
                # 使用geoiplookup寻找ip来源
                slocation=" ".join(location.split(' ')[3:]) 
                # 切除掉地理地址的无用部分
                record(" : ".join([str(c[i]),slocation]))            
                # 人性化展示
            record("=== end of "+dt.split('\n')[0]+" ===")
```

第二部分中的展示方法是读取key的时候使用geoiplookup映射到地理地址，可以在上面第一部分修改不使用``-nn``以及在这里不使用地址映射，来实现**对域名的发包追踪**

第三部分指定记录方式
```python
rec="/tmp/record.log"
os.system("touch "+rec)
# 指定记录地址

# 定义记录方法
def record(str):
    os.system("echo '"+str+"' >> "+rec)
    print(str)
```

第四部分构建并发线程
```python
tcpdumppy = threading.Thread(target=tcpdumppy, name='tcpdumppy',args=("183.236.0.89","183.40.214.231"))
# ban掉流量转发的终点
count= threading.Thread(target=count, name='count',args=("20",))
# 设置统计的时间间隔为20s
tcpdumppy.start()
count.start()
tcpdumppy.join()
count.join()
# 开始线程
```

[最终代码](/files/tcplisten.py)

### 最终运行结果

![](https://visnonline.oss-cn-shenzhen.aliyuncs.com/pics/tcpdump-py/04.png)

可以直观看到这一段时间间隔内流量包的发送情况（没有全部排除转发终点）

## 写在后面

1. 操练了一下拾起python旧知识，用作脚本工具确实相当方便的

2. 分析了几个小时的流量记录，发现其实去往alphabet系的流量大多发往美国国内（毕竟有全球负载均衡），可以考虑重定向一下流量（代理转发到欧洲）。不过也有可能是因为时间不够长，持续分析中

3. 感谢廖雪峰老师文档帮助，以及Chrome插件``OneTap``帮助收纳接近三十个页面