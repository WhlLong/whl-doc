[TOC]



# 简介

Jstat是JDK自带的一个轻量级小工具。全称“Java Virtual Machine statistics monitoring tool”，它位于java的bin目录下，主要利用JVM内建的指令对Java应用程序的资源和性能进行实时的命令行的监控，包括了对Heap size和垃圾回收状况的监控。



```
jstat -help
Usage: jstat -help|-options
       jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
 
option： 参数选项，例-gc 或者-gcutil 查看gc情况
-t： 可以在打印的列加上Timestamp列，用于显示系统运行的时间【ps 非必须】
-h： 指定输出多少行以后输出一次表头【ps 非必须】
vmid： Virtual Machine ID（ 进程的 pid）
interval： 执行每次的间隔时间（单位为毫秒）
count： 用于指定输出多少次记录，缺省则会一直打印
 
打印当前gc情况
 
jstat -gc pid
 
需要循环打印，设置时间间隔即可
 
jstat -gc pid 5000
```





# options

options包含很多个选项，下面个是各个选项及其功能的描述，再往下会挑几个比较重要的对其输出做详细描述。

| name             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| class            | 类加载器的行为统计                                           |
| compiler         | HotSpot即时编译器的行为统计                                  |
| gc               | 堆的垃圾回收器的行为统计                                     |
| gccapacity       | Java各代区域以及对应空间的容量统计                           |
| gccause          | 垃圾回收的摘要信息(等同于-gcutil)， 以及最后的和当前的(如适用)垃圾回收事件的原因。 |
| gcnew            | new generation的行为统计                                     |
| gcnewcapacity    | new generation及其对应空间的大小统计。                       |
| gcold            | old和permanent generation的行为统计。                        |
| gcoldcapacity    | old generation的大小统计。                                   |
| gcpermcapacity   | permanent generation的大小统计。                             |
| gcutil           | 垃圾回收统计的摘要信息。                                     |
| printcompilation | HotSpot汇编方法统计。                                        |





## -class

jstat -class 49250
Loaded  Bytes  Unloaded  Bytes     Time   
 22264 39522.3      286   401.7      36.93

| name     | 描述                             |
| -------- | -------------------------------- |
| Loaded   | 加载的类的数量                   |
| Bytes    | 加载的类的总量，单位：KB         |
| Unloaded | 卸载的类的数量                   |
| Bytes    | 卸载的类的总量，单位：KB         |
| Time     | 执行类加载和卸载操作所花费的时间 |



## -compiler

-jstat -compiler 49250

Compiled Failed Invalid   Time   FailedType FailedMethod
   22675      4       0   338.71          1 org/aspectj/weaver/World resolve

| name         | 描述                     |
| ------------ | ------------------------ |
| Compiled     | 执行的编译任务数         |
| Failed       | 编译任务失败次数         |
| Invalid      | 已失效的编译任务数       |
| Time         | 执行编译任务所花费的时间 |
| FailedType   | 编译上次失败编译的类型   |
| FailedMethod | 上次失败编译的类名和     |



## -gc

 jstat -gc 55330  1000 10

S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT   
26176.0 26176.0  0.0   24590.5 209792.0 51874.7   262144.0   69304.3   117632.0 111855.6 15388.0 14417.9     89    2.897   8      0.677    3.574



| name | 描述                                                |
| ---- | --------------------------------------------------- |
| S0C  | 年轻代中第一个survivor（幸存区）的容量 (kb)         |
| S1C  | 年轻代中第二个survivor（幸存区）的容量 (kb)         |
| S0U  | 年轻代中第一个survivor（幸存区）目前已使用空间 (kb) |
| S1U  | 年轻代中第二个survivor（幸存区）目前已使用空间 (kb) |
| EC   | 年轻代中Eden（伊甸园）的容量 (kb)                   |
| EU   | 年轻代中Eden（伊甸园）目前已使用空间 (kb)           |
| OC   | Old代的容量 (kb)                                    |
| OU   | Old代目前已使用空间 (kb)                            |
| MC   | 元空间容量（kB）。                                  |
| MU   | Metacspace使用量（kB）                              |
| CCSC | 压缩类空间容量（kB）。                              |
| CCSU | 使用的压缩类空间（kB）                              |
| PC   | Perm(持久代)的容量 (kb)                             |
| PU   | Perm(持久代)目前已使用空间 (kb)                     |
| YGC  | 从应用程序启动到采样时年轻代中gc次数                |
| YGCT | 从应用程序启动到采样时年轻代中gc所用时间(s)         |
| FGC  | 从应用程序启动到采样时old代(全gc)gc次数             |
| FGCT | 从应用程序启动到采样时old代(全gc)gc所用时间(s)      |
| GCT  | 从应用程序启动到采样时gc用的总时间(s)               |



## -gcutil

jstat -gcutil 55330  1000 10

 S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT   
 81.85   0.00  85.91  29.30  95.01  93.18     92    3.008     8    0.677    3.686

| name | 描述                                                     |
| ---- | -------------------------------------------------------- |
| S0   | 年轻代中第一个survivor（幸存区）已使用的占当前容量百分比 |
| S1   | 年轻代中第二个survivor（幸存区）已使用的占当前容量百分比 |
| E    | 年轻代中Eden（伊甸园）已使用的占当前容量百分比           |
| O    | old代已使用的占当前容量百分比                            |
| M    | 元空间利用率占空间当前容量的百分比                       |
| CCS  | 压缩的类空间利用率百分比                                 |
| YGC  | 年轻代GC事件的数量                                       |
| YGCT | 年轻代垃圾收集时间                                       |
| FGC  | fullgc次数                                               |
| FGCT | fullgc耗时(s)                                            |
| GCT  | 从应用程序启动到采样时gc用的总时间(s)                    |



