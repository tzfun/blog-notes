# 前言
相信搞Java开发的同学都经常会接触到Class类文件，了解了JVM虚拟机之后也会大量接触到class字节码，那么它到底是什么样的文件？内部由什么构成？虚拟机又是如何去识别它的？这篇文章就来学习一下Class类文件的结构。

ps：我在面试蚂蚁的时候被问到过这个问题！你没看错，面试也有可能会问。

# 一、什么是Class文件

Class文件又称字节码文件，一种二进制文件，它是由某种语言经过编译而来，注意这里**并不一定是Java语言**，还有可能是Clojure、Groovy、JRuby、Jython、Scala等，Class文件运行在Java虚拟机上。Java虚拟机不与任何一种语言绑定，它只与Class文件这种特定的二进制文件格式所关联。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/1.png)

虚拟机具有语言无关性，它不关心Class文件的来源是何种语言，它只关心Class文件中的内容。Java语言中的各种变量、关键字和运算符号的语义最终都是由多条字节码命名组合而成的，因此字节码命令所能提供的语义描述能力比Java语言本身更加强大。

# 二、Class文件的结构

虚拟机可以接受任何语言编译而成的Class文件，因此也给虚拟机带来了安全隐患，为了提供语言无关性的功能就必须做好安全防备措施，避免危险有害的类文件载入到虚拟机中，对虚拟机造成损害。所以在类加载的第二大阶段就是验证，这一步工作是虚拟机安全防护的关键所在，其中检查的步骤就是对class文件按照《Java虚拟机规范》规定的内容来对其进行验证。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/870109-20160503213708857-429280187.png)

## 1.总体结构

Class文件是一组以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑地排列在Class文件之中，中间没有添加任何分隔符，Class文件中存储的内容几乎全部是程序运行的必要数据，没有空隙存在。当遇到需要占用8位字节以上空间的数据项时，就按照高位在前的方式分割成若干个8位字节进行存储。

Class文件格式采用类似于C语言结构体的伪结构来存储数据，这种伪结构只有两种数据类型：无符号数和表。
* 无符号数属于基本的数据类型，以u1、u2、u4、u8来分别代表1个字节、2个字节、4个字节、8个字节的无符号数，无符号数可以来描述数字、索引引用、数量值或者按照UTF-8编码构成字符串值。
* 表是由多个无符号数或者其他表作为数据项构成的复合数据类型，所有表都习惯性的以“_info”结尾。表用于描述有层次关系的复合结构的数据，整个Class文件本质上就是一张表，它的数据项构成如下图。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/2.png)

## 2.魔数（Magic Number）

每一个Class文件的头4个字节成为魔数（Magic Number），**它的唯一作用是确定这个文件是否是一个能被虚拟机接收的Class文件**。很多文件存储标准中都是用魔数来进行身份识别，比如gif、png、jpeg等都有魔数。使用魔数主要是来识别文件的格式，相比于通过文件后缀名识别，这种方式准确性更高，因为文件后缀名可以随便更改，但更改二进制文件内容的却很少。Class类文件的魔数是Oxcafebabe，cafe babe？咖啡宝贝？至于为什么是这个， 这个名字在java语言诞生之初就已经确定了，它象征着著名咖啡品牌Peet's Coffee中深受欢迎的Baristas咖啡，Java的商标logo也源于此。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/3.png)

## 3.文件版本（Version）

在魔数后面的4个字节就是Class文件的版本号，第5和第6个字节是次版本号（Minor Version），第7和第8个字节是主版本号（Major Version）。Java的版本号是从45开始的，JDK1.1之后的每个JDK大版本发布主版本号向上加1（JDK1.0~1.1使用的版本号是45.0~45.3），比如我这里是十六进制的Ox0034，也就是十进制的52，所以说明该class文件可以被JDK1.8及以上的虚拟机执行，否则低版本虚拟机执行会报`java.lang.UnsupportedClassVersionError`错误。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/4.png)

## 4.常量池（Constant Pool）

