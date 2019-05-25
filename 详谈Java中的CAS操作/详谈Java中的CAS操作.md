# 问题引入
我们知道，在高并发的环境下如果要操作一个数，保证数据的正确性是我们首要关心的，要达到这个目的就需要满足操作的原子性、有序性、可见性，相信很多人看到这三个特点首先想到的就是使用synchronized，它不正好满足条件吗？

synchronized固然可以满足，但是再仔细想想，在高并发处理时，如果只是需要对一个数进行加1操作就需要对其加锁，整个流程要经历等待锁、申请锁、操作数（读+写）、销毁锁，我相信这样去做完全没有必要。于是JDK提供了一系列原子操作类：AtomicInteger、AtomicLong、AtomicBoolean、AtomicReference等，它们都是基于CAS去实现的，下面我们就来详细看一看原子操作类。

# value++操作是原子的吗？
我们平时喜欢使用的*i++*操作可以用在并发环境下吗？答案是**不可以**，因为它不是原子操作，就算配合volatile使用让其线程间可见也是不行的，并发数量一多就很容易出现问题，下面用一段简单的代码来验证一下。

> 什么是原子操作？
>
> 所谓原子操作是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间不会有任何context switch （切换到另一个线程）。简单粗暴的说就是对对象的操作只有一次，拿+1这种场景来说，直接对原值+1就是原子操作，如果是先获取原来数值的值然后再+1就不是原子操作。

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E7%BA%BF%E7%A8%8B%E6%B1%A0/20190514224938.jpg)

按正常情况来讲最后value的值应该是1000000，但是实际运行得出的结果却是**995932**，它是小于1000000的，我们可以推断*value++*的过程应该是先获取value值然后再执行++，为了验证我们使用*javap -c*命令编译出该段代码的字节码来看看，因为字节码文件较长，这里只关注最关键的add()方法部分

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E7%BA%BF%E7%A8%8B%E6%B1%A0/20190514230444.jpg)

图中红色框中的部分就是对value值的操作，可以看到它的步骤是：①获取value的值；②然后入栈；③+1操作；④写入value值。那么现在就可以解释为什么实际运行结果是小于理论值1000000的，在很多的线程中，某一时刻存在两个或多个线程同时获取到value的值，也就是说此时每个线程value值都是一样的，都进行加一之后再写入value值，那么实际的效果只是加了一次1，而却有两个或多个线程去操作了，所以最后结果是小于理想值的。

# AtomicInteger在并发环境的表现
在上面的情景下我们使用AtomicInteger来代替*++*操作看看效果如何。

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E7%BA%BF%E7%A8%8B%E6%B1%A0/20190514231339.jpg)

最后控制台出现了理想的1000000，并且为了避免偶然性，笔者反复运行多次均是这个结果。

# AtomicInteger介绍
AtomicInteger是concurrent包下的atomic包的一个类，在该包中还提供了很多其他的原子操作类，比如AtomicInteger、AtomicLong、AtomicBoolean、AtomicReference等，这里介绍以AtomicInteger为例。你大可把它就看做是一个int类型的数，只是这个类赋予了它一些“安全”操作的能力而已，它的构造函数如下：
```
public AtomicInteger()  //  默认value值为0
public AtomicInteger(int initialValue)  //  设置初始值
```

在类中提供了很多操作方法（不包含jdk8新增的函数式方法）：
```
//  取得当前值
public final int get()  

//  设置当前值
public final void set(int newValue) 
//  设置新值并返回旧值
public final int getAndSet(int newValue)

//  如果当前值为expect，则设置为update
public final boolean compareAndSet(int expect, int update)

//  如果当前值为expect，则设置为update，可能失败，不提供保障
public final boolean weakCompareAndSet(int expect, int update)

//  当前值加1，返回旧值 
public final int getAndIncrement()

//  当前值减1，返回旧值
public final int getAndDecrement()

//  当前值加delta，返回旧值
public final int getAndAdd(int delta)

//  当前值加1，返回新值
public final int incrementAndGet()

//  当前值减1，返回新值
public final int decrementAndGet()

//  当前值加delta，返回新值
public final int addAndGet(int delta)
```

