---
title: "Java Rebase"
date: 2018-01-26
type: ["笔记"]
weight: 8
tags: ["JAVA","计算机"]
categories: ["笔记"]
featuredImage: "http://blog.experts-exchange.com/wp-content/uploads/2012/02/java1.jpg"
description: "大学两年关于java入门计算机语言的总结"
---

# 写在前面
---

## Java囧境

作为面向对象炒得最火的语言，Java曾依靠虚拟机的跨平台特性撑起计算机半臂江山。

时至今日，Java已经略显疲惫，有些高不成低不就的感觉。

1. 试图培养成一种面向通用的语言，在py、R、Go等面向专业语言诞生之后，java的通用优势不足。
	1. 在安卓领域，第二代第三代JVM语言横空出世，Kotlin快速夺取市场。
	2. 在服务器领域，因为历史遗留问题JavaSRE目前依然为主流，但不是长久之计。新的服务器架构快速迭代，后起之秀Py、nodejs也在快速增长。
	3. 桌面领域Java一直有力使不出来，底部依赖大、自家GUI又刚不过qt系列
	4. 语言语法及其体系相对冗长、功能迭代速度相对较慢，底层相对重量级
2. 甲骨文的收购、商业化与闭源一定程度上延迟了技术层面的增长
3. 曾经的跨平台特性已经不再是其核心竞争力
4. Java的体系相当庞大，光是服务器框架就可以学完大学四年

