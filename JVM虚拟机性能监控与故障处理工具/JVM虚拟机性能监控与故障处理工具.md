因为文章过长，来个目录吧
> * 一、概述
> * 二、命令行工具
>     * 1.JPS：JVM Process Status Tool
>     * 2.jstat：JVM Statistics Monitoring Tool
>         * 2.1 类加载
>         * 2.2 垃圾收集
>         * 2.3 编译
>         * 2.4 堆内存
>         * 2.5 新生代垃圾回收
>         * 2.6 新生代内存
>         * 2.7 老年代垃圾回收
>         * 2.8 老年代内存
>         * 2.9 元数据空间
>         * 2.10 垃圾回收统计
>         * 2.11 编译方法
>     * 3.jinfo：Configuration Info for Java
>     * 4.jmap：Memory Map for Java
>     * 5.jhat：JVM Heap Analysis Tool
>     * 6.jstack：Stack Trace for Java
> * 三、可视化工具
>     * 1.JConsole
>     * 2.Visual VM
> * 四、参考资料

# 一、概述
在平时开发时，往往会对程序进行测试，在定位问题时查看运行日志、查看GC日志、Debug、JVM监控等都是需要用到的，今天来介绍一些JDK自带的JVM性能监控与故障处理的工具。Sun公司在JDK中附赠了很多监控工具，每个工具的功能都很强大而且很实用，能在处理应用程序性能问题、定位故障时发挥很大的作用。

当然不同的jdk所提供的监控工具细节上有所差别，笔者所测试的系统是Windows 10，jdk环境是Oracle JDK 1.8，虚拟机是64位的HotSpot，。

# 二、命令行工具

在你所下载的jdk包下面的bin目录下，有非常多的exe文件，接下来要看的工具就在其中。

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526180531.png)

不难发现大部分exe文件都是在17KB左右，其实这并不是巧合，而是这些工具程序都是调用的jsk/lib/tools.jar类库下的函数，仅仅是对这个类库进行简单的封装，所以大小几乎都稳定在17KB左右。其实其他第三方监控工具也都是使用这个类库进行改造的，如果细细读其中的代码，你也可以自己写一个监控工具。

接下来的演示均是在本地演示，如果需要监控远程JVM的话，需要在启动时开启JMX管理功能（参数是：-Dcom.sun.management.jmxremote），当然还需要设置监控端口、用户名、密码、身份认证开关、SSL验证开关等，具体参数读者可自行百度或谷歌哦~

> 回答一个很多人都有的一个疑惑：系统只安装了一个jdk或jre，那么是多个程序共用一个虚拟机还是一个程序一个虚拟机？答案是一个程序一个jvm实例，你可以把它们的关系想象成类和对象的关系，jre就像是一个类，每运行一个程序时会实例化出一个jvm实例，同时根据参数为它分配内存空间。

## 1.JPS：JVM Process Status Tool

虚拟机进程状况工具，**可以列出正在运行的虚拟机进程，并显示虚拟机执行主类名称，以及这些进程的本地虚拟机唯一ID（LVMID）。**这个应用非常多，一般用于查看有哪些运行在虚拟机上的程序，比如后面要讲的JConsle.exe在选择监控程序时就是用的jps，当然追根揭底还是调用的tools.jar中的方法。

如果你配置了jdk/bin下的环境变量，可直接在cmd中输入jps命令进行使用，否则你需要进入这个目录才能使用该工具。

命令格式：
> jps [ options ] [ hostid ]

说明：如果不指定hostid就默认为当前主机或服务器，参数可以联合使用，比如*jps -lv*。

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526183347.png)

```
-q 输出虚拟机唯一ID，不输出类名、Jar名和传入main方法的参数
-m 输出传入main方法的参数
-l 输出main类或Jar的全名
-v 输出虚拟机进程启动时JVM的参数
```

## 2.jstat：JVM Statistics Monitoring Tool

虚拟机统计信息监视工具，**用于监视虚拟机各种运行状态信息的命令行工具，可以显示本地或远程虚拟机进程的类装载、内存、垃圾收集、JIT编译等运行数据。**它是运行期定位虚拟机性能问题的首选工具。

命令格式：
> jstat \[ option vmid \[interval \[s | ms\] \[count\] \] \]
>
> jstat \[-命令选项\] \[vmid\] \[间隔时间/毫秒\] \[查询次数\]

说明：如果是本地虚拟机进程，VMID和LVMID是一样的，如果是远程虚拟机进程，那VMID格式应该是：
> \[protocol:\]\[ // \] lvmid \[@hostname \[:port\] /servername \]

例如每250毫秒查询一次1200垃圾收集状况，一共查询20次，命令和结果如下：

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526185417.png)

jstat详细参数说明如下：

### 2.1 类加载

```
-class  监视类装载、卸载数量、总空间以及类装载所耗费的时间
```

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526190101.png)

