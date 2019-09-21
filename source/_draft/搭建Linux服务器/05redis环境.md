---
title: 05redis环境
date: 2019-09-20 00:00:05
tags: [Linux,服务器,redis安装]
categories: 搭建Linux服务器
---
## 应用场景

缓存（数据查询、短链接、新闻内容、商品内容等）（最多）

聊天室的在线好友列表

任务队列（秒杀、抢购、12306等）

应用排行榜

网站访问统计

数据过期处理（可以精确到毫秒）

分布式集群框架中的session分离

## 安装

redis由C语言开发，安装redis需要先将官网下载的源码进行编译，编译依赖gcc环境。

1.安装gcc

```
yum list installed | grep gcc
yum list | grep gcc
yum install gcc
```

2.解压redis压缩包

```
cd /usr/soft/
tar -zxvf redis-5.0.4.tar.gz
```

3.编译

```
cd redis-5.0.4
make

编译结束
Hint: It's a good idea to run 'make test' ;)
```

4.安装

```
make install PREFIX=/usr/soft/redis
```

安装结束后，/usr/soft/redis/bin下有几个可执行文件

redis-benchmark——性能测试工具
redis-check-aof——AOF文件修复工具
redis-check-rdb——RDB文件检查工具（快照持久化工具）
redis-cli——命令行客户端
redis-sentinel（链接文件 到server）
redis-server——redis服务器启动命令

5.配置文件

cp /usr/soft/redis/redis-5.0.4/redis.conf /usr/soft/redis/bin/redis.conf

vim /usr/soft/redis/bin/redis.conf

```
#IP绑定 69行
bind 192.168.1.111
#是否开启保护模式，由yes该为no 88行
protected-mode no
#数据库的数量(默认是16个 相互隔离 但是可以使用flushall一次清空所有的库) 根据需求修改 186行
databases 16
```

6.启动

前端模式启动（测试）

```
/usr/redis/bin/redis-server

ctrl+c结束
```

后端模式启动

```
vim /usr/redis/bin/redis.conf
#后端模式 136行
daemonize yes
保存
./redis-server redis.conf

./redis-cli shutdown——关闭
./redis-cli -h 192.168.1.111 -p 6379 shutdown
```

## 存储结构及指令

使用
Redis支持类型
1．String（字符串）
命令：
	增：set 键 字符串值
	查询：GET 键
	返回key对应字符串值的指定字符：GETRANGE 键 开始 结束（包含）
	设置并返回旧值：GETSET 键 新值
	获取/设置多个key：MGET 键1 键2 ..../MSET 键1 值1 键2 值2 ....
	当value为整数数据时，可以设为自增：INCR 键 / INCRBY 键 步长
				自减：DECR 键 / DECR 键 步长
	向键值的末尾追加value。如果键不存在则将该键的值设置为value，即相当于 SET key value。返回值是追加后字符串的总长度：APPEND 键 值
	获得字符串长度 键不存在则返回0：STRLEN 键

2．Hash（哈希，key-value，适合存储对象 key是对象 value是属性 字段值只能是字符串类型）
命令：
	增：hmset 键 字段1 字段值1 字段2 字段值2 ....
	      HSET 键 字段 字段值（只能设置一个字段值 不区分插入和更新操作 插入返回1 更新返回0）
	删：HDEL 键 字段1 字段2 ....
	查：HGET 键 字段
	      HMGET 键 字段1 字段2 ....（获取多个字段值）
	      HGETALL 键（获取所有的字段和字段值）
	判断字段是否存在：HEXISTS 键 字段
	只获取字段名或字段值：HKEYS 键 /  HVALS 键
	获取字段数量：HLEN 键

3．List（列表 本质是双向链表）
命令：
	向列表左/右增加元素：lpush/RPUSH 键 值1 值2 ....
	移出并获取列表的第一个/最后一个元素，等待时间设置为秒，阻塞：BLPOP/BRPOP 键 时间
	求长度：LLEN 键
	移出并获取列表的第一个/最后一个元素：LPOP/RPOP 键
	查看列表：LRANGE 键 开始 结束

4．Set（集合，无序不重复）
命令：
	增：sadd 键 值1 值2 ....
	获取数量：SCARD 键
	获得所有元素：SMEMBERS 键
	删除：SREM 键 值1 值2 ....
	判断元素是否在集合中：SISMEMBER 键 值1
	差集/交集/并集：SDIFF/SINTER/SUNION 键1 键2
	返回并移除指定set中的随机某个元素：SPOP 键

