
# 什么是线程池？
今天我们来聊一聊Java中的线程池，首先来看看什么是线程池。
> 线程池就是以一个或多个线程（循环执行）多个应用逻辑的线程集合.

为了避免系统频繁地创建和销毁线程，我们可以让创建的线程进行复用。其实和数据库连接池是一样的道理，为了避免每次数据库查询都重新建立和销毁数据库连接，我们可以使用数据库连接池维护一些数据库连接，让他们长期保持一个激活状态。当系统需要使用数据库时，并不是创建一个新的连接，而是从连接池中获得一个可用的连接。

线程池中总有那么几个活跃线程，当你需要使用线程时，可以从池子中拿一个空闲线程，当完成工作时，并不急着关闭线程，而是将这个线程回收入池，等待下一个任务的执行。 

# JDK支持的几种线程池
在JDK中提供了一套Executor框架，可以方便开发者很好的控制线程。

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E7%BA%BF%E7%A8%8B%E6%B1%A0/20190510094502.jpg)

JDK中提供了五类线程池可供使用，其中newWorkStealingPool是1.8之后出来的，其他是之前就有的。

* newFixedThreadPool：提供一个固定线程数量的线程池。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务则会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的队列。
* newSingleThreadExecutor：一个只有一个线程的线程池。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，该线程空闲，按先入先出的顺序执行队列中的任务。
* newCachedThreadPool：一个可根据实际情况调整线程数量的线程池。线程池的线程数量不确定，但若有空闲线程可以复用，则会优先使用可复用的线程。若所有线程均在工作，又有新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执行完毕后，将返回线程池进行复用。
* ScheduleThreadPool：
    * newSingleThreadScheduleExecutor()：返回一个ScheduleExecutorService对象，线程池大小为1.ScheduleExecutorService接口在ExecutorService接口之上扩展了在给定时间执行某任务的功能，如在某个固定的延时之后执行，或周期性执行某个任务。
    * newScheduleThreadPool()：返回一个ScheduleExecutorService对象，但线程池可以指定线程数量。
* newWorkStealingPool：基于Fork/Join框架的一种线程池，将提交的一个任务拆分成多个子任务让多个线程并行执行。如果某个线程任务先执行完，就从其他线程的任务队列的尾部窃取任务执行，保证大部分线程有任务可工作，同时提升执行效率。工作线程从任务队列的首部获取任务执行，空闲线程从任务队列的尾部窃取任务执行。

提供一个示例：
```
package thread;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPool {

    public static class MyTask implements Runnable{

        @Override
        public void run() {
            System.out.println(System.currentTimeMillis()+":Thread ID:"+Thread.currentThread().getId());
            try{
                Thread.sleep(500);
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        ExecutorService fixedThreadPool = Executors.newFixedThreadPool(5);
        for (int i=0;i<10;i++){
            fixedThreadPool.submit(new MyTask());
        }
        fixedThreadPool.shutdown();
    }
}
```

# 核心线程池的内部实现
除了ForkJoin外的其他几类线程池的核心，其实都是由一个ThreadPoolExecutor类实现的，他们的源码内部其实都调用了这个类。
```
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
    
public static ExecutorService newWorkStealingPool(int parallelism) {
        return new ForkJoinPool
            (parallelism,
             ForkJoinPool.defaultForkJoinWorkerThreadFactory,
             null, true);
    }
```

我们知道那四类线程池实际上只是知道了一个外壳，这明显还不够，我们还需要知道ThreadPoolExecutor这个类，首先来看看类的构造函数的定义：
```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler)
```

参数含义如下：
* corePoolSize：指定线程池中的线程数量。
* maximumPoolSize：指定了线程池中的最大线程数量。
* keepAliveTime：当现成池线程数量超过corePoolSize时，多余的空闲线程的存活时间。
* unit：keepAliveTime的单位。
* workQueue：任务队列，被提交但尚未被执行的任务。
* ThreadFactory：线程工厂，用于创建线程（一般默认即可）。
* handler：拒绝策略，当任务太多来不及处理，如何拒绝任务。

# 几种不同的任务队列
workQueue参数是一个BlockingQueue对象，仅用于存放Runnable对象。根据使用场景不同一般可有四种任务队列。

