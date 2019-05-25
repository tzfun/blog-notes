这篇博客主要记录一下我平时怎么搭建服务端的，从大二开始接触到服务器，经过了一年踩了很多坑，现在摸索到了一些门路，当然肯定我的处理办法不是最好的，学习ing，欢迎朋友们和我多多交流。
# 一个端口一个服务
大一的时候我主要使用的是Apache和Tomcat两种服务器，当时因为不太了解linux操作系统（服务器最好使用linux），所以是用的Windows Server 2016。记得当时在用PHP做网站时，用的是wampserver这个软件，而且它只能运行在windows环境下。安装软件后运行apache就可以了，如果要修改端口只需要配置相应的配置文件就可以了。

加入WingStudio之后我转向了Java，平时用的更多的就是Tomcat了，这个也很简单，将java程序的war包放进Tomcat的webapp目录下，然后再在其bin目录下运行startup.bat就可以了（linux运行startup.sh），如果要修改访问端口只需要在conf/server.xml里面找到相应配置就行了。

一个简单的web服务最好理解，应该也是初学者最容易接收的。
![20180923201743.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180923201743.jpg)

# 多个端口多个服务
其实这个就是搭建多个简单的服务，这里提一下主要是引出后面nginx和docker的一些优势。
如果我现在有五个服务需要放在服务器上，在不使用任何其他工具的情况下，那么我就需要在服务器上启动五个服务，并且为每个服务开启一个端口。这样很浪费资源，但是也是我曾经尝试过的，没尝试过怎么知道其中的利与弊呢？来看看它的一个结构图：
![20180923202846.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180923202846.jpg)

# 同一端口多个服务
当我们有很多服务需要挂载在服务器上，但是我又想通过一个端口去访问不同服务，并且在一个服务器（以Tomcat为例）的前提下，就可以将多个服务放在服务器下，并且做端口映射（如果是8080端口Tomcat自带，如果是其他端口需要在server.xml中配置，这个配置贼恼火，亲测！！），这种方式也是我之前用的最多的。其结构图如下：
![20180923203457.jpg](
https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180923203457.jpg)

使用这种方式很容易遇到一个问题，也是我曾经遇到过的，就是Tomcat内存不足！！！当时看到这个错误的时候也很懵逼，尝试着给Tomcat多分配一点内存，虽然这个能解决问题，但是还是会出现一些问题，比如Tomcat服务器压力太大，几个服务同时运行的时候，如果你的服务还很大，Tomcat很可能会GG，为了解决这个问题可以用消耗操作系统资源空间的方法来增强你服务的性能。

# 使用nginx服务器
如果你现在有两个服务需要上线，只有一个备案域名，一台服务器主机，个人又有强迫症，不喜欢在访问时带上端口访问，为了减轻Tomcat压力不能使用上面“同一端口多个服务”的方法，那么这种情况下，一个Tomcat服务器就无法满足需求。我之前在配置服务器的时候也是这种情况，通过师兄介绍Nginx服务器可以做反向代理，这就很不错了呀，于是尝试着去学了一下。

## nginx简介
先copy一下百度百科对nginx的介绍
> Nginx (engine x) 是一个高性能的HTTP和反向代理服务，也是一个IMAP/POP3/SMTP服务。Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点（俄文：Рамблер）开发的，第一个公开版本0.1.0发布于2004年10月4日。

> Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，并在一个BSD-like 协议下发行。其特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。

如果要做服务运维的童鞋nginx是必学的，但是作为开发者来说可能说不必学得很深入，但是基本原理与应用还是需要掌握的，接下来谈谈我对nginx的理解。

从我平时应用来讲，nginx最大的优点在于`负载均衡`、`反向代理`和`静态文件分发`，接下来说一下我平时如何使用它的，肯定有很多不足之处，望大佬们提点。

## nginx反向代理

### 什么是反向代理
反向代理（Reverse Proxy）方式是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器；并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。 通常的代理服务器，只用于代理内部网络对Internet的连接请求，客户机必须指定代理服务器,并将本来要直接发送到Web服务器上的http请求发送到代理服务器中。当一个代理服务器能够代理外部网络上的主机，访问内部网络时，这种代理服务的方式称为反向代理服务。
![20180923210837.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180923210837.jpg)

### nginx反向代理工作原理
（此部分内容转自https://www.cnblogs.com/anruy/p/4989161.html）

 反向代理服务器通常有两种模型，它可以作为内容服务器的替身，也可以作为内容服务器集群的负载均衡器。

#### 一、作内容服务器的替身

