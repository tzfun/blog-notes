今天介绍一个非常好用的DOM操作工具——Jsoup，它可以用JS类似的函数来操作DOM元素，是Java爬虫里面使用非常广泛的一个工具。（PS：因为没有找到合适的Jsoup的API文档，并且库函数的注释也比较少，所以它的类学习不能像以前一样细致，简单了解其中的用途即可）
# 导入Jsoup
使用Maven项目，在pom.xml的依赖中加入下面代码：
```
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.11.2</version>
</dependency>
```
# 解析Html
## 字符串解析
通过Jsoup.parse(String html)方法可以将字符串解析成DOM对象，它的解析功能非常强大，如果dom元素中的标签首位不匹配（比如标签有头无尾），解析器也可以自动将不完整的标签补充上，如果是某些标签必须填父元素而没有，解析器也会自动补上。
```
package com.example.demo.net;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.select.Elements;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 18:35 2019/2/2
 * @Description:
 */
public class JsoupHtmlTest {
    public static void main(String[] args) throws Exception{
        String html = "<!DOCTYPE html>\n" +
                "<html>\n" +
                "<head>\n" +
                "\t<meta charset=\"utf-8\">\n" +
                "\t<meta http-equiv=\"X-UA-Compatible\" content=\"IE=edge\">\n" +
                "\t<title>测试网页</title>\n" +
                "\t<link rel=\"stylesheet\" href=\"\">\n" +
                "</head>\n" +
                "<body>\n" +
                "\t<p>这是p标签的内容</p>\n" +
                "\t<a href=\"http://blog.beifengtz.com\" title=\"beifengtz's blog\">beifeng blog</a>\n" +
                "</body>\n" +
                "</html>";
        Document document = Jsoup.parse(html);  //  将字符串解析成Document对象
        System.out.println(document);
    }
}
```
在Jsoup中处理HTML文档的部分类的继承结构如下：
```
java.lang.Object
    org.jsoup.nodes.Node
        org.jsoup.nodes.Element
            org.jsoup.nodes.Document
```
```
java.lang.Object
    org.jsoup.nodes.Node
        org.jsoup.nodes.LeafNode
            org.jsoup.nodes.TextNode
```
## 从网络加载html文档
Jsoup可以使用Jsoup中提供的一些静态方法从网络获取html文档并进行解析成Document对象
```
package com.example.demo.net;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;


/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 19:09 2019/2/2
 * @Description:
 */
public class JsoupNetTest {
    public static void main(String[] args) throws Exception{
        Document document = Jsoup.connect("http://blog.beifengtz.com/").get();
        System.out.println(document);
    }
}
```
如果在请求过程中需要传入参数或者传入cookie等等操作可以这样：
```
Document doc = Jsoup.connect("http://blog.beifengtz.com")  //  链接地址
  .data("query", "Java")    //  参数数据（可以传入Map对象）
  .userAgent("Mozilla")     //  使用代理
  .cookie("auth", "token")  //  传入cookie（可以传入Map对象）
  .timeout(3000)            //  请求超时时间
  .post();                  //  请求方法（post/get）
```
## 从文件加载html文档
jsoup还支持从文件中加载html文档并解析成Document对象：
```
package com.example.demo.net;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;

import java.io.File;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 19:24 2019/2/2
 * @Description:
 */
public class JsoupFileTest {
    public static void main(String[] args) throws Exception{

        File input = new File("C:"+File.separator+"Users"+File.separator+"TZplay"+File.separator+"Desktop"+File.separator+"text"+File.separator+"index.html");
        Document doc = Jsoup.parse(input, "UTF-8");
        System.out.println(doc);
    }
}
```
# 遍历Document对象
Jsoup中遍历对象的方法与JavaScript遍历的方法非常类似，方法名都几乎相同，对于有过Js开发经验的应该比较熟悉这些方法：

