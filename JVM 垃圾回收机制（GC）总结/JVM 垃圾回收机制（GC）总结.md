# 一、概述
说起垃圾收集（Garbage Collection），大多数人都会想起Java，这项技术从始至终伴随着Java的成长，但事实上GC的出现要早于Java，它诞生于1960年MIT的使用动态分配和垃圾回收技术的语言Lisp。经过近60年的发展，目前内存的动态分配和内存回收技术已经非常成熟了，所有的垃圾回收已经自动化，经过迭代更新，自动回收也经过反复优化，效率和性能都非常可观。

**为什么要了解GC？**

在你排查内存溢出、内存泄漏等问题时，以及程序性能调优、解决并发场景下垃圾回收造成的性能瓶颈时，就需要对GC机制进行必要的监控和调节。
# 二、怎样标识哪些对象“已死”？

既然名叫垃圾回收，那么哪些对象成为“垃圾”呢？已经不再被使用的对象便视为“已死”，就应该被回收。在Java中，GC只针对于堆内存，Java语言中不存在指针说法，而是叫引用，**在堆内存中没有被任何栈内存引用的对象应该被回收**。

## 1.引用计数算法
引用计数算法是判断对象是否存活的算法之一：**它给每一个对象加一个引用计数器，每当有一个地方引用它时，计数器值就加1；当引用失效时，计数器值就减1；任何时刻计数器为0的对象就是不可能被使用的，即将被垃圾回收器回收。**

<font color=red><strong>缺点：</strong></font>**无法解决对象减互相循环引用的问题。**即当两个对象循环引用时，引用计数器都为1，当对象周期结束后应该被回收却无法回收，造成**内存泄漏**。

```
public class GcTest {

    public static void main(String[] args) {
        MyObject myObject_1 = new MyObject();
        MyObject myObject_2 = new MyObject();
        
        myObject_1.instance = myObject_2;
        myObject_2.instance = myObject_1;

        myObject_1 = null;
        myObject_2 = null;  

        //  对象循环引用，当时用引用计数算法时，无法回收这两个对象
        System.gc();
    }
    
    static class MyObject{
        Object instance;
    }
}
```

## 2.可达性分析算法
目前主流使用的都是可达性分析算法来判断对象是否存活。算法基本思路：**以“GC Roots”作为对象的起点,从此节点开始向下搜索，搜索所走过的路径成为引用链（Reference Chain），当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的。**

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190523194625.png)

**哪些对象可作为GC Roots？**
* 虚拟机栈（栈帧中的本地变量表）中引用的对象；
* 方法区中类静态属性引用的对象；
* 方法区中常量引用的对象；
* 本地方法栈中JNI（Native方法）引用的对象；
* 活跃线程的引用对象。

# 三、Java中四种引用

在JDK1.2之前，Java中的引用定义很单一：如果reference类型的数据中储存的数值代表的是另一块内存的起始地址，就称这块内存代表着一个引用。但是这种定义太过狭隘，如果某个对象介于被引用和未被引用两种状态之间，那么这种定义就显得无能为力。在JDK1.2后Java对引用的概念进行了扩充，将引用分为强引用（Strong Reference）、软引用（Soft Reference）、弱引用（Weak Reference）、虚引用（Phantom Reference），这四种引用强度依次逐渐减弱。

* **强引用（Strong Reference）**

强引用就是值在程序代码中普遍存在的，用new关键字创建的对象都是强引用，**只要强引用还存在，垃圾收集器永远不会回收掉被引用的对象**。

* **软引用（Soft Reference）**

软引用是用来描述一些还有用但并非必需的对象，在系统将要发**生内存溢出**之前，将会吧这些对象列入回收范围之中进行第二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。可用来实现高速缓存。软引用对象在回收时会被放入引用队列（ReferenceQueue）。

```
//  软引用
SoftReference<String> softReference = new SoftReference<>("北风IT之路");
```

* **弱引用（Weak Reference）**