5．zset(sorted set：有序值不重复  序/分数可以相同)
命令：
	增：zadd 键 序1 值 序2 值2 ....
	获得有序集合数量：ZCARD 键
	获得有序集合指定范围的从小到大的值：ZRANGE 键 开始 结束（包含）
	获得有序集合指定范围的从大到小的值：ZREVRANGE 键 开始 结束（包含）WITHSCORES（获得分数）
	获取元素的分数：ZSCORE 键 值
	删除：ZREM 键 值1 值2 ....

通用：
	删除键：DEL 键
	键是否存在：EXISTS 键
	查看所有满足条件的key：KEYS 键开头*（模板）
	重命名：RENAME 旧键 新键
	显示指定key的数据类型：type key
	设置key的有效期，单位为seconds：EXPIRE 键 时间
	删除key的有效期，变为永久有效：PERSIST 键
	返回key的剩余有效时间，pttl返回毫秒，ttl返回秒：TTL/PTTL 键

事务
	开启事务：MULTI
	触发事务，提交：EXEC
	取消事务，清除：DISCARD
	当某个事务需要按条件执行时，就要使用这个命令将给定的键设置为受监控的状态：watch 键1 键2 ....（可以实现乐观锁）
	清除所有先前为一个事务监控的键：unwatch

Redis不支持事务回滚（？ 回滚：discard）
1.大多数事务失败是因为语法错误或者类型错误，这两种错误，在开发阶段都是可以预见的（？ 如果出现异常 则忽略）
2.为了性能方面就忽略了事务回滚

## 持久化

Redis是一个**内存**数据库，为了保证数据的持久性，它提供了两种持久化方案:

RDB方式（默认 通过**快照**完成的，当**符合一定条件**时Redis会自动将内存中的数据进行快照并持久化到硬盘）

AOF方式 （每执行一条会**更改Redis中的数据的命令**，Redis就会将该命令写入硬盘中的AOF文件）

### RDB方式

#### Redis会在指定的情况下触发快照

**1.**     **符合自定义配置的快照规则**

**2.**     **执行save或者bgsave命令**

**3.**     **执行flushall命令**

**4.**     **执行主从复制操作**

#### 在redis.conf中设置自定义快照规则

save <seconds> <changes> 

save 900 1：表示15分钟（900秒钟）内至少1个键被更改则进行快照

可以配置多个条件（每行配置一个条件），每个条件之间是“或”的关系

#### 配置dir指定rdb快照文件的位置

dir ./

#### 配置dbfilename指定rdb快照文件的名称

dbfilename dump.rdb

#### 优缺点

缺点：一旦Redis异常退出，就会**丢失最后一次快照以后更改的所有数据**

优点：RDB可以最大化Redis的性能，保存时会分出子进程，但数据量大会耗时变缺点

### AOF方式

开启：redis.conf配置文件中的appendonly参数开启

appendonly yes

保存位置

dir ./

默认的文件名

appendfilename **appendonly.aof**

## 主从复制和切换主从机

主redis不需要配置 只需要修改从redis配置

修改从服务器上的redis.conf文件
```
slaveof 主服务器IP 主服务器端口
```
只有从机第一次连接上主机是全量同步
断线重连有可能触发全量同步也有可能是增量同步（master判断runid是否一致）
除此之外的情况都是增量同步

### Sentinel 哨兵进程

监控redis集群中Master主服务器工作的状态

作用：监控 提醒 自动迁移（自动改变主从机的配置文件 以及 哨兵监控文件）

修改从机的sentinel.conf

