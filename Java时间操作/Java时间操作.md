在博客开始之前呢先分享一个在线的java doc
文档（jdk8）
* 英文版：[http://tool.oschina.net/apidocs/apidoc?api=jdk_7u4](http://tool.oschina.net/apidocs/apidoc?api=jdk_7u4)
* 中文版：[http://tool.oschina.net/apidocs/apidoc?api=jdk-zh](http://tool.oschina.net/apidocs/apidoc?api=jdk-zh)

OK，本文的主题是java中常用的时间操作，在平时开发过程中经常会使用到这些时间操作类，但是大部分使用都是其他工具包提供的类或者就那么几个常用的方法，对其中的方法也都并没有深入学习。所以这篇博客就记录一下我对jdk8中有关常用的时间操作的学习，在此过程中会用到jdk文档。

# Date类
首先我们打开java.util这个包，找到Date类，先看到类的定义
```
public class Date
extends Object
implements Serializable, Cloneable, Comparable<Date>
```
可以发现它是Object的直接子类，并且它实现了Serializable和Cloneable接口，说明它的属性是可存储和转换的，能够序列化，同时Date类支持克隆。

相信对于java开发者Date类是经常使用的，我们可以通过它来获取时间信息，也可以对时间进行格式化输出（此类方法已经过时），接下来看一下具体使用。
## 构造方法
回归文档，先看一下它的构造方法
```
Date()
Allocates a Date object and initializes it so that it represents the time at which it was allocated, measured to the nearest millisecond.

Date(int year, int month, int date)
Deprecated. 
As of JDK version 1.1, replaced by Calendar.set(year + 1900, month, date) or GregorianCalendar(year + 1900, month, date).

Date(int year, int month, int date, int hrs, int min)
Deprecated. 
As of JDK version 1.1, replaced by Calendar.set(year + 1900, month, date, hrs, min) or GregorianCalendar(year + 1900, month, date, hrs, min).

Date(int year, int month, int date, int hrs, int min, int sec)
Deprecated. 
As of JDK version 1.1, replaced by Calendar.set(year + 1900, month, date, hrs, min, sec) or GregorianCalendar(year + 1900, month, date, hrs, min, sec).

Date(long date)
Allocates a Date object and initializes it to represent the specified number of milliseconds since the standard base time known as "the epoch", namely January 1, 1970, 00:00:00 GMT.

Date(String s)
Deprecated. 
As of JDK version 1.1, replaced by DateFormat.parse(String s).
```
我们发现只有**Date()**、**Date(long date)** 两个方法目前是可以正常使用的，其他的方法都已经被设定为过时，并且都是从jdk1.1开始不推荐使用，而是推荐其他方法（下面学习），所以我们只需要关注这两个方法即可。

如果这样new一个对象，将会获得当前时间。
```
Date date = new Date()
```
如果传入一个long型数据（时间戳）可以获取指定时间的Date
```
Date date = new Date(1547778462);
```

## 成员函数

```$xslt
Date date = new Date();
```
* boolean	after(Date when)测试传入的时间when是否在date之后；
* boolean	before(Date when)测试传入的时间when是否在date之前；
* Object    clone()继承自Object类，因为Date类实现了Cloneable接口所以支持对象克隆，相当于new一个和它一样的Date对象；
* int	compareTo(Date anotherDate)Date时间比较，当小于anotherDate时为-1，等于为0，大于为1；
* boolean	equals(Object obj)对象比较，继承自Object
* int	hashCode()返回哈希值
* void	setTime(long time)重新设定Date的时间
* String	toString()字符串输出继承自Object
* long	getTime()获取Date的时间戳（最为常用）

其余的类似于getYear、getMonth、getDate方法都已经过时，官方不推荐使用，所以暂时不用管它。
# SimpleDateFormat类
## 文档解读
我们在文档中找到java.text包，然后选择其中的SimpleDateFormat类。java.text包是一个实现国际化程序的开发包，SimpleDateFormat类是一个专门处理时间格式的类。首先我们看一下它的继承关系

```
java.lang.Object
    java.text.Format
        java.text.DateFormat
            java.text.SimpleDateFormat
```
它是继承自Format和DateFormat的，Format类是text包中的核心类，它主要负责格式化及类型转换，我们的重点不是它，需要关注的是它的直接父类DateFormat类。
DateFormat类是一个时间格式转换的抽象类，其中提供了很多的方法，它可以将Date转换为String型**public final String format(Date date)**，也可以将String转换为Date型**public Date parse(String source) throws ParseException**

对于SimpleDateFormat类我们先看一下它的构造函数
```
SimpleDateFormat()
Constructs a SimpleDateFormat using the default pattern and date format symbols for the default locale.

SimpleDateFormat(String pattern)
Constructs a SimpleDateFormat using the given pattern and the default date format symbols for the default locale.

SimpleDateFormat(String pattern, DateFormatSymbols formatSymbols)
Constructs a SimpleDateFormat using the given pattern and date format symbols.

SimpleDateFormat(String pattern, Locale locale)
Constructs a SimpleDateFormat using the given pattern and the default date format symbols for the given locale.
```

在这四个构造函数中第二个是使用的最多的（就我目前的经验），所以我的重点也关注于它，其中的参数pattern是字符串形式的时间格式，它的格式各种各样，我们的重点应该放在这个格式上。

> 在时间格式转换上，常见的转换单位有：年（yyyy）、月（MM）、日（dd）、时（HH）、分（mm）、秒（ss）、毫秒（SSS）。

这里严格区分大小写，以上是常见的转换单位，在文档里面有列出详细的时间单位转换。

* **符号  详情描述  介绍  示例**
    * G	Era designator	Text	AD
    * y	Year	Year	1996; 96
    * Y	Week year	Year	2009; 09
    * M	Month in year	Month	July; Jul; 07
    * w	Week in year	Number	27
    * W	Week in month	Number	2
    * D	Day in year	Number	189
    * d	Day in month	Number	10
    * F	Day of week in month	Number	2
    * E	Day name in week	Text	Tuesday; Tue
    * u	Day number of week (1 = Monday, ..., 7 = Sunday)	Number	1
    * a	Am/pm marker	Text	PM
    * H	Hour in day (0-23)	Number	0
    * k	Hour in day (1-24)	Number	24
    * K	Hour in am/pm (0-11)	Number	0
    * h	Hour in am/pm (1-12)	Number	12
    * m	Minute in hour	Number	30
    * s	Second in minute	Number	55
    * S	Millisecond	Number	978
    * z	Time zone	General time zone	Pacific Standard Time; PST; GMT-08:00
    * Z	Time zone	RFC 822 time zone	-0800
    * X	Time zone	ISO 8601 time zone	-08; -0800; -08:00
    
## 测试示例

1. 将日期格式化显示（Date型数据变为String型数据）
```
package language;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 10:26 2019/1/18
 * @Description:
 */
public class TimeTest {
    public static void main(String[] args) {
        Date date = new Date ();
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
        String str = simpleDateFormat.format(date);
        System.out.println(str);
    }
}
```

测试结果：
```
2019-01-18 12:24:45.960
```

2. 将字符串转换为日期（String型数据变为Date型数据）
```
package language;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 10:26 2019/1/18
 * @Description:
 */
public class TimeTest {
    public static void main(String[] args) throws ParseException {
        String str = "2019-1-1 12:12:12.123";
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
        Date date = simpleDateFormat.parse(str);
        System.out.println(date);
    }
}
```
测试结果：
```
Tue Jan 01 12:12:12 CST 2019
```
在将字符串变为日期型数据的时候，如果日期型的月或者日等不对，超出正常范围，例如1月只有31天，在写的时候写成1月41天，此时它会自动进位，相当于2月10日，不会抛出异常。仅当匹配的字符串格式和构造函数中的pattern格式不匹配时才会抛异常。

**另外：DateFormat 和 SimpleDateFormat 类不都是线程安全的，在多线程环境下调用 format() 和 parse() 方法应该使用同步代码来避免问题。文档说明如下：**
```
Synchronization：
　　Date formats are not synchronized. 
　　It is recommended to create separate format instances for each thread. 
　　If multiple threads access a format concurrently, it must be synchronized externally.
```
对于具体原因我也不是很清楚，在一篇博客中看到说：
> SimpleDateFormat继承了DateFormat,在DateFormat中定义了一个protected属性的 Calendar类的对象：calendar。只是因为Calendar累的概念复杂，牵扯到时区与本地化等等，Jdk的实现中使用了成员变量来传递参数，这就造成在多线程的时候会出现错误。

> 总结：关于数据类型的转换
>
>       在数据表的操作里面的几个常用类型：VARCHAR2(String)、CLOB（String）、Number（Double、int）、Date（java.util.Date)
>
>       Date与String类之间的转换依靠的是SimpleDateFormat；
>
>       String与基本类型之间的转换依靠的是包装类与String.valueOf()方法；
>
>       long与Date转换依靠的是Date类提供的构造以及getTime()方法。

# Calendar类
Date类和SimpleDateFormat类两个往往是一起使用的，但是Calendar这个类主要是进行一些简单的日期计算的。首先我们从文档中看一下Calendar类的定义
```
public abstract class Calendar
extends Object
implements Serializable, Cloneable, Comparable<Calendar>
```
可以发现Calendar类是一个抽象类，那么它应该依靠子类来进行实例化操作。但是在这个类里面它提供有一个static方法，此方法返回的就是本类对象：public static Calendar getInstance()。
我们可以用下面方法来获取Calendar对象：
```
Calendar calendar = Calendar.getInstance();
```
前面我在读Date的文档时有提到过Calendar类，因为Calendar类的主要作用就是来取代Date类中的一些时间输出的方法，从jdk1.1后不再推荐使用Date来获取年、月、日等等，而是使用Calendar类，下面测试一下使用Calendar类来获取时间。

```
package language;
import java.util.Calendar;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 10:26 2019/1/18
 * @Description:
 */
public class TimeTest {
    public static void main(String[] args) {
        StringBuffer sb = new StringBuffer();
        Calendar calendar = Calendar.getInstance();
        sb.append(calendar.get(Calendar.YEAR)).append("-");
        sb.append(calendar.get(Calendar.MONTH) + 1).append("-");   // 月是从0开始计
        sb.append(calendar.get(Calendar.DAY_OF_MONTH)).append(" ");
        sb.append(calendar.get(Calendar.HOUR)).append(":");
        sb.append(calendar.get(Calendar.MINUTE)).append(":");
        sb.append(calendar.get(Calendar.SECOND)).append(".");
        sb.append(calendar.get(Calendar.MILLISECOND));
        System.out.println(sb);
    }
}

```
输出结果：
```
2019-1-18 0:59:36.694
```
这种方式获取时间可以很方便的进行计算，比如我要获取两个月后的时间，只需要在Calendar.MONTH加个3就可以了。至于官方为何取消Date类中获取年月日的这些操作，而使用Calendar类替换，我想主要原因就是在不同地区调用获取的时间不一样，这些牵扯到时区和本地化的操作，将方法写在Date类又太冗杂而且不符合类聚原则，所以干脆新增一个Calendar类来专门获取年月日，同时附带时区和本地化的设置方法。对于具体原因不太清除，这也只是我的猜测。总的来说当在编码是需要单独获取年、月、日等操作时尽量使用Calendar类不要使用Date类。