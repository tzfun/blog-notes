# 简介
  Maven是一款强大的项目构建工具，能帮助用户建立一套有效的自动化构建体系，是Apache的一个顶级项目（官网地址：http://maven.apache.org/）。maven的核心文件是`pom.xml`，其中的pom也就是Project Object Modal，在构建项目过程中，主需要配置`pom.xml`中项目模块间的相互依赖关系和使用相关命令就可以了，省去了之前繁琐的建构任务。

# Maven模型
![maven模型][1]

  [1]: https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180907180004.jpg

如上图所示,maven通过核心文件pom.xml项目对象模型，管理其中项目的依赖，maven搜索依赖的顺序如下：

> 1. 第1步-搜索本地存储库中的依赖关系，如果没有找到，则移动到步骤2，如果找到，则执行进一步的处理。
> 2. 步骤2-如果未找到中央存储库中的搜索依赖项，并且提到远程存储库/存储库，则移动到步骤4，如果找到，则将其下载到本地存储库以供将来参考。
> 3. 步骤3-如果没有提到远程存储库，Maven只是停止处理并抛出错误（无法找到依赖项）。
> 4. 步骤4-在远程存储库或存储库中搜索依赖项，如果找到，则将其下载到本地存储库以供将来参考，否则Maven将按预期停止处理并抛出错误（无法找到依赖项）。

最后maven会通过插件进行项目清理、编译、测试生成报告、打包，当然相应的操作需要相应的命令去执行。

# Maven基本概念

* **Project**：任何你想构建的实物，Maven都可以认为它们是工程。这些工程都会被定义为工程对象模型，也就是Project Object Modal。一个工程可以依赖其他的工程，当然它也可以有多个子工程构成。
* **POM**：`pom.xml`是Maven的核心文件，她是指示Maven如何工作的元数据文件。POM文件位于每个工程的根目录中。
* **GroupId**：`GroupId`是一个工程在全局中的唯一标识符，一般它就是工程名。GroupId有利于使用一个完全的包名将一个工程从其他有类似名字的工程中区别出来。
* **Artifact**：构件，是工程将要产生或需要使用的文件，它可以是`.jar`文件、源文件、二进制文件、`.war`文件，甚至可以是`.pom`文件。每个Artifact都由GroupId和ArtifactId组合的标识符唯一识别。需要被使用的Artifact都要放在仓库中，否则Maven无法找到它们。
* **Dependency**：为了能够构件或运行，一个java工程一般都会依赖其他包。在Maven中，这些被依赖的包被称为Dependency。Dependency一般是其他工程的Artifact。
* **Plug-in**：可以说Maven是一堆插件的集合，她的每一个功能都是由插件完成的。插件提供goal。并根据在POM中找到的元数据去完成工作。主要的Maven插件是由Java编写而成的。
* **Repository**：仓库，即放置Artifact的地方，有中央仓库、公共仓库、私有仓库及本地仓库之分。为了提高Artifact的下载速度，一般我们都会使用国内的私有仓库或者开放的共有仓库，默认是从中央仓库中拉取。

顺便提供几个Maven仓库地址：

https://search.maven.org/

http://mvnrepository.com/

http://maven.aliyun.com/mvn/view

#Maven安装

 1. 从官网下载Maven包，地址为：http://maven.apache.org/download.cgi
 2. 把下载下来的zip压缩包解压到指定目录（一般不要放在C盘）
 3. 设置“JAVA_HOME”环境变量，指定JDK安装目录。（java程序员一般在安装jdk的时候都会配置）
 4. 设置“M2_HOME”环境变量，这里指向刚刚解压的目录。
 5. 配置“Path”环境变量，把解压的Maven目录中的bin目录（%M2_HOME%\bin）添加到当前环境变量中，系统变量和用户变量都行，两者的区别在于用户使用权限，放在系统变量里所有用户都能使用，放在用户变量就只能是当前用户能使用，一般Windows都是个人电脑，不会有多用户的情况，两者均可。添加好之后在cmd中输入关键字`mvn`即可使用mvn相关命令。
 6. 为了防止内存溢出（一般个人电脑不会出现这种情况），需要设置`MAVEN_OPTS=-Xms 512m -XMX
    1024m`，这样相当于手动给Maven添加内存空间。

