本文接着上一篇文章讲：[详解Class类文件的结构（上）](http://blog.beifengtz.com/article/59)

本文继续使用上次的Test.class文件，它是由下面单独的一个类文件编译而成的，没有包。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/12.png)

## 6. 索引（Index）

索引又分类索引、父类索引和接口索引集合，类索引（this_class）和父类索引（super_class）都是一个u2类型的数据，而接口索引集合（interfaces）是一组u2类型的数据的集合，**Class文件依靠这些索引数据来确定这个类的继承关系**。所有类（除了java.lang.Object）都只有一个父类索引（Java的单继承），即父类索引不为0，只有java.lang.Object的父类索引为0。接口索引用来描述该类实现了哪些接口，它们的出现顺序是按照implements语句后接口的先后顺序出现的，如果这个类是一个接口就按照extends后面出现的顺序来。

类索引和父类索引各自指向一个CONSTANT_Class_info的类描述符常量，然后通过CONSTANT_Class_info可以定位到一个CONSTANT_Utf8_info类型的常量中的*全限名*字符串。而接口索引集合则以接口计数器开头，和前面常量池类似，若计数器表示n则后面紧跟着的n个u2数据是表示该类实现的n个接口的类索引，分别指向对应的类描述符常量。

> 全限名："java/lang/Object"表示Object类的全限名，将类全名中的“.”替换成“/”而已，多个全限名之间是“;”分隔。

仍然以我上次的那个Test.class文件为例，这里三个u2类型的值分别为Ox0005、Ox0006、Ox0000，前两个分别表示的是类索引、父类索引所指向的常量描述符。第三个表示接口集合的个数，这里为0即没有实现任何接口。假设为2，则表示接下来的2个u2数据表示实现的两个接口，每个u2数据也指向的是常量描述符。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/11.png)

## 7.字段表集合（Field Info）
字段表（field_info）用于描述接口或者类中声明的变量。**字段包括类级变量以及实例级变量，但不包括在方法内部声明的局部变量**。字段包含的信息比较多，包含以下内容：
* 字段的作用域：public、private、protect修饰符
* 变量类型（类变量or实例变量）：static
* 可变性：final
* 并发可见性：volatile
* 可否序列化：transient
* 数据类型：基本数据类型、对象、数组
* 字段名称

上面的这些信息除了字段数据类型和字段名称其他都是以**布尔值**来描述的，有就是true且对应一个标志位，没有则false，这种表示方法和上一节的Access Flags一样。字段数据类型和字段名称是引用的常量池中的常量来描述，可能是CONSTANT_Class_info也可能是CONSTANT_Utf8_info。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/13.png)

根据Java语言的语法我们可以知道，ACC_PUBLIC、ACC_PRIVATE、ACC_PROTECTED三个标志只能选一个，ACC_FINAL、ACC_VOLATILE不能同时存在，接口必须有ACC_PUBLIC、ACC_STATIC、ACC_FINAL标志。

**描述符**

描述符的作用是用来描述字段的数据类型、方法的参数列表（数量、类型、顺序）和返回值。其中基本数据类型以及void返回值类型都是用一个大写字母来表示的，对象的类型由一个L加对象全限名表示。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/14.jpg)

基本数据类型和普通类型都已经知道怎么表示了，但Java中有一个特殊类型就是数组类型，它是在编译期产生的，它的描述符是在变量描述符前面加一个"\["，如果是二维则加两个\[，比如"\[\["。例如一个`String\[\]\[\]`记录为`\[\[Ljava/lang/String`，一个`int\[\]`记录为`\[I`。

如果是描述一个方法则在描述符前面加一个括号“()”，如果有参数则在其中按顺序添加描述符即可。例如一个`String toString(char[] c,int a,String[] b)`的描述符为：“`([CI[Ljava.lang.String)Ljava.lang.String`”。

这里同样以Test.class文件来验证，第一个u2数据是容量技术器fields_count，这里是Ox0000，说明没有字段表数据，看文章开头的java代码，确实没有定义任何字段。由于在编译class文件开始没有考虑周全，没有定义字段，这里容量技术器为0也就看不到后面的字段描述内容，这里先假设是Ox0001，即有一个字段。第二个u2数据是访问标识符access_flags，假设这里是Ox0002，说明字段标志为ACC_PRIVATE。第三个u2数据是字段名称name_index，假设值为Ox0005，指向#5的常量池CONSTANT_Utf8_info字符串。第四个u2数据是字段描述符，这里是Ox0007，指向#7的常量池字符串。

## 8. 方法表集合

方法表的描述和字段表集合描述形式一样，只需要按照对应的表格对照就可以了。方法表结构依次包含了access_flags（访问标志）、name_index（方法名索引）、descriptor_index（描述符索引）、attribute（属性表集合）几项。方法内的具体代码存放在属性表集合attribute的名为“Code”的属性里面。

方法表结构表：

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/15.jpg)

方法访问标志表：

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/16.jpg)

继续以Test.class文件分析，容量计数器methods_count的值为Ox0002，表示由两个方法，疑惑？看文章开头的代码只有一个main方法啊，为什么会有两个？其实字节码中包含了平时省略了的无参构造方法\<init\>。紧跟着的是2个方法描述集合，这里以第一个无参构造来解释，首先是访问标志access_flags，值是Ox0001，查表可知是ACC_PUBLIC类型的，然后是方法名索引name_index，值是Ox0007，指向的是常量池CONSTANT_Utf8_info字符串，即#7，我们查看反编译的代码可以看到#7确实是\<init\>。然后是描述符索引descriptor_index，值是Ox0008指向的是常量项#8，反编译后看到是`()V`，构造方法无返回值，所以用的void的标识字符V，但是在书写代码时不能显式加void，因为其验证是在编译期。紧接着的是属性表集合的属性计数量attributes_count，这里是Ox0001，说明只有一个属性，即前面说的“Code”属性。接下来的就是分别表示每一个属性的具体指向，这里只有一个当然就只需看一个u2数据，这里是Ox0009，指向的是常量项#9，反编译结果#9确实是Code。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/17.png)

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/18.png)

**如果方法在子类中没有被重写，方法表集合中就不会出现来自父类的信息。**

从方法表集合可以看出，Class文件对一个方法的特征识别（《Java虚拟机规范》称之为特征签名）有很多，比如方法描述符、访问控制标志、返回值、属性表等。这里我想起来了之前腾讯一个面试官问我的问题“重载的验证是在哪个阶段？”，当时我没回答好这个问题，看了《深入理解Java虚拟机》这一节的内容才知道，对于Java方法的重载是在编译器验证的，在Java语义里规定：只要方法名、参数内容及顺序相同则视为非法重载，而对返回值、修饰符等没有严格要求。而在Class文件里对一个方法的特征签名比编译期的多，也就是说如果两个方法有相同的名称和特征签名，但返回值不同，那么也是可以合法存在于同一个Class文件的。

## 9.属性表集合

属性表（attribute_info）存在于Class文件、字段表、方法表等，它用于描述某些场合专有的信息。在class文件中对属性表的限定并不是很严格，只要不要与已有属性名重复，任何不人实现的编译器都可以向属性表中写入自己定义的属性信息，虚拟机在运行时会忽略掉它不认识的属性。这一部分内容较多并且不固定，建议读者阅读最新的《Java虚拟机规范》或《深入理解Java虚拟机——周志明 著》。


本文是笔者阅读《深入理解Java虚拟机》一书时的简单总结和实践。参考文献：《Java虚拟机规范（第二版）》、《深入理解Java虚拟机》