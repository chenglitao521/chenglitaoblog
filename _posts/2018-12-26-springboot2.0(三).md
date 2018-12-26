---
layout: post
title: springboot2.0(三)热部署devtoos
categories: [springboot]
description:  springboot2.0(三)热部署devtoos
keywords: springboot

---



我们在开发中经常修改代码后，要重启才能生效。这样比较耗时，idea下Springboot提供了几种热部署的方式，使我们在不重启服务的情况下加载修改后的代码。这里是介绍devtoos的方式。

## 热部署插件Spring-boot-devtoos的介绍

首先，上官网地址，比较容易阅读：

[devtoos](https://blog.csdn.net/Angry_Mills/article/details/80492068)

## 配置方式

1.pom文件

```xml
		<!-- 热部署模块 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<!-- 这个需要为 true 热部署才有效 -->
			<optional>true</optional>
		</dependency>
		
		
		<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<!--如果没有该项配置，devtools不会起作用，即应用不会restart -->
					<fork>true</fork>
				</configuration>
			</plugin>
		</plugins>
	    </build>
```

2.idea配置

ctr+alt+s快捷键打开设置，勾选自动编译。

 ![](https://oscimg.oschina.net/oscnet/f4314c7f7c0cae4e3df55e50b9b8e25affa.jpg)

ctr+shift+alt+/ 快捷键 ，点击register,勾选如图。

![](https://oscimg.oschina.net/oscnet/5a2118f7525d6435e37c87c97066c5f052d.jpg)

![](https://oscimg.oschina.net/oscnet/a9d625461f9f8a657cd6ff8270e7ebceb4d.jpg)



## 测试

修改完成后，测试是否生效。修改某个java文件后，1-2秒后idea自动重启，编译该文件实时生效。

**注意：这里不推荐，idea自动编译的方式。可以在写完代码后手动ctr+F9的方式，编译修改的文件。**



------

作者：狂奔的熊二   

出处：[https://kbdxe.cn](https://kbdxe.cn)
