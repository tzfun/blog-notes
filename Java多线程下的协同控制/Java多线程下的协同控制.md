协同控制是并发程序必不可少的重要手段。主要分为两大控制方法，一个是JDK提供的最基础的协同控制方法，一个是java.util.concurrent包下的拓展类控制，接下来我们将会介绍这两种方法有哪些操作可以进行同步控制。

# 一、基础的协同控制

## 线程基础知识
因为加锁涉及到多线程，所以有必要先说一下线程的基础知识（定义那些就不必多说了吧~~）。

首先线程是有生命周期的，在Java中它有6个状态来表示，分别是NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED

* 新建（NEW）：创建后尚未启动的线程的状态
* 运行（RUNNABLE）：正在运行或在准备的线程，包含Running和Ready
* 无限期等待（WAITING）：不会被分配CPU执行时间，需要显式被唤醒
* 限期等待（TIMED_WAITING）：在一定时间后会由系统自动唤醒
* 阻塞（BLOCKED）：等待获取排它锁
* 结束（TERMINATED）：已终止线程的状态，线程已经结束执行，并且不可再次唤醒。

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E7%BA%BF%E7%A8%8B%E6%B1%A0/20190511170255.jpg)

线程的启动是由start()方法启动的，至于结束stop()方法可以关闭，但是它是强制性关闭，也就是说你不管你线程的任务有没有执行完都立马停止，不推荐这种方法，取而代之的是interrupt()方法，它的原理就是多加了一个中断标志位，在线程执行中不断去判断是否中断，当中断设置为true时，当前任务执行完之后就结束线程。

## synchronized
如果某一个资源被多个线程共享，为了避免因为资源抢占导致资源数据错乱，我们需要对线程进行同步，那么synchronized关键字就可以实现线程间的同步。它可以保证线程的原子性、可见性、有序性。

synchronized的用法大致有三种：
* 对象加锁：对指定对象加锁，进入同步代码钱要获得给定对象的锁。
* 实例方法加锁：相当于对当前实例加锁，进入同步代码前要获得当前**实例**的锁。
* 静态方法加锁：相当于对当前类加锁，进入同步代码前要获得当前**类**的锁。

其中对实例方法加锁时容易出现错误的使用，比如下面的伪代码：
```
// 假设SyncClass中的非静态increase方法加了同步锁
Thread t1 = new Thread(new SyncClass());
Thread t2 = new Thread(new SyncClass());    //  未作用到同一个实例，加锁无效
```

因为在运行的时候因为不是同一个实例，每new一个对象就是一个新的实例，锁对方法的同步并未作用到同一个实例，所以加锁无效。有效的加锁方法如下：
```
// 假设SyncClass中的非静态increase方法加了同步锁
SyncClass sc = new SyncClass();
Thread t1 = new Thread(sc);
Thread t2 = new Thread(sc);     //  作用到同一个实例，有效的加锁
```

## 等待与通知
为了保证多个线程之间的协作，JDK提供了两个重要方法可以修改线程的状态：wait()和notify()。这两个类是Object类的方法，意味着任何对象都可以调用，但这两个方法必须在同步块中调用。

在线程A中，调用了obj.wait()方法，那么线程A就会停止继续执行，而转为等待状态，进入等待池（或等待队列）。

等待何时结束呢？要么在wait方法调用时传入等待时间，那么它就会进入TIMED_WAITING状态，然后经过等待时间后自动唤醒，如果没有传入等待时间就会进入WAITING状态，只有调用notify()或notifyAll()方法时才会被唤醒（两个方法区别后面介绍）。

如果线程所操作的资源被synchronized加了锁，并且此时锁被其他线程占用，那么该线程就会转为BLOCKED状态，进入锁池（或阻塞队列）。只有占用锁的这个线程释放了锁并且当前线程抢占到了这个锁，才会转为RUNNABLE状态继续运行任务。

**notify()和notifyAll()的区别是什么？**

notify()方法会从等待池（或等待队列）中随机选择一个线程进行唤醒，当然最恶劣的情况发生就是某一个线程运气很不好，每次都没有被选中，这样就容易造成线程饥饿，当然这种情况发生的可能性还是很小的。而notifyAll()方法会将等待池（等待队列）中所有的线程全部唤醒，进而抢占资源。

下面我简单画了一个示意图帮助大家理解

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E7%BA%BF%E7%A8%8B%E6%B1%A0/20190511173714.jpg)

