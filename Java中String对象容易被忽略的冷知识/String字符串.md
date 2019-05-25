# String类的两种定义方法
String是一个字符串类型的类，使用`""`定义的内容都是字符串，但是String在使用上有一点特殊，它有两种定义方式，相信所有java程序员都知道，但是有些细节却很容易被忽略，我们接下来从内存关系上来分析一下。


## 直接赋值（匿名类）
相信很多人在初学程序的时候都写过`hello word！`，它是一个字符串，那么我们通过第一种直接赋值的方式来定义一个`hello world！`
```
public class StringTest {
    public static void main(String args[]){
        String str = "hello world!";    //直接赋值
        System.out.println(str);
    }
}
```
运行结果图
```
hello world!
```
通过这样一个代码String类对象已经实例化了，并且其中有内容，就是`hello world！`。

## new实例化（构造方法）
String对象也是可以通过关键字`new`来进行实例化的，接下来我们看个简单的例子。
```
public class StringTest {
    public static void main(String args[]){
        String str = new String("hello world!");
        System.out.println(str);
    }
}
```
运行结果图
```
hello world!
```

不难看出来这个是通过构造方法来给String对象赋值的，在String类中的构造方法是这样写的：
```
/**
 * Initializes a newly created {@code String} object so that it represents
 * the same sequence of characters as the argument; in other words, the
 * newly created string is a copy of the argument string. Unless an
 * explicit copy of {@code original} is needed, use of this constructor is
 * unnecessary since Strings are immutable.
 *
 * @param  original
 *         A {@code String}
 */
public String(String original) {
    this.value = original.value;
    this.hash = original.hash;
}
```
对于哈希我们后面再仔细解读，先看这个value，可以看出String对象的属性有一个是value，当通过构造函数传入一个字符串时该对象的value将被赋值，并且构造方法传入的对象也是String类，相当于自己作为参数传进去，这样的做法在java中是允许的，那么传进去的String又是哪儿来的呢？我们继续来看。

在解决这个问题之前我们先看一下字符串比较，通过这个来引入。
# 字符串比较
我们来看一下代码
```
public class StringTest {
    public static void main(String args[]){
        String str1 = "hello";
        String str2 = new String("hello");
        String str3 = str2;                 //引用传递
        System.out.println(str1 == str2);
        System.out.println(str2 == str3);
        System.out.println(str3 == str1);
    }
}
```
运行结果
```
false
true
false
```

以上三个String类对象的内容完全一样，但是结果有的是true有的是false，原因就是在java中String类的比较用`==`并不是比较其内容，而是比较其所在堆内存中的地址值，并非比较其数值。我们来分析一下内存关系图
![20180914185657.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180914185657.jpg)

我们通过最后一个内存图可以看出str1和str2指向的地址不一样，所以第一个输出false；str2和str3指向的地址都是`XO0020`，所以第二个输出true，str3和str1指向的地址不一样，所以第三个输出false。

如果在String中想比较大小要用到String类中的`equals()`方法，该方法比较的就是对象中所存的值。
```
public class StringTest {
    public static void main(String args[]){
        String str1 = "hello";
        String str2 = new String("hello");
        String str3 = str2;                 //引用传递
        System.out.println(str1.equals(str2));
        System.out.println(str2.equals(str3));
        System.out.println(str3.equals(str1));
    }
}
```
运行结果
```
true
true
true
```
> 在平时使用的时候很容易对这两个搞混淆,一般使用eauals的会比较多。

不难看出在字符串比较时有比较内存地址和内容值之分，回顾之前写的一篇文章`java实例化对象过程中的内存分配`，我们继续来通过内存分配的方式分析上面讲的两个String定义的方式。

# 两种实例化方式的区别

## 直接赋值过程

在java中，如果直接用双引号里面加上字符串，就是实例化了一个String匿名类对象，此过程就会在堆内存中开辟一个空间。
```
String str = "hello world！";
```
那么在这个过程中会开辟几个堆内存几个栈内存呢？我们来看一下内存图
![20180914223745.jpg](
https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180914223745.jpg)

此过程中开辟了一块堆内存一块栈内存，但是如果我们声明多个字符串并且都赋值同一内容的字符串呢？

## new实例化过程

```
String str = new String("hello world！");
```
以上的代码其实可以将其初略地划分为三个步骤：
1. 先实例化一个String类对象`str`
2. 再实例化一个匿名对象`"hello world！"`
3. 将str对象栈内存指向`"hello world!"`的堆内存空间

