---
title: Redis的使用
date: 2019-09-25 22:00:00
tags: [redis]
categories: redis
---
## NOSQL
### NOSQL和关系型数据库比较
#### 优点
1. 成本：nosql数据库简单易部署，基本都是开源软件，不需要像使用oracle那样花费大量成本购买使用，相比关系型数据库价格便宜。
2. 查询速度：nosql数据库将数据存储于缓存之中，关系型数据库将数据存储在硬盘中，自然查询速度远不及nosql数据库。
3. 存储数据的格式：nosql的存储格式是key，value形式、文档形式、图片形式等等，所以可以存储基础类型以及对象或者是集合等各种格式，而数据库则只支持基础类型。
4. 扩展性：关系型数据库有类似join这样的多表查询机制的限制导致扩展很艰难。

#### 缺点
1. 维护的工具和资料有限，因为nosql是属于新的技术，不能和关系型数据库十几年的技术同日而语。
2. 不提供对sql的支持，如果不支持sql这样的工业标准，将产生一定用户的学习和使用成本。
3. 不提供关系型数据库对事务的处理。

### 非关系型数据库的优势：
1. 性能NOSQL是基于键值对的，可以想象成表中的主键和值的对应关系，而且不需要经过SQL层的解析，所以性能非常高。
2. 可扩展性同样也是因为基于键值对，数据之间没有耦合性，所以非常容易水平扩展。

### 关系型数据库的优势：
1. 复杂查询可以用SQL语句方便的在一个表以及多个表之间做非常复杂的数据查询。
2. 事务支持使得对于安全性能很高的数据访问要求得以实现。对于这两类数据库，对方的优势就是自己的弱势，反之亦然。

### 总结
关系型数据库与NoSQL数据库并非对立而是互补的关系，即通常情况下使用关系型数据库，在适合使用NoSQL的时候使用NoSQL数据库，让NoSQL数据库对关系型数据库的不足进行弥补。一般会将数据存储在关系型数据库中，在nosql数据库中备份存储关系型数据库的数据

## 主流的NOSQL产品
- 键值(Key-Value)存储数据库
  - 相关产品： Tokyo Cabinet/Tyrant、Redis、Voldemort、Berkeley DB
  - 典型应用： 内容缓存，主要用于处理大量数据的高访问负载。 
  - 数据模型： 一系列键值对
  - 优势： 快速查询
  - 劣势： 存储的数据缺少结构化
- 列存储数据库
  - 相关产品：Cassandra, HBase, Riak
  - 典型应用：分布式的文件系统
  - 数据模型：以列簇式存储，将同一列数据存在一起
  - 优势：查找速度快，可扩展性强，更容易进行分布式扩展
  - 劣势：功能相对局限
- 文档型数据库
  - 相关产品：CouchDB、MongoDB
  - 典型应用：Web应用（与Key-Value类似，Value是结构化的）
  - 数据模型： 一系列键值对
  - 优势：数据结构要求不严格
  - 劣势： 查询性能不高，而且缺乏统一的查询语法
- 图形(Graph)数据库
  - 相关数据库：Neo4J、InfoGrid、Infinite Graph
  - 典型应用：社交网络
  - 数据模型：图结构
  - 优势：利用图结构相关算法。
  - 劣势：需要对整个图做计算才能得出结果，不容易做分布式的集群方案。

## 应用场景
### 网上
- 缓存（数据查询、短链接、新闻内容、商品内容等）（最多）
- 聊天室的在线好友列表
- 任务队列（秒杀、抢购、12306等）
- 应用排行榜
- 网站访问统计
- 数据过期处理（可以精确到毫秒）
- 分布式集群框架中的session分离

### 推测
- 当前登录人员信息（统计所有登陆人员）————> 所有的用户及其权限 和组织机构
- 数据字典
- 菜单
- union表（相当于统计计算 但要求可靠性）
- 当前流程环节
- 通知
- session
- 发送邮件 短信的验证码及其发送间隔期限
- 有期限的东西
- 实时聊天
- 所有的统计登录访问次数
- 简单的消息队列
- 放到缓存中的东西

>不能用redis：
>		数据量太大、数据访问频率非常低的业务都不适合使用Redis，数据太大会增加成本，访问频率太低，保存在内存中纯属浪费资源。


## 存储结构及指令