# AtomicInteger如何实现原子操作？
所有Atomic相关类的实现都是通过**CAS**（Compare And Swap）去实现的，它是一种乐观锁的实现。对于乐观锁来说，总是会把事情往乐观的方向想，他们认为所有事情总是不太容易发生问题，出错几率很小。当然与之相反的就是悲观锁，也就是synchronized锁，它总是很严谨，认为出错是一种常态，所以无论大小，都考虑的很全面，不允许一点错误发生。

CAS技术就是乐观锁的一种形式，Compare And Swap顾名思义比较交换，它会比较操作之前的值和预期的值是否一致，一致才进行操作，否则什么都不做，然后循环去CAS。它是放在Unsafe这个类中的，这个类是不允许更改的，而且也不建议开发者调用，它只是用于JDK内部调用，看名字就知道它是不安全的，因为它是直接操作内存，稍不注意就可能把内存写崩，其内部大部分是native方法。

CAS的过程是这样的：它包含三个参数CAS(V,E,N)。V表示要更新的变量，E表示预期值，N表示新值。仅当V值等于E值时，才会对V的值设为N，否则当前线程什么都不做。
```
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

我们去看AtomicInteger的内部实现可以发现，全是调用的Unsafe类中的方法

![](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E7%BA%BF%E7%A8%8B%E6%B1%A0/20190515001417.jpg)

与锁想比，使用CAS会使程序看起来更复杂一些，但由于其非阻塞性，它对死锁又天生免疫，并且线程间的相互影响也远远比基于锁的方式要小。更为重要的是，使用无锁的方式完全没有锁竞争带来的系统开销，也没有线程频繁调度带来的开销，因此，它要比基于锁的方式拥有更优越的性能。

# 简单CAS操作的弊端
我们可以设想一个场景：你要向银行卡中存入1000元钱，在存之前有2000，存之后应该是3000元。如果在存之前确认了是1000元，好没问题，于是你开始存钱，恰恰在存的过程中出现了另外一个人因为操作失误向你的账号转入了500元，在很短时间内又联系银行工作人员将这500转回，此时你存入1000之后仍然是3000元，但是，你并不知道中间有这500元转入和转出的过程。这种情况在之前所说的AtomicInteger等简单原子操作来说是极有可能发生的，而且是很危险的。

下面笔者引用《Java高并发程序设计》一书中提供的贵宾卡充值消费的场景来给大家演示。

场景：如果有一家蛋糕店，为了挽留客户，决定为贵宾卡里余额小于20的客户一次性赠送20元，刺激消费者充值和消费。但条件是每一位客户只能被赠送一次。
```
package thread;

import java.util.concurrent.atomic.AtomicReference;

/**
 * @author beifengtz
 * <a href='http://www.beifengtz.com'>www.beifengtz.com</a>
 * <p>location: thread.javase_learning</p>
 * Created in 0:33 2019/5/15
 */
