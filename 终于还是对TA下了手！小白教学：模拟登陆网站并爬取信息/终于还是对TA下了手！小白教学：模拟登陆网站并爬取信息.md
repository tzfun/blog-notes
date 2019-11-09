# 前言

相信很多读者多多少少都玩过模拟登陆某某网站、爬取某某网站数据等等，对于高手来说这篇文章简直是小菜一碟，不过对于入门级程序猿来说可能将会是ta跨入网络编程的第一步，相信对于小白的你看了这篇文章你肯定会对网络编程产生极大的兴趣。

<p>
<font color=red>
<strong>注意！！！本文仅仅是技术分享，请不要滥用该技术做任何非法的事情！</strong>
</font>
</p>

# 研究目标网站

因为小编之前发了一条朋友圈，模拟登陆了学校的教务网，获取到个人信息，引发了很多刚入门编程的师弟师妹关注，所以这里也以学校教务网作为目标网站作为示例，不过还是得提醒一下广大小白程序员：**非法爬取政府、教育、商业等机构网站信息是属于违法行为**！这里虽以教务网作为示例网站，但是获取的信息仅是小编本人的个人信息，不涉及任何其他信息，所以小白在学的时候注意哦，不要随便爬别人网站的数据~~~

我们先使用chrome浏览器打开目标网站`http://jiaowu.****.cn`，找到它的登录模块，按`F12`打开浏览器控制台，使用元素选择器选中登录模块。