这里顺便说一下Maven目录结构

* **bin**：Maven的运行脚本。
* **boot**：Maven自己的类装载器。
* **conf**：全局配置文件（本地仓库地址在这里配置）。
* **lib**：Maven运行时所需的类库。

# Maven配置

## settings.xml

maven的本地仓库默认是在一个`.m2`的目录下，Windows用户默认是在`C:\Users\你的PC用户名\.m2`,如果说你不想使用默认地址，你需要在maven目录下的`conf`目录中找到`settings.xml`（保存的是本地所有项目所共享的全局配置信息）,然后修改如下代码段：

```
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
  <localRepository>你想要设置的本地仓库存放路径</localRepository>

```

当然其中settings.xml文件中还有很多配置项，这里保存一下笔记，相信在以后构建工程中会使用到。

`localRepository`：本地仓库位置，默认在.m2/repository/，可以人为更改

`offline`：离线开关，是否每次构建都从远程仓库下载，默认false

`servers`：对应POM文件的distributionManagement元素里定义id,和登陆服务器的用户名、密码

`mirrors`：定义仓库镜像，将仓库地址指向自定义仓库地址(id：新的镜像ID，name：镜像名称，url：镜像地址，mirrorOf：以那个地址做镜像，默认为central)

`proxies`：设置HTTP代理

`parent`     给出父项目的位置，如果存在上一级父项目的话。如果没有特别指出的话，值就是父项目对于当前项目而言。位置是一个 as a group ID, artifact ID 和version元素的组合。

`modelVersion`     描述这个POM文件遵从哪个版本的项目描述符.

`groupId`     针对一个项目的普遍唯一识别符。通常用一个完全正确的包的名字来与其他项目的类似名字来进行区分（比如：org.apache.maven)

`artifactId`     在给定groupID 的group里面为artifact 指定的标识符是唯一的 artifact 代表的是被制作或者被一个project应用的组件。对于Maven项目的artifact 的例子包括：一些JARs, 原代码以及二进制的发布以及WARs. 

`packaging`     这个项目生产出来的artifact 类型，举个例子 jar war  pom Plugins 能够创建他们自己的包，包括包的类型，所以这个列表不可能包含所有可能的类型 

`name`     当前项目的全称

`version`     当前项目产生的artifact的当前版本

`description`     当前项目的一个细节描述，当需要描述这个项目的时候被Maven所用，比如在web 站点中。 这个元素能够被指定为CDATA 类型，允许在描述器中HTML的tags, 并不鼓励用空白文本来表示。 如果你需要去修改生成的web 站点的索引页，你能够用你自己的索引来代替自动生成的文本。 

`url`     当前项目的主页的URL 

`prerequisites`     描述当前项目的编译环境的先决条件 

`issueManagement`    当前项目的发布管理信息。

`ciManagement`     当前项目的连续集成信息。

`inceptionYear`     当前项目开始的年份, 用4位数字描述. 涉及到介绍情况时用作提供版权信息 

`mailingLists`     包含的信息包括邮件列表

`developers`     描述当前的项目的开发人员的信息

`contributors`     描述对当前项目有贡献的人员的信息，不特指开发人员

`licenses`     这个元素描述了当前项目的所有的许可文件。每一个许可文件用一个许可元素来描述，然后描述额外的元素. 通常只列出适用于这个项目的许可文件以及适用于 依赖的非licenses。如果多个licenses都列出来了，那么假设这个用户选择其中的所需的，而不是接受所有的许可文件。 

`scm`     指定当前项目中的版本控制工具，比如CVS, Subversion, 等等。

`organization`     这个元素描述这个项目所属组织的各种属性的描述。这些属性应用于文档创建的时候 (版权通知和链接). 

`build`     创建项目时必须的信息。

`profiles`     本地项目编译档案文件时的列表，被激活时会修改build的过程 

`modules`     模块 (有时被叫做子项目)作为当前项目的一部分.每一个被列出来的子模块都指向包含这个模块的目录文件的相对路径 

`repositories`     发现依赖和扩展的远程资源库

`pluginRepositories`     发现plugins 的远程资源库的列表，主要是为了编译和报告

