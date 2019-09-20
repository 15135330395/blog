---
title: 08nginx集群
date: 2019-09-20 00:00:08
tags: [Linux,服务器,nginx集群]
categories: 搭建Linux服务器
---
nginx是一款高性能的http 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器。官方测试nginx能够支支撑5万并发链接，并且cpu、内存等资源消耗却非常低，运行非常稳定。

常见反向代理服务器：

Nginx、lvs、F5（硬件）、haproxy

应用场景：

1、http服务器。Nginx是一个http服务可以独立提供http服务。可以做网页静态服务器。

2、虚拟主机。可以实现在一台服务器虚拟出多个网站。例如个人网站使用的虚拟主机。

3、反向代理，负载均衡。当网站的访问量达到一定程度后，单台服务器不能满足用户的请求时，需要用多台服务器集群可以使用nginx做反向代理。并且多台服务器可以平均分担负载，不会因为某台服务器负载高宕机而某台服务器闲置的情况。

优点：

占内存小，可以实现高并发连接、处理响应快。

可以实现http服务器、虚拟主机、反向代理、负载均衡。

nginx配置简单

可以不暴露真实服务器IP地址



## 1.安装

在Centos下，yum源不提供nginx的安装，可以通过切换yum源的方法获取安装。也可以通过直接下载安装包的方法，以下命令均需root权限执行：

首先安装必要的库（nginx 中gzip模块需要 zlib 库，rewrite模块需要 pcre 库，ssl 功能需要openssl库）

yum list installed | grep zlib

yum list installed | grep pcre

yum list installed | grep openssl

（最小版都装上了）

新增yum源

```
vim /etc/yum.repos.d/nginx.repo
新增
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
保存
```

yum install nginx

## 2.配置和常用命令

```
服务
systemctl enable nginx.service
systemctl restart nginx.service

vim /etc/nginx/conf.d/default.conf
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
启动服务器
nginx -c /etc/nginx/nginx.conf
关闭服务器
nginx -s quit
检测配置文件
nginx -t
重启服务器/重新加载配置文件
./nginx -s reload
```

## 3.反向代理

启动两个 Tomcat  8080 8081

修改配置文件
```
server {
	listen       80;
	server_name  8080.localhost;
	location / {
		proxy_pass  http://127.0.0.1:8080;
		index  index.html index.htm;
	}
}
server {
	listen       80;
	server_name  8081.localhost;
	location / {
		proxy_pass  http://127.0.0.1:8081;
		index  index.html index.htm;
	}
}
```

./nginx -s reload


## 4.负载均衡

### 1.轮询（默认）

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。 
upstream backserver { 
server 192.168.0.14; 
server 192.168.0.15; 
} 

### 2.指定权重

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。 
upstream backserver { 
server 192.168.0.14
weight=10; 
server 192.168.0.15
weight=10; 
}

### 3.IP绑定 ip_hash

每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。 
upstream backserver { 
ip_hash; 
server 192.168.0.14:88; 
server 192.168.0.15:80; 
} 

## 高并发解决方案

业务数据库  -》 数据水平分割(分区分表分库)、读写分离

业务应用 -》 逻辑代码优化(算法优化)、公共数据缓存

应用服务器 -》 反向静态代理、配置优化、负载均衡(apache分发，多tomcat实例)

系统环境 -》 JVM调优

页面优化 -》 减少页面连接数、页面尺寸瘦身

1、动态资源和静态资源分离；

2、CDN；

3、负载均衡；

4、分布式缓存；

5、数据库读写分离或数据切分（垂直或水平）；

6、服务分布式部署。

 