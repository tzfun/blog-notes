前段时间我在看《深入理解Java虚拟机》一书，打算好好学学虚拟机的知识，在看到《第4章 虚拟机性能监控与故障处理工具》时产生了诸多灵感。首先是了解这些监控工具能干嘛？然后发现了其中一些局限性，接着视图解读部分命令源码，自己也想做一个虚拟机监控工具，但是想想做工具应用很简单，如果能将jdk提供的调试库进行改造然后封装成API，那么每个开发者岂不都可以很容易的写自己的虚拟机监控工具了吗？于是我开发了自己的第一个API：VmConsole-Api

* github：https://github.com/tzfun/VmConsole-Api

求颗星星鼓励一下(ಥ_ಥ) 

# VmConsole简介
VmConsole-Api是一个jvm虚拟机性能监控API，将oracle jdk提供的tools.jar、sa-jdi.jar包进行了功能拓展，对一些监控命令结果进行了封装，你可以方便地从对象中读取每一个参数和结果。该类库并不是简单地使用运行时exec()调用jps、jstat、jstack等命令，而是从底层深度拓展而来，所以你不需要配置java环境变量就可以通过Java代码对虚拟机进行监控。

目前VmConsole已经发布到Maven中央仓库了，使用者可以直接引入Maven依赖即可，当然非Maven项目也可从我的Github中下载Jar包导入项目

当然可能有其他公司或者大佬开发出了虚拟机监控API，如果有就当是我学习练手啦，同时希望有相关信息的朋友告知作者我一下。

# jdk监控工具的局限性
在jdk中提供有很多的监控工具，比如jps、jstack、jstat等，还有图像界面的jconsole、visualvm等，但是这些工具**仅用于调试**，如果在自己的项目中使用确很难，除非调用exec()方法，但是这种方法很影响程序性能。

在tools.jar包里面查看这些命令类库源码可以发现，里面全都是一次性执行就结束的函数体，大量使用了System.exit()关闭虚拟机和System.out.println()输出结果，**可用性很低**。读者可以看我的这篇文章了解一些命令执行流程：[从源码角度深度分析JVM虚拟机监控工具](https://mp.weixin.qq.com/s?__biz=MzU3OTkyMDAxNg==&mid=2247483979&idx=1&sn=27a64699f00639f4172dae2b16a1e32f&chksm=fd5f8bb4ca2802a283a787cc5a51e835b33628f50c26de106718fddd39d609ef34d250588da1&token=1596384866&lang=zh_CN#rd)

# 对tools.jar功能拓展

tools.jar和sa-jdi.jar是虚拟机监控最主要的两个类库，jdk/bin目录下的jps、jconsole、visualvm等程序都是基于这两个包进行开发的，在tools.jar和sa-jdi.jar中有很多可以直接使用的方法，它们相当于是对虚拟机信息获取类进行的另一层封装，但是这些封装都仅仅是**一次执行就结束**，重用性很低，类中的执行方法无法直接调用。我通过方法重载、类继承等方式去拓展了一些类，有些类的方法和属性是private的，无法去拓展，就直接重构了整个类，比如`beifengtz.vmconsole.tools.MyTool`类就是对sa-jdi.jar中的Tool类方法进行了重构。

我在编写时考虑到了可拓展性，在原jdk的类方法固定死了输出方式为System.out.println()，改编之后提供了打印流和对象接收结果两种方式，更方便开发者功能拓展。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190610122248.png)

# 简单使用

首先你需要将vmconsole-api.jar包导入到项目中，可以是直接导入jar也可以是Maven。

然后就可以直接使用其中封装的命令了，详细使用说明请前往github查看。这里以jstat命令为例。

如果你是在本地使用jstat命令查看vmId为8804的gc情况，在控制台结果是这样的：

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190610120300.png)

如果你用vmconsole调用函数，只需要在自己的代码中这样调用：

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190610120810.png)

命令执行的结果是一个对象，为了方便观看，将其格式处理之后：

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190610121219.png)

除了众多查询虚拟机信息的接口之外还有**向其他虚拟机执行命令**的函数，封装于jcmd命令。比如你现在使用的程序是虚拟机A，另一个程序在虚拟机B，在虚拟机A中想要对虚拟机B执行GC，那么你只需要这样调用：

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190610121846.png)

# 使用场景

如果你想写一个桌面版的虚拟机监控可视化程序，VmConsole能给你提供许多方便。如果你想写一个web版的虚拟机监控工具，那么VmConsole是你很好的选择，因为它不是简单地执行exec()命令，而是从jdk命令底层封装，相比于执行exec，它能给你的程序提升很大的性能。