* SynchronousQueue：直接提交的队列。该对象是一种特殊的BlockingQueue，它没有容量，每个插入操作都要等待一个相应的删除操作，反之，每一个删除操作都要等待相应的插入操作。如果使用SnchronousQueue提交的任务不会被真实的保存，而总是将新任务提交给线程执行。使用SynchronousQueue队列，通常要设置很大的maximumPoolSize值，否则很容易执行拒绝策略。
* ArrayBlockingQueue：有界的任务队列。该类的构造函数必须带一个参数**capacity**，用于表示最大容量，从类名即可看出内部是用数组实现的，显然必须确定最大容量。有界队列仅当在任务队列装满时，才可能将线程数量提升到corePoolSize以上。
* LinedBlockingQueue：无界的任务队列。与有界队列想比，除非系统资源耗尽，否则无界的队列不存在任务入队失败的情况。其内部使用链表实现的。
* PriorityBlockingQueue：优先任务队列。可以控制任务的执行先后顺序，它是一个特殊的无界队列，ArrayBlockingQueue和LinkedBlockingQueue都是按照先进先出的算法处理任务的，而PriorityBlockingQueue则可以根据任务自身的优先级顺序先后执行，在确保系统性能的同时，也能有很好的质量保证（总是确保高优先级的任务先执行）。

# 拒绝策略
在ThreadPoolExecutor的构造函数最后一个参数指定了拒绝策略。当任务数量超过系统实际承载能力时，就会使用拒绝策略。在JDK中内置了四种拒绝策略。

* AboritPolicy策略：该策略会直接抛出异常，组织系统正常工作，也是默认的拒绝策略。
* CallerRunsPolicy策略：只要线程池未关闭，该策略直接在调用者线程中，运行当前被丢弃的任务。这样做不会真的丢弃任务，但是任务提交的仙鹤草呢个的性能极有可能会急剧下降。
* DiscardOldestPolicy策略：该策略将丢弃最老的一个请求，也就是即将被执行的一个任务，并尝试再次提交当前任务。
* DiscardPolicy策略：直接默默丢弃无法处理的任务，不予任何处理。

当然也可以自定义拒绝策略，可以实现RejectExecuntionHandler接口，该接口定义如下：
```
package java.util.concurrent;

/**
 * A handler for tasks that cannot be executed by a {@link ThreadPoolExecutor}.
 *
 * @since 1.5
 * @author Doug Lea
 */
public interface RejectedExecutionHandler {

    /**
     * Method that may be invoked by a {@link ThreadPoolExecutor} when
     * {@link ThreadPoolExecutor#execute execute} cannot accept a
     * task.  This may occur when no more threads or queue slots are
     * available because their bounds would be exceeded, or upon
     * shutdown of the Executor.
     *
     * <p>In the absence of other alternatives, the method may throw
     * an unchecked {@link RejectedExecutionException}, which will be
     * propagated to the caller of {@code execute}.
     *
     * @param r the runnable task requested to be executed
     * @param executor the executor attempting to execute this task
     * @throws RejectedExecutionException if there is no remedy
     */
    void rejectedExecution(Runnable r, ThreadPoolExecutor executor);
}
```

# 自定义线程创建
我相信大家在学习这一部分的时候会有一个和我一样的疑惑：线程池的线程是从哪里来的？

其实从ThreadPoolExecutor的构造函数中不难发现有一个ThreadFactory类的参数，ThreadFactory是一个接口
```
public interface ThreadFactory {

    /**
     * Constructs a new {@code Thread}.  Implementations may also initialize
     * priority, name, daemon status, {@code ThreadGroup}, etc.
     *
     * @param r a runnable to be executed by new thread instance
     * @return constructed thread, or {@code null} if the request to
     *         create a thread is rejected
     */
    Thread newThread(Runnable r);
}
```

