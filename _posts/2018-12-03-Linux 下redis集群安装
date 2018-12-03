---
layout: post
title: Linux redis的集群安装
categories: [redis]
description: redis集群安装
keywords: redis集群安装

---



Linux环境安装redis集群

[TOC]

### 安装环境

系统：CentOS release 6.5 (Final)

环境要求：

```
gcc；

    Ruby:

Rubygems:

   redis-3.X   (需要3.0以上版本支持集群)
```

注意：**我测试时连接外网了，如果没有连接外网就需要离线下载所需的依赖，然后上传安装**。

### 安装单个redis

注意：这里**需要gcc和ruby的环境**。安装前需要检查系统是否有gcc和ruby的环境：

* gcc环境

  gcc用来编译redis的源码，使用gcc  -v 查看，

![](https://oscimg.oschina.net/oscnet/83c6aadd2916b43a95b69016cfaf2e98a57.jpg)

* ruby 环境

  ruby 用来执行集群的命名，使用ruby -v，gem -v 查看：

![](https://oscimg.oschina.net/oscnet/1302e07cec729a4bbdda208056358c78edb.jpg)



安装好上述两个依赖后，现在安装redis.

1. 创建存放安装包的文件夹，makdir redis。然后cd到该目录，上传下载好的tar.gz文件。然后，执行解压命令

```
tar zxvf redis-3.2.12.tar.gz 
```

![](https://oscimg.oschina.net/oscnet/f45ecea96561e2f3ad9102cb6fe589ae9dd.jpg)



1. cd到redis的解压目录，然后执行make && make install 命令安装：

   ```
   $ cd /home/redis/redis-3.2.12
   $ make && make install  //make 这里如果不指定PREFIX，默认将安装在/usr/local/bin下，保持默认就好
   ```

![](https://oscimg.oschina.net/oscnet/9bcc56062a19f4371b952e3ecc7e601f776.jpg)

安装成功后，出现如下图：



![](https://oscimg.oschina.net/oscnet/b293a393b8fbc97b5c5f362031cdf7d567a.jpg)

也可以在/usr/local/bin下查看redis的安装文件。

1. cd到解压目录，指定配置文件启动。

   ```
   redis-server redis.conf
   ```

   ![](https://oscimg.oschina.net/oscnet/3537e19e4ba36d5b3680912a66c92fdcf67.jpg)

注意，这里启动的redis不是后台启动，退出会话后redis会自动退出。通常单机redis我们需要修改如下配置：



```properties
 daemonize yes  //默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程
 requirepass foobared   //设置客户端连接密码
 port 6379  //指定端口
 bind 127.0.0.1  //绑定的主机地址，如果需要远程连接，则需修改该参数
 protected-mode  yes // 保护模式，如果需要远程连接，则需要修改该参数
 
```

### 安装Redis集群

安装集群至少需要三个主节点，三个对应的从节点。防止某个节点挂掉，其他两个节点可以推举其对应得从节点为主节点，一定程度的容错。

这里在同一台机器上安装，生产环境至少需要三台机器安装。创建6个文件分别存放配置信息，文件夹名以端口命名。比如7001-7006。

1. 使用mkdir命令批量创建文件夹。

![](https://oscimg.oschina.net/oscnet/e1f72b618e59f8d789dfe29dbc9d3fa58fe.jpg)

1. 然后我们需要在安装目录复制redis-cli、redis-server、redis.conf 三个配置文件到每个文件夹，用来启动单个redis.  如果安装时没有指定目录，则默认在usr/local/bin 目录下复制redis-cli、redis-server文件，redis.conf在解压的目录可以复制，也可以自己创建。

![](https://oscimg.oschina.net/oscnet/94f1a8ba45f5283e1afd25baf20475d1b01.jpg)

   在复制redis.conf配置文件

![](https://oscimg.oschina.net/oscnet/bf15f0531ddbe1894430047e1057d85503d.jpg)

1. 修改每个redis.conf文件

```properties
bind 0.0.0.0   //0.0.0.0表示允许所有连接
protected-mode no //保护模式，yes表示不允许远程连接
port 7001  //端口，这里每个配置文件不同
daemonize yes  //以后台方式启动
appendonly yes   //redis 将每一次写操作请求都追加到appendonly.aof 文件中redis重新启动时,会从							该文件恢复出之前的状态。
cluster-enabled yes  //启用集群
cluster-node-timeout 5000  //节点超时时间
```

1. 在每个文件下执行redis-server  redis.conf 命令。这个可以创建脚本执行比较快。

```
vim startall.sh 就会打开vim编辑器，创建一个空的文本
```

这里我每个分别启动，脚本执行不起作用（心累）

![](https://oscimg.oschina.net/oscnet/f0e8a6dd5158b16a948848f056bc35adfe1.jpg)



1. 此时，我们进入解压的目录src下，复制redis-trib.rb 文件到redis目录；
   ![](https://oscimg.oschina.net/oscnet/6226be1c25be1014fa6b1cc75af7c637e35.jpg)

1. 在redis目录执行命令：

   注意：**这里--replicas 1 表示每个主数据库拥有从数据库个数为1。master节点不能少于3个。**

   **如果需要远程访问，这里的命令中127.0.0.1 需要换成服务器的ip地址。否则不能远程访问。**

   ```xml
   ./redis-trib.rb  create --replicas  1  127.0.0.1:7001  127.0.0.1:7002  127.0.0.1:7003  127.0.0.1:7004  127.0.0.1:7005  127.0.0.1:7006
   ```



#### 各种报错解决

执行./redis-trib.rb  create --replicas 命令后报错：

![](https://oscimg.oschina.net/oscnet/21502017beefd578bcdc137f7f2c1a5b520.jpg)

然后，ruby-v 和gem -v 查看ruby环境发现都已经安装了。其实这个错误是缺少redis对ruby的接口，我们需要下载redis的gem包；

执行命令：gem install redis  有报错了。

![](https://oscimg.oschina.net/oscnet/d5a990fd9b3725ea3b6cdd82ee76ef38844.jpg)

ruby版本太低了。。。。

执行如下命令安装rvm,用他来安装ruby更新的版本：

```
//具体RVM安装命令地址：http://rvm.io/
　　[root@linux ~]# gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB 

　　[root@linux ~]# curl -sSL https://get.rvm.io | bash -s stable

　　[root@linux ~]# find / -name rvm -print
　　[root@linux ~]# source /usr/local/rvm/scripts/rvm
```

安装好rvm,就可以看到很多rvm的文件。

```xml

```

![](https://oscimg.oschina.net/oscnet/ede400b0acb196df8d95826bf7157548332.jpg)

设置默认的ruby版本

```

```

![](https://oscimg.oschina.net/oscnet/45cb5c1ad8b186807a7b91500e639f28286.jpg)

在执行gem install redis，就会安装成功了。

```
gem install redis
```

![](https://oscimg.oschina.net/oscnet/adbd64ff72ad19c646c9c624ba71b403e16.jpg)

接下来执行命令安装集群：

```
./redis-trib.rb  create --replicas  1  127.0.0.1:7001  127.0.0.1:7002  127.0.0.1:7003  127.0.0.1:7004  127.0.0.1:7005  127.0.0.1:7006
```

![](https://oscimg.oschina.net/oscnet/f8d0646eb3fe6c46860771d39f66f6ce7ed.jpg)

输入yes,回车。



终于成功了，显示显示的主从节点信息。

#### 集群测试

切换到任意的节点目录下，连接添加数据测试：

```
 redis-cli -p 7001  -c
```

![](https://oscimg.oschina.net/oscnet/7ebc094d784710b6b2b2c0dee51b5dae7f3.jpg)

这里就可以看到set name 集群已经hash了key分布在哪个槽上，自动跳转到对应的节点上。

------

作者：狂奔的熊二   

出处：[https://kbdxe.cn](https://kbdxe.cn)
