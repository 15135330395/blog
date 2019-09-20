---
title: 07mysql集群和mycat
date: 2019-09-20 00:00:07
tags: [Linux,服务器,mysql集群]
categories: 搭建Linux服务器
---
# mysql 集群

## 主从复制（自带）

方式有多种：基于基于日志（binlog）的主从复制方式、

原理

- Master将数据改变记录到二进制日志(binary log)中，也就是配置文件log-bin指定的文件，
  这些记录叫做二进制日志事件(binary log events)；

- Slave 通过 I/O 线程读取 Master 中的 binary log events 并写入到它的中继日志(relay log)；

- Slave 重做中继日志中的事件， 把中继日志中的事件信息一条一条的在本地执行一次，完
  成数据在本地的存储， 从而实现将改变反映到它自己的数据(数据重放)。

主从配置需要注意的点
- 主从服务器操作系统版本和位数一致；
- Master 和 Slave 数据库的版本要一致；
- Master 和 Slave 数据库中的数据要一致；
- Master 开启二进制日志， Master 和 Slave 的 server_id 在局域网内必须唯一；

步骤：

### Master配置

1. 安装数据库；

2. 修改数据库配置文件， 指明 server_id， 开启二进制日志(log-bin)；

   vim /etc/my.cnf

   [mysqld]

   skip-name-resolve
   log-bin=mysql-bin
   log_timestamps=SYSTEM
   server-id=111

3. 启动数据库， 查看当前是哪个日志， position 号是多少；

   SHOW MASTER STATUS;

   show variables like 'server_id'; 

   set global server_id=111; #此处的数值和my.cnf里设置的一样就行 

4. 登录数据库， 授权数据复制用户（IP 地址为从机 IP 地址， 如果是双向主从， 这里的还需要授权本机的 IP 址， 此时自己的 IP 地址就是从 IP 地址)；

5. 备份数据库（记得加锁和解锁）；

6. 传送备份数据到 Slave 上；

7. 启动数据库；

主机查看从机
    show slave hosts;

### Slave 上的配置

1. 安装数据库

   （如果服务器完全复制 需要删掉/var/lib/mysql/auto.cnf文件并重启mysql 重新生成mysql的uuid）

2. 修改数据库配置文件， 指明 server_id（如果是搭建双向主从的话， 也要开启二进制日志 log-bin）；

   vim /etc/my.cnf

   [mysqld]
   log-bin=mysql-bin
   log_timestamps=SYSTEM
   server-id=112

   show variables like 'server_id'; 
   log_timestamps=SYSTEM
   set global server_id=112; #此处的数值和my.cnf里设置的一样就行 

3. 启动数据库， 还原备份；

4. 查看当前是哪个日志， position 号是多少（单向主从此步不需要， 双向主从需要）；

   SHOW MASTER STATUS;

5. 指定 Master 的地址、 用户、 密码等信息；（show的内容）

   CHANGE MASTER TO
   MASTER_HOST='192.168.1.111',
   MASTER_USER='root',
   MASTER_PASSWORD='Root123.',
   MASTER_LOG_FILE='mysql-bin.000001',
   MASTER_LOG_POS=155;

6. 开启同步， 查看状态。
   show slave status\G;
   start slave;
   show slave status\G;

确保Slave_IO_Running=Yes
　　Slave_SQL_Running=Yes 否则重新配置从服务器配置文件（或查看上次错误原因Last_IO_Error）

## 读写分离（mysql-proxy 第三方）

1.配置文件

vim /etc/my.cnf

主

```
[mysqld]
log-bin=mysql-bin #从库会基于此log-bin来做复制
binlog-do-db=mytest #用于读写分离的具体数据库，这里我创建了mytest作测试
binlog_ignore_db=mysql #不用于读写分离的具体数据库
binlog_ignore_db=information_schema #和binlog-do-db一样，可以设置多个
#选择row模式 
binlog-format=ROW
server-id=1
```