public class AtomicTest {
    static AtomicReference<Integer> money = new AtomicReference<>();
    public static void main(String[] args) {
        //  设置账户初始值小于20，这是一个需要被充值的客户
        money.set(19);

        // 模拟多个线程同时更新后台数据库，为用户充值
        for (int i=0;i<3;i++){
            new Thread(){
                public void run(){
                    while(true){
                        while (true){
                            Integer m =money.get();
                            if (m<20){
                                if (money.compareAndSet(m,m+20)){
                                    System.out.println("余额小于20，充值成功，余额："+money.get()+"元");
                                    break;
                                }
                            }else{
                                //  余额大于20，无需充值
                                break;
                            }
                        }
                    }
                }
            }.start();
        }

        //  用户消费线程，模拟消费行为
        new Thread(){
            public void run(){
                for (int i=0;i<100;i++){
                    while (true){
                        Integer m =money.get();
                        if (m>10){
                            System.out.println("大于10元");
                            if (money.compareAndSet(m,m-10)){
                                System.out.println("消费10元，余额："+money.get()+"元");
                                break;
                            }
                        }else {
                            System.out.println("没有足够的金额");
                            break;
                        }
                    }
                    try{
                        Thread.sleep(100);
                    }catch (InterruptedException e){
                        e.printStackTrace();
                    }
                }
            }
        }.start();
    }
}
```
最后运行的结果如下：
```
余额小于20，充值成功，余额：39元
大于10元
消费10元，余额：29元
大于10元
消费10元，余额：19元
余额小于20，充值成功，余额：39元
大于10元
消费10元，余额：29元
大于10元
消费10元，余额：19元
余额小于20，充值成功，余额：39元
大于10元
消费10元，余额：29元
大于10元
消费10元，余额：19元
余额小于20，充值成功，余额：39元

......一直循环
```
可以看到这个账户被先后反复多次充值，其原因正是因为账户余额被反复修改，修改后的值等于原来的值，是的CAS操作无法正确判断当前的数据状态。

# 带时间戳的CAS操作类AtomicStampedeReference

为了解决这种问题，JDK提供了一个带有时间戳的CAS操作类AtomicStampedeReference，它内部不仅维护了对象的值，还维护了一个时间戳，当AtomicStampedeReference对应的值被修改时，除了更新数据本身外，还必须更新时间戳，当AtomicStampedeReference设置对象值时，对象值以及时间戳都必须满足期望，写入才会成功，因此即使对象值被反复读写，写会原值，只要时间戳发生变化，就能防止不恰当的写入。

下面使用AtomicStampedeReference来修改上面的贵宾卡充值的问题吧：

```
package thread;

import java.util.concurrent.atomic.AtomicStampedReference;

/**
 * @author beifengtz
 * <a href='http://www.beifengtz.com'>www.beifengtz.com</a>
 * <p>location: thread.javase_learning</p>
 * Created in 0:33 2019/5/15
 */
public class AtomicTest {
    static AtomicStampedReference<Integer> money = new AtomicStampedReference<>(19,0);
    public static void main(String[] args) {

        // 模拟多个线程同时更新后台数据库，为用户充值
        for (int i=0;i<3;i++){
            final int timestamp = money.getStamp();
            new Thread(){
                public void run(){
                    while(true){
                        while (true){
                            Integer m =money.getReference();
                            if (m<20){
                                if (money.compareAndSet(m,m+20,timestamp,timestamp+1)){
                                    System.out.println("余额小于20，充值成功，余额："+money.getReference()+"元");
                                    break;
                                }
                            }else{
                                //  余额大于20，无需充值
                                break;
                            }
                        }
                    }
                }
            }.start();
        }

        //  用户消费线程，模拟消费行为
        new Thread(){
            public void run(){
                for (int i=0;i<100;i++){
                    while (true){
                        int timeStamp = money.getStamp();
                        Integer m =money.getReference();
                        if (m>10){
                            System.out.println("大于10元");
                            if (money.compareAndSet(m,m-10,timeStamp,timeStamp+1)){
                                System.out.println("消费10元，余额："+money.getReference()+"元");
                                break;
                            }
                        }else {
                            System.out.println("没有足够的金额");
                            break;
                        }
                    }
                    try{
                        Thread.sleep(100);
                    }catch (InterruptedException e){
                        e.printStackTrace();
                    }
                }
            }
        }.start();
    }
}
```

此时运行结果：
```
余额小于20，充值成功，余额：39元
大于10元
消费10元，余额：29元
大于10元
消费10元，余额：19元
大于10元
消费10元，余额：9元
没有足够的金额
没有足够的金额
没有足够的金额
```

可以看到账户只被赠送了一次，达到了所要的需求。