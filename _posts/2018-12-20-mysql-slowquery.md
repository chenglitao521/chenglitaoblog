---
layout: post
title: mysql慢查询开启
categories: [mysql]
description: mysql慢查询开启
keywords: mysql,慢查询

---

​	数据库的查询快慢是影响系统瓶颈的一大关键因素，所以我们需要找出查询比较慢的sql语句进行优化。开启慢日志是个很不错的方法，找出超过执行时间的语句分析。

#### 参数说明

slow_query_log ： 查看mysql的慢日志是否开启；OFF为关闭，ON为开启

slow_query_log_file ：日志的存放位置，默认放在Data目录下， mysql会自动生成[hostname]-slow.log 文件

long_query_time ：超过多少秒记录

#### 开启慢日志

1.命令行查询参数

```sql
mysql> show variables like 'slow_query%';
+---------------------+--------------------------+
| Variable_name       | Value                    |
+---------------------+--------------------------+
| slow_query_log      | ON                       |
| slow_query_log_file | DESKTOP-HD47B8V-slow.log |
+---------------------+--------------------------+
2 rows in set (0.02 sec)

mysql> show variables like 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.03 sec)
```



2.设置方法

方法一：全局变量设置，**不需要重启服务**

将 slow_query_log 全局变量设置为“ON”状态

```sql
mysql>  set global slow_query_log='ON'; 
Query OK, 0 rows affected (0.00 sec)
```

查询超过1秒就记录

```sql
mysql> set global long_query_time=1;
Query OK, 0 rows affected (0.00 sec)
```

设置慢查询日志存放的位置

```sql
mysql>  set global slow_query_log_file='C:/ProgramData/MySQL/MySQL Server 8.0/Data/slow.log';
Query OK, 0 rows affected (0.00 sec)
```

方法二：设置配置文件

win下修改my.ini文件，linux修改my.cnf文件，然后重启mysql服务生效

```sql
[mysqld]
slow_query_log = ON
slow_query_log_file = /usr/local/mysql/data/slow.log
long_query_time = 1
```

#### 验证慢日志

执行命令验证

```sql
mysql> select sleep(2);
+----------+
| sleep(2) |
+----------+
|        0 |
+----------+
1 row in set (2.02 sec)
```

然后再查询上述慢日志目录，日志是否生成。
