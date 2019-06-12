面试Java必定会问到SE部分的基础知识，我也被问过很多次，这篇文章记录一些常问的问题和答案。

# 一、理解JDK、JRE、JVM

* **JDK（Java Development Kit）：**Java开发工具包，是整个Java开发的核心，其中包含了JRE，即Java运行时环境，拥有编译器和工具（javadoc、jdb）等。如果是开发Java程序只需安装JDK即可。

* **JRE（Java Runtime Environment）：**Java运行时环境，其中包含了JVM标准实现、Java类库和一些基础构件。JRE适用于运行Java程序，而不能创建和开发Java程序，但是如果运行的程序含有需编译的程序（例如JSP需转换为Servlet）就需要安装jdk。

* **JVM（Java Virtual Machine）：**Java虚拟机，它能够将 class 文件中的字节码指令进行识别成机器码并调用操作系统上的 API 完成动作。JVM有针对不同操作系统的具体实现，这是Java跨平台的关键所在。

所有Java开发和运行环境不一定都是同一个厂商提供的，大部分是Sun公司（现被Oracle收购）提供的，JDK、JRE、JVM等都有不同的实现。比如JDK有Oracle JDK、Open JDK以及其他公司提供的JDK等，JVM有Sun HotSpot VM、IBM J9 VM、Google Android Dalvik VM以及其他VM等。一般我们使用的是Oracle JDK + HotSpot VM。

# 二、重载和重写

* **重载（Overload）：**发生在同一个类中，方法名必须相同，参数类型、参数个数、参数顺序、返回值、访问权限修饰符可以不同。

* **重写（Override）：**发生在父子类中，方法名、参数必须相同，返回值必须是父类返回值或者其子类，异常必须是父类异常或其子类，访问修饰符权限必须大于等于父类，除了private，被private修饰的方法无法被重写。

私有（private）方法和构造方法无法被重写，但是可以被重载，一个类可以有多个被重载的构造方法。

# 三、Java的三大特性

* **封装：**提供访问权限修饰符来控制属性和方法的访问可见性，Java中有四大访问修饰符：private、default、protect、public。其中特别注意：外部类可以访问内部类的private/protected变量，在编译时，外部类和内部类不再是嵌套结构，而是变为一个包中的两个类，然后对于private变量的访问，编译器会生成一个accessor函数。
    * private：同一个类可访问
    * default：同一个类和同一个包可访问
    * protect：同一个类、同一个包、子类可访问
    * public：同一个类、同一个包、子类、其他包可访问

* **继承：**子类继承父类，子类拥有父类所有的属性和方法，但是只能够访问非private的属性和方法，子类的属性和方法对父类不可见，子类可以重写父类的非private和非final的方法。在开发中合理使用继承可以方便地复用代码，重构时继承提供了很大的方便。

* **多态：**程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量倒底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。多态有三种实现形式：
    * 继承：在多态中必须存在有继承关系的子类和父类
    * 重写/实现接口：子类对父类中某些方法进行重新定义，在调用这些方法时就会调用子类的方法
    * 向上转型：在多态中需要将子类的引用赋给父类对象，只有这样该引用才能够具备技能调用父类的方法和子类的方法

# 四、接口（interface）和抽象类（abstract）的区别

* 抽象类可以有构造方法，接口中不能有构造方法。
* 接口中的抽象方法默认是public，但也只能是public，抽象方法不能具体实现，jdk8之后可以有default方法实现，而抽象类既可以有抽象方法又可以有非抽象方法。从某种意义上讲接口是一种特殊的抽象类。
* 抽象类中可以包含静态方法，在 JDK1.8 之前接口中不能不包含静态方法，JDK1.8 以后可以包含。
* 接口的实例变量默认是final类型，不可修改，而抽象类不一定。
* 一个类可以实现多个接口，但只能继承一个抽象类，接口不可以实现接口，但可以继承接口，并且可以继承多个接口。
* 接口不能用new实例化，但是可以声明变量，其变量必须引用该接口实例化的一个对象。
    
# 五、==和equals区别

**==**：如果比较的是基本数据类型，判断其值是否相等；如果比较的是引用类型，判断两个对象的地址是否相等。

**equals**：equals是在Object类中定义的方法，在Object类中仅比较两个对象的地址是否相同。

```java
public boolean equals (Object x){
    return this == x;
}
```

大家都知道所有类都是Object的子类，所以可以选择是否重写equals方法，以String类的equals方法为例，equals方法用于判断字符串的值是否相同。

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

在重写equals方法时需要满足几点规则：
* 自反性。对于任何非null的引用值x，x.equals(x)应返回true。
* 对称性。对于任何非null的引用值x与y，当且仅当：y.equals(x)返回true时，x.equals(y)才返回true。
* 传递性。对于任何非null的引用值x、y与z，如果y.equals(x)返回true，y.equals(z)返回true，那么x.equals(z)也应返回true。
* 一致性。对于任何非null的引用值x与y，假设对象上equals比较中的信息没有被修改，则多次调用x.equals(y)始终返回true或者始终返回false。
* 对于任何非空引用值x，x.equal(null)应返回false。

