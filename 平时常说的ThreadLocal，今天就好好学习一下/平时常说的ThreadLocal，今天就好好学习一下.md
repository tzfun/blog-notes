# 前言

ThreadLocal是多线程处理中非常重要的一个工具，比如数据库连接池存放Connection、存放本地参数等作用，面试也经常会问到它的应用及原理，本文就将从外到内地学习一下ThreadLocal。

# 一、了解ThreadLocal的作用

ThreadLocal顾名思义是线程私有的局部变量存储容器，可以理解成每个线程都有自己专属的存储容器，它用来存储线程私有变量，其实它只是一个外壳，内部真正存取是一个Map，后面会仔细讲解。每个线程可以通过`set()`和`get()`存取变量，多线程间无法访问各自的局部变量，相当于在每个线程间建立了一个隔板。只要线程处于活动状态，它所对应的ThreadLocal实例就是可访问的，线程被终止后，它的所有实例将被垃圾收集。总之记住一句话：**ThreadLocal存储的变量属于当前线程**。

# 二、ThreadLocal简单使用

话不多说先看一下ThreadLocal的一个简单案例

**Code**:
```java
public class Test implements Runnable {
    private static AtomicInteger counter = new AtomicInteger(100);
    private static ThreadLocal<String> threadInfo = new ThreadLocal<String>() {
        @Override
        protected String initialValue() {
            return "[" + Thread.currentThread().getName() + "," + counter.getAndIncrement() + "]";
        }
    };

    @Override
    public void run() {
        System.out.println("threadInfo value:" + threadInfo.get());
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(new Test());
        Thread thread2 = new Thread(new Test());

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.println("threadInfo value in main:" + threadInfo.get());
    }
}
```

**Output**:
```
threadInfo value:[Thread-0,100]
threadInfo value:[Thread-1,101]
threadInfo value in main:[main,102]
```

上述代码中我用ThreadLocal来存储线程的信息，其格式为`[线程名，线程ID]`，定义的变量是静态的。从运行结果可以看出来每个线程包括主线程访问到的`threadInfo`获取到的值都是不一样的，而且存放的信息就是本线程的信息，也应证了上面那句话**ThreadLocal存储的变量属于当前线程**。

# 三、ThreadLocal原理

## 3.1 ThreadLocal的存取过程
解析原理先从源码开始，首先看一下`ThreadLocal.set()`方法
```java
//  ThreadLocal中set方法
public void set(T value) {
    //  获取当前线程对象
    Thread t = Thread.currentThread();
    //  获取该线程的threadLocals属性（ThreadLocalMap对象）
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

//  Thread类中的threadLocals定义
ThreadLocal.ThreadLocalMap threadLocals = null;

//  ThreadLocal中getMap方法
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

//  ThreadLocal中createMap方法
//  为线程创建ThreadLocalMap对象并赋值给threadLocals
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

从源码中看到在set方法里面就是把值存入`ThreadLocalMap`类中，这个类是属于`ThreadLocal`的内部类，但是在`Thread`类中也有定义`threadLocals`变量，get方法的操作对象也是`ThreadLocalMap`，也就是说关键的存储和获取实质上在于`ThreadLocalMap`类。其中是以ThreadLocal类为key，存入的值为value，而ThreadLocal又是定义在每个线程的属性中，这也就实现了“ThreadLocal线程私有化”的功能，**每次都是先从当前线程获取到threadLocals属性，也就是获得ThreadLocalMap对象，以ThreadLocal对象作为key存取对应的值**。

## 3.2 探究ThreadLocalMap对象

`ThreadLocalMap`对象是ThreadLocal类的内部类，其中它就是一个简单的Map，每个存的值被封装成Entry进行存储，下面是Entry的源码

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

Entry是ThreadLocalMap的内部类，仔细观察其源码发现，它是继承了一个ThreadLocal的弱引用。回忆一下Java中的四种引用：强引用、软引用、弱引用、幻象引用。强引用是new创建出来的对象，只要强引用存在，垃圾收集器永远不会回收该引用；**软引用**（SoftReference）是在内存即将被占满时被回收；**弱引用**（WeakReference）用来描述非必需对象的，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次GC发生之前，当垃圾收集器工作时，无论当前内存是否足够，都会回收掉该类对象；**幻象引用**又称虚引用或幽灵引用（Phantom Reference），它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得对象实例，任何时候都可能被回收。更多的解释和示例读者可前往我之前写的这篇文章阅读：

* [JVM 垃圾回收机制（GC）总结 ](https://mp.weixin.qq.com/s/nkTBD2YmJ0CnhNckwDIU2g)

## 3.3 ThreadLocal对象的回收

Entry是一个弱引用，是因为它不会影响到ThreadLocal的GC行为，如果是强引用的话，在线程运行过程中，我们不再使用ThreadLocal了，将ThreadLocal置为null，但ThreadLocal在线程的ThreadLocalMap里还有引用，导致其无法被GC回收。而Entry声明为WeakReference，ThreadLocal置为null后，线程的ThreadLocalMap就不算强引用了，ThreadLocal就可以被GC回收了。

```java
//  ThreadLocal中的remove方法
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if (m != null)
        m.remove(this);
}

//  ThreadLocalMap中的remove方法
private void remove(ThreadLocal<?> key) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
            e != null;
            e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            e.clear();
            expungeStaleEntry(i);
            return;
        }
    }
}
```

**可能存在的内存泄漏问题**

ThreadLocalMap的生命周期是与线程一样的，但是ThreadLocal却不一定，可能ThreadLocal使用完了就想要被回收，但是此时线程可能不会立即终止，还会继续运行（比如线程池中线程重复利用），如果ThreadLocal对象只存在弱引用，那么在下一次垃圾回收的时候必然会被清理掉。

如果ThreadLocal没有被外部强引用的情况下，在垃圾回收的时候会被清理掉的，这样一来ThreadLocalMap中使用这个ThreadLocal的key也会被清理掉。但是，value 是强引用，不会被清理，这样一来就会出现key为null的value

在ThreadLocalMap中，调用 set()、get()、remove()方法的时候，会清理掉key为null的记录。在ThreadLocal设置为null之后，ThreadLocalMap中存在key为null的值，那么就可能发生内存泄漏，只有手动调用`remove()`方法来避免，**所以我们在使用完ThreadLocal变量时，尽量用threadLocal.remove()来清除，避免threadLocal=null的操作**。remove方法是彻底地回收该对象，而通过`threadLocal=null`只是释放掉了ThreadLocal的引用，但是在ThreadLocalMap中却还存在其Entry，后续还需处理。

# 四、ThreadLocal应用场景

* 处理较为复杂的业务时，使用ThreadLocal代替参数的显示传递。
* ThreadLocal可以用来做数据库连接池保存Connection对象，这样就可以让线程内多次获取到的连接是同一个了（Spring中的DataSource就是使用的ThreadLocal）。
* 管理Session会话，将Session保存在ThreadLocal中，使线程处理多次处理会话时始终是一个Session。