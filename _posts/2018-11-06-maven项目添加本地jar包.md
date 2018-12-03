---
layout: post
title: maven项目添加本地jar包
categories: [maven]
description: maven项目添加本地jar包
keywords: maven

---



```
    项目需要把本地的jar包，安装到本地仓库中，先介绍安装命令格式，使用db2jar安装示例。
```

## 操作步骤

1.首先要检查本地maven环境变量，cmd输入mvn -v如下图。

![maven](https://oscimg.oschina.net/oscnet/efaac37e2dd503edb138c25efb1c60ae627.jpg)

2.确认好mvn环境变量后，执行安装命令。规则如下：

注意：这个命令不能换行，中间用空格来分割的

```
安装命令实例：
mvn install:install-file -DgroupId=com.baidu -DartifactId=ueditor -Dversion=1.0.0 -Dpackaging=jar -Dfile=ueditor-1.1.2.jar

------------



-DgroupId=<groupId>       : 设置项目代码的包名(一般用组织名)

-DartifactId=<artifactId> : 设置项目名或模块名 

-Dversion=1.0.0           : 版本号

-Dpackaging=jar           : 什么类型的文件(jar包)

-Dfile=<myfile.jar>       : 指定jar文件路径与文件名(同目录只需文件名)

```



## 安装db2jar包到本地仓库

 cmd下切换目录到jar所在目录，执行命令：

### db2jcc 

`mvn install:install-file "-DgroupId=com.ibm.db2" "-DartifactId=db2jcc" "-Dversion=8.1" "-Dpackaging=jar" "-Dfile=db2jcc.jar"`

### db2jcc_license_cu

`mvn install:install-file "-DgroupId=com.ibm.db2" "-DartifactId=db2jcc_license_cu" "-Dversion=9.7" "-Dpackaging=jar" "-Dfile=db2jcc_license_cu.jar"`

------

作者：狂奔的熊二   

出处：[https://kbdxe.cn](https://kbdxe.cn)