如果您的内容服务器具有必须保持安全的敏感信息，如信用卡号数据库，可在防火墙外部设置一个代理服务器作为内容服务器的替身。当外部客户机尝试访问内容服务器时，会将其送到代理服务器。实际内容位于内容服务器上，在防火墙内部受到安全保护。代理服务器位于防火墙外部，在客户机看来就像是内容服务器。

当客户机向站点提出请求时，请求将转到代理服务器。然后，代理服务器通过防火墙中的特定通路，将客户机的请求发送到内容服务器。内容服务器再通过该通道将结果回传给代理服务器。代理服务器将检索到的信息发送给客户机，好像代理服务器就是实际的内容服务器（参见图 2）。如果内容服务器返回错误消息，代理服务器会先行截取该消息并更改标头中列出的任何 URL，然后再将消息发送给客户机。如此可防止外部客户机获取内部内容服务器的重定向 URL。

这样，代理服务器就在安全数据库和可能的恶意攻击之间提供了又一道屏障。与有权访问整个数据库的情况相对比，就算是侥幸攻击成功，作恶者充其量也仅限于访问单个事务中所涉及的信息。未经授权的用户无法访问到真正的内容服务器，因为防火墙通路只允许代理服务器有权进行访问。

![20180923211817.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180923211817.jpg)
 可以配置防火墙路由器，使其只允许特定端口上的特定服务器（在本例中为其所分配端口上的代理服务器）有权通过防火墙进行访问，而不允许其他任何机器进出。
#### 二、作为内容服务器的负载均衡器
可以在一个组织内使用多个代理服务器来平衡各 Web 服务器间的网络负载。在此模型中，可以利用代理服务器的高速缓存特性，创建一个用于负载平衡的服务器池。此时，代理服务器可以位于防火墙的任意一侧。如果 Web 服务器每天都会接收大量的请求，则可以使用代理服务器分担 Web 服务器的负载并提高网络访问效率。

对于客户机发往真正服务器的请求，代理服务器起着中间调停者的作用。代理服务器会将所请求的文档存入高速缓存。如果有不止一个代理服务器，DNS 可以采用“循环复用法”选择其 IP 地址，随机地为请求选择路由。客户机每次都使用同一个 URL，但请求所采取的路由每次都可能经过不同的代理服务器。

可以使用多个代理服务器来处理对一个高用量内容服务器的请求，这样做的好处是内容服务器可以处理更高的负载，并且比其独自工作时更有效率。在初始启动期间，代理服务器首次从内容服务器检索文档，此后，对内容服务器的请求数会大大下降。
![20180923211923.jpg](
https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180923211923.jpg)

### nginx配置
如果有多个服务的话，不建议直接在`nginx.conf`这个配置文件下配置你的服务，最好在nginx根目录下的`conf.d`文件中添加你的配置文件，当然这里需要吧`conf.d`文件下的所有配置文件引入到`nginx.conf`中去。在`nginx.conf`中http配置中加入这句话
```
    include /etc/nginx/conf.d/*.conf;
```
比如我有两个服务分别是VR360和blog，那么我只需要在`conf.d`中创建两个文件`VR360.conf`和`blog.conf`就可以了，各自服务的配置分别放在各自的配置文件里就可以了，方便管理，而且还不容易出错。下面引入刚刚那位大佬博客中的配置模板
```
server {
	    listen   80;
	    root /root/nmapp2_venv;
	    index index.py index.html;
	  
	    server_name server;
	  
	    location / {
	        #if (!-e $request_filename) {
	        #    rewrite ^/(.*)$ /index.py/$1 last;
	        #}
            # 这里配置转发到哪个服务
            proxy_pass  http://localhost:8080/;
	  
	        #Proxy Settings
	        proxy_redirect     off;
	        proxy_set_header   Host             $host;
	        proxy_set_header   X-Real-IP        $remote_addr;
	        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
	        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
	        proxy_max_temp_file_size 0;
	        proxy_connect_timeout      90;
	        proxy_send_timeout         90;
	        proxy_read_timeout         90;
	        proxy_buffer_size          4k;
	        proxy_buffers              4 32k;
	        proxy_busy_buffers_size    64k;
	        proxy_temp_file_write_size 64k;
	    }
	  
	    location ~ /index\.py {
	        proxy_pass_header Server;
	        proxy_set_header Host $http_host;
	        proxy_set_header X-Real-IP $remote_addr;
	        proxy_set_header X-Scheme $scheme;
	        proxy_pass http://tornado;
	    }
```

