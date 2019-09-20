---
title: 03tomcat环境
date: 2019-09-20 00:00:03
tags: [Linux,服务器,tomcat安装]
categories: 搭建Linux服务器
---
## 1.解压安装包
```
tar -zxvf /usr/tomcat/apache-tomcat-9.0.24.tar.gz
```

## 2.测试启动Tomcat

进入tomcat的bin目录

```
sh ./startup.sh
```

如果报错-bash: startup.sh: command not found

则chmod u+x *.sh