（其实感觉有点像[Apache与Nginx port80之争](https://linux.cn/article-8292-1.html)那份漫画，老一辈与晚辈各自的优势）

**所提的点不是贬低Java，是较理性地描述目前的问题**



## 个人发展瓶颈

本身自己是学习Java内容居多的，从高中一本《从如坑到弃疗》，到大学起来接触各种教新的技术。

**大学生迷茫直观上来说，无非分为两种**

- 了解得太少
- 了解得太多

中学的时候不知道java是什么，一直摇摆不定学不学计算机，特迷茫。报考了志愿进了计算机这一大坑，诶，目标算是定下来了，不会太迷茫，java就继续学下去。直到接触了更多更先进的技术，再次陷入迷茫。

[断舍离](https://book.douban.com/subject/24749465/)提供了一些方法论来解决问题，做出了决定：放弃java，转向其他语言

上升到另一个层面，变成语言无非只是解决问题的方法而以，所以这个决定并不是什么大事。我现在要做的是把大学这一两年来断断续续学到的Java的内容进行归纳总结，**毕竟编程设计语言是具有共性的，在Java学到的解决问题的思路，完全可以应用到其他语言、学科和领域**。所以在此开了这篇文章。

所以标题引用了git中一个专业名词**Rebase**，更直观的感受是**更换基底**的意思。

文所涉及的内容，是个人**从Java中学习而来的**，并不是**Java独有的**，请勿混淆


# Java总览
---
语言特性

- 面向对象（封装继承多态，不允许多重继承）
- 跨平台(once code, ~~run~~ debug everywhere)
- 底层解释型（虚拟机机器码）
- 安全检查（你可能需要Rust）
- [强类型、静态语言](https://www.zhihu.com/question/19918532)
- 内存垃圾gc
- 大小写严格、默认驼峰命名法

## 技术与名词

JVM(Java Virtual Machine) Java实体运行软件，通过编程Java代码，编译成class机器码，由JVM适配平台，执行机器码内容（以此实现跨平台）。当然，JVM不一定由Java编写，使用Ruby语言+JRuby编译器等照样可以调用JVM。具体的计算机程序编译过程请自行了解。

JRE(Java Runtime Environment)Java运行环境，包含一个JVM和一系列标准类库

JDK(Java Development Kit) 程序开发库，包含了接口等一系列开发必备软件，通过Java语法导入、调用

JDK的实体文件结构

目录        |描述
---|---
 \\bin     | 编译器、测试与调用等工具
 \\demo    | 代码示例
 \\docs    | html的类库文档
 \\include | 用于编译本地库、本地方法的文件
 \\jre     |运行环境所需的文件
 \\lib     |类库文件
 \\src     |类库源文件


SE、ME、EE Java的三个不同版本，分别对应小型设备、桌面与简单服务器、企业及服务器三种平台



### clone()

> 相关接口：Cloneable
> 相关模型：原型模式

深克隆：刨根问底精确到每一个基本单位全部复制一遍，需要使用递归，每一个子类别都要实现克隆方法

浅克隆：只克隆表面对象，而被克隆对象的属性与原对象属性可能还是同一个引用，具体看你写到有多浅

克隆可用于大规模相同引用、相同数值的对象（如粒子），或者创建耗时的大规模对象，结合[原型模式](https://visnz.gitbooks.io/ood-oodp/content/chuang-jian-xing-mo-shi/yuan-xing-mo-shi.html)完成原型克隆，可以实现如git分支的思路、预防恶意修改等功能

可以使用序列化进行对象拷贝（见下端（迷之奇技淫巧

### 接口与回调

> 相关模型：责任链模式

回调（callback）是常见的设计模式，通俗话来说类似“触发器”。规定一组接口，并规定一个事件信号（非必须），编写回调内容。在编程过程部署这些回调接口，当有事件信号传入的时候，接口的实例就调用编写好的回调内容。这种模式适合把**响应逻辑和响应时机**分开部署，维护也相对方便。

最广泛的应用在窗口的MVC模式，用户发送event（控制，比如鼠标点击），程序接受并通过根节点递归（[责任链模式](https://visnz.gitbooks.io/ood-oodp/content/xing-wei-xing-mo-shi/ze-ren-lian-mo-shi.html)），寻找到可以响应事件的模组上，模组做出响应动作

### awt与Swing

UI的包里面大致结构可以划分为：框架、组件、事件（接口与回调）

awt的基本思路是希望能抽象各个平台的组件，**基于平台的对等体实现**，各平台大佬~~敷衍~~配合，勉强完成了“一次编写，处处运行”。由于各平台对于视窗的库都不一样，java也只能取其最小公分母。窗口组件的类型、风格受限于各种平台

后来swing是基于awt的统一了接口与规格（并开发了一些新的组件（~~反正后来也不新了~~）），通过为顶层组件提供对等体，使用纯 Java 代码来模拟的。提供更强大的UI组件，对底层平台依赖少，不同平台一致UI，所以可以看到swing的接口是相对统一且较易开发。Swing 无法充分利用硬件 GUI 加速器和专用主机 GUI 操作的优点，所以**慢**。Swing用的是lightweight的API，Qt用的也是他的API。

即，awt面向底层，swing面向上层。

SWT结合两者优点避开两者缺点，部分采用本地对等体（awt），为没有提供的特定主机创建模拟控件（Swing）。相对较为中和，在移植方面表现出色。

### lambda表达式


#### 表达式与闭包

lambda表达式是无状态的，不接受传参，无法基于自身构建闭包。通过与其词法范围来完成闭包。例如``a.b(e -> e*2)``是可以实现的，但``a.c((e,f) -> e*f)``不起作用

通常会使用在lambda外面的词法范围定义变量，使之成为闭包如
```
func(){
  int a=0;
  x.y(e -> e*a);
}
```

词法范围和实际逻辑是低耦合的，变量和引用就很难绑定，如
```
run(event){
  print("setup");
  //do event (print(a))
}
main(){
  int a=4;  //first here
  run(      //second here
    () -> print(a)  //third here
  );  
}
```
跨越了一个级别堆栈的引用不可达

### 异常与异常链

异常的出现加速软体的工程化，将问题的反馈具体到类，抛出到指定的接受者，做出不同的响应。可以此基础上设立底层自动执行、仲裁者等角色，模块与模块之间统一异常管理

如果处理不了就继续抛出给上层仲裁者，无需理会也最好出log，尽量不要丢弃异常或无声处理，软体一大就很难调试了

尽可能具体地描述异常而不是根类Exception，借助异常链可以具体地追踪到产生错误的位置（~~包是不会出问题的，最多是你调用得不好~~）

构造函数如果需要异常抛出，推荐使用工厂模式、静态生成器模式等

- 记得调用finally清除资源 ~~以防内存泄漏~~

- try尽量小，分散、精准地抓住每个异常

- 把发生异常的对象通过异常构造器传出，方便上层仲裁

### log

尽量避免用``System.out.println()``而是采用专门的Log模块来输出日志信息，[两者区别](https://juejin.im/entry/5987d73c5188256dcc11bb9f)，日志文件的特性是面向管理和记录。

使用专门模块实现如异步调用、后台分析监控等，专门的Log模块如log4j、logback，JDK中也有自带的Logger包（推荐log4j）

> 技术细节：分等级输出、格式化、过滤器、重定向输出


### spring

spring boot

spring cloud

### JavaBean

EJB(Enterprise JavaBean)

POJO(Plain Ordinary Java Object)


## 线程

### 大三元

Runnable提供异步任务，通常提交给Thread对象
```java
Runnable{
  void run();
}
```

Callable提供返回值
```java
Callable<V>{
  V call();
}
```

Future保存计算结果，可以启动某个线程，将该对象交给它，通过引用调用运行结果（任务与结果反馈的分离）。
```java
Future<V>{
  V get();
  V get(long timeout, TimeUnit unit);
  boolean cancel(boolean mayInterruptIfRunning);
  boolean isCancelled();
  boolean isDone();
}
```

``FutureTask implements RunnableFuture``，构建一个可以运行的对象，可以将callable对象传入封装成一个Runnable或Future对象。实际上就是把任务本身和任务反馈结合在一起，类似一个扩展的Callable。


### 同步锁

> synchronized同步锁、Lock锁对象、Condition条件对象、volatile

synchronized同步锁作为关键字对类、对象、**部分代码块** 进行修饰，假设被修饰的部分拥有一个内部锁，执行方法时候对象获得锁，结束时候自动释放。发生异常时会自动释放锁，没办法设置获取超时。

Lock是一个接口，由需要同步的对象或代码块调用``lock()``和``unlock()``来完成。相对关键字Lock能具体到每个步骤，提供更多调度可能性（中断等待线程、中断执行线程、了解锁情况、优化性能调度、异常处理）

条件对象是针对某一个条件进行锁的管理。某一线程申请了锁，却因为条件不符需要其他资源，此时条件对象回收他的锁，进行重新分配。当符合该条件的线程完成工作，调用唤醒阻塞在这个条件的所有线程。

volatile是轻量级的同步锁，不提供原子性但保证可见性，在每次访问的时候会获取最新的值，但不适合以其作为判断来完成逻辑（如在高并发情况下，读取的时候是一个数，实际执行到代码块的时候已经变了）

**不过通常优先选择已有的执行模型（concurrent包中提供），小范围的锁功能也尽量使用简单的用法**

### 执行器（执行模型）

创建线程开销大，调度复杂，总结前人经验与使用频度，jdk提供了非常基础的**线程池调度执行器**

线程池里拥有活跃或等待的一定数量的线程，按需选择新增或删除线程，任务对象（Runnable、Callable、Runnable+result）交付过来，线程池接收调度。记得调用shutdown关掉。

执行器提供静态工厂创建

方法|描述
---|---
CachedThreadPool|按需创建新线程；线程空闲生命周期为60秒，未被使用则销毁
FixedThreadPool|包含固定数量线程，持续保留
SingleThreadExecutor|单个线程的池，每次顺序执行一个任务（比如责任链事件响应，不允许同时多次触发）
ScheduledThreadPool|预定执行线程池，结合Timer计时器
SingleThreadScheduledExecutor|预定单线程池

#### 控制任务组
维护一个任务对象的集合，对其进行管理（
  invokeAny方法，只要有一个完成即可汇报，比如挖矿中，只要求出一个正确答案，其他线程即可停止工作。
  invokeAll方法，需要全部完成才进行汇报，比如分块/分层渲染
  等等
）

#### 同步器

控制障栅：如大量线程必须等待同步又必须在提交之前完成的情况，即设置一个障栅，当指定的一系列线程都到达障栅，障栅撤销，线程继续。比如游戏资源的后台准备

倒计时门栓：设置倒计时，门栓为1的时候只允许准备线程执行，准备完毕调用计时器，工作进程入场

交换器：生产者消费者问题的经典场景，维护一个交换对象，两个线程都准备好的时候执行交换过程



## 流
字符设备提供数据流，以顺序按需读取（如调制解调器、键鼠），块设备提供随机访问，以块为单位反馈（如磁盘）。以类似字符设备的读取方式去读文件被形容为流。

- 缓冲流``Buffered[In/Out]putStream``：维护一个缓冲区，可以分别指派读取和使用，规定大小、清空流等功能（满的时候则清空、输出）
- [随机访问流](http://www.cnblogs.com/ysocean/p/6870250.html)``RandomAccessFile``：以字符为单位的游标指针，表名权限后对其进行随机读写（多线程、断点下载可用）
- Closeable接口：源自忘记关流程序员们的疏忽，特设置AutoClosable交给系统管理，底层或管理器可以对流进行检查与关闭，以确保系统安全性。使用try-with-resource语法糖，将会在try块结束的时候自动关闭（通常关闭动作在finally里执行）
  ```java
  try( someObj which implements AutoClosable )
  { ...dosomething }
  ```

- [装饰流](http://www.codeceo.com/article/java-io-stream-decorators.html)：运用[装饰器模式](https://visnz.gitbooks.io/ood-oodp/content/jie-gou-xing-mo-shi/zhuang-shi-mo-shi.html)，允许向一个现有的对象添加新的功能，同时又不改变其接口结构。用一个新的装饰对象来囊括一个被装饰对象，且实现同样的接口（如``Buffered[In/Out]putStream``只是``[In/Out]putStream``的一个加了缓冲模块的装饰流）

- 缓冲区结构：0...标记...位置...界限...容量，标记表示当前已读数据的位置，位置来区分是否已读，界限表示所有已读取但未被读写的部分，界限以外的区域上限到容量

### 对象流与序列化
将对象以特定（可自定义）的格式完成**对一系列对象的编码**，可以实现对象的存储（实现如运行状态暂存和持久化、远程对象交流、版本管理等功能）。Serializable接口是一个没有方法的标志接口，表示这个类可以被序列化，底层也会自动遍历类对象的所有属性并进行序列化。jvm有指定默认的编码方式（推荐）。

使用序列化进行克隆：在完成序列化编码的时候，Serializable接口会具体到每一个对象都拔出他的根本引用，作为参考写入。可以以此作为一个迭代的克隆（卧槽代码瞬间就少了（不过涉及寻址、取值、记录装载和编码，这样克隆会相对慢很多

### RMI
RMI（Remote Method Invocation）Java中的远程调用，可以跨越不同平台，把对象以流的方式传输（如在本地调用云端的方法，云端返回对象回到本地，实现的框架比如Netty）。多端需要有相同的类或接口，版本控制也是一个较大的问题。通常为明文编码，有特定编解码规范，自行造个加密轮子（雾）

网络通信代理模型

1. CORBA（通用对象请求代理），支持任何语言的对象之间的通信，使用IIOP协议

2. web服务架构（WS-\*），基于xml文件，传输简单对象的访问协议（SOAP）

rmi属于定制版的对象传输，针对java对象进行编码。rmi也可以进行分布式的部署任务

### 读写文件
随机访问流可以访问文件、复制移动和删除文件，Java有自带的包API（不劳烦您解决生产者消费者问题了）、也包含创建目录、递归创建，获取文件具体信息等基本功能。

运用正则表达式+File提供获取整个递归目录的方法=搜索文件（使用NIO的FileVisitor进行遍历文件）

内存映射文件：通过虚拟内存来实现，把文件映射到内存地址访问，实现快速的文件读写

文件通道：对外设文件的一种抽象，交付给内核以完成文件管理（加锁、传输、读写、内存映射等）


### xml
java提供了两种xml解析器，DOM支持将xml解析为树形结构，SAX解析成字符流。``DocumentBuilder``实现生成器模式，``DocumentBuilderFactory``实现工厂模式，后者生成前者，前者构建解析器，支持传入文档、url或一个输入流，由根结点开始向内遍历（会把空白包含在内）

dtd文档类型定义，通常在文档头、xsd文件具体描述dtd内容

蜜汁使用xml构建凭空生成svg图像

### NIO
非阻塞式，通过映射来处理数据块的io包，channel和buffer是核心对象。buffer作为一个抽象类，本质是一个数组，作为容器使用；创建成本高，用法与传统buffer类没有太大区别。channel的读写交互都经由buffer完成。

提供了FileLock的文件锁功能（lock()阻塞方法、tryLock()非阻塞的尝试方法），锁是虚拟机持有，两个java程序不能对同一文件加两把锁

WatchService创建一个后台线程负责对一些文件进行变化监控

调用java.nio.file.attribute访问文件属性

## socket
半关闭：比如http接受到请求，客户端发出请求后即可关闭输出流

套接字中断：socketchannel

URI统一资源标识符（提供语法结构），在库里主要做解析工作，URL实现对资源的连接。URL及相关类如URLConnection与URLPermission权限管理

## JDBC
jdbc使用标准SQL语句+sql自选扩展+db提供商与中间件开发的驱动

JDBC规范的驱动分类：

1. jdbc翻译到odbc，使用odbc驱动去驱动数据库（早期java）

2. java代码+平台代码+数据库api，需要安装平台相关性代码+类库

3. 纯java驱动，由服务器构件（中间件）完成JDBC请求匹配到指定数据库的API

4. 纯java驱动，由jdbc直接翻译成数据库协议api（高可靠）

## 国际化&多语言
Locale对象表达，可定制格式化内容，就是代码要麻烦一点。formatter格式器系列以工厂方法接受一个locale对象生成格式器。

字符串格式化通过properties属性文件的键值对映射来完成管理（ResourceBundle），编程的时候使用键映射，在properties里完成映射内容。

## 窗口

窗口开发使用JOptionPane提供了几个模板式窗口，确认信息窗口、要求输入窗口、选择窗口，还有三合一的窗口。

## 安全
java.security包中有加密相关工具，sha1、md5之类的在MessageDigest类的静态工厂实现

权限集合分割访问

对称密码流

JNI（java本地接口），用于在java中融合其他语言或本地bash等协同工作。（因为管理机制，java并不推荐使用）

### 类加载器
编译器将java文件转换为虚拟机指令（.class），类加载器即用来在虚拟机运行时加载这些class文件（关联式、运行时加载），协同安全管理器类工作。有引导类、扩展类和系统类三种加载器。父类加载器优先加载，每个线程拥有一个上下文类加载器。可以自己编写加密的类加载器。

加载器将字节码传递给虚拟机的时候会经过**校验器**的一系列检查（如变量初始化、访问规则违反与否、类型匹配等）@编译器检查+数字签名。经过校验器后再送入安全管理器。安全管理器（将检查：创建or销毁虚拟机、访问本地文件socket剪贴板，还有反射再创建等等。默认不开启）

## 杂项
虚拟机提供脚本引擎，得以使用js、ruby等语言对虚拟机和执行程序进行控制（当然脚本是写在外面的）

编译器提供api可以手写动态编译检查和连接

@interface 注释类，提供在编译器检查时候的多分支功能

运用反射创建新对象

高精度：BigInteger（装饰器）

Jenkins：java写成的在线部署工具

高并发与服务器模型（C10K问题）

jsp服务器模型

Junit测试工具（TestNG）

三大框架struts+hibernate+spring

[结构类似生成器模式的流式编程](http://www.cnblogs.com/jun-ma/p/4838260.html)




所看的资料参考程度

1. Core Java：外国友人作，首推。思路非常明确：“其他技术→存在问题→众人的解决方法→java的解决方法”来引出java的某项技术，会从较广范围由浅入深（会很深，容易让人放弃），代码示例算比较易懂暴力，**适合对计算机刚入门不久且懵逼的朋友**。有文档摘抄，**适合当工具书**。

2. 疯狂Java讲义：估计是国内少有得几本拿得出手的java教程，比较**符合国人学习状况**，代码会标出重点啥的。文档有摘抄但没有统一归纳出来。内容太过详细容易磨灭新手自信心，**适合有略微java基础的深入或补漏**。

3. Thinking in Java (4th Edition)：非常细节细致入微，以java特性为基础和基本单位进行讲解，**适合理论研究和补漏**，不太适合入门。

4. Effective Java (2nd Edition)：78条经验设计理论，比较形而上，**适合瓶颈期针对问题进行深入研究**。



---
参考资料

1. [Core Java™ Volume I Fundamentals. (Ninth Edition)](https://www.amazon.com/Core-Java-I-Fundamentals-9th/dp/0137081898)

2. [Core Java™ Volume II Fundamentals. (Ninth Edition)](https://www.amazon.com/Core-Java-II-Advanced-Features-9th/dp/013708160X)

3. [疯狂JAVA讲义 (3rd Edition)](https://book.douban.com/subject/3246499/)

3. [Thinking in Java (4th Edition)](https://www.amazon.com/Thinking-Java-4th-Bruce-Eckel/dp/0131872486)

4. [Effective Java  (2nd Edition)](https://book.douban.com/subject/3360807/)

5. [Github: jobbole/awesome-java-cn](https://github.com/jobbole/awesome-java-cn)

6. [Github: google/guava](https://github.com/google/guava)

7. [google/guava 翻译学习支持](http://ifeve.com/google-guava/)

8. [Gitbook 阿里巴巴Java开发手册](https://goghtsui.gitbooks.io/-java/content/)

9. [如何评价阿里近期发布的Java编码规范？ - 知乎](https://www.zhihu.com/question/55642203)

10. [Java SE API与文档](http://www.oracle.com/technetwork/cn/java/javase/documentation/api-jsp-136079-zhs.html)
