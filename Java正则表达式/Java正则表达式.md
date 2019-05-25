今天复习的内容是正则表达式，上次学习正则还是一年前，在平时的开发中也有一些时间使用正则表达式，但是实现的功能都是基础的，而且发现这些正则符号经常会忘记，每次写的时候才把正则表翻出来找。今天重新把Java正则温习了一遍，也简单的列举一些常用的正则符号，以便平时翻阅。

# Pattern类和Matcher类
## Pattern类
java正则表达式通过java.util.regex包下的Pattern类与Matcher类实现，Pattern类用于创建一个正则表达式,也可以说创建一个匹配模式,它的构造方法是私有的,不可以直接创建,但可以通过Pattern.complie(String regex)简单工厂方法创建一个正则表达式。下面是它常用的几个方法：
> * static Pattern	compile(String regex) 将给定的正则表达式编译到模式中
> * static boolean	matches(String regex, CharSequence input)  编译给定正则表达式并尝试将给定输入与其匹配
> * String	pattern()   返回在其中编译过此模式的正则表达式
> * String[]	split(CharSequence input) 围绕此模式的匹配拆分给定输入序列
```
package language;

import java.util.Arrays;
import java.util.regex.Pattern;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 20:40 2019/1/20
 * @Description:
 */
public class RegexTest {
    public static void main(String[] args) {
        Pattern pattern = Pattern.compile("\\d+");  //  编译正则表达式
        System.out.println(pattern.pattern());  // 返回正则表达式的字符串
        System.out.println(Arrays.toString(pattern.split("456asda6sdasd5")));   // 按照编译好的正则拆分字符串，返回String[]
        System.out.println(Pattern.matches("\\d+","hello123"));   //  快速匹配
        System.out.println(Pattern.quote("123"));   //  返回指定字符串的文本模式字符串
    }
}
```
输出如下：
```
\d+
[, asda, sdasd]
false
\Q123\E
```

## Matcher类
通过解释 Pattern 对 character sequence 执行匹配操作的引擎。Matcher类是平时使用的较少的一个类。它的成员函数会被Pattern、String等类自动调用里面方法（其实不常用，因为这些方法一般是jdk内部调用，可学习使用）。
> * 它提供三个匹配操作方法,三个方法均返回boolean类型,当匹配到时返回true,没匹配到则返回false。
>   * matches()对整个字符串进行匹配,只有整个字符串都匹配了才返回true 
>   * lookingAt()对前面的字符串进行匹配,只有匹配到的字符串在最前面才返回true 
>   * find()对字符串进行匹配,匹配到的字符串可以在任何位置. 
> * 当使用matches(),lookingAt(),find()执行匹配操作后,就可以利用以上三个方法得到更详细的信息.:
>   * start()返回匹配到的子字符串在字符串中的索引位置.
>   * end()返回匹配到的子字符串的最后一个字符在字符串中的索引位置. 
>   * group()返回匹配到的子字符串

# 正则表达式符号
正则表达式的符号在jdk文档的Pattern类里面有详细的说明，不过里面很多符号看着比较复杂，下面我简单罗列一些常用的表达式符号，平常的开发记住这些也就差不多了。
1. 单个字符（匹配数量：1）
    * 字符：表示由一个字符所组成，比如"a"匹配"a"成功而"a"匹配"b"失败；
    * \\\：表示转义字符"\"，在一些特殊字符进行匹配时要使用；
    * \t：表示"\t"符号，实际匹配时要写成"\\\t"，\要进行转义；
    * \n：匹配换行符号，实际匹配时要写成"\\\n"，\要进行转义；
2. 字符集（匹配数量：1）
    * \[abc\]：表示可能是a或者b或者c的其中一个；
    * \[^abc\]：表示不是a、b、c的任何一位；
    * \[a-z\]：表示小写字母；
    * \[a-zA-Z\]：表示英文字母，不区分大小写；
    * \[0-9\]：表示0到9的任意一个数字；
3. 简化的字符集表达式（匹配数量：1）
    * .：表示任意的一位字符；
    * \d：表示0-9的任意一个数字，等价于"\[0-9\]"；
    * \D：表示不是0到9的任意一个数字，等价于"\[^0-9\]"；
    * \s：表示任意的空白字符，等价于"\t"、"\n"；
    * \S：表示任意的非空白字符；
    * \w：等价于"\[a-zA-Z_0-9\]"，表示任意的字母、数字、下划线；
    * \W：等价于"\[^a-zA-Z_0-9\]"，表示不是由任意的字母、数字、下划线所组成；
4. 边界匹配（与JavaScript对应）
    * ^：表示正则表达式的开始；
    * $：表示正则表达式的结束；
5. 数量匹配
    * 正则?：表示此正则可以出现0次或1次；
    * 正则+：表示此正则可以出现1次及以上；
    * 正则*：表示此正则可以出现0次、1次或多次；
    * 正则{n}：表示此正则正好出现n次；
    * 正则{n,}：表示此正则至少出现n次；
    * 正则{n,m}：表示此正则出现n到m次；
