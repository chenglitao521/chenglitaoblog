---
layout: post
title: Java中注解到底是什么
categories: [Jav]
description: Java中注解到底是什么
keywords: 注解

---
上次在学习@Transactional注解的时候，思考到个问题。Java中的注解到底是什么？有是如何实现的？

## 注解是什么

注解也叫元数据，例如我们常见的@Override和[@Deprecated](https://my.oschina.net/jianhuaw)，注解是JDK1.5版本开始引入的一个特性，用于对代码进行说明，可以对包、类、接口、字段、方法参数、局部变量等进行注解。

注解可分为三类：

* 一类是Java自带的标准注解，包括@Override（标明重写某个方法）、@Deprecated（标明某个类或方法过时）和@SuppressWarnings（标明要忽略的警告），使用这些注解后编译器就会进行检查。
* 一类为元注解，元注解是用于定义注解的注解，包括@Retention（标明注解被保留的阶段）、@Target（标明注解使用的范围）、@Inherited（标明注解可继承）、@Documented（标明是否生成javadoc文档）
* 一类为自定义注解，可以根据自己的需求定义注解

### 元注解

元注解是可以注解到注解上的注解，或者说元注解是一种基本注解，但是它能够应用到其它的注解上面。明白点说，就是我们定义注解时用的注解就是元注解。

```java
 @Target(ElementType.METHOD)
 @Retention(RetentionPolicy.RUNTIME)
 public @interface Test {
 

 }
```

除了@符号，注解很像是一个接口。定义注解的时候需要用到元注解。

在注解中一般会有一些元素以表示某些值。注解的元素看起来就像接口的方法，唯一的区别在于可以为其制定默认值。没有元素的注解称为标记注解，上面的@Test就是一个标记注解。 

   注解的可用的类型包括以下几种：所有基本类型、String、Class、enum、Annotation、以上类型的数组形式。元素不能有不确定的值，即要么有默认值，要么在使用注解的时候提供元素的值。而且元素不能使用null作为默认值。注解在只有一个元素且该元素的名称是value的情况下，在使用注解的时候可以省略“value=”，直接写需要的值即可。

元注解一共有5种， @Retention、@Documented、@Target、@Inherited、@Repeatable。

1. Retention 

英文意为保留期的意思。当 @Retention 应用到一个注解上的时候，它解释说明了这个注解的的存活时间。

它的取值如下： 

* RetentionPolicy.SOURCE 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。 
* RetentionPolicy.CLASS 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。 
* RetentionPolicy.RUNTIME 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。

1. @Documented

这个元注解肯定是和文档有关。它的作用是能够将注解中的元素包含到 Javadoc 中去

1. Target 

目标的意思，@Target 指定了注解运用的地方

ElementType.ANNOTATION_TYPE 可以给一个注解进行注解
ElementType.CONSTRUCTOR 可以给构造方法进行注解
ElementType.FIELD 可以给属性进行注解
ElementType.LOCAL_VARIABLE 可以给局部变量进行注解
ElementType.METHOD 可以给方法进行注解
ElementType.PACKAGE 可以给一个包进行注解
ElementType.PARAMETER 可以给一个方法内的参数进行注解
ElementType.TYPE 可以给一个类型进行注解，比如类、接口、枚举

1. @Inherited

Inherited 是继承的意思，但是它并不是说注解本身可以继承，而是说如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解

1. @Repeatable

Repeatable 自然是可重复的意思。@Repeatable 是 Java 1.8 才加进来的，所以算是一个新的特性。

什么样的注解会多次应用呢？通常是注解的值可以同时取多个。

## 注解处理器

上面定义的注解，java如何去处理这些注解？这里用到了java中的反射机制。

需要关注lang包下Class类和AnnotatedElement 接口；

![](https://oscimg.oschina.net/oscnet/06282942c6d59c7ab1f853bf1ee04ba3db3.jpg)

Java.lang.reflect.AnnotatedElement 接口是所有程序元素（Class、Method和Constructor）的父接口，所以程序通过反射获取了某个类的AnnotatedElement对象之后，程序就可以调用该对象的如下四个个方法来访问Annotation信息：

　 方法1：<T extends Annotation> T getAnnotation(Class<T> annotationClass): 返回改程序元素上存在的、指定类型的注解，如果该类型注解不存在，则返回null。

　 方法2：Annotation[] getAnnotations():返回该程序元素上存在的所有注解。

　 方法3：boolean is AnnotationPresent(Class<?extends Annotation> annotationClass):判断该程序元素上是否包含指定类型的注解，存在则返回true，否则返回false.

 　方法4：Annotation[] getDeclaredAnnotations()：返回直接存在于此元素上的所有注释。与此接口中的其他方法不同，该方法将忽略继承的注释。（如果没有注释直接存在于此元素上，则返回长度为零的一个数组。）该方法的调用者可以随意修改返回的数组；这不会对其他调用者返回的数组产生任何影响.

Class对象也实现了该接口，即可以获取目标对象的注解，继而可以根据注解的属性，获得对象的成员变量、方法等。

------

作者：狂奔的熊二   

出处：[https://kbdxe.cn](https://kbdxe.cn)