![](https://s2.ax1x.com/2019/09/05/numWU1.jpg)

然后分析其中登录时需要传输的数据，传输形式不同网站会有所不同，有的是前后端分离的网站，请求数据可能是通过Ajax异步请求，而有的则可能是form表单请求，有的或许是使用ES6的Fetch请求。模拟登录的过程实际就是模拟请求的过程，只要是按照后台请求的格式以及对方的信任数据就可以模拟此过程（当然对于有身份验证的无法模拟，比如https）。

就拿目标网站的登录模块来说，它是使用最老的一种方式：form表单请求，这种也是最容易模拟的，所以比较方便演示。我们先展开登录的form表单所有的html代码。

![](https://s2.ax1x.com/2019/09/05/nun1aR.jpg)

在input标签中name则为请求数据的key，其对应的value就是需要传的值，不难发现其中的无非就是以下几个参数：

1. **user**：你的学号或工号
2. **pwd**：你的密码
3. **lb**：你的登录角色
4. **sign**：隐藏表单域，用于验证身份（一般网站通过定期修改这个值来避免脚本登陆）

然后咱们再来看form表单身份认证请求的地址和类型，这些信息都在form表单中，你直接看html就可以获取到。稍微麻烦一点的话可能有的网站会把这些信息隐藏在js文件里面，然后对js文件编码压缩，可能你找的过程需要麻烦一些，不过嘿嘿，再复杂再麻烦都难不倒咱们程序员的，只要你的这些数据放在前端文件里，总能找到的。

![](https://s2.ax1x.com/2019/09/05/nuuqht.jpg)

好了这里的数据便一目了然：
1. 请求地址：`http://jiaowu.****.cn/jiaoshi/bangong/check.asp`
2. 请求方法：POST

我们拿到了这些信息之后还不够，你需要查看其中获取的cookie，所有的这些数据传输、cookie信息、请求头信息等都可以通过网络抓包去查看，只要请求的数据包没有经过加密，你都能看到。网上抓包工具非常多，你们百度或谷歌一下就知道啦，这里就不展示抓包的过程。

经过小编探路，已经得知，身份认证的cookie数据需要来源于当前页面下发的cookie，你可以通过在控制台点击`Application -> Cookies`查看这个页面所有的cookie，在下发的cookie中有一个是用于时间验证，只要在有效时间内认证均是被允许的。除此之外你还可以通过获取下发当前页面的网络请求头和响应头数据查看cookie信息，位置在控制台`Network -> index.asp`。

好啦，基本上需要模拟登录的信息已经掌握了，接下来我们就开始编码吧

# 代码实现

不同语言实现不一样，不过逻辑思路都是一致的，无论你是使用Java还是Python，还是C++或Go，只要支持网络编程的语言都可以实现。这里小编用Java来做，Java的网络编程API以及各种库实在是太多，为了让小白的你看的更清晰易懂，这里使用Jsoup来展示。

首先你需要创建一个Maven项目（不知道怎么建的自信百度哦~），引入Jsoup的Maven或Gradle依赖，这里以Maven为例（Gradle依赖可自信改写或查看官方文档来添加）

```xml
<dependency>
    <groupId>org.jsoup</groupId>
    <artifactId>jsoup</artifactId>
    <version>1.11.2</version>
</dependency>
```

然后建立你的测试类开始编码，上面分析已经很明白了，要模拟登录需要请求地址、请求方法、请求数据以及Cookie，而cookie是在请求主页的时候才会下发，所以总共我们需要两次请求，第一次请求先获取cookie，第二次请求才是真正的模拟登录，所以显示第一次请求:
```java
String url = "http://jiaowu.****.cn";// 主页地址
Connection con = Jsoup.connect(url); //建立连接

Connection.Response rs = con.postDataCharset("UTF-8").execute();// 获取响应
rs.charset("UTF-8");//  设置响应编码
Map<String, String> cookies = rs.cookies();//   我们需要拿到的cookie
```

然后就是准备好我们的数据以及cookie来请求登录认证的地址:
```java
Connection con2 = Jsoup.connect("http://jiaowu.****.cn/jiaoshi/bangong/check.asp");//   身份认证的地址
Map<String, String> data = new HashMap<>(); //  设置post的数据

data.put("user", "201*****");
data.put("lb", "S");
data.put("pwd", "******");
data.put("sign", "c4f986961b0ffe8d521548678ae5b586");
con.data(data);
Connection.Response login = con2.ignoreContentType(true)
        .postDataCharset("UTF-8")
        .method(Connection.Method.POST)
        .cookies(cookies)
        .data(data)
        .header("Upgrade-Insecure-Requests", "1")
        .execute();
login.charset("UTF-8");
```

好了，现在我们就拿到了响应的对象，为了检测是否登录成功，你可以打印一下响应返回给你的dom树：
```java
Document doc = login.parse();
System.out.println(doc);
```

如果你能打印出正常的dom树说明你就已经完成了模拟登录，通过在自己计算机上就可以登录，而不需要进入网站。

只要你拿到了这个网站的认证数据，那么你就可以在接下来做任何你的权限范围内所做的事情，比如获取课表信息、项目信息等。

因为学校的教务网做的比较简单，个人信息全是放在cookie中，所以你不需要进行第三次请求就可以获取到自己的个人信息。**当然这也就警示广大学弟学妹，不要随便在其他未认证的网站中登录自己的账号**！

接下来是解析第二次请求的cookie内容，学校的教务网对字符串进行了url编码，有小伙伴可能以为这是乱码，其实它就是明文并不是乱码，经过简单的url解码即可。

```java
Map<String,String> cookie = login.cookies();
cookie.forEach(new BiConsumer<String, String>() {
    @Override
    public void accept(String s1, String s2) {
        try {
            System.out.println(String.format("%s = %s", decode(s1), decode(s2)));
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
    }
});

//  url解码
private static String decode(String url) throws UnsupportedEncodingException{
    String prevURL = "";
    String decodeURL = url;
    while (!prevURL.equals(decodeURL)) {
        prevURL = decodeURL;
        decodeURL = URLDecoder.decode(decodeURL, "UTF-8");
    }
    return decodeURL;
}
```

# 完整的代码和执行结果
```java
import org.jsoup.Connection;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;

import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.net.URLDecoder;
import java.net.URLEncoder;
import java.util.HashMap;
import java.util.Map;
import java.util.function.BiConsumer;

/**
 * @Author beifengtz
 * @Site www.beifengtz.com
 * @Date Created in 22:35 2019/9/4
 * @Description:
 */
public class JsoupHtmlTest {
    public static void main(String[] args) throws IOException {
        String url = "http://jiaowu.****.cn/web/web/web/index.asp";
        Connection con = Jsoup.connect(url); //建立连接

        Connection.Response rs = con.postDataCharset("UTF-8").execute();// 获取响应
        rs.charset("UTF-8");
        Map<String, String> cookies = rs.cookies();

        Connection con2 = Jsoup.connect("http://****.edu.cn/jiaoshi/bangong/check.asp");
        Map<String, String> data = new HashMap<>(); //  设置post的数据

        data.put("user", "201*****");
        data.put("lb", "S");
        data.put("pwd", "******");
        data.put("sign", "c4f986961b0ffe8d521548678ae5b586");
        con.data(data);
        Connection.Response login = con2.ignoreContentType(true)
                .postDataCharset("UTF-8")
                .method(Connection.Method.POST)
                .cookies(cookies)
                .data(data)
                .header("Upgrade-Insecure-Requests", "1")
                .execute();
        login.charset("UTF-8");
        //  Document doc = login.parse();
        //  System.out.println(doc);
        Map<String, String> cookie = login.cookies();
        cookie.forEach(new BiConsumer<String, String>() {
            @Override
            public void accept(String s1, String s2) {
                try {
                    System.out.println(String.format("%s = %s", decode(s1), decode(s2)));
                } catch (UnsupportedEncodingException e) {
                    e.printStackTrace();
                }
            }
        });
    }

    //  url解码
    private static String decode(String url) throws UnsupportedEncodingException {
        String prevURL = "";
        String decodeURL = url;
        while (!prevURL.equals(decodeURL)) {
            prevURL = decodeURL;
            decodeURL = URLDecoder.decode(decodeURL, "UTF-8");
        }
        return decodeURL;
    }

    //  url编码
    private static String encode(String url) throws UnsupportedEncodingException {
        return URLEncoder.encode(url, "UTF-8");
    }
}
```

执行结果：
![](https://s2.ax1x.com/2019/09/05/nul6KK.jpg)

可以看到里面还是有一些很重要的数据，包括密码！！！通过本文的阅读你不但可以了解模拟登录和爬虫相关的知识，同时你也应该具备有一定的安全意识，不只是学校的网站，千万不要随便在别人网站上登录，因为很可能你一登录自己的信息就被别人记录咯~~~

拓展学习：IP伪造、反爬虫、多线程爬虫等