```
#sentinel monitor <master-name> <master ip> <master port> <quorum>
sentinel monitor mymaster 192.168.10.133 6379 1
```
配置文件说明
```
# 哨兵sentinel实例运行的端口 默认26379
port 26379
# 哨兵sentinel的工作目录
dir /tmp
# 哨兵sentinel监控的redis主节点的 ip port 
# master-name  可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。
# quorum 当这些quorum个数sentinel哨兵认为master主节点失联 那么这时 客观上认为主节点失联了
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 2
# 当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供密码
# 设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster MySUPER--secret-0123passw0rd
 
 
# 指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒
# sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000
 
# 这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行 同步，
这个数字越小，完成failover所需的时间就越长，
但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。
可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。
# sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1

# 故障转移的超时时间 failover-timeout 可以用在以下这些方面： 
#1. 同一个sentinel对同一个master两次failover之间的间隔时间。
#2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
#3.当想要取消一个正在进行的failover所需要的时间。  
#4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了
# 默认三分钟
# sentinel failover-timeout <master-name> <milliseconds>
sentinel failover-timeout mymaster 180000
 
#配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
#对于脚本的运行结果有以下规则：
#若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
#若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
#如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
#一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
 
#通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，
这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，
一个是事件的类型，
一个是事件的描述。
如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。
#通知脚本
# sentinel notification-script <master-name> <script-path>
  sentinel notification-script mymaster /var/redis/notify.sh
  
# 客户端重新配置主节点参数脚本
# 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
# 以下参数将会在调用脚本时传给脚本:
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
# 目前<state>总是“failover”,
# <role>是“leader”或者“observer”中的一个。 
# 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的
# 这个脚本应该是通用的，能被多次调用，不是针对性的。
# sentinel client-reconfig-script <master-name> <script-path>
 sentinel client-reconfig-script mymaster /var/redis/reconfig.sh

```

启动

```
./redis-sentinel sentinel.conf
```

## Jedis

### 普通环境
Jedis 包：jedis-2.9.0.jar + 数据库连接池包：commons-pool2
```
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
```
private Jedis jedis = new Jedis("localhost", 6379);
```
连接池连接
```
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
@Configuration
@EnableCaching
public class RedisConfiguration extends CachingConfigurerSupport {
	private static final Logger logger = LoggerFactory.getLogger(RedisConfiguration.class);
	@Bean
	@Override
	public KeyGenerator keyGenerator() {
		return (target, method, params) -> {
			StringBuilder sb = new StringBuilder();
			sb.append(target.getClass().getName());
			sb.append(":");
			sb.append(method.getName());
			for (Object obj : params) {
				sb.append(":" + String.valueOf(obj));
			}
			String rsToUse = String.valueOf(sb);
			logger.info("自动生成Redis Key -> [{}]", rsToUse);
			return rsToUse;
		};
	}

	@Bean
	public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
		// StringRedisSerializer\JdkSerializationRedisSerializer进行序列化的，springboot是通过Jackson2JsonRedisSerializer进行序列化的
		Jackson2JsonRedisSerializer<Object> redisSerializer = new Jackson2JsonRedisSerializer<Object>(
				Object.class);
		ObjectMapper om = new ObjectMapper();
		om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
		om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
		redisSerializer.setObjectMapper(om);
	
	//JdkSerializationRedisSerializer redisSerializer = new JdkSerializationRedisSerializer();
	
		// 配置redisTemplate
		RedisTemplate<String, Object> redisTemplate = new RedisTemplate<String, Object>();
		redisTemplate.setConnectionFactory(connectionFactory);
		RedisSerializer<?> stringSerializer = new StringRedisSerializer();
		redisTemplate.setKeySerializer(stringSerializer); // key序列化
		redisTemplate.setValueSerializer(redisSerializer); // value序列化
		redisTemplate.setHashKeySerializer(stringSerializer); // Hash key序列化
		redisTemplate.setHashValueSerializer(redisSerializer); // Hash
																			// value序列化
		redisTemplate.afterPropertiesSet();
		return redisTemplate;
	}
	
	@Override
	@Bean
	public CacheErrorHandler errorHandler() {
		// 异常处理，当Redis发生异常时，打印日志，但是程序正常走
		logger.info("初始化 -> [{}]", "Redis CacheErrorHandler");
		CacheErrorHandler cacheErrorHandler = new CacheErrorHandler() {
			@Override
			public void handleCacheGetError(RuntimeException e, Cache cache, Object key) {
				logger.error("Redis occur handleCacheGetError：key -> [{}]", key, e);
			}
	
			@Override
			public void handleCachePutError(RuntimeException e, Cache cache, Object key, Object value) {
				logger.error("Redis occur handleCachePutError：key -> [{}]；value -> [{}]", key, value, e);
			}
	
			@Override
			public void handleCacheEvictError(RuntimeException e, Cache cache, Object key) {
				logger.error("Redis occur handleCacheEvictError：key -> [{}]", key, e);
			}
	
			@Override
			public void handleCacheClearError(RuntimeException e, Cache cache) {
				logger.error("Redis occur handleCacheClearError：", e);
			}
		};
		return cacheErrorHandler;
	}
	}

## 集群