`dependencies`     这个元素描述了所有与当前项目相关的依赖.这些依赖被用作创建一个编译时的路径. 他们被自动的从资源库中下在下来依据当前项目的定义。如需更多信息，参看 the dependency mechanism 

`reports`     Deprecated.禁止适用。现在的版本中被 Maven所忽略掉。

`reporting`     这个元素包括报告的plugins 的指定，用作Maven生成站点的自动生成报告.这些报告将会运行当用户执行mvn site. 所有的报告将会包括在浏览器的导航栏中。 

`dependencyManagement`     缺省的依赖信息将会从这个元素中继承。这些依赖在这一部分中被不立刻被解决的。当一个源于这个POM的元素描述了一个依赖根据匹配的 groupId 和artifactId,这个部分的版本和其他值用作那些还没有指定的依赖。

`distributionManagement`     对于一个项目分配的信息允许对于远程web服务器和资源库的site和artifacts配置。


## pom.xml

**pom.xml**文件是平时接触最多也是使用最多的配置文件，其中允许有如下配置项：

### 依赖配置

`groupId`:项目或者组织的唯一标志，并且配置时生成的路径也是由此生成，如org.codehaus.mojo生成的相对路径为：/org/codehaus/mojo

`artifactId`: 项目的通用名称

`version`:项目的版本

`packaging`: 打包的机制，如pom, jar, maven-plugin, ejb, war, ear, rar, par

`classifier`: 分类

### pom关系

    POM关系主要为**依赖**、**继承**、**合成**

#### 依赖关系

看如下代码

```
<dependencies>
    <dependency>
      <groupId>beifeng.jms<groupId>
      <artifactId>jms<artifactId>
      <version>1.0<version>
      <type>jar<type>
      <scope>test<scope>
      <optional>true<optional>
    <dependency>
    ...
<dependencies>
```
```
mvn install:install-file

-DgroupId=javax.jms      这个和自己的pom.xml中的中的一样就行

-DartifactId=jms         这个和自己的pom.xml中的中的一样就行

-Dversion=1.0          这个和自己的pom.xml中的中的一样就行

-Dfile=D:\jms-1.0.jar   （自己要添加的jar包，例如我放在D盘）

-Dpackaging=jar

```

`type`:相应的依赖产品包形式，如jar，war
`scope`:用于限制相应的依赖范围，包括以下的几种变量：
`compile` ：默认范围，用于编译
`provided`：类似于编译，但支持你期待jdk或者容器提供，类似于classpath
`runtime`:在执行时，需要使用
`test`:用于test任务时使用
`system`:需要外在提供相应得元素。通过systemPath来取得
`systemPath`: 仅用于范围为system。提供相应的路径
`optional`: 标注可选，当项目自身也是依赖时。用于连续依赖时使用

通过以上命令就可以把自己的jar包导入到maven中的repository中

**独占性**
外在告诉maven你只包括指定的项目，不包括相关的依赖。此因素主要用于解决版本冲突问题
```
<dependencies>
    <dependency>
      <groupId>org.apache.maven<groupId>
      <artifactId>maven-embedder<artifactId>
      <version>2.0<version>
      <exclusions>
        <exclusion>
          <groupId>org.apache.maven<groupId>
          <artifactId>maven-core<artifactId>
        <exclusion>
      <exclusions>
    <dependency>
<dependencies>
```
*表示项目maven-embedder需要项目maven-core，但我们不想引用maven-core*

#### 继承关系

先定义父项目如下：
```
<project>
<modelVersion>4.0.0<modelVersion>
<groupId>org.codehaus.mojo<groupId>
<artifactId>my-parent<artifactId>
<version>2.0version>
<packaging>pom<packaging>
<project>
```
packaging 类型，需要pom用于parent和合成多个项目。我们需要增加相应的值给父pom，用于子项目继承。

如果想父项目，配置如下：
```
<project>
<modelVersion>4.0.0<modelVersion>
<parent>
    <groupId>org.codehaus.mojo<groupId>
    <artifactId>my-parent<artifactId>
    <version>2.0<version>
    <relativePath>../my-parent<relativePath>
<parent>
<artifactId>my-project<artifactId>
<project>
```
relativePath可以不需要，因为通过artifactId就可以查询到副项目，但是用于指明parent的目录，用于快速查询。

