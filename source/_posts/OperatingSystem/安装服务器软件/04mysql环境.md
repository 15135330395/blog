---
title: 04mysql环境
date: 2019-09-20 00:00:04
tags: [Linux,服务器,mysql安装]
categories: 搭建Linux服务器
---
## 1.安装

yum安装

```
yum install mysql-server
```
官网源安装

```
wget -ox /usr/mysql/mysql80-community-release-el7-3.noarch.rpm https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm

yum localinstall mysql80-community-release-el7-3.noarch.rpm
查询可以安装的版本
yum repolist all | grep mysql
安装服务器版
yum install mysql-community-server
```

压缩版安装（比较麻烦 不建议）

```
tar –xzvf mysql-5.1.56.tar.gz #对mysql tar包解压缩
cd myql-5.1.56
./configure –prefix=/usr/local/mysql #制定mysql的安装目录
make #编译源代码
make install #安装
cp support-files/my-medium.cnf /etc/my.cnf #复制配置文件模板
cd /usr/local/mysql
bin/mysql_install_db –user=root #初始化安装mysql数据库
bin/mysqld_safe –user=root & #使用用户mysql安全启动mysql程序并放到后台执行

配置环境变量
将以下信息添加到用户下的.base_profile 文件中 
#mysql items begin
export MYSQL_HOME=MySQL 安装目录
export PATH=$MYSQL_HOME/bin:$PATH
export LD_LIBRARY_PATH=$MYSQL_HOME/lib:$LD_LIBRARY_PATH
export LIBPATH=$LD_LIBRARY_PATH:$LIBPATH
#mysql items end
```


## 2.启动mysql

```
service mysqld start
systemctl start mysqld
```

## 3.修改mysqld执行权限（可能用不到）

```
chmod 755 /etc/rc.d/init.d/mysqld 
```
## 4.修改mysql的root账户密码

```
查看临时密码
grep 'temporary password' /var/log/mysqld.log
//进入mysql
mysql -u root -p
//修改密码
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Root123.';
//查看 mysql 初始的密码策略
SHOW VARIABLES LIKE 'validate_password%';
//设置密码的验证强度等级
set global validate_password_policy=LOW;
//设置密码的长度
set global validate_password_length=6;

或

mysqladmin -u root password 'newpwd'（8位+大小写+加数字+特殊字符）
```

## 5.设置mysql开机启动

```
chkconfig mysqld on 
systemctl enable mysqld.service
```

## 6.设置远程连接

让%（所有ip的）用户拥有访问所有库和表的权限。用户名为root 密码为123456
```
mysql -u root -p
use mysql;
select host, user, authentication_string, plugin from user;
update user set host = "%" where user = "root";
```

```
grant all privileges on *.* to root@'%' identified by 'root' with grant option;(8版本用不到了)
```

刷新权限，不用重启立即生效

```
flush privileges;
```

## 7.连接mysql时报caching_sha2_password错误

原因：

mysql8.0和5.x其中一个改动就是加密认证方式发生改变，

caching_sha2_password是8.0
mysql_native_password是5.x

解决方案：

1，更改mysql的加密认证方式

```
use mysql;
update user set plugin='mysql_native_password' where user='root';
flush privileges;
```
2，更改mysql的jdbc版本

## 8. Host is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'报错

原因：

同一个ip在短时间内产生太多（超过mysql数据库max_connection_errors的最大值）中断的数据库连接而导致的阻塞；

解决方法：

解决方法：
   1、提高允许的max_connection_errors数量：
　　① 进入Mysql数据库查看max_connection_errors： show variables like '%max_connect_errors%'; 
　   ② 修改max_connection_errors的数量为1000： set global max_connect_errors = 1000; 
　　③ 查看是否修改成功：show variables like '%max_connect_errors%'; 

   2、使用mysqladmin flush-hosts 命令清理一下hosts文件（不知道mysqladmin在哪个目录下可以使用命令查找：whereis mysqladmin）；
　　① 在查找到的目录下使用命令修改：mysqladmin --socket=/tmp/kkimdb.sock --port=3306 -uhyman -p flush-hosts
　　备注： 配置有master/slave主从数据库的要把主库和从库都修改一遍的（我就吃了这个亏明明很容易的几条命令结果折腾了大半天）；
   flush hosts; 也可以

