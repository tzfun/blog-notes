# 前言

上一篇文章介绍了常用虚拟机监控工具的使用方法以及参数含义，这一篇就从源码角度来看一下它们的内部构造。因为笔者时间有限，目前为止只看了jps、jstat、jstack的源码，同时笔者也准备写一个更方便于开发者调用的API，对其源码进行了相应的改造，至于为什么改造后面看源码的时候给大家讲，最后API做出来了还希望大家多多支持哦~

如果读者有兴趣深入学习虚拟机监控相关的类库，需要提示一下，jdk/lib下面的包是核心包，无论是你看源码还是拓展类库，都离不开他们，目前就笔者看的jps、jstat、jstack三个命令来说，需要把**tools.jar**和**sa-jdi.jar**两个包加入到项目中，其中tools.jar提供较多的命令式调用的类库，也包括参数解析等，jps和jstat完全只用tools.jar就可以了，但是jstack必需sa-jdi.jar这个包，因为其内部使用反射调用了这个包中的JStack类的main方法，如果不加入的话就会报异常。

在jar包中的文件都是class文件，你需要进行手动反编译去看，或者你用idea工具编译，这里笔者是用的idea工具解析的。

因为jstat和jstack命令较为复杂，内容较多，一篇文章不足以讲述，所以这里以jps命令源码解析为主，另外两个命令会说一些大体执行流程，不会系讲

# jps命令

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/1.png)

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/2.png)

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/3.png)

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/4.png)

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/5.png)

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/6.png)

看完jps执行函数之后最关心的就是如何获取这些虚拟机实例的，笔者也很是好奇，于是去看了getMonitoredHost()这个方法。结果发现这些获取虚拟机的信息都是通过System.getProperty()方法获取的，这些都是从系统或虚拟机底层提供的实时数据，这些也是虚拟机的开发者为了方便管理VM提供的一些数据信息，没什么好奇怪的。

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190529101816.png)

# jstat和jstack命令
jstat命令可以获取很多虚拟机参数、类加载、gc、内存情况等等，jstack可以获取各个线程的堆栈信息，这两个命令相比于jps内部相对更加复杂，这里篇幅原因就不详细讲述了。如果有兴趣可以关注笔者开发的API（开发中），里面会有一些代码注释，当然那是笔者改造之后的代码，和源代码有一些差别。

jstat命令因为命令参数较多，而且返回的数据差别较大，所以它是将所有结果进行了Format处理，将结果包装成OutputFormatter对象后统一输出，在其内部是通过虚拟机全路径打开一个输出流（OutputStream），这些时实的信息都是从虚拟机按照特定的协议获取数据。

jstack命令的构造则和前两个有点不太，前两个的相关处理类都是在tools.jar包中，而jstack则没有，在tools包中的Jstack类实际上就只是一个外壳，没有实际的含义，为什么这么说呢？看下面这个图就知道了

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190529103746.png)

是不是恍然大悟？它实际上就是通过反射调用Hotspot包下面的Jstack类的方法，而这个类就在sa_jdi.jar包中，仔细去看里面Jstack的执行函数体，它是通过Tool类注册监视虚拟机的vmId，然后又跑一个多线程去依次遍历被监视虚拟机的各个线程数据信息。

# 总结

通过看源码还是有很多收获的，设计者的代码风格简洁明了，很多是值的借鉴的。对于虚拟机的监控来讲，或许只有通过虚拟机开发商提供的数据或管理接口才能获取，毕竟这东西是别人开发的嘛，规则还得他们来制定。如果想自己写监控工具要么使用System的exec方法调用，但是这个方法有很大弊端，如果同时调用数量多了而且很不幸调用进程发生了阻塞，这样容易造成内存冲爆，因为它的原理实际上是调用系统的其他进程，也就是说它会开一个新的进程去执行，和在命令行执行是一样的，这种开销太大不推荐使用，在《深入理解Java虚拟机》第5章作者也有讲到这个。最好的是使用提供的api或者在此基础上进行改进，而tools这些命令类库直接使用显然是没法的，比如上面看的jps命令，很多地方用到了System.exit()，而且输出全都是写死了的，用的System.out，想获取内容只能通过控制台，其他tools下面的所有工具类都是如此。所以最好的办法就是重构它们啦，笔者也是初次写虚拟机监控API，如果你对此有兴趣欢迎和我交流。