---
layout: post
title: Spring事务系列(四)声明式注解事务实现机制
categories: [事务,spring]
description: Spring 声明式注解事务实现机制
keywords: spring,事务

---



## Spring中注解事务实现机制

在使用[@Transactional](https://my.oschina.net/u/3770144) 注解管理事务时步骤很简单。但是如果对@Transactional理解不够透彻，很容易出现事务不起作用的情况。所以，在对@Transactional的实现机制要有一定的了解。

在应用系统调用声明[@Transactional](https://my.oschina.net/u/3770144) 的目标方法时，Spring Framework 默认使用 AOP 代理，在代码运行时生成一个代理对象，根据[@Transactional](https://my.oschina.net/u/3770144) 的属性配置信息，这个代理对象决定该声明@Transactional 的目标方法是否由拦截器 TransactionInterceptor 来使用拦截，在 TransactionInterceptor 拦截时，会在在目标方法开始执行之前创建并加入事务，并执行目标方法的逻辑, 最后根据执行情况是否出现异常，利用抽象事务管理器(图 2 有相关介绍)AbstractPlatformTransactionManager 操作数据源 DataSource 提交或回滚事务, 如图 1 所示。
![图1 Spring事务实现机制](https://oscimg.oschina.net/oscnet/226bc57b9c058ef4e70ccedb4f5a262d404.jpg)

Spring AOP 代理有 CglibAopProxy 和 JdkDynamicAopProxy 两种，图 1 是以 CglibAopProxy 为例，对于 CglibAopProxy，需要调用其内部类的 DynamicAdvisedInterceptor 的 intercept 方法。对于 JdkDynamicAopProxy，需要调用其 invoke 方法。

正如上文提到的，事务管理的框架是由抽象事务管理器 AbstractPlatformTransactionManager 来提供的，而具体的底层事务处理实现，由 PlatformTransactionManager 的具体实现类来实现，如事务管理器 DataSourceTransactionManager。不同的事务管理器管理不同的数据资源 DataSource，比如 DataSourceTransactionManager 管理 JDBC 的 Connection。

PlatformTransactionManager，AbstractPlatformTransactionManager 及具体实现类关系如图 2 所示。

![图2 TransactionManager 类结构](https://oscimg.oschina.net/oscnet/a970744a0b6bd406bb06d16d009ef30fad9.jpg)

## 注解方式事务使用的注意

当您对 Spring 的基于注解方式的实现步骤和事务内在实现机制有较好的理解之后，就会更好的使用注解方式的事务管理，避免当系统抛出异常，数据不能回滚的问题。

### 正确的设置@Transactional 的 propagation 属性

1. TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
2. TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
3. TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。

### 正确的设置@Transactional 的 rollbackFor 属性

默认情况下，如果在事务中抛出了未检查异常（继承自 RuntimeException 的异常）或者 Error，则 Spring 将回滚事务；除此之外，Spring 不会回滚事务。

如果在事务中抛出其他类型的异常，并期望 Spring 能够回滚事务，可以指定 rollbackFor。例：

@Transactional(propagation= Propagation.REQUIRED,rollbackFor= MyException.class)

通过分析 Spring 源码可以知道，若在目标方法中抛出的异常是 rollbackFor 指定的异常的子类，事务同样会回滚。

### Transactional 只能应用到 public 方法才有效

只有@Transactional 注解应用到 public 方法，才能进行事务管理。这是因为在使用 Spring AOP 代理时，Spring 在调用在图 1 中的 TransactionInterceptor 在目标方法执行前后进行拦截之前，DynamicAdvisedInterceptor（CglibAopProxy 的内部类）的的 intercept 方法或 JdkDynamicAopProxy 的 invoke 方法会间接调用 AbstractFallbackTransactionAttributeSource（Spring 通过这个类获取表 1. @Transactional 注解的事务属性配置属性信息）的 computeTransactionAttribute 方法。

```java
protected TransactionAttribute computeTransactionAttribute(Method method,
    Class<?> targetClass) {
        // Don't allow no-public methods as required.
        if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
return null;}
```

这个方法会检查目标方法的修饰符是不是 public，若不是 public，就不会获取@Transactional 的属性配置信息，最终会造成不会用 TransactionInterceptor 来拦截该目标方法进行事务管理。

### 避免 Spring 的 AOP 的自调用问题

在 Spring 的 AOP 代理下，只有目标方法由外部调用，目标方法才由 Spring 生成的代理对象来管理，这会造成自调用问题。若同一类中的其他没有@Transactional 注解的方法内部调用有@Transactional 注解的方法，有@Transactional 注解的方法的事务被忽略，不会发生回滚。见清单 5 举例代码展示。

```java
@Service
-->public class OrderService {
    private void insert() {
insertOrder();
}
@Transactional
    public void insertOrder() {
        //insert log info
        //insertOrder
        //updateAccount
       }
}
```



insertOrder 尽管有@Transactional 注解，但它被内部方法 insert 调用，事务被忽略，出现异常事务不会发生回滚。

上面的两个问题@Transactional 注解只应用到 public 方法和自调用问题，是由于使用 Spring AOP 代理造成的。为解决这两个问题，使用 AspectJ 取代 Spring AOP 代理。

需要将下面的 AspectJ 信息添加到 xml 配置信息中。



## 总结

通过本文的介绍，相信读者能够清楚的了解基于@Transactional 注解的实现步骤，能够透彻的理解的 Spring 的内部实现机制，并有效的掌握相关使用注意事项，从而能够正确而熟练的使用基于@Transactional 注解的事务管理方式。



[参考链接](https://www.ibm.com/developerworks/cn/java/j-master-spring-transactional-use/#icomments "参考链接")

------

作者：狂奔的熊二   

出处：[https://kbdxe.cn](https://kbdxe.cn)
