title: 基于spring boot创建项目模板
date: 2016-06-03 15:41:25
categories: 'spring boot'
tags: ['java', 'spring boot']
description:
---
Spring这个巨无霸因为其优良的设计和功能的完备, 已经是每个Java程序员必备的技能. 不过由于Spring的配置繁琐, 相关模块众多, 增加了框架整合的复杂度, spring boot的出现很好的解决了这些问题, 其设计目的是用来简化新的Spring应用的初始搭建以及开发过程.
<!--more-->

### spring boot
Spring这个巨无霸因为其优良的设计和功能的完备, 已经是每个Java程序员必备的技能. 不过由于Spring的配置繁琐, 相关模块众多, 增加了框架整合的复杂度, spring boot的出现很好的解决了这些问题, 其设计目的是用来简化新的Spring应用的初始搭建以及开发过程.

**spring boot的特点**
1. 可以创建web和non-web类型的项目
2. 嵌入式servlet容器, 直接使用fat Jar部署, 无需部署War包
3. 自动化分散配置Spring
4. 简化Maven配置
5. 提供production-ready特性, 比如健康指标, 度量数据等

**spring boot常用组件**  
spring boot对常用模块进行了组件化, 整合起来非常容易, 下面是几个常用的spring boot组件  
- spring-boot-starter-web:支持全栈web开发，里面包括了Tomcat和Spring-webmvc。
- spring-boot-starter-mail:提供对javax.mail的支持.
- spring-boot-starter-ws: 提供对Spring Web Services的支持
- spring-boot-starter-test:提供对常用测试框架的支持，包括JUnit，Hamcrest以及Mockito等。
- spring-boot-starter-actuator:支持产品环境下的一些功能，比如指标度量及监控等。
- spring-boot-starter-jetty:支持jetty容器。
- spring-boot-starter-log4j:引入默认的log框架（logback）

### 项目模板的意义
公司内部的各个开发团队因为技术的偏好以及擅长的技术不同, 在开发项目时都有各自的考虑和选型, 这种情况导致了大部分的项目使用的框架是不同的, 增加了新人上手项目的难度, 而且各个开发团队在重复的造轮子, 技术的积累比较薄弱.  
正确的做法是应该有一套合理高效的统一使用的项目模板, 规范开发流程和技术选型, 将主要的精力放在业务的开发上, 而不是重复造轮子.