每个字段说明：
```
Loaded: 加载class的数量
Bytes： 所占用空间大小
Unloaded： 未加载数量
Bytes: 未加载占用空间
Time： 时间
```

### 2.2 垃圾收集
```
-gc  监视Java堆状况，包括Eden区、两个Survivor区、老年代等容量、已用空间、GC时间合计等信息
```

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526185417.png)

每个字段说明：
```
S0C：第一个幸存区的大小
S1C：第二个幸存区的大小
S0U：第一个幸存区的使用大小
S1U：第二个幸存区的使用大小
EC：伊甸园区的大小
EU：伊甸园区的使用大小
OC：老年代大小
OU：老年代使用大小
MC：方法区大小
MU：方法区使用大小
CCSC:压缩类空间大小
CCSU:压缩类空间使用大小
YGC：年轻代垃圾回收次数
YGCT：年轻代垃圾回收消耗时间
FGC：老年代垃圾回收次数
FGCT：老年代垃圾回收消耗时间
GCT：垃圾回收消耗总时间
```

### 2.3 编译

```
-compiler  输出JIT编译器编译过的方法、耗时等信息
```

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526190622.png)

每个字段说明：
```
Compiled：编译数量。
Failed：失败数量
Invalid：不可用数量
Time：时间
FailedType：失败类型
FailedMethod：失败的方法
```

### 2.4 堆内存

```
-gccapacity  监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间
```

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526190910.png)

每个字段说明：
```
NGCMN：新生代最小容量
NGCMX：新生代最大容量
NGC：当前新生代容量
S0C：第一个幸存区大小
S1C：第二个幸存区的大小
EC：伊甸园区的大小
OGCMN：老年代最小容量
OGCMX：老年代最大容量
OGC：当前老年代大小
OC:当前老年代大小
MCMN:最小元数据容量
MCMX：最大元数据容量
MC：当前元数据空间大小
CCSMN：最小压缩类空间大小
CCSMX：最大压缩类空间大小
CCSC：当前压缩类空间大小
YGC：年轻代gc次数
FGC：老年代GC次数
```

### 2.5 新生代垃圾回收

```
-gcnew 监视新生代GC状况
```

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526191105.png)

每个字段说明：
```
S0C：第一个幸存区大小
S1C：第二个幸存区的大小
S0U：第一个幸存区的使用大小
S1U：第二个幸存区的使用大小
TT:对象在新生代存活的次数
MTT:对象在新生代存活的最大次数
DSS:期望的幸存区大小
EC：伊甸园区的大小
EU：伊甸园区的使用大小
YGC：年轻代垃圾回收次数
YGCT：年轻代垃圾回收消耗时间
```

### 2.6 新生代内存

```
-gcnewcapacity  监视内容与-gcnew基本相同，主要输出使用到的最大、最小空间
```

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526191341.png)

每个字段说明：
```
NGCMN：新生代最小容量
NGCMX：新生代最大容量
NGC：当前新生代容量
S0CMX：最大幸存1区大小
S0C：当前幸存1区大小
S1CMX：最大幸存2区大小
S1C：当前幸存2区大小
ECMX：最大伊甸园区大小
EC：当前伊甸园区大小
YGC：年轻代垃圾回收次数
FGC：老年代回收次数
```

### 2.7 老年代垃圾回收

```
-gcold 监视老年代GC状况
```

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526191524.png)

每个字段说明：
```
MC：方法区大小
MU：方法区使用大小
CCSC:压缩类空间大小
CCSU:压缩类空间使用大小
OC：老年代大小
OU：老年代使用大小
YGC：年轻代垃圾回收次数
FGC：老年代垃圾回收次数
FGCT：老年代垃圾回收消耗时间
GCT：垃圾回收消耗总时间
```

### 2.8 老年代内存

```
-gcoldcapacity 监视内容与-gcold基本相同，主要输出使用到的最大、最小空间
```

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526191704.png)

每个字段说明：
```
OGCMN：老年代最小容量
OGCMX：老年代最大容量
OGC：当前老年代大小
OC：老年代大小
YGC：年轻代垃圾回收次数
FGC：老年代垃圾回收次数
FGCT：老年代垃圾回收消耗时间
GCT：垃圾回收消耗总时间
```

### 2.9 元数据空间

```
-gcmetacapacity  监视元数据空间内存情况（jdk1.8之后）
```

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526191901.png)

每个字段说明：
```
MCMN:最小元数据容量
MCMX：最大元数据容量
MC：当前元数据空间大小
CCSMN：最小压缩类空间大小
CCSMX：最大压缩类空间大小
CCSC：当前压缩类空间大小
YGC：年轻代垃圾回收次数
FGC：老年代垃圾回收次数
FGCT：老年代垃圾回收消耗时间
GCT：垃圾回收消耗总时间
```
### 2.10 垃圾回收统计