从
```
[mysqld]
log-bin=mysql-bin #从库会基于此log-bin来做复制
replicate-do-db=mytest #用于读写分离的具体数据库，这里我创建了mytest作测试
#选择row模式 
binlog-format=ROW
server-id=2
```
2.mysql-proxy是官方提供的mysql中间件产品可以实现负载平衡，读写分离（配多个 防止单点失效）

 下载 mysql-proxy-并安装

创建配置文件 安装目录/bin/mysql-proxy.conf
```
[mysql-proxy]
#用于中间件连接的用户
admin-username=root
admin-password=root
#根据存放的文件位置自行调整
admin-lua-script=C:/mysql-proxy-0.8.5-windows-x86-32bit/lib/mysql-proxy/lua/admin.lua 
#主库服务器+端口
proxy-backend-addresses=192.168.103.207:3307
#从库服务器+端口，多个从库用，隔开
proxy-read-only-backend-addresses=192.168.103.208:3307
#日志文件存放位置，如果你指定了一个路径，请确保手动创建了对应的文件夹，否则会报错
log-file=C:/mysql-proxy-0.8.5-windows-x86-32bit/log/mysql-proxy.log
#日志级别
log-level=debug
#以守护进程方式运行
daemon=true
#长连接
keepalive=true
```
启动

```
.\mysql-proxy.exe -P 192.168.103.203:6217 --defaults-file=mysql-proxy.conf
```

3.下载Atlas会有两个版本，其中有个分表的版本，但是这个需要其他的依赖，我这边不需要分表这种需求，所以安装普通的版本