## 等待线程结束
在一些情况下，线程之间的协作可能会存在依赖关系，比如线程B需要线程A的输出才能继续执行，那么就必须等待线程A运行完，此时就可以使用join来实现。
```
public final void join() throws InterruptedException 
public final synchronized void join(long millis) throws InterruptedException
```
join方法调用时可以传入等待时间参数，如果不传入任何参数表示无限等待，它会一直阻塞当前线程，知道目标线程执行完毕。如果传入等待时间，则在超过这个时间之后就停止等待，继续往下执行。

```
Thread threadA = new Thread();
Thread threadB = new Thread();
threadA.start();
threadA.join(); //  等待线程A执行完，线程B才启动
threadB.start();
```

## 线程谦让
在实际应用中有些任务是重要任务，有些任务的重要程度可能低一点，那么在程序运行中有可能存在优先执行重要任务而延后执行次要任务的需求。那么要实现这样的操作就要对线程优先级进行操作了，大家都知道线程是有优先级的，优先级高的就有**可能**优先执行，为什么是可能而不是一定呢？因为现在计算机的运行速度非常快，而且大多数都是多核心的，就算优先级低的线程也可能优先被CPU执行。

如果在运行中想手动让某个线程让出CPU让其他线程优先执行的话，就需要使用yield()方法了。该方法会让出CPU但是不会让出锁，但也不一定调用之后就会让出CPU，因为它只是给一个**暗示**，告诉其他线程我可以晚点执行，你们先执行吧！但是CPU可能会忽视这个暗示，在JDK中的注释有所说明：
```
/**
* A hint to the scheduler that the current thread is willing to yield
* its current use of a processor. The scheduler is free to ignore this
* hint.
*
* <p> Yield is a heuristic attempt to improve relative progression
* between threads that would otherwise over-utilise a CPU. Its use
* should be combined with detailed profiling and benchmarking to
* ensure that it actually has the desired effect.
*
* <p> It is rarely appropriate to use this method. It may be useful
* for debugging or testing purposes, where it may help to reproduce
* bugs due to race conditions. It may also be useful when designing
* concurrency control constructs such as the ones in the
* {@link java.util.concurrent.locks} package.
*/
public static native void yield();
```

## 小小总结
对一些常用的线程协同控制方法进行小小总结：
* wait()：Object类的方法，必须在synchronized同步块中调用，会让出CPU，并让出锁，不传入时间则无限等待，该对象进入等待池中，除非被notify唤醒。
* sleep()：Thread类的方法，可以在任意处调用，目的在让线程休眠，会让出CPU，但不会让出锁。
* notify()：Object类的方法，必须在synchronized同步块中调用，从等待池中随机唤醒一个线程进入锁池去竞争锁。
* notifyAll()：将等待池中所有线程唤醒，全部进入锁池竞争锁。
* yield()：暗示让出CPU的使用权，但是调度器可能会无视该暗示，并不会让出锁。
* stop()：强制停止一个线程（不推荐使用）。
* interrupt()：通知线程应该被中断。如果线程处于阻塞状态就会抛出InterruptException异常；如果线程正常运行就会将中断标志设置为true，线程运行完当前任务之后结束，保证原子性。

# 二、JUC包提供的协同控制拓展
java.util.concurrent包提供了很多并发控制工具，它们几乎可以完全替代前面介绍的基础控制方法，而且可以做到更细粒度化，甚至有些操作上面的方法不能做到，而JUC包下的类可以。由于篇幅的原因这里仅仅介绍一些类方法的作用，实际使用及原理不包含在本文中。

## 可重入锁ReentrantLock
可重入锁可以完全替代synchronized关键字。在JDK5.0的版本重入锁的性能远远超出于synchronized，但从jdk6.0开始，JDK在synchronized关键字做了大量优化（具体优化会在后面我的JVM系列文章介绍，欢迎关注哦~），使得两者性能差距不大。

ReentrantLock实现了Lock接口，lock()方法进行加锁，unlock()方法释放锁，一次原子操作完之后必须调用unlock()。

### 中断响应
对于synchronized来说，如果一个线程在等待锁，那么结果只有要么一直等要么获得锁继续执行。其中不能手动中断，而可重入锁赋予了这个能力，程序可以根据需求取消对锁的请求，某些时候这样做也可以很好地避免用锁时最不希望发生的事情——死锁。

对锁的请求，统一使用lockInterruptibly()方法。这是一个可以对中断进行相应的锁申请动作，即在等待锁的过程中，可以响应中断。
```
void lockInterruptibly() throws InterruptedException;
```

