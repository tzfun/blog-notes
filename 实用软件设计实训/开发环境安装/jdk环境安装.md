> <font color=red>请注意：本次实训jdk均使用1.8版本，如果有同学安装了**非jdk1.8版本**的请先卸载重新安装1.8版本，如果版本不同项目开发过程中会出现不兼容的情况！</font>

# 下载并安装jdk1.8
## 下载
前往[jdk官方下载](https://www.oracle.com/technetwork/java/javase/downloads/index.html)拉到最下面的*Java Archive*

![https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025122554.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025122554.jpg)

然后点击download去下载java SE 8版本，这里注意对应自己的系统位数，windows32位的选择windows x86下载，windows64位的选择windows x64下载。

![https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025122717.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025122717.jpg)
![https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025122827.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025122827.jpg)

## 安装
下载下来之后直接根据程序提示安装即可。第一次是安装 jdk ，第二次是安装 jre 。建议两个都安装在同一个java文件夹中的不同文件夹中。（不能都安装在java文件夹的根目录下，jdk和jre安装在同一文件夹会出错）

安装完之后开始配置系统环境变量，先打开资源管理器，右击此电脑打开属性出现下面界面，点开高级系统设置

![https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025124533.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025124533.jpg)

然后按照下面步骤新建JAVA_HOME变量，这里再系统变量和用户变量里面新建都可以。

![https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025124419.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025124419.jpg)

创建好JAVA_HOME之后开始配置jdk的Path路径，在Path路径中新建一个%JAVA_HOME%\bin，将其映射到bin目录下。

> 注意：如果点击编辑之后没有截图中所示弹框，而是另一种弹框，操作方法不一样，需要在变量值最后加上<b>;%JAVA_HOME%\bin</b> 一定注意分号<b>;</b>是英文分号

![https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025125344.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025125344.jpg)
## 测试
测试编译环境：在cmd中输入javac -version，如果出现*javac 1.8.0_144*（此处版本只要是1.8.0以上就行）就说明编译环境安装成功。

测试运行环境：在cmd中输入java -version，如果出现下面内容，说明运行环境安装成功
```
java version "1.8.0_144"
Java(TM) SE Runtime Environment (build 1.8.0_144-b01)
Java HotSpot(TM) 64-Bit Server VM (build 25.144-b01, mixed mode)
```
# 下载并安装Maven环境
## 下载
这次项目构建是基于Maven的，所以需要安装Maven管理工具。先前往[Maven官方下载](http://maven.apache.org/download.cgi)下载*Binary zip archive*，下载下来zip文件之后解压文件。
![https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025123503.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/%E5%AE%9E%E7%94%A8%E8%BD%AF%E4%BB%B6%E8%AE%BE%E8%AE%A1/20181025123503.jpg)

## 安装
在安装Maven之前请先安装JDK环境，确认安装好之后开始安装Maven环境，上面下载的文件解压后将解压后的文件夹放到你指定的目录，比如这里我放到C:\Program Files (x86)\Java目录下，我是和jdk放一起的。然后就可以根据上面jdk配置环境变量的步骤配置Maven环境变量，这里我就不演示截图，自行根据上面步骤进行。
> * 打开系统高级设置，选择环境变量
> * 新建一个名为*MAVEN_HOME*的环境变量，变量值为你的Maven路径，比如我这里就是C:\Program Files (x86)\Java\apache-maven-3.5.3
> * 配置Path路径，编辑Path后新建，值为%MAVEN_HOME%\bin，将其映射到bin目录下

## 测试
在cmd中输入*mvn -version*命令，如果出现下面内容说明Maven环境已经安装完成
```
Apache Maven 3.5.3 (3383c37e1f9e9b3bc3df5050c29c8aff9f295297; 2018-02-25T03:49:05+08:00)
Maven home: C:\Program Files (x86)\Java\apache-maven-3.5.3\bin\..
Java version: 1.8.0_144, vendor: Oracle Corporation
Java home: C:\Program Files (x86)\Java\jdk1.8.0_144\jre
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
```

# 下载并安装MySQL数据库
## 下载
前往[MySQL官方下载](https://dev.mysql.com/downloads/installer/)mysql 5.7版本，操作系统选择windows，位数根据自己电脑确定，下载*Windows (x86, 32-bit), MSI Installer*版本，点下载会让你登录，你选择*No thanks, just start my download.*即可，为了让大家减少其他配置，直接用此安装程序安装即可。

## 安装
此安装方法比较简单，我就不详细的介绍如何操作了，根据软件提示进行即可。不过有几点需要提示一下：
* 在安装的时候最好选择Developer Default默认安装；
* 操作到Type and Networking的时候会让你选择连接数据库的协议和端口等，这里端口就让它在默认的3306端口即可，如果端口被占用可另外选择一个端口；
* 操作到Accounts and Roles时需要谨慎填写，特别是用户名和密码，这里设置的用户名和密码是每次连接数据库需要用到的。
* 你的角色默认是root一个用户，如果想创建多个数据库用户需要选择Add User添加用户，里面会涉及到一些权限什么的，如果你不会操作可不用添加用户，安装之后可通过命令添加，此次实训也不涉及数据库多用户操作，如果有兴趣可自行学习。

## 测试
在cmd中执行*mysql -u 你的用户名 -p*然后输入你的密码，看能否成功连接进入内容，如果能连接就安装成功了。
```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 5.7.20-log MySQL Community Server (GPL)

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```