## -gccause

jstat -gccause 55330 

S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT    LGCC                 GCC                 
81.85   0.00  91.16  29.30  95.01  93.18     92    3.008     8    0.677    3.686 Allocation Failure   No GC

| name | 描述               |
| ---- | ------------------ |
| LGCC | 上次垃圾回收的原因 |
| GCC  | 当前垃圾回收的原因 |





## -gccapacity

jstat -gccapacity 55330 

NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC 
262144.0 262144.0 262144.0 26176.0 26176.0 209792.0   262144.0   262144.0   262144.0   262144.0      0.0 1155072.0 121832.0      0.0 1048576.0  15644.0     92     8

| name  | 描述                                        |
| ----- | ------------------------------------------- |
| NGCMN | 年轻代(young)中初始化(最小)的大小 (kb)      |
| NGCMX | 年轻代(young)的最大容量 (kb)                |
| NGC   | 年轻代(young)中当前的容量 (kb)              |
| S0C   | 年轻代中第一个survivor（幸存区）的容量 (kb) |
| S1C   | 年轻代中第二个survivor（幸存区）的容量 (kb) |
| EC    | 年轻代中Eden（伊甸园）的容量 (kb)           |
| OGCMN | old代中初始化(最小)的大小 (kb)              |
| OGCMX | old代的最大容量 (kb)                        |
| OGC   | old代当前新生成的容量 (kb)                  |
| OC    | Old代的容量 (kb)                            |
| MCMN  | 最小元空间容量（kB）                        |
| MCMX  | 最大元空间容量（kB）                        |
| MC    | 元空间容量（kB）                            |
| CCSMN | 压缩类空间最小容量（kB）                    |
| CCSMX | 压缩类空间最大容量（kB）                    |
| CCSC  | 压缩类空间容量（kB）                        |
| YGC   | 年轻代GC事件的数量                          |
| FGC   | 从应用程序启动到采样时old代(全gc)gc次数     |



## -gcnew

 jstat -gcnew 55330 

 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT  
26176.0 26176.0    0.0 16579.6 15  15 13088.0 209792.0   6160.0     93    3.030

| name | 描述                                                |
| ---- | --------------------------------------------------- |
| S0C  | 年轻代中第一个survivor（幸存区）的容量 (kb)         |
| S1C  | 年轻代中第二个survivor（幸存区）的容量 (kb)         |
| S0U  | 年轻代中第一个survivor（幸存区）目前已使用空间 (kb) |
| S1U  | 年轻代中第二个survivor（幸存区）目前已使用空间 (kb) |
| TT   | 持有次数限制                                        |
| MTT  | 最大持有次数限制                                    |
| DSS  | 当前需要survivor（幸存区）的容量 (kb)（Eden区已满） |
| EC   | 年轻代中Eden（伊甸园）的容量 (kb)                   |
| EU   | 年轻代中Eden（伊甸园）目前已使用空间 (kb)           |
| YGC  | 从应用程序启动到采样时年轻代中gc次数                |
| YGCT | 从应用程序启动到采样时年轻代中gc所用时间(s)         |





# 使用

jstat在进行jvm调优时用处十分的大，通过该命令可以分析出新生代和老年代对象增长的速率，YGC的触发频率和平均耗时，FGC的触发频率和平均耗时以及YGC以后对象的存活率。

以 jstat -gc 55330  1000 10这条命令为例，它的作用是每秒输出一次gc信息，一共输出10次。案例如下：

![1566364822303](D:\whl\github\whl-doc\jvm\jvm命令\jvm命令之jstat\jstat-gc.png)

这里只是简单的输出了一个本地环境的gc状况，并没有真实模拟出gc的情况，但是这并不妨碍我们分析具体使用状况。

## 新生代对象增长的速率

想要知道新生代对象增长的情况，可以根据EC和EU两个参数的输出结果来看，这里的命令是每秒输出一次，那么EU这个参数就可以看出每隔一秒新生代所用内存占用了多少。

在我们进行压测的时候，使用这个命令，就可以知道在压测流量下，每秒钟产生的对象大概是多少，从而估算出在当前新生代的配置下大概多久会发生一次YGC。



## YGC的触发频率和平均耗时

YGC和YGCT代表的是程序启动以来发生YGC的次数和这些YGC所用时间的总和。那么根据这两个参数的输出结果就能够大概的估算出YGC的触发频率和每次YGC的平均耗时，通过这种形式估算出来的结果要比上面通过新生代对象增长情况估算出来的更为准确。



## YGC以后对象存活和晋升到老年代的数量

通过观察YGC的值，我们可以真实的看到哪一秒发生了YGC，再根据这一秒钟EU和S0/S1占用的情况，可以看出YGC以后还存活在新生代的对象的总量，根据这一秒钟内OU占用的情况，可以看出YGC以后老年代的占用变化了多少（晋升到老年代的对象有多少），综合YGC以后新生代剩余的对象以及晋升到老年代的数量，就可以估算出YGC以后的真正的对象存活率。





## FGC触发频率和平均耗时

FGC的触发频率和平均耗时的估算方法与YGC基本一致，FGC和FGCT代表的事程序启动以来发生FGC的次数和发生FGC的总耗时，再结合程序启动时间，就可以估算出FGC大概多长时间会触发一次，平均每次耗时多久。