> ​     Atlas (普通) : [**Atlas-2.2.1.el6.x86_64.rpm**](https://github.com/Qihoo360/Atlas/releases/download/2.2.1/Atlas-2.2.1.el6.x86_64.rpm)
>
> ​     Atlas (分表) : [**Atlas-sharding_1.0.1-el6.x86_64.rpm**](https://github.com/Qihoo360/Atlas/releases/download/sharding-1.0.1/Atlas-sharding_1.0.1-el6.x86_64.rpm)

下载并安装

/usr/local/mysql-proxy/里有4个文件夹（bin conf lib log）

bin目录下放的都是可执行文件

\1. “encrypt”是用来生成MySQL密码加密的，在配置的时候会用到

\2. “mysql-proxy”是MySQL自己的读写分离代理

\3. “mysql-proxyd”是360弄出来的，后面有个“d”，服务的启动、重启、停止。都是用他来执行的

conf目录下放的是配置文件

\1. “test.cnf”只有一个文件，用来配置代理的，可以使用vim来编辑

进入bin目录，使用encrypt来对数据库的密码进行加密，我的MySQL数据的用户名是buck，密码是hello，我需要对密码进行加密

```
./encrypt hello
```

配置Atlas，使用vim进行编辑

vim test.cnf

```
#管理接口的用户名
admin-username = user
#管理接口的密码
admin-password = pwd


#Atlas后端连接的MySQL主库的IP和端口，可设置多项，用逗号分隔
proxy-backend-addresses = 192.168.246.134:3306

#Atlas后端连接的MySQL从库的IP和端口，@后面的数字代表权重，用来作负载均衡，若省略则默认为1，可设置多项，用逗号分隔
proxy-read-only-backend-addresses = 192.168.246.135:3306@1

#用户名与其对应的加密过的MySQL密码，密码使用PREFIX/bin目录下的加密程序encrypt加密，下行的user1和user2为示例，将其替换为你的MySQL的用户名和加密密码！
pwds = buck:RePBqJ+5gI4=

#Atlas监听的工作接口IP和端口
proxy-address = 0.0.0.0:1234
#Atlas监听的管理接口IP和端口（还可以指定IP，其他的IP无法访问管理员的命令界面）
admin-address = 0.0.0.0:2345
```

启动

```
./mysql-proxyd test start
```
确定mysql本身进不去
```
/etc/init.d/mysqld status
mysql
```
Atlas的管理模式能进去 它会把自己当成一个MySQL数据库
mysql -h127.0.0.1 -P2345 -uuser -ppwd 

```
查看功能
select * from help;
```

通过工作接口来访问

mysql -h127.0.0.1 -P1234 -ubuck -phello

可以让数据库某一台down掉，来测试监控的可用性

select * from backends;

## 读写分离测试

这里测试读写分离需要使用到Jmeter了，它是Java写第一套开源的压力测试工具，因为这个比较方便。他有专门测试MySQL的模块，需要使用MySQL的JDBC驱动jar包，配置很简单，东西很好很强大很好用。

Jmeter下载地址：http://jmeter.apache.org/download_jmeter.cgi

MySQL的JDBC  ：http://dev.mysql.com/downloads/connector/j/

# mycat集群（第三方）

## 简介

支持JDBC连接ORACLE、DB2、SQL Server，将其模拟为MySQL  Server使用

1、**Schema**：逻辑库，与MySQL中的Database（数据库）对应，一个逻辑库中定义了所包括的Table。

2、**Table**：表，即物理数据库中存储的某一张表，与传统数据库不同，这里的表格需要声明其所存储的逻辑数据节点DataNode。**在此可以指定表的分片规则。**

3、**DataNode**：MyCAT的逻辑数据节点，是存放table的具体物理节点，也称之为分片节点，通过DataSource来关联到后端某个具体数据库上

4、**DataSource**：定义某个物理库的访问地址，用于捆绑到Datanode上

## 数据切分介绍

垂直分割（不同表在不同数据库）

优点：
- 拆分后业务清晰，拆分规则明确。 
- 系统之间整合或扩展容易。 
- 数据维护简单。

缺点：
- 部分业务表无法join，只能通过接口方式解决，提高了系统复杂度。
- 受每种业务不同的限制存在单库性能瓶颈，不易数据扩展跟性能提高。 
- 事务处理复杂。

由于垂直切分是按照业务的分类将表分散到不同的库，所以有些业务表会过于庞大，存在单库读写与存储瓶颈，所以就需要水平 拆分来做解决。

水平分割（数据量）

几种典型的分片规则包括：

- 按照用户ID求模，将数据分散到不同的数据库，具有相同数据用户的数据都被分散到一个库中。 

- 按照日期，将不同月甚至日的数据分散到不同的库中。 

- 按照某个特定的字段求摸，或者根据特定范围段分散到不同的库中。

优点有：
- 拆分规则抽象好，join操作基本可以数据库做。 
- 不存在单库大数据，高并发的性能瓶颈。 
- 应用端改造较少。 
- 提高了系统的稳定性跟负载能力。

缺点有：
- 拆分规则难以抽象。 
- 分片事务一致性难以解决。 
- 数据多次扩展难度跟维护量极大。 
- 跨库join性能较差。

前面讲了垂直切分跟水平切分的不同跟优缺点，会发现每种切分方式都有缺点，但共同的特点缺点有：

- 引入分布式事务的问题。 
- 跨节点Join的问题。
- 跨节点合并排序分页问题。 
- 多数据源管理问题。
## 安装

官方网站：

http://www.mycat.org.cn/

github地址

https://github.com/MyCATApache

下载后解压

```
cd /usr/mycat
tar -zxvf Mycat-server-1.6.7.3-release-20190828135747-linux.tar.gz
```

启动命令：./mycat start

停止命令：./mycat stop

重启命令：./mycat restart

注意：可以使用mysql的客户端直接连接mycat服务。默认服务端口为8066

```
mysql -uroot -p -P8066 -h127.0.0.1 -default_auth=mysql_native_password（因为连的mysql8 不写-default_auth会报密码错误 -h必须写上并加上IP）
```

--bin			  启动目录
--conf			配置文件存放配置文件
--lib				MyCAT自身的jar包或依赖的jar包的存放目录。
--logs			 MyCAT日志的存放目录。日志存放在logs/log中，每天一个文件

## 配置

由于mycat不支持mysql8.0，所以需要将mysql8.0配置成5.x

1.修改加密规则

在mysql8之前的版本使用的密码加密规则是mysql_native_password，但是在mysql8则是caching_sha2_password。

```
use mysql;
select user,host,plugin from user;
update user set plugin='mysql_native_password' where User='root';#修改加密规则
ALTER USER 'root'@'localhost' BY 'Root123.';#更新一下用户的密码
FLUSH PRIVILEGES; #刷新权限
```

2.如果是在Linux平台，在首次启动前设置lower_case_table_names = 1（表名大小写不敏感），需要删除数据（不知道需不需要）

```
systemctl stop mysqld.service
rm -rf /var/lib/mysql
vim /etc/my.cnf
lower_case_table_names=1
保存
systemctl restart mysqld.service
```





https://blog.csdn.net/jaysonhu/article/details/52858535

### 配置schema.xml

管理着MyCat的逻辑库、表、分片规则、DataNode以及DataSource

是逻辑库定义和表以及分片定义的配置文件。

数据库8.0 所以需要替换jdbc驱动包和连接方式

```
#删除mycat/lib下的mysql5.x驱动包（不删 可能也行）
cd /usr/mycat/mycat/lig
rm mysql-connector-java-5.1.35.jar
放入mysql8.0驱动包 mysql-connector-java-8.0.17.jar
对文件赋权限
chmod 777 mysql-connector-java-8.0.17.jar

如果mysql unblock with ‘mysqladmin flush-hosts’
解决方法登陆mysql -u root -p 后执行命令 flush hosts;
```

vim schema.xml
```
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://org.opencloudb/">
<schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100">
<!-- auto sharding by id (long) -->
		<table name="TB_ITEM" dataNode="dn1,dn2,dn3" rule="auto-sharding-long" />
		<table name="TB_USER" primaryKey="ID" type="global" dataNode="dn1,dn2" />
	</schema>
	<dataNode name="dn1" dataHost="localhost1" database="db1" />
	<dataNode name="dn2" dataHost="localhost2" database="db2" />
	<dataNode name="dn3" dataHost="localhost1" database="db3" />
	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
	writeType="0" dbType="mysql" dbDriver="#native#改成jdbc" switchType="1" slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<!-- can have multi write hosts -->
		<writeHost host="hostM1" url="#192.168.1.111:3306#改成jdbc:mysql://192.168.1.111:3306?useSSL=false&amp;useUnicode=true&amp;characterEncoding=UTF8&amp;serverTimezone=UTC" user="root"
			password="root">
			<!-- can have multi read hosts -->

		</writeHost>
	</dataHost>
	<dataHost name="localhost2" maxCon="1000" minCon="10" balance="0"
		writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<!-- can have multi write hosts -->
		<writeHost host="hostM1" url="192.168.25.166:3306" user="root"
			password="root">
			<!-- can have multi read hosts -->
		</writeHost>
	</dataHost>
</mycat:schema>
```

### 配置server.xml

保存了所有mycat需要的系统配置信息 是Mycat服务器参数调整和用户授权的配置文件。

```


<user name="test">
    <property name="password">test</property>
    <property name="schemas">TESTDB</property>
    <property name="readOnly">true</property>
</user>
```



```
mysql -uroot -p -P8066 -h127.0.0.1 -default_auth=mysql_native_password
数据系统

mysql -uroot -p -P9066 -h127.0.0.1 -default_auth=mysql_native_password
管理系统常用命令
show @@help;
```



### 配置rule.xml

是分片规则的配置文件，分片规则的具体一些参数信息单独存放为文件，也在这个目录下，配置文件修改需要重启MyCAT。

rule.xml里面就定义了我们对表进行拆分所涉及到的规则定义，可以灵活的对表使用不同的分片算法，或者对表使用相同的算法但具体的参数不同。这个文件里面主要有tableRule和function这两个标签。在具体使用过程中可以按照需求添加tableRule和function。可以不做修改，使用默认配置

### 配置log4j.xml

日志存放在logs/log中，每天一个文件，日志的配置是在conf/log4j.xml中，根据自己的需要可以调整输出级别为debug                           debug级别下，会输出更多的信息，方便排查问题。

autopartition-long.txt,partition-hash-int.txt,sequence_conf.properties， sequence_db_conf.properties 分片相关的id分片规则配置文件

## 测试分片

创建表并插入数据

由于配置的分片规则为“auto-sharding-long”，所以mycat会根据此规则自动分片。

每个datanode中保存一定数量的数据。根据id进行分片

经测试id范围为：

Datanode1：1~5000000

Datanode2：5000000~10000000

Datanode3：10000001~15000000

 

当15000000以上的id插入时报错：

[Err] 1064 - can't find any valid datanode :TB_ITEM -> ID -> 15000001

此时需要添加节点了。



主从复制（同上）



读写分离

Mycat 1.4 支持MySQL主从复制状态绑定的读写分离机制，让读更加安全可靠，配置如下：

```
<dataNode name="dn1" dataHost="localhost1" database="db1" />
	<dataNode name="dn2" dataHost="localhost1" database="db2" />
	<dataNode name="dn3" dataHost="localhost1" database="db3" />
	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
		writeType="0" dbType="mysql" dbDriver="native" switchType="2"  slaveThreshold="100">
		<heartbeat>show slave status</heartbeat>
		<writeHost host="hostM" url="192.168.25.134:3306" user="root"
			password="root">
			<readHost host="hostS" url="192.168.25.166:3306" user="root"
			password="root" />
		</writeHost>
	</dataHost>
```



**(1)** **设置 balance="1"与writeType="0"**

Balance参数设置：

\1. balance=“0”, 所有读操作都发送到当前可用的writeHost上。

\2. balance=“1”，所有读操作都随机的发送到readHost。

\3. balance=“2”，所有读操作都随机的在writeHost、readhost上分发

WriteType参数设置：

\1. writeType=“0”, 所有写操作都发送到可用的writeHost上。

\2. writeType=“1”，所有写操作都随机的发送到readHost。

\3. writeType=“2”，所有写操作都随机的在writeHost、readhost分上发。

 “readHost是从属于writeHost的，即意味着它从那个writeHost获取同步数据，因此，当它所属的writeHost宕机了，则它也不会再参与到读写分离中来，即“不工作了”，这是因为此时，它的数据已经“不可靠”了。基于这个考虑，目前mycat 1.3和1.4版本中，若想支持MySQL一主一从的标准配置，并且在主节点宕机的情况下，从节点还能读取数据，则需要在Mycat里配置为两个writeHost并设置banlance=1。”

**(2)** **设置 switchType="2" 与slaveThreshold="100"**

**switchType** **目前有三种选择：**

**-1****：表示不自动切换**

**1** **：默认值，自动切换**

**2** **：基于MySQL主从同步的状态决定是否切换**

“Mycat心跳检查语句配置为 show slave status ，dataHost 上定义两个新属性： switchType="2" 与slaveThreshold="100"，此时意味着开启MySQL主从复制状态绑定的读写分离与切换机制。Mycat心跳机制通过检测 show slave status 中的 "Seconds_Behind_Master", "Slave_IO_Running", "Slave_SQL_Running" 三个字段来确定当前主从同步的状态以及Seconds_Behind_Master主从复制时延。“



## 监控平台

emmm... 需要用到zookeeper

目前被监控的MySQL版本支持5.7（推荐），5.6。

而且还不支持8.0。。。有点尴尬



# MyCAT自增字段和返回生成的主键ID的经验分享

MyCAT自增字段和返回生成的主键ID的经验分享
说明：
1、mysql本身对非自增长主键，使用last_insert_id()是不会返回结果的，只会返回0.
2、mysql只会对定义自增长主键，可以用last_insert_id()返回主键值。

mycat目前提供了自增长主键功能，但是如果对应的mysql节点上数据表，没有定义auto_increment，
那么在mycat层调用last_insert_id()也是不会返回结果的。
正确使用方式如下：
1、mysql定义自增主键
CREATE TABLE `tt2` (
  `id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `nm` INT(10) UNSIGNED NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MYISAM AUTO_INCREMENT=6 DEFAULT CHARSET=utf8;
2、mycat定义自增
[root@test conf]# vim schema.xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://org.opencloudb/">

        <schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100">
            <!-- random sharding using mod sharind rule -->
        <!-- autoIncrement="true" 属性-->
            <table name="tt2" primaryKey="id" autoIncrement="true" dataNode="dn1,dn2,dn3,dn4,dn5" rule="mod-long" />
            <table name="mycat_sequence" primaryKey="name" dataNode="dn1"/>
        </schema>
    
        <dataNode name="dn1" dataHost="localhost1" database="db1" />
        <dataNode name="dn2" dataHost="localhost1" database="db2" />
        <dataNode name="dn3" dataHost="localhost1" database="db3" />
        <dataNode name="dn4" dataHost="localhost1" database="db4" />
        <dataNode name="dn5" dataHost="localhost1" database="db5" />
    
        <dataHost name="localhost1" maxCon="1000" minCon="20" balance="0" writeType="0" dbType="mysql" dbDriver="native">
                <heartbeat>select user()</heartbeat>
                <writeHost host="hostM1" url="127.0.0.1:3366" user="root" password="123456">
                </writeHost>
        </dataHost>
</mycat:schema>

3、mycat对应sequence_db_conf.properties增加相应设置；
4、mycat的对应mycat_sequence增加对应记录。
5、链接mycat，测试结果如下：

127.0.0.1/root:[TESTDB> insert into tt2(nm) values (99);
Query OK, 1 row affected (0.14 sec)

127.0.0.1/root:[TESTDB> select last_insert_id();
+------------------+
| LAST_INSERT_ID() |
+------------------+
|              101 |
+------------------+
1 row in set (0.01 sec)

127.0.0.1/root:[TESTDB> insert into tt2(nm) values (99);
Query OK, 1 row affected (0.00 sec)

127.0.0.1/root:[TESTDB> select last_insert_id();
+------------------+
| LAST_INSERT_ID() |
+------------------+
|              102 |
+------------------+
1 row in set (0.00 sec)

127.0.0.1/root:[TESTDB> insert into tt2(nm) values (99);
Query OK, 1 row affected (0.00 sec)

127.0.0.1/root:[TESTDB> select last_insert_id();
+------------------+
| LAST_INSERT_ID() |
+------------------+
|              103 |
+------------------+
1 row in set (0.00 sec)

127.0.0.1/root:[TESTDB> insert into tt2(nm) values (99);
Query OK, 1 row affected (0.01 sec)

127.0.0.1/root:[TESTDB> select last_insert_id();
+------------------+
| LAST_INSERT_ID() |
+------------------+
|              104 |
+------------------+
1 row in set (0.00 sec)

127.0.0.1/root:[TESTDB> insert into tt2(nm) values (99);
Query OK, 1 row affected (0.00 sec)

127.0.0.1/root:[TESTDB> select last_insert_id();
+------------------+
| LAST_INSERT_ID() |
+------------------+
|              105 |
+------------------+
1 row in set (0.00 sec)

127.0.0.1/root:[TESTDB> insert into tt2(nm) values (99);
Query OK, 1 row affected (0.00 sec)

127.0.0.1/root:[TESTDB> select last_insert_id();
+------------------+
| LAST_INSERT_ID() |
+------------------+
|              106 |
+------------------+
1 row in set (0.00 sec)