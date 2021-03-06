---
layout: post
title: redis的集群模式和哨兵模式对比
categories: [redis]
description: redis集群和哨兵模式对比
keywords: redis集群
---

redis的集群模式和哨兵模式

## redis中集群模式

redis集群模式配置支持3.0及以上的版本。目的提高redis的可用性，但是只能保证**一定程度的高可用**。

#### redis-cluster原理

Redis 集群有16384个哈希槽,每个key通过CRC16校验后对16384取模来决定放置哪个槽.集群的每个节点负责一部分hash槽,举个例子,比如当前集群有3个节点,那么:

* 节点 A 包含 0 到 5500号哈希槽.
* 节点 B 包含5501 到 11000 号哈希槽.
* 节点 C 包含11001 到 16384号哈希槽.

这种结构无论添加删除或者改变某个节点的哈希槽的数量都不会造成集群不可用的状态。

因为这种key分布在不同的节点，所以不能使用多keys的操作。（其实也可以变相的实现，但是在高负载的情况下存在风险）。

#### redis-cluster的主从复制模型

为了实现一定程度的高可用，比如某个节点挂掉的情况下，服务仍然能够正常使用。redis使用了主从复制模型，例如创建三个节点的集群A,B,C,在集群创建的时候或者过段时间为三个节点，添加从节点A1,B1,C1.此时整个集群有三个master节点和三个slave节点。A节点down掉的情况，集群推举A1为主节点继续服务。当然，如果A,A1都挂掉的情况，集群则无法使用。所以这也是为什么集群只能保证一定程度的高可用。

#### redis集群安装配置

[Linux Redis集群安装](https://my.oschina.net/kbdxe/blog/2963868 "Linux Redis集群安装")

### redis-cluster在Java中的应用

这里列出spring-boot中的配置方法和spring中的配置方式两种方式，根据自己的情况使用。（**亲测可用**）

环境：

jdk:1.8

spring:

spring-boot:2.0.6

jedis：2.9.0（这个版本支持集群密码的配置，更早的版本不支持）

##### spring中的配置



##### Spring-boot中的配置

pom.xmL中引入jedis2.9依赖：

```xml
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.9.0</version>
        </dependency>
```

application.properties中redis的配置：



```properties
# Redis服务器地址
spring.redis.clusterNodes=ip:port,ip1:port1
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=XXXX
# 连接超时时间（毫秒）
spring.redis.timeout=3600
# 连接池最大连接数（使用负值表示没有限制）
spring.redis.jedis.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.jedis.pool.max-wait=-1
# jedis超时
spring.redis.jedis.shutdown-timeout=100
# 连接池中的最大空闲连接
spring.redis.jedis.pool.max-idle=8
# 连接池中的最小空闲连接
spring.redis.jedis.pool.min-idle=0
```

接下来将这些配置读取到Spring 容器中：

```java
/**
 * @Author:chenglitao
 * @Description:
 * @Date:2018/11/7 16:15
 */

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;

/**
*@Author:chenglitao
*@Description:
*@Params:redis配置类
*@Date:2018/11/28 16:46
*
*/
@Configuration
public class JedisRedisConfig {
    @Value("${spring.redis.clusterNodes}")
    private String clusterNodes;
    @Value("${spring.redis.password}")
    private String password;
    @Value("${spring.redis.port}")
    private int port;
    @Value("${spring.redis.timeout}")
    private int timeout;
    @Value("${spring.redis.jedis.pool.max-idle}")
    private int maxIdle;
    @Value("${spring.redis.jedis.pool.min-idle}")
    private int minIdle;

    @Value("${spring.redis.jedis.pool.max-wait}")
    private long maxWaitMillis;
 
}
```

然后配置RedisCluster:

```java
/**
 * @Author:chenglitao
 * @Description:
 * @Date:2018/11/7 16:13
 */


import org.apache.commons.pool2.impl.GenericObjectPoolConfig;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import redis.clients.jedis.HostAndPort;
import redis.clients.jedis.JedisCluster;

import java.util.HashSet;
import java.util.Set;

/**
*@Author:chenglitao
*@Description:
*@Params: 
*@Date:2018/11/28 16:44
*
*/
@Configuration

public class RedisClusterConfig {
    @Autowired
    private JedisRedisConfig redisProperties;

    @Bean
    public JedisCluster jedisCluster() {
        String[] serverArray = redisProperties.getClusterNodes().split(",");
        //获取服务器数组，逗号分隔
        Set<HostAndPort> nodes = new HashSet<>();

        for (String ipPort : serverArray) {
            String[] ipPortPair = ipPort.split(":");
            nodes.add(new HostAndPort(ipPortPair[0].trim(), 									Integer.valueOf(ipPortPair[1].trim())));
        }

        GenericObjectPoolConfig config = new
                GenericObjectPoolConfig();
        config.setMaxTotal(3);
        return new JedisCluster(nodes, redisProperties.getTimeout(), 1000, 100, 			redisProperties.getPassword(), config);//需要密码连接的创建对象方式，如果没有密码，就使用没有密码的方法，这里有很多重载的构造方法。
    }

}
```



最后，写test方法测试下就ok了。

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import redis.clients.jedis.JedisCluster;

@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoApplicationTests {
    @Autowired
    private JedisCluster jedisCluster;

    @Test
    public void contextLoads() {
        jedisCluster.set("aa", "111");
        String aa = jedisCluster.get("aa");
        System.out.println(aa);
    }
}

```

当然，在开发中需要封装基础的工具类操作。下面是基本的操作的封装，根据自己的需要修改即可。
----------
 

作者：狂奔的熊二   

出处：[https://kbdxe.cn](https://kbdxe.cn)