### demo项目介绍
#### spring-boot-base目录结构
![image](http://7ls0sn.com1.z0.glb.clouddn.com/spring-boot-base.jpg-w980)
**补充说明**
- 过滤器  
添加过滤器的方法, 除了demo中提供的配置方式, 也可以在过滤器上直接添加@WebFilter注解实现
- 内嵌jetty
spring boot默认内嵌的是tomcat, 通过修改pom.xml文件, 替换tomcat
 ```xml
......
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
......
```
**jetty高级配置**
```java
@Configuration
public class JettyConfig {

    @Bean
    public EmbeddedServletContainerFactory servletContainerFactory(
            @Value("${jetty.threadPool.maxThreads:256}") final String maxThreads,
            @Value("${jetty.threadPool.minThreads:8}") final String minThreads,
            @Value("${jetty.threadPool.idleTimeout:60000}") final String idleTimeout) {
        JettyEmbeddedServletContainerFactory jettyEmbeddedServletContainerFactory = new JettyEmbeddedServletContainerFactory();
        jettyEmbeddedServletContainerFactory.addServerCustomizers(new JettyServerCustomizer() {

            @Override
            public void customize(Server server) {
                final QueuedThreadPool threadPool = server.getBean(QueuedThreadPool.class);
                threadPool.setMaxThreads(Integer.valueOf(maxThreads));
                threadPool.setMinThreads(Integer.valueOf(minThreads));
                threadPool.setIdleTimeout(Integer.valueOf(idleTimeout));
            }
        });

        return jettyEmbeddedServletContainerFactory;
    }

}
```
- 打包  
spring boot配合maven可以很容易构建可执行jar包, 如果要构建一个普通的jar包, 非可执行的, 需要在pom.xml中增加以下代码
```xml
......
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <classifier>exec</classifier>
    </configuration>
</plugin>
......
```
执行 *mvn clean package* 会生成两个jar包, xxxx-exec.jar是可执行jar包, xxxx.jar是普通的jar包

#### 整合Druid数据库连接池以及配置相关监控
目前来看, Druid是Java语言中最好的数据库连接池，并且能够提供强大的监控和扩展功能。
在repository模块的pom.xml引入Druid的依赖
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.0.20</version>
</dependency>
```
使用spring-boot的自动配置, 将默认数据库连接池替换为Druid
```java
//MyBatisConfig.java
......
@Bean
public DataSource dataSource() {
    DruidDataSource druidDataSource = new DruidDataSource();
    try {
        druidDataSource.setUrl(jdbcUrl);
        druidDataSource.setUsername(username);
        druidDataSource.setPassword(password);
        druidDataSource.setMaxActive(maxActive);

        druidDataSource.setFilters("stat");
        druidDataSource.setInitialSize(1);
        druidDataSource.setMaxActive(60000);
        druidDataSource.setMinIdle(1);
        druidDataSource.setTimeBetweenEvictionRunsMillis(60000);
        druidDataSource.setMinEvictableIdleTimeMillis(300000);
        druidDataSource.setTestWhileIdle(true);
        druidDataSource.setTestOnBorrow(false);
        druidDataSource.setTestOnReturn(false);
        druidDataSource.setValidationQuery("select 'x'");
        druidDataSource.setPoolPreparedStatements(true);
        druidDataSource.setMaxOpenPreparedStatements(20);

    } catch (SQLException e) {
        e.printStackTrace();
    }

    return druidDataSource;
}
......

```
配置Druid的监控Controller, 在web模块下找到DruidStatViewController
```java
@WebServlet(urlPatterns = "/druid/*",
        initParams={
                @WebInitParam(name="allow",value=""),// IP白名单 (没有配置或者为空，则允许所有访问)
                @WebInitParam(name="deny",value=""),// IP黑名单 (存在共同时，deny优先于allow)
                @WebInitParam(name="loginUsername",value="atlas"),// 用户名
                @WebInitParam(name="loginPassword",value="atlas"),// 密码
                @WebInitParam(name="resetEnable",value="false")// 禁用HTML页面上的“Reset All”功能
        })
public class DruidStatViewController extends StatViewServlet {

}
```
在浏览器打开*http://localhost:9090/boom/druid/index.html*
![image](http://7ls0sn.com1.z0.glb.clouddn.com/druid.jpg-w980)

#### 整合mybatis
在repository模块下找到config包
```java
//MyBatisConfig.java
......
@Bean(name = "sqlSessionFactory")
public SqlSessionFactory sqlSessionFactoryBean() throws Exception {
    SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
    sqlSessionFactoryBean.setDataSource(dataSource());
    PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();

    sqlSessionFactoryBean.setMapperLocations(resolver.getResources("classpath:/mybatis/*.xml"));

    return sqlSessionFactoryBean.getObject();
}
......

```
```java
//MyBatisMapperScannerConfig.java
@Configuration
@MapperScan("com.boom.base.repository.mapper")
@AutoConfigureAfter({ MyBatisConfig.class, MyBatisConfigDevelopment.class })
public class MyBatisMapperScannerConfig {

    public MapperScannerConfigurer mapperScannerConfigurer() {
        MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
        mapperScannerConfigurer.setSqlSessionFactoryBeanName("sqlSessionFactory");
        mapperScannerConfigurer.setBasePackage("com.boom.base.repository.mapper");
        return mapperScannerConfigurer;
    }
}

```

### 最后
项目位置: [spring-boot-base](https://github.com/boomya/spring-boot-base)   
这个项目只是用来抛砖引玉, 如果能给大家带来一点想法是最好的, 希望大家多提宝贵意见, 谢谢.
