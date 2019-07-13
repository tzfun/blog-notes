# 前言
使用缓存已经是开发中老生常谈的一件事了，常用专门处理缓存的工具比如Redis、MemCache等，但是有些时候可能需要一些简单的缓存处理，没必要用上这种专门的缓存工具，那么自己写一个缓存类最合适不过了。
# 一、分析
首先分析一下缓存类该如何设计，这里我以一种非常简单的方式来实现一个缓存类，这也是我一直以来使用的设计方案。

为了明确功能，首先定义一个接口类**CacheInt**，然后是缓存实现的工具类**CacheUtil**。然后再看其中的功能，为了存取方便，缓存应是以键值对的形式存取，为了适应更多的场景，所以在存取的时候可以加一个缓存过期时间，然后再加上其他常见的添加、获取、删除、缓存大小、是否存在key、清理过期缓存等方法，整个缓存工具的方法差不多就是这些。

**缓存类需要注意的问题：**
1. 缓存对象应该是唯一的，也就是单例的；
2. 缓存的操作方法要同步，在多线程并发条件下防止出错；
3. 缓存的容器应该具有较高的并发性能，ConcurrentHashMap是一个不错的选择。

# 二、具体实现

## 1. CacheInt接口的定义

CacheInt接口的定义如下：

```java
public interface CacheInt {
    /**
     * 存入缓存，此过程始终会清除过期缓存
     * @param key 键
     * @param value 值
     * @param expire 过期时间，单位秒，如果小于1则表示长期保存
     */
    void put(String key, Object value, int expire);

    /**
     * 获取缓存，1/3的概率清除过期缓存
     * @param key 键
     * @return Object对象
     */
    Object get(String key);

    /**
     * 获取缓存大小
     * @return int
     */
    int size();

    /**
     * 是否存在缓存
     * @param key 键
     * @return boolean
     */
    boolean isContains(String key);

    /**
     * 获取所有缓存的键
     * @return Set集合
     */
    Set<String> getAllKeys();

    /**
     * 删除某个缓存
     * @param key 键
     */
    void remove(String key);
}
```

## 2. CacheUtil的具体实现

缓存实现的核心就是CacheUtil，下面结合注释进行说明，为了避免文章篇幅冗杂，以下截图就是完整源码截图，并且保持先后顺序。

首先是类定义和其属性定义，其中本类实例对象用*volatile*进行修饰提高可见性，初始化缓存容量用于初始化ConcurrentHashMap缓存容器的大小，**此大小根据实际应用场景进行优化**。

```java
import java.util.Iterator;
import java.util.Map;
import java.util.Random;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

/**
 * @author beifengtz
 * <a href='http://www.beifengtz.com'>www.beifengtz.com</a>
 * Created in 13:24 2019/7/13
 */
public class CacheUtil implements CacheInt{

    //  本类实例对象，单例
    private static volatile CacheUtil instance;

    //  初始化缓存容量
    private static final int CACHE_INITIAL_SIZE = 8;

    //  缓存容器定义
    private static ConcurrentHashMap<String, Entry> cacheMap;

    //  然后是内部类Entry的定义，该类是用来存储实际数据的，为了方便处理过期时间，添加初始化时间戳、过期时间等属性。
    //  存储内容的结构定义
    static class Entry {
        long initTime;//    存储时间
        int expire; //  单位：秒
        Object data;//  具体数据

        Entry(long initTime, int expire, Object data) {
            this.initTime = initTime;
            this.expire = expire;
            this.data = data;
        }
    }

    //  然后是使用双检锁单例方式获取本类实例对象，因为单例只能存在唯一的特点，所以注意构造函数需要设为private
    private CacheUtil() {
        cacheMap = new ConcurrentHashMap<>(CACHE_INITIAL_SIZE);
    }

    public static CacheUtil getInstance() {
        if (instance == null) {
            synchronized (CacheUtil.class) {
                if (instance == null) {
                    instance = new CacheUtil();
                }
                return instance;
            }
        }
        return instance;
    }

    //  接下来是存入缓存数据`put()`方法，
    //  这里的`clearExpiredCache()`是清理过期缓存，
    //  后面会看到方法体，因为在我项目中存入缓存的情况较少，
    //  所以这里我固定了每次存之前先清理一次过期时间缓存，
    //  这里可以根据自己项目实际情况进行优化。
    public synchronized void put(String key, Object value, int expire) {
        clearExpiredCache();

        Entry entry = new Entry(System.currentTimeMillis(), expire, value);
        cacheMap.put(key, entry);
    }

    //  然后是获取缓存`get()`方法，因为获取数据的时间较为多数，
    //  所以这里我设定了三分之一的概率清理过期缓存，适当地释放堆内存，
    //  并且在获取时检测是否过期，如果已过期然而还获取到了，就删除并返回空。
    public synchronized Object get(String key) {

        //  构造三分之一的几率清除过期缓存
        if(new Random().nextInt(12) > 8){
            clearExpiredCache();
        }

        if (cacheMap.containsKey(key)) {
            Entry entry = cacheMap.get(key);
            if (entry.expire > 0 && System.currentTimeMillis() > entry.expire * 1000 + entry.initTime) {
                cacheMap.remove(key);
                return null;
            } else {
                return entry.data;
            }
        } else {
            return null;
        }
    }

    //  然后就是比较常规的一些方法，具体可以看代码

    public int size() {
        return cacheMap.size();
    }

    @Override
    public boolean isContains(String key) {
        return cacheMap.containsKey(key);
    }

    @Override
    public Set<String> getAllKeys() {
        return cacheMap.keySet();
    }

    @Override
    public void remove(String key) {
        cacheMap.remove(key);
    }

    //  最后一个方法就是清理过期缓存，这里你可以选择启动一个监听
    //  线程实时地清理缓存，也可以选择在适当时机进行一次清理，
    //  比如我这里就是在存在put和get操作时固定或概率地清理缓存。
    private synchronized void clearExpiredCache() {
        Iterator<Map.Entry<String, Entry>> iterator = cacheMap.entrySet().iterator();
        while (iterator.hasNext()) {
            Map.Entry<String, Entry> entry = iterator.next();
            if (entry.getValue().expire > 0 &&
                    System.currentTimeMillis() > entry.getValue().expire * 1000 + entry.getValue().initTime) {
                iterator.remove();
            }
        }
    }
}
```

