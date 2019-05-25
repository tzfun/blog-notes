简单地记录一下mybatis、druid在springboot中的配置，有时候记性不好容易搞忘，特别是新学的druid，记录下来方便以后翻阅。
# Mybatis
Mybatis就不用介绍了，直接上配置流程吧。

## 一、 引入Maven依赖
```
    <!-- mysql连接 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- mybatis依赖 -->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>1.3.2</version>
    </dependency>

    <!-- 分页插件 -->
    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper-spring-boot-starter</artifactId>
        <version>1.2.3</version>
    </dependency>
```
## 二、 创建mybatis-config.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--配置全局属性-->
    <settings>
        <!--使用jdbc的getGeneratedKeys获取数据库自增主键值-->
        <setting name="useGeneratedKeys" value="true"/>

        <!--使用列标签替换列别名 默认true-->
        <setting name="useColumnLabel" value="true"/>

        <!--开启驼峰命名转换-->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>

    <!--配置分页插件-->
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageInterceptor">
            <property name="helperDialect" value="mysql"/>
        </plugin>
    </plugins>
</configuration>
```
## 三、 创建entity和mapper
在resource目录下创建mapper，对应dao的xml文件就放在这里

![20180921212303.jpg](
https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180921212303.jpg)

UserDao.xml文件如下：
```
<!--头部信息，确定xml规范-->
<?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE mapper
                PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
                "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--此处是对应UserDao接口-->
<mapper namespace="org.sicau.iotworksexhibition.dao.UserDao">
    <!--此处的Student对应POJO实体，id对应定义方法名-->
<select id="queryStudentList" resultType="org.sicau.iotworksexhibition.entity.POJO.Student">
        select * from user_tb;
    </select>
</mapper>

```

创建实体存放目录，在源代码存放目录下创建entity包：
![20180921213030.jpg](
https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180921213030.jpg)

## 四、 配置application.yml文件
```
# mybatis 配置
mybatis:
  mapper-locations: classpath:mapping/*.xml # 配置mapper路径
  type-aliases-package: org.sicau.iotworksexhibition.PO # 配置实体对象路径,这样在涉及到实体类写sql语句的时候，可以不用写包名
  config-location: classpath:mybatis-config.xml # 配置配置文件路径
```

## 五、 配置MapperScan
```
@SpringBootApplication
@MapperScan("org.sicau.iotworksexhibition.dao") //扫描mybatis的mapper
public class IotWorksExhibitionApplication {
    public static void main(String[] args) {
        SpringApplication.run(IotWorksExhibitionApplication.class, args);
    }
}
```

## 六、配置数据源
可以用Spring推荐的DBCP，也可以用c3p0，但是这个是比较老的一个工具，而且各方面来说都快过时了，推荐使用阿里的druid。这里记录一下配置dbcp的application.yml配置（一般配置）
```
spring:
# 配置数据源
  datasource:
    url: jdbc:mysql://localhost:3306/iot_works_exhibition?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root  # 数据库用户名
    password: 123456 # 数据库密码
    dbcp2:
      default-auto-commit: false  # 是否自动提交
    tomcat:
      max-wait: 60000 # 等待超时时间
```

# druid
druid是阿里巴巴团队开发的一个非常优秀的数据库连接池,能提供强大的监控和拓展功能。官方地址：[http://druid.io/](http://druid.io/)
![alibaba druid](
https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180921215036.jpg)

第一次配置这个还是折腾了很久，还是记录一下吧，后面再慢慢学习。
## 一、 pom依赖
```
    <!--阿里连接池druid-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.0.18</version>
    </dependency>
```

## 二、 application.yml配置
```
spring:
# 配置数据源
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/iot_works_exhibition?useUnicode=true&characterEncoding=utf-8&useSSL=false
      username: root
      password: 123456

      # 下面为连接池的补充设置，应用到上面所有数据源中
      # 初始化大小，最小，最大
      initialSize: 5
      minIdle: 5
      maxActive: 20
      # 配置获取连接等待超时的时间
      maxWait: 60000
      # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
      timeBetweenEvictionRunsMillis: 60000
      # 配置一个连接在池中最小生存的时间，单位是毫秒
      minEvictableIdleTimeMillis: 300000
      validationQuery: SELECT 1 FROM DUAL
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false
      # 打开PSCache，并且指定每个连接上PSCache的大小
      poolPreparedStatements: true
      maxPoolPreparedStatementPerConnectionSize: 20
      # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
      filters: stat,wall #,log4j
      # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
      slowSqlMillis: 5000
      # 合并多个DruidDataSource的监控数据
      useGlobalDataSourceStat: true
```

## 三、 注入DruidDataSource
在config.dao目录下创建DruidConfiguration.java
```
package org.sicau.iotworksexhibition.config.dao;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import javax.sql.DataSource;

/**
 * @Author beifengtz
 * @Date Created in 20:26 2018/9/21
 * @Description:
 */
