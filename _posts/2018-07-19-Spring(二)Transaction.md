---
layout: post
title: Spring事务系列(二)Spring事务的原理
categories: [spring , 事务]
description: spring事务的原理，使用
keywords: Spring, 事务
---
## spring中事务的原理

### 核心接口

首先要看的接口为PlatformTransactionManager，spring管理事务的核心接口。

``` java
package org.springframework.transaction;

public interface PlatformTransactionManager {
    TransactionStatus getTransaction(TransactionDefinition var1) throws TransactionException;

    void commit(TransactionStatus var1) throws TransactionException;

    void rollback(TransactionStatus var1) throws TransactionException;
}

```
### 事务管理器
spring并不直接管理事务，而是提供了许多事务管理器，他们将事务管理的职责委托给Hibernate或者JDBC等持久化机制所提供的相关平台框架的事务来实现。
这个接口就是上述提到的PlatformTransactionManager，具体的事务管理机制对于spring是透明的。Spring事务管理的一个优点就是为不同的事务API提供一致
的编程模型，如JTA、JDBC、Hibernate、JPA。

#### JDBC事务
如果使用JDBC实现持久化，使用DataSourceTransactionManager，我们要配置如下的XML代码：


``` xml
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource">
            <ref bean="dataSource" />
        </property>
    </bean>
```
在来看看该类的继承关系，间接的实现了PlatformTransactionManager接口。    
查看DataSourceTransactionManager类源代码，实际上，DataSourceTransactionManager是通过调用java.sql.Connection来管理事务，而后者是通过
DataSource获取到的。通过调用连接的commit()方法来提交事务，同样，事务失败则通过调用rollback()方法进行回滚。

#### Hibernate事务
如果应用程序的持久化是通过Hibernate实现的，那么你需要使用HibernateTransactionManager。对于Hibernate3，需要在Spring上下文定义中添加
如下的<bean>声明：

``` xml
 <bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
```
sessionFactory属性需要装配一个Hibernate的session工厂，HibernateTransactionManager的实现细节是它将事务管理的职责委托给
org.hibernate.Transaction对象，而后者是从HibernateSession中获取到的。当事务成功完成时，HibernateTransactionManager 将会调用Transaction对象的commit()方法，反之，将会调用rollback()方法。


#### Java持久化API事务（JPA）

Hibernate多年来一直是事实上的Java持久化标准，但是现在Java持久化API作为真正的Java持久化标准进入大家的视野。如果你计划使用JPA的话，
那你需要使用Spring的JpaTransactionManager来处理事务。你需要在Spring中这样配置JpaTransactionManager：

``` xml
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
```
JpaTransactionManager只需要装配一个JPA实体管理工厂（javax.persistence.EntityManagerFactory接口的任意实现）。JpaTransactionManager
将与由工厂所产生的JPA EntityManager合作来构建事务。

#### Java原生API事务
如果你没有使用以上所述的事务管理，或者是跨越了多个事务管理源（比如两个或者是多个不同的数据源），你就需要使用JtaTransactionManager：

``` xml
    <bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager">
        <property name="transactionManagerName" value="java:/TransactionManager" />
    </bean>
```

JtaTransactionManager将事务管理的责任委托给javax.transaction.UserTransaction和javax.transaction.TransactionManager对象，其中事务成功完
成通过UserTransaction.commit()方法提交，事务失败通过UserTransaction.rollback()方法回滚。

### 基本事务属性
事务管理器接口PlatformTransactionManager通过getTransaction(TransactionDefinitiondefinition)方法来得到事务，这个方法里面的参数是
TransactionDefinition类 ，这个方法里面的参数是TransactionDefinition类，这个类就定义了一些基本的事务属性。

 1. 传播行为
 2. 隔离规则
 3. 回滚规则
 4. 是否只读
 5. 事务超时

TransactionDefinition 接口定义了上述的事务属性

``` java
    int getPropagationBehavior();

    int getIsolationLevel();

    int getTimeout();

    boolean isReadOnly();

    String getName();
```
#### 传播行为

事务的第一个方面是传播行为（propagationbehavior）。当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有
事务中运行，也可能开启一个新事务，并在自己的事务中运行。Spring定义了七种传播行为：

| PROPAGATION_REQUIRED      | 表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务                                                                                  |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PROPAGATION_SUPPORTS      | 表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会在这个事务中运行                                                                                              |
| PROPAGATION_MANDATORY     | 表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常                                                                                                                  |
| PROPAGATION_REQUIRED_NEW  | 表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION_NOT_SUPPORTED | 表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager                               |
| PROPAGATION_NEVER         | 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常                                                                                                    |
| PROPAGATION_NESTED        |	表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样                                                                                                                                                                                   |

嵌套事务一个非常重要的概念就是内层事务依赖于外层事务。外层事务失败时，会回滚内层事务所做的动作。而内层事务操作失败并不会引起外层事务的回滚。
PROPAGATION_REQUIRED应该是我们首先的事务传播行为。它能够满足我们大多数的事务需求。

#### 隔离级别

见上一篇文章

#### 只读
事务的第三个特性是它是否为只读事务。如果事务只对后端的数据库进行该操作，数据库可以利用事务的只读特性来进行一些特定的优化。通过将事务设置为只读，你就可以给数据库一个机会，让它应用它认为合适的优化措施。

#### 事务超时
为了使应用程序很好地运行，事务不能运行太长的时间。因为事务可能涉及对后端数据库的锁定，所以长时间的事务会不必要的占用数据库资源。事务超时就是事务的一个定时器，在特定时间内事务如果没有执行完毕，那么就会自动回滚，而不是一直等待其结束

#### 回滚规则
默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚（这一行为与EJB的回滚行为是一致的） 
但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

### 事务状态
PlatformTransactionManager接口的getTransaction()的方法得到的是TransactionStatus接口的一个实现，这个接口的内容如下：

``` java
public interface TransactionStatus extends SavepointManager, Flushable {
    boolean isNewTransaction();

    boolean hasSavepoint();

    void setRollbackOnly();

    boolean isRollbackOnly();

    void flush();

    boolean isCompleted();
}

```



## 参考链接
[参考链接](https://blog.csdn.net/trigl/article/details/50968079)
----------

作者：狂奔的熊二   

出处：[https://kbdxe.cn](https://kbdxe.cn)