当线程创建时就是用的这个这个方法去创建线程的，使用这一个方法我们可以跟踪线程池中的所有被创建的线程，以及定义其名称、优先级、组等。下面给一个简单的自定义线程示例：
```
package thread;

import java.util.concurrent.*;

public class ThreadPool {

    public static class MyTask implements Runnable{

        @Override
        public void run() {
            System.out.println(System.currentTimeMillis()+":Thread ID:"+Thread.currentThread().getId());
            try{
                Thread.sleep(500);
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        MyTask myTask = new MyTask();
        ExecutorService es = new ThreadPoolExecutor(10, 10, 0L,
                TimeUnit.MILLISECONDS, new SynchronousQueue<Runnable>(),
                new ThreadFactory() {
                    @Override
                    public Thread newThread(Runnable r) {
                        Thread t = new Thread(r);
                        System.out.println("create Thread:"+t);
                        return t;
                    }
                });
        for (int i=0;i<10;i++){
            es.submit(myTask);
        }
    }

}
```
# 线程池中的异常处理
在实际使用线程池中我们很容易遇到一些幽灵错误，没有得到理想的结果而控制台又没有任何错误信息，甚至包括一些异常都不会抛出，感觉像是异常被线程池吞并了一样。比如下面这段代码
```
package thread;

import java.util.concurrent.*;

public class ThreadPool {

    public static class MyTask implements Runnable{
        int a;
        int b;

        public MyTask(int a, int b) {
            this.a = a;
            this.b = b;
        }

        @Override
        public void run() {
            double c = a/b;
            System.out.println(c);
        }
    }

    public static void main(String[] args) throws InterruptedException{
        ExecutorService es = Executors.newCachedThreadPool() ;
        for (int i=0;i<5;i++){
            es.submit(new MyTask(100,i));
        }
        Thread.sleep(1000);
        es.shutdown();
    }

}
```
控制台输出如下：
```
50.0
100.0
33.0
25.0
```

很明显第一个除0操作会抛出一个异常，但是并没有在控制台打印出。其实有一个很好的解决方法就是把submit()方法改为execute()即可，改了之后就会得到下面的结果：
```
Exception in thread "pool-1-thread-1" java.lang.ArithmeticException: / by zero
	at thread.ThreadPool$MyTask.run(ThreadPool.java:18)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
100.0
50.0
25.0
33.0
```

这也是submit()方法和execute()方法很重要的一个区别，submit会“吞并”错误异常堆栈，而execute不会。追根问底为什么呢？其实看这两个方法定义就一目了然了
```
/**
* Submits a Runnable task for execution and returns a Future
* representing that task. The Future's {@code get} method will
* return {@code null} upon <em>successful</em> completion.
*
* @param task the task to submit
* @return a Future representing pending completion of the task
* @throws RejectedExecutionException if the task cannot be
*         scheduled for execution
* @throws NullPointerException if the task is null
*/
Future<?> submit(Runnable task);


/**
* Executes the given command at some time in the future.  The command
* may execute in a new thread, in a pooled thread, or in the calling
* thread, at the discretion of the {@code Executor} implementation.
*
* @param command the runnable task
* @throws RejectedExecutionException if this task cannot be
* accepted for execution
* @throws NullPointerException if command is null
*/
void execute(Runnable command);
```

submit是一个有结果返回的方法，并且返回对象是Future，返回结果以及异常堆栈都放到了Future中，如果不做处理我们当然看不到了，而execute没有接收异常的对象，所以会直接抛出。

如果我们将主方法这样改一下，就能看到正常的异常信息了
```
public static void main(String[] args) throws InterruptedException{
        ExecutorService es = Executors.newCachedThreadPool() ;
        Future[] futures = new Future[5];
        for (int i=0;i<5;i++){
            futures[i] = es.submit(new MyTask(100,i));
        }
        Thread.sleep(1000);
        for (Future future : futures){
            try{
                future.get();
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        es.shutdown();
    }
```

运行结果：
```
100.0
50.0
25.0
33.0
java.util.concurrent.ExecutionException: java.lang.ArithmeticException: / by zero
	at java.util.concurrent.FutureTask.report(FutureTask.java:122)
	at java.util.concurrent.FutureTask.get(FutureTask.java:192)
	at thread.ThreadPool.main(ThreadPool.java:32)
Caused by: java.lang.ArithmeticException: / by zero
	at thread.ThreadPool$MyTask.run(ThreadPool.java:18)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
```

好了，今天的线程池学习就到这里啦~