弱引用也是用来描述非必需对象的，但是它的强度比软引用更弱一些，**被弱引用关联的对象只能生存到下一次GC发生之前，当垃圾收集器工作时，无论当前内存是否足够，都会回收掉该类对象。**弱引用对象在回收时会被放入引用队列（ReferenceQueue）。

```
//  弱引用
WeakReference<String> weakReference = new WeakReference<>("北风IT之路");
```

* **虚引用（Phantom Reference）**

虚引用被称为幽灵引用或幻象引用，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得对象实例。**任何时候都可能被回收**，一般用来跟踪对象被垃圾收集器回收的活动，起哨兵作用。必须和引用队列（ReferenceQueue）联合使用。

```
//  虚引用，必须配合引用队列使用
ReferenceQueue<String> referenceQueue = new ReferenceQueue<>();
PhantomReference<String> phantomReference = new PhantomReference<>("北风IT之路",referenceQueue);
```

# 四、finalize()赋予对象重生

在可达性分析算法中被标记为不可达的对象，也不一定是一定会被回收，它还有第二次重生的机会。每一个对象在被回收之前要进行两次标记，一次是没有关联引用链会被标记一次，第二次是判断该对象是否覆盖finalize()方法，如果没有覆盖则真正的被定了“死刑”。

如果这个对象被jvm判定为有必要执行finalize()方法，那么这个对象会被放入F-Queue队列中，并在烧毁由一个由虚拟机自动创建的、低优先级的finalizer线程去执行它。但是这里的“执行”是指虚拟机会触发这个方法，但是**并不代表会等它运行结束。**虚拟机在此处是做了优化的，因为如果某个对象在finalize方法中长时间运行或者发送死循环，将可能导致F-Queue队列中其他对象永远处于等待，甚至可能会导致整个内存回收系统崩溃。如果要在finalize方法中重生这个对象你可以按照下面代码做：

```
public class GcTest {
    public static GcTest instance = null;

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("收集器检测到finalize方法，对象即将获得一次重生的机会");
        instance = this;
    }

    public static void main(String[] args) throws InterruptedException{
        instance = new GcTest();
        //  引用置为空，堆内对象将视为垃圾
        instance = null;
        //  执行gc
        System.gc();
        Thread.sleep(500);
        //  虽然执行了gc，但是可能在finalize方法中获得重生，
        //  因此可能会打印出myObject的地址
        System.out.println(instance);
        //  最后打印出jvm.GcTest@7cc355be
    }
}
```

注意！finalize()方法只会被系统调用一次，多次被gc只有第一次会被调用，因此只有**一次**的重生机会。

# 五、回收方法区

假如一个字符串“abc”已经进入了常量池中，但是当前系统没有任何一个String对象是“abc”，那么这个对象就应该回收。方法去（HotSpot虚拟机中的永久代）的垃圾收集主要回收两部分内容：废弃常量和无用的类。比如上述的“abc”就是属于废弃常量，那么哪些类是无用的类呢？

* 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例；
* 加载该类的ClassLoader已经被回收；
* 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

# 六、垃圾收集算法

## 1.标记-清理算法（Mark-Sweep）

算法思路：**算法分为“标记”和“清理”两个步骤，首先标记处所有需要回收的对象，在标记完成后再统一回收所有被标记的对象。**

缺陷：
1. 标记和清理的两个过程效率都不高；
2. 容易产生内存碎片，碎片空间太多可能导致无法存放大对象。

<font color=red><strong>适用于存活对象占多数的情况。</strong></font>

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/1ewzykxlrj.png)

<font size=1>图片来源：https://cloud.tencent.com/developer/article/1336613</font>

## 2.复制算法（Copy）

算法思路：**将可用内存划分为大小相等的两块，每次只使用其中的一块。当这一块内存用完后，就将还存活的对象复制到另一块去，然后再把已使用过的内存空间一次清理掉。**

缺陷：
1. 可用内存缩小为了原来的一半