### 锁申请等待限时
除了中断响应可以避免死锁外还可以使用可重入锁的tryLock()方法，这是一个对申请锁时间进行限制的方法，在限定时间内会自旋式地重复申请锁，直到申请成功返回true，当超出限定时间还未获取到锁就会返回false。当然如果不传入限制时间则只会尝试申请一次，如果这一次未申请成功就返回false。
```
boolean tryLock();
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
```

### 实现公平锁
synchronized锁抢占是随机的，哪个线程抢占到锁就具有锁的使用权，而这个抢占过程是不公平的，也就是说有可能造成某个线程一直抢不过其他线程，而一直处于阻塞状态，那么这个过程就是不公平的获取锁。为了不让这种现象发生可以使用公平锁让每个线程都有机会获得锁，而且是公平的。

在ReentrantLock实例化时可传入一个boolean类型的参数fair，它就代表是否实现公平锁，传入true就表示公平锁，false代表非公平锁。当然如果什么都不传就是非公平锁。
```
public ReentrantLock() {
    sync = new NonfairSync();
}

public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

ReentrantLock中还提供了其他方法可以获取锁的状态以及等待线程队列等，这里就不一一介绍了，**总而言之ReentrantLock是把锁对象化，能更细粒度地操作锁。**

## Condition
一般和ReentrantLock一起使用的还有Condition，它是一个接口，它的作用和wait()、notify()方法的作用是大致相同的。区别就是wait和notify是配合synchronized使用的，Condition是配合Lock接口使用的。

其方法含义如下：
* await()：使当前线程等待，同时释放当前锁，当其他线程调用signal()或者signalAll()时，线程会重新获得锁继续执行。或者当线程被中断时，也能跳出等待。
* awaitUninterruptibly()：该方法与await()方法基本相同，但是它不会在等待过程中响应中断。
* signal()：唤醒一个在等待的线程。**在调用前要先获得锁**
* signalAll()：唤醒所有在等待的线程。**在调用前要先获得锁**

其实在ArrayBlockingQueue源码里面就是使用的Condition来控制队列满和空时的操作限制的。

## Semaphore
信号量Semaphore为多线程协作提供了更多强大的控制方法。它是锁的扩展，无论是synchronized还是ReentrantLock，一次都只允许一个线程访问一个资源，**而信号量却可以指定多个线程同时访问一个资源**。

```
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```
在构造信号量对象时，必须指定**准入数**（同时最多允许多少个线程访问资源），同时可以指定是否公平。其主要方法如下：
```
// 尝试获得一个准入的许可。若无法获得线程就会等待，直到有线程释放一个许可或当前线程被中断
public void acquire()

// 和acquire方法类似，但是不响应中断
public void acquireUninterruptibly()

// 尝试获取一个许可，它不会进行等待，立即返回
public boolean tryAcquire()

// 限时尝试
public boolean tryAcquire(long timeout, TimeUnit unit)

// 线程访问资源结束后，释放一个许可
public void release()
```

## CountDownLatch
倒计时器CountDownLatch是一个非常实用的多线程控制工具类。它通常用来控制线程等待，可以让某一个线程等待直到倒计时结束，再开始执行。
```
public CountDownLatch(int count)
```
在实例化对象时可传入计时线程数，当一个线程完成任务时可用countDown()方法申明该项任务已完成，用await等待全部任务完成之后才会继续执行，当然该方法也可以限定等待时间。
```
public void countDown()
public void await()
public boolean await(long timeout, TimeUnit unit)
```

## CyclicBarrier
这是一个比CountDownLatch功能更强大更复杂的工具类，称之为循环栅栏。栅栏是一种障碍物，用来组织线程继续执行，要求线程在栅栏处等待。循环一词便表示了它可以反复地进行倒计时。比如，假设我们将倒计时数设为10，那么10个线程任务都完成之后，计数器就会清零，然后再计算下一批10个任务。

## ReadWriteLock
ReadWriteLock看名字就知道是一种读写锁，其实它和数据库中的锁实现非常类似，将对数据操作的锁分为两类锁：读锁和写锁。这两类锁之间遵循一定的规则：
* 读-读不互斥
* 读-写互斥
* 写-写互斥
了解数据库中锁的同学就很容易理解了，这里因为篇幅原因就不过多地介绍，简单的把获取锁的伪代码放出来吧：
```
ReadWriteLock readWriteLock = new ReentrantReadWriteLock();

Lock readLock = readWriteLock.readLock();
Lock writeLock = readWriteLock.writeLock();
```

OK，对于线程协同控制基本上就介绍这些了，如果能熟练地运用上面的这些方法，相信你对多线程的理解又会进入更深的层次。