6. 逻辑运算
    * 正则1正则2：正则1判断完之后继续判断正则2；
    * 正则1|正则2：正则1或者是正则2有一组满足即可；
    * (正则)：将多个正则作为一组，可以为这一组单独设置出现的次数。
# String类的正则表达式支持
在String对象中有很多方法支持正则表达式，下面列举出常用方法：
* public boolean matches(String regex)  正则验证，使用指定的正则表达式结构；
* public String replaceAll(String regex,String replacement) 全部替换；
* public String replaceFirst(String regex,String replacement)   替换首个；
* public String\[\] split(String regex) 全部拆分；
* public String\[\] split(String regex,int limit)   部分拆分。

在Java中String的这些正则验证的方法都会调用Pattern和Matcher相关的方法，所以平时这两个类用的比较少，但是面试可能会出相关考题所以还是要熟悉一下，String类的这些方法平时用的非常多，应该熟练掌握。下面以一个例子来说明正则表达式的用法，以及我的分析思路。

> 邮箱验证是开发中经常会使用的，如果我们用正则表达式来验证如何进行呢？因为不同的场景对邮箱的限制规则不一样，所以这里我以自定义的邮箱规则来进行演示。
>
> 邮箱结构可简单分为（不严谨）：    用户名@组织名 后缀
>
> 验证邮箱规则：
> * 邮箱用户名只能由**数字**、**字母**、**_(下划线)**和**.(点)** 组成；
> * 邮箱用户名的首和尾必须由**数字** 和**字母**组成；
> * 邮箱用户名不超过30位；
> * 组织名只能由**数字**、**字母**、**_(下划线)** 组成；
> * 后缀只能是 **.com**、**.net**、**.cn**、**.com.cn**、**.edu.cn**、**.edu**、**.org** 。

好了，规则我们已经知道了，现在开始写正则表达式，下面是我的分析步骤：
1. 因为用户名是数字、字母、下划线、点组成，所以不能只用“\w”，可以用“\[a-zA-Z_\\\\.0-9\]”来匹配，注意其中的点需要用\\来进行转义；
2. 对于用户名的首位只能是数字和字母，所以我们可以在第1步的首位加上“\[a-zA-Z0-9\]”，然后是用户名位数限制30位，现在就是这样：“\[a-zA-Z0-9\]\[a-zA-Z_\\\\.0-9\]{0,28}\[a-zA-Z0-9\]”
3. 加上邮箱的@符号，成为“\[a-zA-Z0-9\]\[a-zA-Z_\\\\.0-9\]{0,28}\[a-zA-Z0-9\]@”；
4. 组织名是数字、字母和下划线，所以可以直接用“\\\w+”
5. 后缀给定了特定的，所以可以直接进行分组，表达式为“\\.(com|net|cn|com\\.cn|edu\\.cn|edu|org)”;
6. 最后将上面步骤拼凑在一起就是“\[a-zA-Z0-9\]\[a-zA-Z_\\\\.0-9\]{0,28}\[a-zA-Z0-9\]@\\\w+\\\\.(com|net|cn|com\\\\.cn|edu\\\\.cn|edu|org)”。

最后来验证一下上述的表达式是否正确
```
package language;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 20:40 2019/1/20
 * @Description:
 */
public class RegexTest {
    public static void main(String[] args) {
        String regexStr = "[a-zA-Z0-9][a-zA-Z_\\.0-9]{0,28}[a-zA-Z0-9]@\\w+\\.(com|net|cn|com\\.cn|edu\\.cn|edu|org)";
        System.out.println("1246886075@qq.com："+"1246886075@qq.com".matches(regexStr));
        System.out.println("tz1112tz@163.com："+"tz1112tz@163.com".matches(regexStr));
        System.out.println("201603959@sicau.edu.cn："+"201603959@sicau.edu.cn".matches(regexStr));
        System.out.println("beifengtz_hah123@sicau.edu.cn："+"beifengtz_hah123@sicau.edu.cn".matches(regexStr));
        System.out.println("beifengtz_hah123_@sicau.edu.cn："+"beifengtz_hah123_@sicau.edu.cn".matches(regexStr));
        System.out.println("beifengtz@wingstudio.org："+"beifengtz@wingstudio.org".matches(regexStr));
        System.out.println("123beifengtz_hah123_@beifengtz.com："+"123beifengtz_hah123_@beifengtz.com".matches(regexStr));
        System.out.println("beifengTZasdASa_sdas_sdef_gtz_hah123OSsdw@beifengtz.com："+"beifengTZasdASa_sdas_sdef_gtz_hah123OSsdw@beifengtz.com".matches(regexStr));
    }
}
```
输出结果
```
1246886075@qq.com：true
tz1112tz@163.com：true
201603959@sicau.edu.cn：true
beifengtz_hah123@sicau.edu.cn：true
beifengtz_hah123_@sicau.edu.cn：false
beifengtz@wingstudio.org：true
123beifengtz_hah123_@beifengtz.com：false
beifengTZasdASa_sdas_sdef_gtz_hah123OSsdw@beifengtz.com：false
```
OK，正则顺利的通过验证。最后总结一下：正则表达式的符号要熟记，平时多联系正则多读一些正则表达式，孰能生巧。