<font color=red><strong>算法执行效率高，适用于存活对象占少数的情况。</strong></font>

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/1k9hd934j5.png)

<font size=1>图片来源：https://cloud.tencent.com/developer/article/1336613</font>

## 3.标记-整理算法（Mark-compact）

算法思路：**标记过程和标记-清理算法一样，而后面的不一样，它是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存**

<font color=red><strong>有效地避免了内存碎片的产生。</strong></font>

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190523205351.png)

## 4.分代收集算法（Generational Collection）
当前大多数垃圾收集都采用的分代收集算法，这种算法并没有什么新的思路，只是根据对象存活周期的不同将内存划分为几块，每一块使用不同的上述算法去收集。**在jdk8以前分为三代：年轻代、老年代、永久代。在jdk8以后取消了永久代的说法，而是元空间取而代之。**一般年轻代使用复制算法（对象存活率低），老年代使用标记整理算法（对象存活率高）。

### 4.1 年轻代（复制算法为主）
尽可能快的收集掉声明周期短的对象。整个年轻代占1/3的堆空间，年轻代分为三个区，Eden、Survivor-from、Survivor-to，其内存大小默认比例为**8:1:1**（可调整），大部分新创建的对象都是在Eden区创建。当回收时，先将Eden区存活对象复制到一个Survivor-from区，然后清空Eden区，存活的对象年龄+1；当这个Survivor-from区也存放满了时，则将Eden区和Survivor-from区存活对象复制到另一个Survivor-to区，然后清空Eden和这个Survivor-from区，存活的对象年龄+1；此时Survivor-from区是空的，然后将Survivor-from区和Survivor-to区交换，即保持Survivor-from区为空（此时的Survivor-from是原来的Survivor-to区）， 如此往复。**年轻代执行的GC是Minor GC。**

年轻代的迭代更新很快，大多数对象的存活时间都比较短，所以对GC的效率和性能要求较高，因此使用复制算法，同时这样划分为三个区域，保证了每次GC仅浪费10%的内存，内存利用率也有所提高。

### 4.2 老年代（标记-整理算法为主）
在年轻代经过很多次垃圾回收之后仍然存活的对象（默认15岁），就会被放入老年代中，因为老年代中的对象大多数是存活的，所以使用算法是标记-整理算法。**老年代执行的GC是Full GC。**

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190523211844.png)

### 4.3 永久代/元空间

**jdk8以前：**

永久代用于存放静态文件，如Java类、方法等。该区域回收与上述“方法区内存回收”一致。但是永久代是使用的堆内存，如果创建对象太多容易造成内存溢出OOM（OutOfMemory）。

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190523211037.png)

**jdk8以后：**

jdk8以后便取消了永久代的说法，而是用元空间代替，所存内容没有变化，只是存储的地址有所改变，元空间使用的是主机内存，而不是堆内存，元空间的大小限制受主机内存限制，这样有效的避免了创建大量对象时发生内存溢出的情况。

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/20190523214226.png)

# 七、Minor GC和Full GC

之前多次提到Minor GC和Full GC，**那么它们有什么区别呢？**

* Minor GC即新生代GC：发生在新生代的垃圾收集动作，因为Java有朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。
* Major GC / Full GC：发生在老年代，经常会伴随至少一次Minor GC。Major GC的速度一般会比Minor GC慢倍以上。

**Minor GC发生条件：**

* 当新对象生成，并且在Eden申请空间失败时；

**Full GC发生条件：**

* 老年代空间不足
* 永久带空间不足（jdk8以前）
* System.gc()被显示调用
* Minor GC晋升到老年代的平均大小大于老年代的剩余空间
* 使用RMI来进行RPC或管理的JDK应用，每小时执行1次Full GC

# 八、常见的垃圾收集器（jdk8及以前）

一张图即可清除看到不同垃圾收集器之间的关系，连线表示可以配合使用。

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/GC.png)

* **Serial收集器（复制算法)**

