---
layout: post
title: Spring事务系列(一)事务概念
categories: [事务,mysql]
description: 事务学习
keywords: spring, 事务 
---
## 事务概念
我们在实际业务场景中，经常会遇到数据频繁修改读取的问题。在同一时刻，不同的业务逻辑对同一个表数据进行修改，这种冲突
很可能造成数据不可挽回的错乱，所以我们需要用事务来对数据进行管理。


什么事务？数据库中事务的四大特性：

 - 原子性(Atomicity):操作这些指令时，要么全部执行成功，要么全部不执行。只要其中一个指令执行失败，所有的指令都执行失败                    ，数据进行回滚，回到执行指令前的数据状态。
 - 一致性(Consistency):事务的执行使数据从一个状态转换为另一个状态，但是对于整个数据的完整性保持稳定。
 - 隔离性(Isolation)：在该事务执行的过程中，无论发生的任何数据的改变都应该只存在于该事务之中，对外界不存在任何影响。                   只有在事务确定正确提交之后，才会显示该事务对数据的改变。其他事务才能获取到这些改变后的数据。
 - 持久性(Durability):当事务正确完成后，它对于数据的改变是永久性的。

通俗点讲，事务就是一系列指令的集合。


----------


## 并发事务导致的问题

在许多事务处理同一个数据时，如果没有采取有效的隔离机制，那么并发处理数据时，会带来一些的问题。
问题分为5类，包括3类数据读问题：脏读、不可重复读和幻读。两类数据更新问题：第一类丢失更新、第二类丢失更新。

 1.脏读：一个事务读取到另一个事务未提交的更新数据。
  
``` 
小明的银行卡余额里有100元。现在他打算用手机点一个外卖饮料，需要付款10元。但是这个时候，他的女朋友看中了一件衣服95元  
，她正在使用小明的银行卡付款。于是小明在付款的时候，程序后台读取到他的余额只有5块钱了，根本不够10元，所以系统拒绝了
他的交易，告诉余额不足。但是小明的女朋友最后因为密码错误，无法进行交易。小明非常郁闷，明明银行卡里还有100元，怎么会 
余额不足呢？（他女朋友更郁闷。。。）
```
 2.不可重复读：一个事务两次读取同一行的数据，结果得到不同状态的结果，中间正好另一个事务更新了该数据，两次结果相异，
    不可被信任。
```
小明在手机上购买起购价为1W元理财产品。系统首先要判断他的余额够不够购买理财产品，如果足够再获取当前的余额，进行申请。
系统第一次读取到小明的余额还剩1W元，刚好足够购买产品。但是这个时候刚好他女朋友又看中了一个包包5000元，这次密码终于
不会再错误的女朋友毫不犹豫刷了小明的银行卡买下了这个包包。但是这个系统刚好在进行第二次确认，发现小明的余额上只有5
000元，根本不够购买。于是系统很生气，拒绝了小明的购买行为，告诉他，你真是个骗子！！！
```
 3.虚读(幻读):一个事务读取到另一事务已提交的insert数据。
 

```  
公司财务A在进行员工薪资核算业务，需要对小明的工资进行计算并录入系统，必须查询两次明细信息，然后将后一次的明细信息计
算总数出来。财务在第一次明细查询时，基本工资2000元，全勤奖1000元，提成2000元，共计5000元。在进行第二次计算时，财务
B突然接到通知，需要把下个月的节日福利也在这个月的工资中发放，每人100元。于是财务B在每个人的工资明细中又加了一条节
日福利100元。而此时财务A获得第二次查询小明的工资明细后，发现工资明细变成了4条数据，总数是5100元。两次计算结果相差
100元，财务A奇怪这多出来的一条明细100元是哪里来的呢？（都怪财务B不告诉A。。。）
```

注意：不可重复读和幻读的区别是：**前者是指读到了已经提交的事务的更改数据（修改或删除），后者是指读到了其他已经提交事务的新增数据**。
对于这两种问题解决采用不同的办法，防止读到更改数据，只需对操作的数据添加行级锁，防止操作中的数据
发生变化；二防止读到新增数据，往往需要添加表级锁，将整张表锁定，防止新增数据（oracle采用多版本数据的方式实现）。
    
    
 4.第一类丢失更新：撤销一个事务时，把其他事务已提交的更新数据覆盖。


```  
小明去银行柜台存钱，他的账户里原来的余额为100元，现在打算存入100元。在他存钱的过程中，银行年费扣了5元，余额只剩95元
。突然他又想着这100元要用来请女朋友看电影吃饭，不打算存了。在他撤回存钱操作后，余额依然为他存钱之前的100元。所以那5块钱到底扣了谁的？
```


 5.第二类丢失更新：是不可重复读的特殊情况。如果两个事物都读取同一行，然后两个都进行写操作，并提交，第一个事物所做的改变就会丢失。
 

``` 
小明和女朋友一起去逛街。女朋友看中了一支口红，（对，女朋友就是用来表现买买买的）小明大方的掏出了自己的银行卡，告诉女朋友：
亲爱的，随便刷，随便买，我坐着等你。然后小明就坐在商城座椅上玩手机，等着女朋友。这个时候，程序员的聊天群里有人推荐了一本书，
小明一看，哎呀，真是本好书，还是限量发行呢，我一定更要买到。于是小明赶紧找到购买渠道，进行付款操作。而同时，小明的女朋友也
在不亦乐乎的买买买，他们同时进行了一笔交易操作，但是这个时候银行系统出了问题，当他们都付款成功后，却发现，银行只扣了小明的
买书钱，却没有扣去女朋友此时交易的钱。哈哈哈，小明真是太开心了！
```

为了解决上述问题，数据库通过锁机制解决并发访问的问题。根据锁定对象不同：分为行级锁和表级锁；根据并发事务锁定的关系上看：分为共享锁定和独占锁定，
共享锁定会防止独占锁定但允许其他的共享锁定。而独占锁定既防止共享锁定也防止其他独占锁定。为了更改数据，数据库必须在进行更改的行上施加行独占锁定，
insert、update、delete和selsct for update语句都会隐式采用必要的行锁定。

但是直接使用锁机制管理是很复杂的，基于锁机制，数据库给用户提供了不同的事务隔离级别，只要设置了事务隔离级别，数据库就会分析事务中的sql语句然后
自动选择合适的锁。
 
### mysql中的事务隔离级别
    

 - Serializable (串行化)：可避免脏读、不可重复读、幻读的发生。
 - Repeatable read (可重复读)：可避免脏读、不可重复读的发生。
 - Read committed (读已提交)：可避免脏读的发生。
 - Read uncommitted (读未提交)：最低级别，任何情况都无法保证。
 
在MySQL数据库中，支持上面四种隔离级别，默认的为Repeatable read (可重复读)；而在Oracle数据库中，只支持Serializable (串行化)级别和Read committed (读已提交)这两种级别，其中默认的为Read committed级别。

----------

作者：狂奔的熊二   

出处：[https://kbdxe.cn](https://kbdxe.cn)