### 使用
#### Redis支持类型
##### 1．String（字符串）
命令：
>增：set 键 字符串值
>查询：GET 键
>返回key对应字符串值的指定字符：GETRANGE 键 开始 结束（包含）
>设置并返回旧值：GETSET 键 新值
>获取/设置多个key：MGET 键1 键2 ..../MSET 键1 值1 键2 值2 ....
>当value为整数数据时，可以设为自增：INCR 键 / INCRBY 键 步长
>														自减：DECR 键 / DECR 键 步长
>向键值的末尾追加value。如果键不存在则将该键的值设置为value，即相当于 SET key value。返回值是追加后字符串的总长度：APPEND 键 值
>获得字符串长度 键不存在则返回0：STRLEN 键

##### 2．Hash（哈希，key-value，适合存储对象 key是对象 value是属性 字段值只能是字符串类型）
命令：
>增：hmset 键 字段1 字段值1 字段2 字段值2 ....
>		HSET 键 字段 字段值（只能设置一个字段值 不区分插入和更新操作 插入返回1 更新返回0）
>删：HDEL 键 字段1 字段2 ....
>查：HGET 键 字段
>			HMGET 键 字段1 字段2 ....（获取多个字段值）
>			HGETALL 键（获取所有的字段和字段值）
>判断字段是否存在：HEXISTS 键 字段
>只获取字段名或字段值：HKEYS 键 /  HVALS 键
>获取字段数量：HLEN 键

##### 3．List（列表 本质是双向链表）
命令：
>向列表左/右增加元素：lpush/RPUSH 键 值1 值2 ....
>移出并获取列表的第一个/最后一个元素，等待时间设置为秒，阻塞：BLPOP/BRPOP 键 时间
>求长度：LLEN 键
>移出并获取列表的第一个/最后一个元素：LPOP/RPOP 键
>查看列表：LRANGE 键 开始 结束

##### 4．Set（集合，无序不重复）
命令：
>增：sadd 键 值1 值2 ....
>获取数量：SCARD 键
>获得所有元素：SMEMBERS 键
>删除：SREM 键 值1 值2 ....
>判断元素是否在集合中：SISMEMBER 键 值1
>差集/交集/并集：SDIFF/SINTER/SUNION 键1 键2
>返回并移除指定set中的随机某个元素：SPOP 键

##### 5．zset(sorted set：有序值不重复  序/分数可以相同)
命令：
>增：zadd 键 序1 值 序2 值2 ....
>获得有序集合数量：ZCARD 键
>获得有序集合指定范围的从小到大的值：ZRANGE 键 开始 结束（包含）
>获得有序集合指定范围的从大到小的值：ZREVRANGE 键 开始 结束（包含）WITHSCORES（获得分数）
>获取元素的分数：ZSCORE 键 值
>删除：ZREM 键 值1 值2 ....

##### 通用：
命令：
>删除键：DEL 键
>键是否存在：EXISTS 键
>查看所有满足条件的key：KEYS 键开头*（模板）
>重命名：RENAME 旧键 新键
>显示指定key的数据类型：type key
>设置key的有效期，单位为seconds：EXPIRE 键 时间
>删除key的有效期，变为永久有效：PERSIST 键
>返回key的剩余有效时间，pttl返回毫秒，ttl返回秒：TTL/PTTL 键

#### 事务
>开启事务：MULTI
>触发事务，提交：EXEC
>取消事务，清除：DISCARD
>当某个事务需要按条件执行时，就要使用这个命令将给定的键设置为受监控的状态：watch 键1 键2 ....（可以实现乐观锁）
>清除所有先前为一个事务监控的键：unwatch

### Redis不支持事务回滚
1.大多数事务失败是因为语法错误或者类型错误，这两种错误，在开发阶段都是可以预见的
（旧版本 如果出现异常 则忽略）
2.为了性能方面就忽略了事务回滚

从 Redis 2.6.5 开始, 在命令排队期间发生错误，Redis会拒绝执行 EXEC，并返回一个错误，然后自动放弃这个事务。
在 Redis 2.6.5 之前，EXEC调用后，会执行排队成功的命令，忽略失败的命令。

单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。
事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。

### RedisUtils.java
```

```
## Redis可视化连接工具
花钱的 但比较火
redis desktop manager（RDM）

不花钱的
https://github.com/qishibo/AnotherRedisDesktopManager/ （比较简洁好看）
https://github.com/caoxinyu/RedisClient （丑了点）
Web Redis Manager（管理页面是layui） https://github.com/yswenli/WebRedisManager

## Jedis