在主版本号紧接着的就是常量池的入口，它是Class文件结构中与其他项目关联最多的数据类型，也是占用空间最大的数据之一。常量池的容量由后2个字节指定，比如这里我的是Ox001d，即十进制的29，这就表示常量池中有29项常量，而常量池的索引是从1开始的，这一点需要特殊记忆，因为程序员习惯性的计数法是从0开始的，而这里不一样，所以我这里常量池的索引范围是1~29。设计者将第0项常量空出来是有目的的，这样可以满足后面某些指向常量池的索引值的数据在特定情况下需要表达“不引用任何一个常量池项目”的含义。

通过`javap -v`命令反编译出class文件之后，我们可以看到常量池的内容：
![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/5.png)

常量池中主要存放两大类常量：**字面量**和**符号引用**。比如文本字符、声明为final的常量值就属于字面量，而符号引用则包含下面三类常量：
* 类和接口的全限名
* 字段的名称和描述符
* 方法的名称和描述符

在之前的文章（[详谈类加载的全过程](http://blog.beifengtz.com/article/56)）中有详细讲到，在加载类过程的第二大阶段连接的第三个阶段**解析**的时候，**会将常量池中的符号引用替换为直接引用**。相信很多人在开始了解那里的时候也是一头雾水，作者我也是，当我了解到常量池的构成的时候才明白真正意思。**Java代码在编译的时候，是在虚拟机加载Class文件的时候才会动态链接，也就是说Class文件中不会保存各个方法、字段的最终内存布局信息，因此这些字段、方法的符号引用不经过运行期转换的话无法获得真正的内存入口地址，也就无法直接被虚拟机使用。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析、翻译到具体的内存地址之中**。

常量池中每一项常量都是一张表，这里我只找到了JDK1.7之前的常量池项目类型表，见下图。

* 常量池项目类型表：
![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/6.png)
* 常量池常量项的结构总表：
![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/7.png)

比如我这里测试的class文件第一项常量，它的标志位是Ox0a，即十进制10，即表示tag为10的常量项，查表发现是CONSTANT_Methodref_info类型，和上面反编译之后的到的第一个常量是一致的，Methodref表示类中方法的符号引用。查上面《常量池常量项的结构总表》可以看到Methodref中含有3个项目，第一个tag就是上述的Ox0a，那么第二个项目就是Ox0006，第三个项目就是Ox000f，分别指向的CONSTANT_Class_info索引项和CONSTANT_NameAndType_info索引项为6和15，那么反编译的结果该项常量指向的应该是#6和#15，查看上面反编译的图应证我们的推测是对的。后面的常量项就以此类推。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/8.png)

这里需要特殊说明一下utf8常量项的内容，这里我以第29项常量项解释，也就是最后一项常量项。查《常量池常量项的结构总表》可以看到utf8项有三个内容：tag、length、bytes。tag表示常量项类型，这里是Ox01，表示是CONSTANT_Utf8_info类型，紧接着的是长度length，这里是Ox0015，即十进制21，那么再紧接着的21个字节都表示该项常量项的具体内容。**特别注意length表示的最大值是65535，所以Java程序中仅能接收小于等于64KB英文字符的变量和变量名，否则将无法编译**。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/9.png)

## 5.访问标志（Access Flags）

在常量池结束后，紧接着的两个字节代表访问标志（Access Flags），该标志用于识别一些类或者接口层次的访问信息，其中包括：Class是类还是接口、是否定义为public、是否定义为abstract类型、类是否被声明为final等。

访问标志表
![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/9.jpg)

标志位一共有16个，但是并不是所有的都用到，上表只列举了其中8个，没有使用的标志位统统置为0，access_flags只有2个字节表示，但是有这么多标志位怎么计算而来的呢？它是由标志位为true的标志位值取或运算而来，比如这里我演示的class文件是一个类并且是public的，所以对应的ACC_PUBLIC和ACC_SIPER标志应该置为true，其余标志不满足则为false，那么access_flags的计算过程就是：Ox0001 | Ox0020 = Ox0021

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/jvm/Class%20file/10.png)


篇幅原因，未完待续......