# 前言

一个Java类从被加载到虚拟机内存开始，到卸载出内存为止，它经过了哪些步骤呢？这篇文章就来简述一下关于Java类生命周期相关的知识，其中每个生命周期的具体内容不会细讲，因为内容太多，我准备专门花一篇文章介绍类生命周期中的详细步骤，期待下一篇文章吧~

# 概述

一个Java类从开始到结束整个生命周期会经历7个阶段：加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化（Initialization）、使用（Using）和卸载（Unloading）。其中验证、准备、解析三个部分又统称为连接（Linking）。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/870109-20160503213708857-429280187.png)

这里我所说的Java类是已经编译好的类，也就是说它已经是class字节码了，如果要从`.java`文件算起的话应该还有个编译过程。

# 每个阶段的顺序

并不是所有时候这七个阶段都是顺序进行的，其中加载、验证、准备、初始化、卸载是固定顺序开始的，解析阶段不一定。解析在某些情况下可以在初始化阶段之后再开始，这也是为了支持**运行时绑定**（也成为动态绑定）。刚刚说的五个阶段是固定顺序开始，但是不一定会按部就班地“进行”或“完成”，是因为这些阶段通常是互相交叉地混合进行的，通常会在一个阶段执行的过程中调用激活另一个阶段。

# 简述七个阶段

这里先简单介绍一下各个阶段所做的事，每个阶段详细的过程在后面会有专门的文章介绍。

1. 加载:加载过程就是把class字节码文件载入到虚拟机中，至于从哪儿加载，虚拟机设计者并没有限定，你可以从文件、压缩包、网络、数据库等等地方加载class字节码。

* 通过类的全限定名来获取定义此类的二进制字节流
* 将此二进制字节流所代表的静态存储结构转化成方法区的运行时数据结构
* 在内存中生成代表此类的java.lang.Class对象,作为该类访问入口.

2. 验证:验证的目的是确保class文件的字节流中信息符合虚拟机的要求，不会危害虚拟机安全，使得虚拟机免受恶意代码的攻击，这一步至关重要。

* 文件格式验证
* 源数据验证
* 字节码验证
* 符号引用验证

3. 准备:准备阶段的工作就是为类的静态变量分配内存并设为jvm默认的初值，对于非静态的变量，则不会为它们分配内存。静态变量的初值为jvm默认的初值，而不是我们在程序中设定的初值。(仅包含类变量,不包含实例变量).　　

4. 解析:虚拟机将常量池中的符号引用替换为直接引用，解析动作主要针对类或接口，字段，类方法，方法类型等等。

5. 初始化:在该阶段，才真正意义上的开始执行类中定义的java程序代码，该阶段会执行类构造器，并且在Java虚拟机规范中有明确的规定，在下面5种情况下必须对类进行初始化：

* 遇到new、getstatic、putstatic、invokestatic这4条字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。
* 使用java.long.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
* 当初始化一个类的时候，如果发现其父类没有进行过初始化，则需要先触发其父类的初始化。
* 当虚拟机启动时，需要制定一个执行的主类（即main方法的类），虚拟机必须先初始化这个类。
* 使用动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后解析结果是REF_getStatic、REF_putStatic、REF_invokeStatic的方法句柄，并且这个方法句柄对应的类没有进行初始化，则需要先触发其初始化。

6. 使用:使用该类所提供的功能，其中包括主动引用和被动引用。

主动引用：
* 通过new关键字实例化对象、读取或设置类的静态变量、调用类的静态方法。
* 通过反射方式执行以上三种行为。
* 初始化子类的时候，会触发父类的初始化。
* 作为程序入口直接运行时（也就是直接调用main方法）。

被动引用：
* 引用父类的静态字段，只会引起父类的初始化，而不会引起子类的初始化。
* 定义类数组，不会引起类的初始化。
* 引用类的常量，不会引起类的初始化。

7. 卸载:从内存中释放，在我之前写的**垃圾回收机制（GC）总结**一文中有介绍到方法区内存回收中对类的回收条件，这里再贴出来一下：

* 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例；
* 加载该类的ClassLoader已经被回收；
* 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

# 引用类时容易忽略的点

在平时做面试题中很有可能会考察对类加载流程的理解，有的是直接给你几个描述让你选择，有的是给出一段代码，让你判断输出结果。第一种方式偏向于理论，相信看了本文上面的介绍大多都知道多多少少，第二种往往是很多Java程序员容易犯错的，接下来给出几段代码来讲解。（环境是jdk1.8）

## No.1

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190603171132.png)

上面的代码执行完成之后除了123被输出外，在此之前还输出了“父类被初始化”，并没有输出子类被初始化。对于静态字段，只有直接定义这个字段的类才会被初始化，这里通过子类来访问父类中定义的静态字段，只会触发父类的初始化而不会触发子类的初始化。这里可以开启`-XX:+TraceClassLoading`虚拟机参数查看加载内容。

## No.2

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190603171438.png)

上面的代码执行完之后并没有任何输出！也就是说此过程并没有触发`jvm.FatherClass`的初始化阶段，但是实际上这个过程触发了另一个名为`[Lorg.FatherClass`的类的初始化，它是一个由虚拟机自动生成的、直接继承于Object的子类，创建动作由字节码指令anewarray触发。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190603173142.png)

`[Lorg.FatherClass`这个类代表了一个元素类型为`jvm.FatherClass`的一维数组，数组中应有的属性和方法（用户可直接使用的只有被修饰为public的length属性和clone方法）都实现在这个类里。

## No.3

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190603173639.png)

上面的代码执行结束之后控制台只输出了123，并没有输出“父类被初始化”，说明此时FatherClass并没有被触发初始化，这里和No.1里面value唯一不同就在于多加了一个final关键字。这是因为在编译阶段以及做了**常量传播优化**，在编译时就将常量value存储到了Test类的常量池中，这里对value的引用其实就是对本类（Test）常量池的引用，所以这里无需初始化FatherClass类。也就是说，实际上Test的class文件之中并没有FatherClass类的符号引用入口，这两个类在编译成class之后就不存在任何联系了。

为了佐证该过程，我反编译出No.1和No.3两个测试代码的Test字节码文件

No.1情况下的Test.class

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190603175126.png)
![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190603175137.png)

No.3情况下的Test.class

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190603175242.png)
![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190603175259.png)

从字节码中不难看出No.1情况获取value值是通过`getstatic #3`获取，其中#3是`#3 = Fieldref #18.#19        // jvm/FatherClass.value:I`，也就是说它仍然需要从FatherClass的引用获取vlaue。而No.3情况获取value值是`bipush 123`,这个123是直接从常量池中取的，无需从FatherClass类中获取。