### 普通环境
Jedis 包：jedis-2.9.0.jar + 数据库连接池包：commons-pool2
```java
// 1.设置IP地址和端口
Jedis jedis = new Jedis("localhost", 6379);
// 2.设置数据
jedis.set("key1", "value1");
// 3.获得数据
String value = jedis.get("key1");
// 4.释放资源
jedis.close();
```
可以封装成Utils工具类
```java
private Jedis jedis = new Jedis("localhost", 6379);
```
连接池连接
```java
// 1.获得连接池配置对象，设置配置项
JedisPoolConfig config = new JedisPoolConfig();
// 最大连接数和最大空闲连接数
config.setMaxTotal(30);
config.setMaxIdle(10);
// 2.创建连接池对象
JedisPool jedisPool = new JedisPool(config,"127.0.0.1", 6379);
// 3.从连接池中获得连接
Jedis jedis = null;
try{
	jedis = jedisPool.getResource();
	// 4.设置数据
	jedis.set("key1", "value1");
	// 5.获得数据
	String value = jedis.get("key1");
} catch (Exception e){
	e.printStatckTrace();
} finally {
	// 6.关闭连接
	if(jedis != null){
		jedis.close();
	}
	// 7.关闭连接池
	if(jedisPool != null){
		jedisPool.close();
	}
}
```

### spring整合（本身线程不安全 需要数据库连接池的支持）
Jedis+pool

spring环境的高级封装
RedisTemplate配置
redis.properties：
#访问地址 
redis.host=127.0.0.1 
#访问端口
redis.port=6379 
#注意，如果没有password，此处不设置值，但这一项要保留 
redis.password=
#最大空闲数，数据库连接的最大空闲时间。超过空闲时间，数据库连接将被标记为不可用，然后被释放。设为0表示无限制。 
redis.maxIdle=300 
#连接池的最大数据库连接数。设为0表示无限制 
redis.maxTotal=600 
#最大建立连接等待时间。如果超过此时间将接到异常。设为-1表示无限制。 
redis.maxWait=1000 
#在borrow一个jedis实例时，是否提前进行alidate操作；如果为true，则得到的jedis实例均是可用的； 
redis.testOnBorrow=true

spring-redis.xml：
<!--加载外部数据库配置-->
<context:property-placeholder location="classpath:redis.properties" file-encoding="utf-8" ignore-unresolvable="true"/>
<!-- 配置redis池，依次为最大实例数，最大空闲实例数，(创建实例时)最大等待时间，(创建实例时)是否验证 -->
<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <property name="maxTotal" value="${redis.maxTotal}"/>
    <property name="maxIdle" value="${redis.maxIdle}"/>
    <property name="maxTotal" value="${redis.maxTotal}" />
    <property name="maxWaitMillis" value="${redis.maxWaitMillis}"/>
    <property name="testOnBorrow" value="${redis.testOnBorrow}"/>
</bean>
<bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <property name="poolConfig" ref="jedisPoolConfig" />
    <property name="hostName" value="${redis.host}" />
    <property name="port" value="${redis.port}" />
    <property name="password" value="${redis.password}" />
    <property name="timeout" value="${redis.timeout}" />
</bean>
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
    <property name="connectionFactory" ref="jedisConnectionFactory" />
    <!-- 序列化方式 建议key/hashKey采用StringRedisSerializer -->
    <property name="keySerializer">
        <bean
            class="org.springframework.data.redis.serializer.StringRedisSerializer" />
    </property>
    <property name="valueSerializer">
        <bean
            class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
    </property>
    <property name="hashKeySerializer">
        <bean
            class="org.springframework.data.redis.serializer.StringRedisSerializer" />
    </property>
    <property name="hashValueSerializer">
        <bean
            class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer" />
    </property>
     <!-- 开启REIDS事务支持 -->
     <property name="enableTransactionSupport" value="false" />
</bean>
在applactionContext.xml引入spring-redis.xml
 <import resource="classpath:spring/spring-redis.xml" />

### springboot环境整合
可以直接注入RedisTemplate + reids配置类
```
<!--  redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <exclusion>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
        </exclusion>
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
application.yml：
spring:
    redis: 
      database: 0
      host: localhost
      port: 6379
      password: 
      timeout: 0
      jedis:
        pool:
          max-active: 8
          max-wait: 20000
          max-idle: 20

RedisConfiguration.java：
序列化的配置
// StringRedisSerializer\JdkSerializationRedisSerializer进行序列化的，springboot是通过Jackson2JsonRedisSerializer进行序列化的