## nginx负载均衡

网站的访问量越来越大，服务器的服务模式也得进行相应的升级，比如分离出数据库服务器、分离出图片作为单独服务，这些是简单的数据的负载均衡，将压力分散到不同的机器上。有时候来自web前端的压力，也能让人十分头痛。怎样将同一个域名的访问分散到两台或更多的机器上呢？这其实就是另一种负载均衡了，nginx自身就可以做到，只需要做个简单的配置就行。

nginx不单可以作为强大的web服务器，也可以作为一个反向代理服务器，而且nginx还可以按照调度规则实现动态、静态页面的分离，可以按照轮询、ip哈希、URL哈希、权重等多种方式对后端服务器做负载均衡，同时还支持后端服务器的健康检查。
Nginx负载均衡一些基础知识：

nginx 的 upstream目前支持 4 种方式的分配 
> 1. 轮询（默认） 
　　每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。 
> 2. weight 
　　指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。 
> 3. ip_hash 
　　每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。  
> 4. fair（第三方） 
　　按后端服务器的响应时间来分配请求，响应时间短的优先分配。
![20180923221612.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180923221612.jpg)
当然做负载均衡一般都是在服务比较大、访问请求多的情况下会用到。我也只有一台服务器，暂时需求量没那么大，所以暂时会用不到，做个笔记相信以后会用到的
```
upstream  vr.beifengtz.com
{
    server   主机IP1:端口;
    server   主机IP2:端口;
}
 
upstream  blog.beifengtz.com   
{
    server   主机IP1:端口;
    server   主机IP2:端口;
}
 
server
{
    listen  80;
    server_name  vr.beifengtz.com;
 
    location / {
        proxy_pass        https://vr.beifengtz.com;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}
 
server
{
    listen  80;
    server_name  blog.beifengtz.com;
 
    location / {
        proxy_pass        http://blog.beifengtz.com;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}
```

## nginx文件分发

### MVC与MVVC
一般做网站目前有两种主要模型：MVC、MVVC，两者目前使用的都非常多，VR360是MVVC模型，本站博客系统是直接用的tale的，他是使用的MVC模型。我平时更喜欢使用MVVC模型，当然各有各的好处吧，先解释一下MVC和MVVC模型有什么区别：
> MVC模型是Model、View、Control三个单词的首字母，很明显它将整个系统分成了三个模块，做过开发的人都清楚这个我就不细说了，我们重点关注V和MC之间吧，MVC模型是将View层的数据渲染直接放在服务端，渲染好了之后直接将静态文件发送给客户端进行显示。而MVVC模型则有所不同，它又称双MVC模型，相当于客户端、服务端均是一个MVC模型，View层更多的是在客户端进行渲染。**归根结底一句话：MVC相对更安全，但是会增加服务器压力；MVVC没有MVC安全，但是把数据渲染的工作交给每个客户端，为服务器减轻了很大的压力。**

![20180923222236.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180923222236.jpg)
### nginx解决跨域问题
上面介绍MVC和MVVC区别的时候就有一点，MVC模型下的系统服务器压力会比较大，这是因为静态文件和服务处理都是放在同一个服务器下的，那么如果不想单独开一个Tomcat服务器，这时候就可以用nginx来分发静态文件了，直接将静态文件放在nginx服务器下，Tomcat的任务就只有服务处理了，它的压力会减轻一点。

平时在做网站开发的时候（特别是前后台分离开发）都会遇到一个头疼的问题：*请求跨域被拒绝！*。

首先什么是跨域？
> 指的是浏览器不能执行其他网站的脚本。它是由浏览器的同源策略造成的，是浏览器施加的安全限制。域名、端口不一样都属于跨域，XMLHttpRequest请求属于跨域（也就是AJAX）。

那么如何解决跨域问题呢？一般有三种方法：
> 1. 服务端在配置过滤器的时候配置允许所有域访问，不推荐，因为不安全。
> 2. 前后台数据用**JSONP**格式传输，它是将数据包装在JS脚本中，让浏览器以为此数据是js文件，浏览器就会放行。
> 3. 使用nginx代理，静态文件放在nginx中，服务端与nginx放在同一个服务器主机里，这时请求的域和服务端的域是在同一主机下并且用nginx做过反向代理，就相当于是同一个域。

服务器这一块我是一直采坑过来的，搞不好自己现在还在某个坑里，还有很多的内容需要学习，如果上述的内容有什么不对的地方欢迎向我指出，同时欢迎和我一起学习交流。ヾ(◍°∇°◍)ﾉﾞ