我们来看一下其内存关系
![20180914230300.jpg](
https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180914230300.jpg)

**通过这个图可以看出此种方法创建String对象的缺陷，每次都会产生一块垃圾空间，所以建议在平时开发中尽量使用第一种方式。**

# JVM对象池
我们来看一下代码
```
public class StringTest {
    public static void main(String args[]){
        String str1 = "hello";
        String str2 = "hello";
        String str3 = str2;                 //引用传递
        System.out.println(str1 == str2);
        System.out.println(str2 == str3);
        System.out.println(str3 == str1);
    }
}
```
运行结果
```
true
true
true
```

我们发现上面代码运行结果都是true，说明三个String对象所指向的堆内存是同一个地址，我们来看一下其内存关系图
![20180914224436.jpg](
https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180914224436.jpg)
那么如果我们再加一个str4指向“world”呢？
```
public class StringTest {
    public static void main(String args[]){
        String str1 = "hello";
        String str2 = "hello";
        String str3 = str2;                 //引用传递
        String str4 = "world";
        System.out.println(str1 == str2);
        System.out.println(str2 == str3);
        System.out.println(str3 == str1);
        System.out.println(str3 == str4);
    }
}
```
运行结果
```
true
true
true
false
```
我们可以得到，实例化的匿名对象如果内容相同则使用的是同一个堆内存空间，并不是说实例化了三个"hello"就会开辟三个堆内存空间，如果内容不同则会开辟新的堆内存空间。

但是按道理来说应该是每实例化一个对象就开辟一个空间，之所以会出现这种情况是因为JVM对象池的原因，它用到了一个`共享设计模式`，目的是为了节省资源消耗。

> 共享设计模式：
  在JVM的底层实际上会存在有一个对象池（不一定只保存String对象，其他对象也可保存），当代码之中通过直接赋值的方式定义了String对象时，会将此字符串对象所使用的匿名对象入池保存，而后如果后续还有其他String对象也采用了直接赋值的方式，并且设置了同样内容的时候并不会开辟新的堆内存空间，而是使用已有的对象进行引用的分配，从而继续使用。

那么如果通过构造方法来创建String对象能使用对象池吗？我们来看一段代码
```
public class StringTest {
    public static void main(String args[]){
        String str1 = new String("hello");
        String str2 = new String("hello");
        System.out.println(str1 == str2);
    }
}
```
运行结果
```
false
```

很明显通过构造方法来赋值的方式并没有将其存入对象池，其原因是使用了关键字new开辟的新内存。如果希望开辟的新内存也可以利用对象池，这个时候我们就需要手动入池，用String类中的方法`intern()`。

jdk8中intern()方法的定义
```
    /**
     * Returns a canonical representation for the string object.
     * <p>
     * A pool of strings, initially empty, is maintained privately by the
     * class {@code String}.
     * <p>
     * When the intern method is invoked, if the pool already contains a
     * string equal to this {@code String} object as determined by
     * the {@link #equals(Object)} method, then the string from the pool is
     * returned. Otherwise, this {@code String} object is added to the
     * pool and a reference to this {@code String} object is returned.
     * <p>
     * It follows that for any two strings {@code s} and {@code t},
     * {@code s.intern() == t.intern()} is {@code true}
     * if and only if {@code s.equals(t)} is {@code true}.
     * <p>
     * All literal strings and string-valued constant expressions are
     * interned. String literals are defined in section 3.10.5 of the
     * <cite>The Java&trade; Language Specification</cite>.
     *
     * @return  a string that has the same contents as this string, but is
     *          guaranteed to be from a pool of unique strings.
     */
    public native String intern();
```
我们来看看使用手工入池的代码

```
public class StringTest {
    public static void main(String args[]){
        // 使用构造方法定义了新的内存空间，而后入池
        String str1 = new String("hello").intern();
        String str2 = new String("hello").intern();
        String str3 = "hello";
        System.out.println(str1 == str2);
        System.out.println(str1 == str3);
    }
}
```
运行结果
```
true
true
```

通过上面的分析，如果以后面试问到“String类对象两种实例化方式区别是什么？”，那么我们就能清楚的回答啦~