equals方法用于判断两个对象在实际意义上是否是同一个对象，比如有两张照片判断其中的人是否是同一个人，虽然两张中的穿着、所在环境都不一样，但是在实际意义上是同一个人。equals方法常常和hashCode()一起使用。

# 六、hashCode和equals

equals方法上面有介绍，hashCode()定义于Object类中，该方法用于获取哈希散列码，它返回一个int类型的值，哈希散列码的作用是确定该对象在哈希表中的索引位置，目的是为了支持Map接口。在Object类中hashCode是一个native方法，它是由在虚拟机堆的位置唯一确定，一般在重写该方法时需要自己定义其中的算法。

**重写equals时必须重写hashCode方法？**

其实是不一定的，网上很多文章都说必须同时重写，这是建立在设计合理的基础上。如果一个类不涉及HashSet、Hashtable、HashMap等内部使用哈希表的数据结构的类时，可以不必重写hashCode方法，因为如果不涉及哈希表hashCode就毫无意义。但是在实际编码时又要求同时重写，因为你无法预测该类是否会应用到含有哈希表的类，所以通常会有“重写equals时必须重写hashCode方法”的说法。

* 两个对象相等，则hashCode一定也是相等的；
* 两个对象相等，互相调用equals方法也都返回true；
* 两个对象有相同的hashCode值，它们互相调用equals也不一定返回true（可能发生Hash碰撞）。

# 七、String、StringBuilder、StringBuffer区别

**可变性**