新生代**单线程**收集器，标记和清理都是单线程，优点是简单高效。是client级别默认的GC方式，可以通过-XX:+UseSerialGC来强制指定。

* **Serial Old收集器(标记-整理算法)**

老年代单线程收集器，Serial收集器的老年代版本。

* **ParNew收集器(复制算法)**

新生代收集器，Serial收集器的多线程版本，在多核CPU情况时表现更好。

* **Parallel Scavenge收集器(复制算法)**

并行收集器，追求高吞吐量，高效利用CPU。适合后台应用等对交互相应要求不高的场景。是server级别默认采用的GC方式，可用-XX:+UseParallelGC来强制指定，用-XX:ParallelGCThreads=2来指定线程数。

* **Parallel Old收集器(复制算法)**
Parallel Scavenge收集器的老年代版本，并行收集器，吞吐量优先。

* **CMS(Concurrent Mark Sweep)收集器（标记-清理算法）**
高并发、低停顿，追求最短GC回收停顿时间（Stop The World），cpu占用比较高，响应时间快，停顿时间短，多核cpu追求高响应时间的选择，但是因为使用标记清理算法，容易产生内存碎片。

* **G1收集器**

G1是一款面向服务端应用的垃圾收集器，支持并行与并发、分代收集、空间整合和可预测停顿的能力，即可适用于年轻代又可适用于老年代。

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/rlwkbq3la9.png)

<font size=1>图片来源：https://cloud.tencent.com/developer/article/1336613</font>

# 九、垃圾收集器参数总结：

* **UseSerialGC**：虚拟机运行在Client模式下的默认值，打开此开关后，使用Serial+Serial Old的收集器组合进行内存回收
* **UseParNewGC**：打开此开关后，使用ParNew+Serial Old的收集器组合进行内存回收
* **UseConcMarkSweepGC**：打开此开关后，使用ParNew+CMS+Serial Old的收集器组合进行内存回收。Serial Old收集器将作为CMS收集器出现Concurrent Mode Failure失败后的后备收集器使用
* **UseParallelGC**：虚拟机运行在Server模式下的默认值，打开此开关后，使用Parallel Scavenge + Serial Old（PS MarkSweep）的收集器组合进行内存回收
* **UseParallelOldGC**：打开此开关后，使用Parallel Scavenge + Parallel Old的收集器组合进行内存回收
* **SurvivorRatio**：新生代中Eden区域与Survivor区域的容量比值，默认值为8，代表Eden：Survivor=8：1
* **PretenureSizeThreshold**：直接晋升到老年代的对象大小，设置这个参数后，大于这个参数的对象将直接在老年代分配
* **MaxTenuringThreshold**：晋升到老年代的对象年龄，每个对象在坚持过一次Minor GC之后，年龄就增加1，当超过这个参数时就进入老年代
* **UseAdaptiveSizePolicy**：动态调整Java堆中各个区域的大小以及进入老年代的年龄
* **HandlePromotionFailure**：是否允许分配担保失败，即老年代的剩余空间不足以应付新生代的整个Eden和Survivor区的所有对象都存活的极端情况
* **ParallelGCThreads**：设置并行GC时进行内存回收的线程数
* **GCTimeRatio**：GC时间占总时间的比率，默认值为99，即允许1%的GC时间。仅在使用Parallel Scavenge收集器时生效
* **MaxGCPauseMillis**：设置GC的最大停顿时间，仅在使用Parallel Scavenge收集器时生效
* **CMSInitingOccupancyFraction**：设置CMS收集器在老年代空间被使用多少后触发垃圾收集。默认值为68%，仅在使用CMS收集器时生效
* **UseCMSCompactAtFullCollection**：设置CMS收集器在完成垃圾收集后是否要进行一次内存碎片整理，仅在使用CMS收集器时生效
* **CMSFullGCsBeforeCompaction**：设置CMS收集器在进行若干次垃圾收集后再启动一次内存碎片整理。仅在使用CMS收集器时生效

参考文献：《深入理解Java虚拟机》