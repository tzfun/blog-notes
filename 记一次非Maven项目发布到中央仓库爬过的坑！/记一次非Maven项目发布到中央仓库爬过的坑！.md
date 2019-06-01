今天有朋友和我聊天说笔者已经有两天没有写文章了，都去干嘛了？我很无奈地回答说被maven弄自闭了。到底是什么样的问题导致我花了这么长时间去解决一个问题？这篇文章告诉你。

# 一、我想要做的事

笔者前几天一直在看Hotspot提供的tools包的源码，并试图去进行了改造和拓展，最后小有成就完成了第一个版本的虚拟机监控API：vmconsole，这不是重点，重点是在后面。我将它打成了jar包，并且想分享出去，让其他开发者也能使用，第一时间想到的就是Maven，如果我把它发布到Maven仓库不就可以了吗？于是开始笔者便开始了爬坑之路。

笔者在这里先为自己的作品打个预广告，欢迎广大编程爱好者使用以及提供意见和建议，同时笔者希望能得到你的小星星鼓励O(∩_∩)O，后续也会有专门的文章介绍该作品。

**JVM虚拟机监控API：VmConsole-Api**

* Github：[https://github.com/tzfun/VmConsole-Api](https://github.com/tzfun/VmConsole-Api)
* API Document：[doc.vmconsole.beifengtz.com](http://doc.vmconsole.beifengtz.com/)

# 二、遇到的问题

这是笔者第一次将自己的项目发布到Maven仓库中，其中的步骤都不清楚，于是我就各种谷歌百度，实在解决不了就找其他人求助，简单的问题成功得以解决，可最关键的问题最后还是自己解决的，接下来就抛出我遇到的问题吧，答案以及解决办法后面会给出。

* **发布到哪个Maven仓库？**
> Maven项目的信息及配置由pom.xml来存储，其中包含了项目信息、开发者信息、依赖、插件等，如果想要获取其他人或组织开发的jar包，那么就要引入我们再熟悉不过的**依赖**了，依赖需要从仓库下载，下载流程如下：
> 1. 检索本地仓库，如果有该依赖返回成功，否则进入下一步；
> 2. 检索私有仓库，如果有该依赖返回成功，否则进入下一步；
> 3. 检索中央仓库，如果有该依赖返回成功，否则返回失败。
>
> 本地仓库是在每个开发者电脑上，一般在`c:/user/.m`文件目录下；私有仓库是个人或组织建立的仓库，需要在全局settings.xml中配置仓库地址；中央仓库算是整个Maven存储的核心仓库，所有开发者都可以在其中获取资源。那么这个问题就解决了，为了让开发者方便地获取该依赖，我需要把jar包发布到中央仓库。

* **如何将一个非Maven项目发布到中央仓库？**

这个问题是笔者遇到的主要问题，我的项目并不是Maven项目，网上一大堆的教程全都是**Maven项目**如何发布到中央仓库，如果我的是Maven项目也不至于忙活这么久，我的项目正是非Maven项目，一个纯Java的，其中还包含了jdk的jar包，正因为jdk提供的这些jar包在Maven仓库中没有，所以我无法把我的项目改成Maven项目，其中的很多类必须依托于这些jar包。所以现在的问题就转换成了**如何把已经打包好了的jar包发布到中央仓库**。

> 简述一下我的做法吧，下面会给出完整的步骤，**这也是笔者目前为止想到的唯一的办法，如果读者有更好的方法，希望你和我联系一下，笔者也学习学习**。要传到中央仓库还是得需要Maven项目，所以我新建了一个空的Maven项目，然后将jar包先安装到本地仓库，在这个空的Maven项目中引入这个jar的本地依赖，再在pom中配置assembly插件把依赖包合并到一个jar包然后发布（启发来源于Springboot，因为Springboot的Maven项目中有配置这个插件），最后将整个项目发布到nexus中（仓库管理中心），在引入依赖的时候不能单纯的引入这个项目，而是引入项目中合并的那个jar，在依赖中配置classifier即可。

# 三、完整步骤

接下来贴出我的完整步骤，其中代码部分有点长，贴出代码的目的是方便有遇到同样情况的人粘贴复制内容，读者大可略过，只看文字或图片。

## 1. 注册sonatype账号并创建issue
首先需要在[https://issues.sonatype.org/secure/Dashboard.jspa](https://issues.sonatype.org/secure/Dashboard.jspa)注册账号，Sonatype通过JIRA来管理OSSRH仓库。

需要填写Email, Full Name, Username以及password，**其中Username与Password后面的步骤需要用到**，请记下来，后面需要用到。

注册成功之后创建一个issue

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20190601192233.png)

* Project：项目默认值，我的选择是：Community Support - Open Source Project Repository Hosting (OSSRH)
* IssueType：默认值x，我的选择是：New Project
* Summary：说明，请介绍一下项目
* GroupId：项目id，根据你的网站域名来，我第一次也是在这里出错，比如我的github是github.com/tzfun，那么groupId就是com.github.tzfun。
* ProjectURL：访问项目的URL，我写的是github中这个项目的地址。
* SCMurl：访问项目的URL，我写的是github仓库地址。
* 其他的根据提示填写

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20190601192315.png)

填写完成之后就提交，中间可能你填写的issue会有问题，需要和客服交流一下，最后出现下面的回复说明已经创建成功了。现在你就可以往nexus中上传项目了

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20190601193028.png)