String类不可变，它每次申请固定长度的char字符数组`final char value[]`，并且不可修改，平时所使用的+号字符串拼接实际上是开辟了多个内存空间，最后结果字符串的堆内存可用，其余的空间全部成为垃圾，读者可阅读我曾经写的一篇文章了解：[Java中String对象最容易被忽略的知识](http://blog.beifengtz.com/article/11)。

StringBuffer和StringBuilder都是可变型字符串类，它们都继承自`AbstractStringBuilder`类，其中的字符数组定义是可变的`char[] value`，在其中每次字符串拼接如果容量充足就在当前堆内存改变，如果不足才开辟新的空间，其中每次扩容是原来容量的2倍+2，源码中是这样实现的：`(value.length << 1) + 2`，最大容量是`Integer.MAX_VALUE - 8`，为什么减8呢？因为对象头需要占用一定空间，实际占用大小因虚拟机位数而定。

**多线程安全**

String和StringBuffer是多线程安全的，String的字符数组是final的，所以它不存在修改也就天然线程安全，而StringBuffer则是通过同步锁实现线程安全的，它的所有方法都是使用的synchronized修饰保证其线程安全性。而StringBuilder则是非线程安全的。

**适用条件**

当字符串拼接很少时适合String类。当字符串拼接很频繁时，如果仅在单线程操作变量，适合StringBuilder；如果在多线程情况下，使用StringBuffer能更好保证其安全性。在单线程情况下，StringBuilder相比于StringBuffer有15%左右的性能提升。

# 八、"abc"与new String("abc")

通常创建字符串有两种方法，一种是直接使用双引号创建`"abc"`，一种是new一个String类。两种方法都能创建字符串，但其流程却有所差别，详细内容可阅读这篇文章：[Java中String对象最容易被忽略的知识](http://blog.beifengtz.com/article/11)。

这两种方法涉及到String的intern方法实现，在jdk6和jdk6以后具体实现有所差别，这里只讲jdk6以后的实现。

* 双引号创建会先检查常量池是否存在该字符串，如果常量池有则直接返回常量池的引用，如果没有则检查该字符串是否存在于堆中，如果存在则将堆中对此对象的引用添加到常量池中，并返回该引用，如果堆中不存在，就在池中创建字符串并返回其引用。
* new一个String类是直接在堆内存中创建一个新对象，但是构造函数传入的字符串又是一个String对象，如果对象池中没有这个字符串就会在堆内存中多一块垃圾，所以平常使用时推荐使用第一种双引号创建。

# 九、构造函数、构造代码块、静态代码块

先看一下这三个在代码中的样子
```java
public class Test1 {
    Test1() {
        System.out.println("构造函数");
    }

    {
        System.out.println("构造代码块");
    }

    static {
        System.out.println("静态代码块");
    }

    public static void main(String[] args) {
        System.out.println("main函数执行");
        new Test1();
    }
}
```
上面代码的运行结果是：
```
静态代码块
main函数执行
构造代码块
构造函数
```

* **构造函数**
    * 对象一建立，就会调用与之相应的构造函数，不实例化对象，构造函数不会运行。
    * 构造函数的作用是用于给对象进行初始化。
    * 一个对象建立，构造函数只运行一次，而一般方法可以被该对象调用多次。

* **构造代码块**
    * 构造代码块的作用是给对象进行初始化。
    * 对象一建立就运行构造代码块，而且**优先于构造函数执行**。有对象实例化，才会运行构造代码块，类不能调用构造代码块。
    * 构造代码块与构造函数的**区别**是：构造代码块是给所有对象进行统一初始化，而构造函数是给对应的对象初始化，因为构造函数是可以多个的，运行哪个构造函数就会建立什么样的对象，**但无论建立哪个对象，都会先执行相同的构造代码块**。也就是说，构造代码块中定义的是不同对象共性的初始化内容。

* **静态代码块**
    * 它是随着类的加载而执行，只执行一次，并优先于主函数。该过程发生在类加载生命周期的**初始化阶段**读者可以阅读我之前写的这篇文章[浅谈一个Java类的生命周期](https://mp.weixin.qq.com/s?__biz=MzU3OTkyMDAxNg==&mid=2247483993&idx=1&sn=c5702da78bfe7204a1fabc4e2c8f5448&chksm=fd5f8ba6ca2802b019e081a15d5bd844393c01e4108ba10cbc4c39b8209103f61c7305d06dfc&token=5331431&lang=zh_CN#rd)。
    * 静态代码块是给类初始化，构造代码块是给对象初始化。
    * 静态代码块中的变量是局部变量，与普通函数中的局部变量性质没有区别。
    * 一个类中可以有多个静态代码块

# 十、final、finally、finalize区别

* **final：**Java关键字，可以用来修饰类、方法、变量，分别有不同的意义，final修饰的class代表不可以继承扩展，final的变量是不可以修改的，而final的方法也是不可以重写的（override）。
* **finally：** Java保证重点代码一定要被执行的一种机制。我们可以使用`try-finally`或者`try-catch-finally`来进行类似关闭 JDBC连接、保证unlock锁等动作。
* **finalize：**基础类Object的一个方法，它的设计目的是保证对象在被垃圾收集前完成特定资源的回收。finalize机制现在已经不推荐使用，并且在JDK9开始被标记为deprecated。

# 十一、Java中的值传递与引用传递

因为这部分知识容易饶，所以我将结合代码描述。

**值传递**

方法传递对象是基本数据类型，方法得到的是参数值的拷贝，无论该方法对其传递变量做什么样的修改，其原本的值均不会改变，因为方法体操作的是拷贝的数据。
```java
public static void main(String[] args) {
    int a = 1, b = 2;
    change(a, b);
    System.out.println("a = " + a + ",b = " + b);
    // 运行结果是:   a = 1,b = 2
}

static void change(int a, int b) {
    a = 3;
    b = 4;
}
```

**引用传递**

引用传递的对象是引用数据类型或数组类型，方法得到的是对象的堆内存地址，**方法可以改变堆内存中对象的内容**，但是它和值传递有一点很容易弄混淆，我相信看下面的代码就不会混淆了。
```java
static class User{
    String name;
    User(String name){
        this.name = name;
    }
}

public static void main(String[] args) {
    User user1 = new User("北风");
    User user2 = new User("tz");

    swap(user1,user2);
    //  该方法是交换user1和user2的堆内存，结果明显是会失败的，
    //  因为它们两个的栈内存并没有改变，仍然指向的是原来的堆内存
    System.out.println("user1:"+user1.name+"; user2:"+user2.name);
    //  运行结果是：    user1:北风; user2:tz

    change(user1,user2);
    //  该方法是改变user1和user2的name属性，会成功，
    //  因为引用传递能修改堆内存的内容
    System.out.println("user1:"+user1.name+"; user2:"+user2.name);
    //  运行结果是：    user1:AAAAAAA; user2:BBBBBBB
}
//  交换user1和user2地址
static void swap(User user1,User user2) {
    User temp = user1;
    user1 = user2;
    user2 = temp;
}
//  修改user1和user2的内容
static void change(User user1,User user2) {
    user1.name = "AAAAAAA";
    user2.name = "BBBBBBB";
}
```

# 十二、获得一个类的实例有哪些方法

* **new对象**，使用关键字new直接在堆内存创建一个对象实例。
* **clone()**，clone()方法定义于Object类，用于从堆内存中克隆一个一模一样的对象到新的堆内存中，被克隆的对象必须实现Cloneable接口，该接口无任何实际定义，仅用于标识。
* **反射**，在运行状态中对任意一个类进行实例化，并且可以调用其所有属性和方法，甚至可以打破访问控制权限的规则，即private定义也可以被访问。忧点是动态加载类，可以提高代码灵活度。缺点是容易造成性能瓶颈，类解释过程交由JVM去做，增加JVM负担。
* **反序列化ObjectInputStream**，，将二进制流转换为类对象，二进制流必须是由Java序列化而来。具体实现可阅读我之前写的文章[序列化与反序列化](http://blog.beifengtz.com/article/36)