#### 合成（或多个模块）

 一个项目有多个模块，也叫做多重模块，或者合成项目。

```
<project>
<modelVersion>4.0.0<modelVersion>
<groupId>org.codehaus.mojo<groupId>
<artifactId>my-parent<artifactId>
<version>2.0<version>
<modules>
    <module>my-project1<module>
    <module>my-project2<module>
<modules>
<project>
```
这里由`my-project1`和`my-project2`两个项目模型合成

#### build配置

主要用于编译设置，包括两个主要的元素，build和report，主要分为两部分，基本元素和扩展元素集合

##### 基本元素
```
<build>
<defaultGoal>install<defaultGoal>
<directory>${basedir}/targetdirectory>
<finalName>${artifactId}-${version}finalName>
<filters>
    <filter>filters/filter1.properties<filter>
<filters>
...
<build>
```
`defaultGoal`: 定义默认的目标或者阶段。如install
`directory`: 编译输出的目录
`finalName`: 生成最后的文件的样式
`filter`: 定义过滤，用于替换相应的属性文件，使用maven定义的属性。设置所有placehold的值

##### 资源（resource）

当你项目中需要指定的资源时，就需要对其进行相应的配置，如spring配置文件,log4j.properties
```
<project>
<build>
    ...
    <resources>
      <resource>
        <targetPath>META-INF/plexus<targetPath>
        <filtering>falsefiltering>
        <directory>${basedir}/src/main/plexus<directory>
        <includes>
          <include>configuration.xml<include>
        <includes>
        <excludes>
          <exclude>**/*.properties<exclude>
        <excludes>
      <resource>
    <resources>
    <testResources>
      ...
    <testResources>
    ...
<build>
<project>
```
`resources`: resource的列表，用于包括所有的资源
`targetPath`: 指定目标路径，用于放置资源，用于build
`filtering`: 是否替换资源中的属性placehold
`directory`: 资源所在的位置
`includes`: 样式，包括那些资源
`excludes`: 排除的资源
`testResources`: 测试资源列表

#### 插件

在build时，执行的插件，比较有用的部分

```
<project>
<build>
    ...
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins<groupId>
        <artifactId>maven-jar-plugin<artifactId>
        <version>2.0<version>
        <extensions>false<extensions>
        <inherited>true<inherited>
        <configuration>
          <classifier>test<classifier>
        <configuration>
        <dependencies>...<dependencies>
        <executions>...<executions>
      <plugin>
    <plugins>
<build>
<project>
```

`extensions`: true or false，是否装载插件扩展。默认false
`inherited`: true or false，是否此插件配置将会应用于poms，那些继承于此的项目
`configuration`: 指定插件配置
`dependencies`: 插件需要依赖的包
`executions`: 用于配置execution目标，一个插件可以有多个目标。
如下代码：

```
<plugin>
        <artifactId>maven-antrun-plugin<artifactId>

        <executions>
          <execution>
            <id>echodirid>
            <goals>
              <goal>run<goal>
            <phase>verify<phase>
            <inherited>false<inherited>
            <configuration>
              <tasks>
                <echo>Build Dir: ${project.build.directory}<echo>
              <tasks>
            <configuration>
          <execution>
        <executions>
     <plugin>
```
`id`:规定execution 的唯一标志
`goals`: 表示目标
`phase`: 表示阶段，目标将会在什么阶段执行
`inherited`: 和上面的元素一样，设置false maven将会拒绝执行继承给子插件
`configuration`: 表示此执行的配置属性

#### Organization 配置组织信息

```
<organization>
    <name>wingstudio</name>
    <url>http://www.wingstudio.org</url>
organization>
```

#### Developers 配置开发者信息

```
<developers>
    <developer>
      <id>beifengtz<id>
      <name>beifeng<name>
      <email>tz1112tz@163.com<email>
      <url>http://blog.beifengtz.com<url>
      <organization>wingstudio</organization>
      <Url>http://www.wingstudio.org<Url>
      <roles>
        <role>architect<role>
        <role>developer<role>
      <roles>
      <timezone>-6timezone>
      <properties>
        <picUrl>http://blog.beifengtz.com<Url>
      <properties>
    <developer>
<developers>
```