## 2.安装并配置GPG
发布到Maven仓库中的所有文件都要使用GPG签名，以保障完整性。如果没有GPG前面的项目无法发布到中央仓库，在后面的**close阶段**会被阻止。

IOS系统请下载GPG Suite：[https://gpgtools.org/](https://gpgtools.org/)

Windows系统请下载gpg4win：[https://www.gpg4win.org/download.html](https://www.gpg4win.org/download.html)

安装好了之后你可以通过图形界面生成秘钥，也可以通过命令行生成，图形界面操作简单易懂就不演示了，下面演示命令行：

1. 生成密钥对：`gpg --gen-key`，此过程会让你输入用户名密码，用户名随便填即可，**密码请务必记住，因为后面每次部署发布的时候都会要输入此密码**；
2. 查看公钥id：`gpg --list-keys`，下一步需要用到；
3. 将公钥id发布到GPG秘钥服务器：`gpg--keyserver hkp://keyserver.ubuntu.com:11371 --send-keys xxxxxxxxxxxxxxxxxxxx`，假设我的公钥id是xxxxxxxxxxxxxxxxxxxx；
4. 检查是否发送成功：`gpg --keyserverhkp://pool.sks-keyservers.net --recv-keys xxxxxxxxxxxxxxxxxxxx`，发布成功该步骤完成。

## 3.Maven全局配置
在Maven的根目录出去找`settings.xml`这个配置文件在apache-maven-3.5.3\conf目录下，打开然后找到servers标签，添加一个服务，这里的id用来唯一标识server，后面配置部署的时候会用到，用户名和密码是你在第一步注册的sonatype账号密码。因为此处填写的是明文，所以请一定注意保管好！被别人拿到了删库跑路都不是问题。
```
<servers>
    <server>
        <id>oss</id>
        <username>Fundebug</username>
        <password>passsword</password>
    </server>
</servers>
```

## 4.项目pom.xml配置
我的做法和网上一大堆的教程做法不同就在于这一步，最关键的配置也在这一步。

如果你是一个Maven项目，那么只需要在正在编写的项目中的pom.xml进行配置即可，Maven在打包上传的时候不会把依赖项打包，只会把你的项目代码打包然后上传，这些依赖项都是通过你的pom文件进行记录的，别人引入了你的依赖之后Maven会自动根据你pom的配置进行下载相关包。所以归根结底部署这一步**只会把你的项目打包成jar并上传到仓库，依赖项的jar并不会被上传！**

如果你是一个纯Java项目，**有一些依赖只有class字节码文件或者其他提供的jar包（即你所需的库在Maven仓库中找不到，但是又有相关的字节码文件或jar包），无法将你的项目改为Maven项目**，也就是和笔者情况一样，具体操作如下。

1. 创建一个新的Maven项目，其中不需要写任何代码，也不需要创建任何类，只需要配置pom中项目信息、作者信息、开源协议等信息。
```
<!-- 组织Id、项目Id、版本号 -->
<groupId>com.github.tzfun</groupId>
<artifactId>vmconsole</artifactId>
<version>1.0.0</version>

<!-- 项目描述 -->
<name>vmconsole</name>
<url>https://github.com/tzfun/VmConsole-Api</url>
<description>An API that makes it easy to monitor virtual machines</description>

<!-- 开源协议 -->
<licenses>
    <license>
        <name>Apache 2</name>
        <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
        <distribution>repo</distribution>
        <comments>A business-friendly OSS license</comments>
    </license>
</licenses>

<!-- 项目地址，这里我填写的github -->
<scm>
    <url>https://github.com/tzfun/VmConsole-Api</url>
    <connection>https://github.com/tzfun/VmConsole-Api.git</connection>
</scm>

<!-- 编码格式 -->
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>

<!-- 开发者信息 -->
<developers>
    <developer>
        <name>beifengtz</name>
        <id>beifengtz</id>
        <email>beifengtz@163.com</email>
        <roles>
            <role>Developer</role>
        </roles>
        <timezone>+8</timezone>
    </developer>
</developers>
```
2. 将你的依赖安装到本地仓库：

    1. 如果你有一些class字节码文件需要打成jar包，很简单，随便使用一个压缩工具，把你的依赖项目工程压缩成**zip**格式，注意一定是zip，因为zip压缩格式和jar格式是一样的，打包之后直接把后缀名改成jar即可。

    2. 如果你只是一些jar包，没有“散装”的class字节码文件，那么就可以直接进入下一步操作。

    3. 用Maven命令行将jar包安装到本地仓库，命令如下：
    ```
    mvn install:install-file -Dfile=D:\vmconsoel-api.jar -DgroupId=com.github.tzfun -DartifactId=vmconsole-local -Dversion=1.0.0 -Dpackaging=jar
    ```

    * -- DgroupId和DartifactId构成了该jar包在pom.xml的坐标， 对应依赖的DgroupId和DartifactId
    * -- Dfile表示需要上传的jar包的绝对路径
    * -- Dversion表示版本号
    * -- Dpackaging 为安装文件的种类，这里是jar
3. 在pom中添加本地仓库依赖

执行成功之后本地仓库就有刚刚那个项目了，接下里把这个依赖添加到pom文件中，这里的groupId、artifactId以及version和上面命令的内容一致。
```
<dependencies>
    <dependency>
        <groupId>com.github.tzfun</groupId>
        <artifactId>vmconsole-local</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>
```

4. 配置发布插件和仓库地址。

一般只需要三个插件即可，但是这里我多加了一个assembly插件。如果你本身就是Maven项目，不需要将诸多的依赖打包一起上传（通过pom可以下载依赖，没必要把文件放到仓库），所以不可以不加此插件，将这个插件注释即可。如果你和我情况一样不是Maven项目，那么就需要加入这个插件，它的作用是将所有的依赖jar包合并成一个jar包。

因为这部分内容较多所以我将我的整个pom.xml一起贴出来，不关心此处的读者可以略过看下文。
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- 组织Id、项目Id、版本号 -->
    <groupId>com.github.tzfun</groupId>
    <artifactId>vmconsole</artifactId>
    <version>1.0.0</version>

    <!-- 项目描述 -->
    <name>vmconsole</name>
    <url>https://github.com/tzfun/VmConsole-Api</url>
    <description>An API that makes it easy to monitor virtual machines</description>

    <!-- 开源协议 -->
    <licenses>
        <license>
            <name>Apache 2</name>
            <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>
            <distribution>repo</distribution>
            <comments>A business-friendly OSS license</comments>
        </license>
    </licenses>

    <!-- 项目地址，这里我填写的github -->
    <scm>
        <url>https://github.com/tzfun/VmConsole-Api</url>
        <connection>https://github.com/tzfun/VmConsole-Api.git</connection>
    </scm>

    <!-- 编码格式 -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!-- 开发者信息 -->
    <developers>
        <developer>
            <name>beifengtz</name>
            <id>beifengtz</id>
            <email>beifengtz@163.com</email>
            <roles>
                <role>Developer</role>
            </roles>
            <timezone>+8</timezone>
        </developer>
    </developers>

    <!-- 依赖 -->
    <dependencies>
        <dependency>
            <groupId>com.github.tzfun</groupId>
            <artifactId>vmconsole-local</artifactId>
            <version>1.0.0</version>
        </dependency>
    </dependencies>

    <profiles>
        <profile>
            <id>release</id> <!-- 部署要用到 -->
            <build>
                <plugins>
                    <!-- 将其他依赖打包成一个jar的插件 -->
                    <plugin>
                        <artifactId>maven-assembly-plugin</artifactId>
                        <configuration>
                            <descriptorRefs>
                                <descriptorRef>jar-with-dependencies</descriptorRef>
                            </descriptorRefs>
                        </configuration>
                        <executions>
                            <execution>
                                <id>make-assembly</id>
                                <phase>package</phase>
                                <goals>
                                    <goal>single</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-source-plugin</artifactId>
                        <version>2.2.1</version>
                        <executions>
                            <execution>
                                <phase>package</phase>
                                <goals>
                                    <goal>jar-no-fork</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <!-- Javadoc -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-javadoc-plugin</artifactId>
                        <version>2.9.1</version>
                        <executions>
                            <execution>
                                <phase>package</phase>
                                <goals>
                                    <goal>jar</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                    <!-- GPG -->
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-gpg-plugin</artifactId>
                        <version>1.6</version>
                        <executions>
                            <execution>
                                <phase>verify</phase>
                                <goals>
                                    <goal>sign</goal>
                                </goals>
                            </execution>
                        </executions>
                    </plugin>
                </plugins>
            </build>
            <distributionManagement>
                <snapshotRepository>
                    <id>oss</id>
                    <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
                </snapshotRepository>
                <repository>
                    <id>oss</id>
                    <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
                </repository>
            </distributionManagement>
        </profile>
    </profiles>
</project>
```

## 5.发布到nexus

接下来**在pom.xml目录下**执行命令：
```
mvn clean deploy -P release
```

其中release不是命令，而是你的pom文件中配置的profile id。在build过程中需要让你输入gpg的密码，输入第2步*安装并配置GPG*的时候的密码即可。

## 6.前往nexus管理

登录仓库管理网站nexus：[https://oss.sonatype.org/#stagingRepositories](https://oss.sonatype.org/#stagingRepositories)，这里的用户名密码是第一步sonatype账号密码。

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20190601203040.png)

选中你上传的项目，然后先close，等差不多半分钟后如果成功Release按钮就会点亮，再点击Release按钮即可。注意close和Release过程都有相应的校验规则，必须全部符合才能通过，按照上面的步骤操作一般是没有问题的，如果在第4、5部用deploy-file命令来上传jar包那么close过程可能会失败，因为close过程中javadoc、source、gpg三个缺一不可。

如果Release成功之后等待差不多2小时就可以在中央仓库搜到自己的项目啦，地址：[https://search.maven.org/](https://search.maven.org/)

![http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20190601203740.png](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20190601203740.png)

## 7.添加你项目的依赖
如果你原本就是Maven项目那么仓库提供的依赖代码就可以成功引入了。如果你和我一样从一开始就是非Maven项目，那么你搜索到的依赖确实是你的那个项目，但是引入却不会生效，是因为没有选择classifier，进入你的项目查看项目内容可以看出有几个jar包，你在引入的时候需要选择是哪个，比如我这里就是下面的代码。
```
<dependency>
  <groupId>com.github.tzfun</groupId>
  <artifactId>vmconsole</artifactId>
  <version>1.0.3</version>
  <classifier>jar-with-dependencies</classifier>
</dependency>
```

![](http://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20190601205156.png)

# 四、总结

在不了解一个工具运作原理的情况下很难驾驭它，而我这次就是在没有清楚Maven发布流程的情况下就开始东弄西弄，所以浪费了很多时间。在中间我尝试了各种可能的操作，比如deploy-file、反编译整个jar包上传、传到其他私有云（aliyun）、命令行模拟pom等等，最后没办法还是得从了解其运行机制来想办法，笔者详细看了Maven的官方文档然后了解了Maven的几个生命周期，最后再配合其他框架的插件写法才解决了此问题。再次验证了一句话：没有内功基础的剑士练不出好剑法。编程最能体现基础的重要性，理解计算机或语言运作机理才能融汇贯通。