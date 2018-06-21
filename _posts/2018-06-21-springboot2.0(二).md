---
layout: post
title:  springboot2.0系列(二)：配置属性
categories: [springboot]
description: 从0开始springboot学习系列
keywords: springboot
---
## 前言
Spring Boot中核心思想：约定优于配置。那到底什么是约定优于配置？

约定优于配置（convention over configuration），也称作按约定编程，是一种软件设计范式，旨在减少软件开发人员需做决定的数量，获得简单的好处，而又不失灵活性。

比如，我们想改tomcat的默认8080端口，我们可以在application.properties配置文件中添加 server.port=8081，即我们只需要修改没有按照约定的配置即可。


## 正文

Spring Boot中几乎所有的配置都在 application.properties 中修改，如果你想看全部的配置，可以查看下面官网链接：[属性配置](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)
只需要选择需要修改的属性配置。



### 自定义属性
在application.properties 添加自定义属性   

``` 
com.chenglt.name:城里头
com.chenglt.sex:男

```
然后在使用的地方通过@Value(value=”${config.name}”)即可获取配置属性

``` java

    @Value("${com.chenglt.name}")
    private  String name;
    @Value("${com.chenglt.sex}")
    private  String sex;
    @Test
    public void getName() {
        System.out.println(name+sex);
    }
```
如果字段太多，可以自定义类来保存，比如，我们新建ChengConfig类，顶部需要使用注解@ConfigurationProperties(prefix = “com.chenglt”)来指明使用前缀。

``` java
@ConfigurationProperties(prefix = "com.chenglt")
public class ChengConfig {

    private String name;
    private String sex;

 <!--这里省略了get和set方法-->
}

```
然后我们在入口类前加上@EnableConfigurationProperties({ChengConfig.class}) 指明加载那个类。

``` java

@SpringBootApplication
@EnableConfigurationProperties({ChengConfig.class})
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}

```
最后再使用的地方注入该对象即可。

``` java
    @Autowired
    ChengConfig configBean;
    @Test
    public void getName() {
        System.out.println(configBean.getName());
    }
```

### 自定义配置文件

有时我们我们希望把自定义的属性单独建个文件，比如现在新建myconfig.properties,放在src/main/resource下并添加上文中的两个属性：

然后新建类MyConfig,@PropertySource注解指定配置文件。

``` java
@Configuration
@ConfigurationProperties(prefix = "com.chenglt")
@PropertySource("classpath:myconfig.properties")
public class MyConfig {
    private String name;
    private String sex;
 <!--这里省略了get和set方法-->
}
```
最后再通过该bean对象获取到属性值。

作者：狂奔的熊二  

出处：[www.xiamo.club](www.xiamo.club)