@Configuration
@Primary    // 标记配置，优先实现
public class DruidConfiguration {
    @Bean(name = "druidDataSource") // 此处最好设置bean名，如果是单数据源倒无所谓，但是如果是多数据源很有可能发生冲突，特别是spring内使用的dbcp容易与其发生冲突。
    @ConfigurationProperties(prefix = "spring.datasource.druid") // 配置读取哪一部分的配置
    public DataSource druidConfiguration(){
        return new DruidDataSource();
    }
}

```

## 四、 配置DruidStatViewServlet
```
package org.sicau.iotworksexhibition.config.servlet;

import com.alibaba.druid.support.http.StatViewServlet;

import javax.servlet.Servlet;
import javax.servlet.annotation.WebInitParam;
import javax.servlet.annotation.WebServlet;

/**
 * @Author beifengtz
 * @Date Created in 20:31 2018/9/21
 * @Description:
 */

@WebServlet(
        urlPatterns= {"/druid/*"},
        initParams= {
                @WebInitParam(name="allow",value="127.0.0.1"),
                @WebInitParam(name="loginUsername",value="root"),
                @WebInitParam(name="loginPassword",value="123456"),
                @WebInitParam(name="resetEnable",value="true")// 允许HTML页面上的“Reset All”功能
        }
)
public class DruidStatViewServlet extends StatViewServlet implements Servlet {
    private static final long serialVersionUID = 1L;
}

```

## 五、 配置DruidStatFilter
```
package org.sicau.iotworksexhibition.config.filter;

import com.alibaba.druid.support.http.WebStatFilter;

import javax.servlet.annotation.WebFilter;
import javax.servlet.annotation.WebInitParam;

/**
 * @Author beifengtz
 * @Date Created in 20:29 2018/9/21
 * @Description:
 */

@WebFilter(
        filterName="druidWebStatFilter",
        urlPatterns= {"/*"},
        initParams= {
                @WebInitParam(name="exclusions",value="*.js,*.jpg,*.png,*.gif,*.ico,*.css,/druid/*")//配置本过滤器放行的请求后缀
        }
)
public class DruidStatFilter extends WebStatFilter {
}

```
## 六、 配置扫描
```
package org.sicau.iotworksexhibition;

import org.mybatis.spring.annotation.MapperScan;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;

@SpringBootApplication
@ServletComponentScan //druid用于扫描所有的Servlet、filter、listener+
@MapperScan("org.sicau.iotworksexhibition.dao") //扫描mybatis的mapper
public class IotWorksExhibitionApplication {
    public static void main(String[] args) {
        SpringApplication.run(IotWorksExhibitionApplication.class, args);
    }
}
```

当我们以上配置完了就可以看效果了，我觉得druid最强大的一点就在于数据库的监控，我们启动项目，并访问http://localhost:8080/druid/login.html，会出现下面的页面，我们使用DruidStatViewServlet处配置的用户名密码登录进入。
![20180921220327.jpg](
https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180921220327.jpg)

然后我们就可以使用它强大的监听功能了
![20180921220350.jpg](
https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180921220350.jpg)

![20180921220531.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180921220531.jpg)

![20180921220631.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180921220631.jpg)

![20180921220644.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180921220644.jpg)

![20180921220654.jpg](https://vr360-beifengtz.oss-cn-beijing.aliyuncs.com/beifeng-blog/article/20180921220654.jpg)