```
-gcutil 统计所有垃圾回收的数据
```

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526192127.png)

每个字段说明：
```
S0：幸存1区当前使用比例
S1：幸存2区当前使用比例
E：伊甸园区使用比例
O：老年代使用比例
M：元数据区使用比例
CCS：压缩使用比例
YGC：年轻代垃圾回收次数
FGC：老年代垃圾回收次数
FGCT：老年代垃圾回收消耗时间
GCT：垃圾回收消耗总时间
```

### 2.11 编译方法

```
-printcompilation  输出已经被JIT编译的方法
```

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526192414.png)

每个字段说明：
```
Compiled：最近编译方法的数量
Size：最近编译方法的字节码数量
Type：最近编译方法的编译类型。
Method：方法名标识。
```

## 3.jinfo：Configuration Info for Java

Java配置信息工具，**能实时地查看和调整虚拟机各项参数。**

命令格式：
> jinfo \[option\] pid

说明：如果不传入参数，则输出该进程的所有配置信息

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526193001.png)

## 4.jmap：Memory Map for Java

Java内存映像工具，**用于生成堆存储快照（一般称为heapdump或dump文件），还可以查询finalize执行队列、Java堆的详细信息，如空间使用率、当前用的是哪种收集器等。**如果不适用jmap命令，可以使用-XX:+HeapDumpOnOutOfMemoryError参数，当虚拟机发生内存溢出的时候可以产生快照。或者使用kill -3 pid也可以产生

jmap命令格式：
> jmap \[option\] vmid

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526194240.png)

参数说明：
```
-dump:[live,]format=b,file=<filename> 使用hprof二进制形式,输出jvm的heap内容到文件=. live子选项是可选的，假如指定live选项,那么只输出活的对象到文件.
-finalizerinfo 打印正等候回收的对象的信息.
-heap 打印heap的概要信息，GC使用的算法，heap的配置及wise heap的使用情况.
-histo[:live] 打印每个class的实例数目,内存占用,类全名信息. VM的内部类名字开头会加上前缀”*”. 如果live子参数加上后,只统计活的对象数量.
-permstat 打印classload和jvm heap长久层的信息. 包含每个classloader的名字,活泼性,地址,父classloader和加载的class数量. 另外,内部String的数量和占用内存数也会打印出来.
-F 强迫.在pid没有相应的时候使用-dump或者-histo参数. 在这个模式下,live子参数无效.
-h | -help 打印辅助信息
-J 传递参数给jmap启动的jvm.
```

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526194012.png)

## 5.jhat：JVM Heap Analysis Tool

虚拟机堆转储快照分析工具，可与jmap配合使用，**这个工具是用来分析jmap dump出来的文件。**由于这个工具功能比较简陋，运行起来也比较耗时，所以这个工具不推荐使用，推荐使用MAT或Visual VM。

例如我要解析上面生成的dump文件可以用命令：
```
jhat vote.dump
```

## 6.jstack：Stack Trace for Java

Java堆栈跟踪工具，**用于生成虚拟机当前时刻的线程快照（一般称为threaddump或者javacore文件）。**线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是定位线程出现长时间停顿的原因，比如线程间死锁、死循环、请求外部资源导致的长时间等待等都是其常见原因。

jstack命令格式：
> jstack [option] vmid

参数含义：
```
-F  当正常输出的请求不被响应时，强制输出线程堆栈
-l  除堆栈外，显示关于锁的附加信息
-m  如果调用到本地方法的话，可以显示C/C++的堆栈
```

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526195353.png)

# 三、可视化工具

JDK除了提供上面所讲的几个强大命令行工具外，还提供了一些可视化工具，可以方便的实时监控正在运行的Java程序。主要是两个：JConsole和VisualVM

## 1.JConsole
点击bin目录下的jconsole.exe即可运行工具，可选两种监控对象，一个是本地程序一个是远程程序，如果要监控远程程序需要开启JMX管理，详细步骤因为篇幅原因就不说了，可以自行百度或谷歌。该程序因为是可视化工具，内容比较简洁直观，而且功能和上面的工具基本相同，只是把数据编程图标展示了，所以这里就不赘述，放截图就好啦。

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526200228.png)

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526200331.png)

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526200606.png)

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526200618.png)

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526200636.png)

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526200650.png)

## 2.Visual VM
点击bin目录下的jvisualvm.exe即可运行工具，jvisualvm同jconsole都是一个基于图形化界面的、可以查看本地及远程的JAVA GUI监控工具，Jvisualvm同jconsole的使用方式一样，jvisualvm界面更美观一些，数据更实时。

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526201032.png)

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526201042.png)

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190526201051.png)

# 四、参考资料

* 《深入理解Java虚拟机》
* https://blog.csdn.net/maosijunzi/article/details/46049117