> - 直接赋值（String str = "字符串";）:只会开辟一块堆内存空间，并且会在自动保存在对象池之中以供下次重复使用；
> - 构造方法（String str = new String("字符串");）:会开辟两块堆内存空间，其中有一块空间将成为垃圾，并且不会自动入池，但是可以使用intern()方法手工入池。

# 字符串常量的不可改变性
字符串一旦被定义就不可改变，但是我们不能从平时编写的代码表面地去理解它，要从内存分析上才能理解它为什么是不可改变的。

我们来看一下下面的代码
```
public class StringTest {
    public static void main(String args[]){
        String str = "hello ";
        str = str + "world ";
        str += "!";
        System.out.println(str);
    }
}
```
运行结果
```
hello world !
```

如果按照代码来理解可能认为str的内容被改变了，并且被改变了两次！之前记得有人问过我类似的问题：上面的代码str对象赋值过程中进行了几步操作？当时我也不是很清楚，不过经过这次学习就能解释这个问题了。
![20180914233923.jpg](
https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180914233923.jpg)

以上操作可以看到，所谓的字符串的内容实际上并未改变（Java定义好了String的内容不能改变），改变的是地址的指向。对于字符串对象内容的改变，是利用了引用关系的改变而实现的，但是每一次的变化都会产生垃圾空间。

其实我们可以从jdk中对String对象的定义中找到其注释可以发现这一规定,下面是String类定义完整注释，在前面就可以看到这一句`Strings are constant; their values cannot be changed after they are created. String buffers support mutable strings.`
```
/**
 * The {@code String} class represents character strings. All
 * string literals in Java programs, such as {@code "abc"}, are
 * implemented as instances of this class.
 * <p>
 * Strings are constant; their values cannot be changed after they
 * are created. String buffers support mutable strings.
 * Because String objects are immutable they can be shared. For example:
 * <blockquote><pre>
 *     String str = "abc";
 * </pre></blockquote><p>
 * is equivalent to:
 * <blockquote><pre>
 *     char data[] = {'a', 'b', 'c'};
 *     String str = new String(data);
 * </pre></blockquote><p>
 * Here are some more examples of how strings can be used:
 * <blockquote><pre>
 *     System.out.println("abc");
 *     String cde = "cde";
 *     System.out.println("abc" + cde);
 *     String c = "abc".substring(2,3);
 *     String d = cde.substring(1, 2);
 * </pre></blockquote>
 * <p>
 * The class {@code String} includes methods for examining
 * individual characters of the sequence, for comparing strings, for
 * searching strings, for extracting substrings, and for creating a
 * copy of a string with all characters translated to uppercase or to
 * lowercase. Case mapping is based on the Unicode Standard version
 * specified by the {@link java.lang.Character Character} class.
 * <p>
 * The Java language provides special support for the string
 * concatenation operator (&nbsp;+&nbsp;), and for conversion of
 * other objects to strings. String concatenation is implemented
 * through the {@code StringBuilder}(or {@code StringBuffer})
 * class and its {@code append} method.
 * String conversions are implemented through the method
 * {@code toString}, defined by {@code Object} and
 * inherited by all classes in Java. For additional information on
 * string concatenation and conversion, see Gosling, Joy, and Steele,
 * <i>The Java Language Specification</i>.
 *
 * <p> Unless otherwise noted, passing a <tt>null</tt> argument to a constructor
 * or method in this class will cause a {@link NullPointerException} to be
 * thrown.
 *
 * <p>A {@code String} represents a string in the UTF-16 format
 * in which <em>supplementary characters</em> are represented by <em>surrogate
 * pairs</em> (see the section <a href="Character.html#unicode">Unicode
 * Character Representations</a> in the {@code Character} class for
 * more information).
 * Index values refer to {@code char} code units, so a supplementary
 * character uses two positions in a {@code String}.
 * <p>The {@code String} class provides methods for dealing with
 * Unicode code points (i.e., characters), in addition to those for
 * dealing with Unicode code units (i.e., {@code char} values).
 *
 * @author  Lee Boynton
 * @author  Arthur van Hoff
 * @author  Martin Buchholz
 * @author  Ulf Zibis
 * @see     java.lang.Object#toString()
 * @see     java.lang.StringBuffer
 * @see     java.lang.StringBuilder
 * @see     java.nio.charset.Charset
 * @since   JDK1.0
 */
```

对于这些知识平时使用时可能不太注意，但是了解之后对以后开发是有很大帮助的，最好的学习方法：多读读API和jdk源码吧~~~