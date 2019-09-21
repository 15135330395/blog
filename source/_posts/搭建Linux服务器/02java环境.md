---
title: 02java环境
date: 2019-09-20 00:00:02
tags: [Linux,服务器,Java安装]
categories: 搭建Linux服务器
---
## 0.删除系统预装jdk

可以一条命令直接删除

```
rpm -e --nodeps `rpm -qa | grep java`
```


## 1.解压tar包

```
tar -zxvf jdk-8u211-linux-x64.tar.gz 
```

## 2.编辑java环境变量配置文件

```
vim /etc/profile 
```

```
最底下加上
export JAVA_HOME=/usr/soft/java/jdk1.8.0_221
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=.:$JAVA_LIB/tools.jar:$JAVA_LIB/dt.jar
export PATH=$JAVA_HOME/bin:$PATH
保存退出
```

```
source /etc/profile  使配置生效（可能会需要等一会儿才能生效）
或
. /etc/profile
```

如果报错 -bash: /user/java/jdk1.8.0_221/bin/java: Permission denied
则 chmod 777 /user/java/jdk1.8.0_221/bin/java

## 3.查询java版本
```
java –version
```
## 4.运行jar 

java -jar springboot-0.0.1-SNAPSHOT.jar

## 5.后台运行jar

nohup java -jar springboot-0.0.1-SNAPSHOT.jar &
nohup java -jar springboot-0.0.1-SNAPSHOT.jar  > log.file  2>&1 &



如果是访问云服务器的页面，可能需要配置安全规则 添加所有访问到本服务器的8080端口