查找元素（Document）：
* 根据id查找（结果唯一）：public Element getElementById(String id)
* 根据标签查找（结果不唯一）：public Elements getElementsByTag(String tagName)
* 根据类名查找（结果不唯一）：public Elements getElementsByClass(String className)
* 根据属性名查找（结果不唯一）：public Elements getElementsByAttribute(String key)
* 根据属性键值对查找（结果不唯一）：public Elements getElementsByAttributeValue(String key, String value)
* 查找不含有该键值对的元素（结果不唯一）：public Elements getElementsByAttributeValueNot(String key, String value)
* 根据属性键值对进行模糊查找，支持正则表达式匹配（结果不唯一）：public Elements getElementsByAttributeValueMatching(String key, Pattern pattern) 

数据操作（Element）：
* 获取元素的文本内容：public String text()
* 设置元素的文本内容：public String text(String value)
* 设置节点的属性：public Node attr(String attributeKey, String attributeValue)
* 获取节点的属性：public String attr(String attributeKey)
* 移除节点的属性：public Node removeAttr(String attributeKey)
* 判断节点是否含有属性：public boolean hasAttr(String attributeKey)
* 获取所有属性：public Attributes attributes()
* 获取元素的id名：public String id()
* 获取元素的class名（返回字符串）：public String className() 
* 获取元素的class名（返回集合）：public Set<String> classNames()
* 获取元素内的html代码：public String html()
* 获取元素以外的html代码：public String outerHtml()
* 获取数据内容：public String data()

操作HTML（Element）：
* 在元素后追加html代码：public Element append(String html)
* 在元素前追加html代码：public Element prepend(String html)
* 在元素后追加文本内容：public Element appendText(String text) 
* 在元素钱追加文本内容：public Element prependText(String text)
* 替换元素内所有的html代码：public Element html(String html)
* 清空元素内容：public Element empty()

# 选择器支持
Jsoup同样支持类似于CSS的选择器语法，可以在开发中达到快速的DOM选择（选择器是前端CSS的知识，这里不赘述）
```
package com.example.demo.net;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.select.Elements;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 18:35 2019/2/2
 * @Description:
 */
public class JsoupHtmlTest {
    public static void main(String[] args) throws Exception{
        String html = "<!DOCTYPE html>\n" +
                "<html>\n" +
                "<head>\n" +
                "\t<meta charset=\"utf-8\">\n" +
                "\t<meta http-equiv=\"X-UA-Compatible\" content=\"IE=edge\">\n" +
                "\t<title>测试网页</title>\n" +
                "\t<link rel=\"stylesheet\" href=\"\">\n" +
                "</head>\n" +
                "<body>\n" +
                "\t<div id=\"id\">\n" +
                "\t\t这是id为“id”的标签内容\n" +
                "\t</div>\n" +
                "\t<p>这是p标签的内容</p>\n" +
                "\t<a href=\"http://blog.beifengtz.com\" title=\"beifengtz's blog\">beifeng blog</a>\n" +
                "\t<div class=\"father\">\n" +
                "\t\t<div class=\"children\">\n" +
                "\t\t\t这是div的子元素\n" +
                "\t\t</div>\n" +
                "\t</div>\n" +
                "\t<div class=\"father\">\n" +
                "\t\t<div class=\"children\">\n" +
                "\t\t\t<a href=\"http://www.sicau.edu.cn\">四川农业大学</a>\n" +
                "\t\t</div>\n" +
                "\t</div>\n" +
                "</body>\n" +
                "</html>";
        Document document = Jsoup.parse(html);  //  将字符串解析成Document对象
        Elements elements = document.select(".children a");   //  选择children类的子元素a标签
        System.out.println(elements.get(0).text());
    }
}
```
最后会输出“四川农业大学”几个字样，这里就简单演示一下如何支持选择器的，它是通过*public Elements select(String cssQuery)*方法实现的。

现在知道有如此方便的工具后，用Java操作html文档轻而易举，用java开发前端页面也会像写JS脚本一样。如果以后有爬虫、网页邮件、模板引擎等开发需求时，想必Jsoup这个工具会很方便吧~~

PS：Jsoup的源码一点注释也没有，看得我脑壳痛┭┮﹏┭┮