# 三、并发测试
普通的实现测试这里就不展示了，肯定是没问题的，读者简单写一些测试样例即可，这里主要展示一下并发测试，因为在实际情况中存在并发处理缓存情况，为了确保其正确性，所以并发测试是必须要做的，下面放出我的测试样例。

```java
@Test
public void concurrentCacheTest() {

    final int LOOP_TIMES = 1000;//  循环次数

    //  线程池，启动10个子线程进行处理
    ExecutorService es = Executors.newFixedThreadPool(10);

    //  存放随机生成的key
    ConcurrentLinkedQueue<String> clq = new ConcurrentLinkedQueue<>();

    //  定义两个计数器，用于计量两次并发过程
    CountDownLatch count1 = new CountDownLatch(LOOP_TIMES);
    CountDownLatch count2 = new CountDownLatch(LOOP_TIMES);

    //  缓存操作过程的计数
    AtomicInteger atomicInteger = new AtomicInteger(0);

    //  测试并发情况下put表现
    for (int i = 0; i < LOOP_TIMES; i++) {
        es.execute(new Runnable() {
            @Override
            public void run() {
                String key = String.valueOf(new Random().nextInt(1000));
                Object value = new Random().nextInt(1000);
                int expire = new Random().nextInt(100);
                clq.add(key);
                cacheUtil.put(key, value, expire);
                System.out.println(atomicInteger.incrementAndGet() +
                        ".存入缓存成功,key=" + key +
                        ",value=" + value +
                        ",expire=" + expire);
                count1.countDown();
            }
        });
    }

    try {
        count1.await();//   等待所有的put执行完
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    //  测试并发情况下的get表现
    atomicInteger.set(0);
    for (int i = 0; i < LOOP_TIMES; i++) {
        es.execute(new Runnable() {
            @Override
            public void run() {
                try {
                    //  随机等待时间
                    Thread.sleep(new Random().nextInt(100));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                String key = clq.poll();
                System.out.println(atomicInteger.incrementAndGet() +
                        ".从缓存中获取key=" + key +
                        "的值：" + cacheUtil.get(key));
                count2.countDown();
            }
        });
    }
    try {
        count2.await();//   等待所有的get执行完
    } catch (InterruptedException e) {
        e.printStackTrace();
    }

    es.shutdown();
    while (true){
        if (es.isTerminated()){
            System.out.println("所有任务均执行完");
            System.out.println("缓存大小：" + cacheUtil.size());
            return;
        }
    }
}
```

**最后测试的表现是很好，没有出现不正确的情况，部分测试结果截图如下：**

![](https://s2.ax1x.com/2019/07/13/Z4vjoT.png)

![](https://s2.ax1x.com/2019/07/13/Z4vzYF.png)

![](https://s2.ax1x.com/2019/07/13/Z4vxFU.png)

# 四、拓展

该类只是简单的实现了缓存的过程，但是在实际应用中不见得能很好地表现，首先它的容量肯定有限，不能存太多缓存，因为使用的是JVM堆内的内存，优化的话可以使用直接内存进行存储，其次其功能也较为简单，比如不支持LRU淘汰等，这个可以用双链表+Map或者是LinkedHashMap去实现，更多功能都可以拓展。