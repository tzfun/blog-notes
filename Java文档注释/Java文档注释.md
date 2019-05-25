# Java中的三种文档注释
Java支持有三种文档注释，分别是：
* 行注释：//
* 段注释：/* */
* 说明注释：/** 开始 */结束

行注释和段注释大多数都不陌生，而说明注释了解的可能少一点，因为它支持有很多标签，说明注释允许在程序中嵌入相关程序信息并使用HTML标签。
# 说明注释标签
在说明注释中支持有很多标签，并且可以用工具软件进行识别，在开源项目里可以看到大量的说明注释，特别是jdk源码中非常多，里面有非常多的标签，下面介绍一下JavaDoc标签：
* @author：表示一个类的作者
```
/**
 * @author beifengtz
 */
```
* @deprecated：知名一个过期的类或成员
```
/**
 * @deprecated 从v1.1版本开始
 */
```
* {@docRoot}：知名当前文档的根目录路径
```
/**
 * @docRoot c:java/language
 */
```
* @exception：标志一个类抛出的异常
```
/**
 * @exception FileNotFoundException
 */
```
* {@link}：插入一个到另一个主题的链接
```
/**
 * {@link java.util.Date}
 */
```
* {@linkplain}：插入一个到另一个主题的链接，但是该链接显示纯文本字体
```
/**
 * {@linkplain java.util.Date}
 */
```
* @param：说明一个方法的参数
```
/**
 * @param args 传入的参数
 */
```
* @return：说明返回值类型
```
/**
 * @return Integer
 */
```
* @see：指定到另一个类的链接
```
/**
 * @see java.util.Date
 */
```
* @serial：说明一个序列化属性
```
/**
 * @serial title将不会被保存值，其余均会被序列化
 */
```
* @serialData：说明通过writeObject()和writeExternal()方法写的数据
```
/**
 * @serialData description
 */
```
* @serialField：说明一个ObjectStreamField组件
```
/**
 * @serialField name type description
 */
```
* @since：标记当前引入一个特定的变化时
```
/**
 * @since jdk1.7
 */
```
* @throws： （与@exception一样）
* {@value}：显示常量的值该常量必须是static属性
```
/**
 * {@value PI 3.1415926}
 */
```
* @version：指定类的版本
```
/**
 * @version jdk1.8
 */
```
# 说明注释示例
说明注释允许使用html标签，比如：
```
/**
 * <p>这是一个关于时间操作的处理类，作者是<a href="http:www.beifengtz.com">beifengtz</a></p>
 */
```
在使用写文档注释是可以进行联合使用，并且不一定只能使用这些标签，也可以自定义标签，当然如果是为了规范当然还是统一标签最好，比如下面是String类的说明注释：
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
现在常用的Java开发工具eclipse和idea都对说明注释支持的很好，对不同的标签会有高亮显示，并且在创建类或者方法时可以使用快捷键自动生成，对于你需要用到其中哪些标签，或者自定义一些标签，只需要提前在IDE中设置好模板即可，比如我的类说明注释就是设置的模板，每次创建类的时候会自动生成。