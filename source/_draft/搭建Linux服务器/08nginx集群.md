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

1、http服务器。Nginx是一个http服务可以独立提供http服务。可以做**网页静态**服务器。

2、虚拟主机。可以实现在一台服务器虚拟出多个网站。例如个人网站使用的虚拟主机。

3、反向代理，负载均衡。当网站的访问量达到一定程度后，单台服务器不能满足用户的请求时，需要用多台服务器集群可以使用nginx做反向代理。并且多台服务器可以平均分担负载，不会因为某台服务器负载高宕机而某台服务器闲置的情况。

> 正向代理（客户端）：客户端通过代理服务器 访问服务器（服务器只知道是代理服务器请求 如vpn访问google）
> 反向代理（服务器端）：服务器端的代理服务器接收到客户请求 指向真实服务器（客户只知道访问的是代理服务器 ）

优点：
占内存小，可以实现高并发连接、处理响应快。
可以实现http服务器、虚拟主机、反向代理、负载均衡。
nginx配置简单
可以不暴露真实服务器IP地址


## 1.安装

在Centos下，yum源不提供nginx的安装，可以通过切换yum源的方法获取安装。也可以通过直接下载安装包的方法，以下命令均需root权限执行：

### 1.yum源方式

#### 1.必要的库
首先安装必要的库（nginx 中gzip模块需要 zlib 库，rewrite模块需要 pcre 库，ssl 功能需要openssl库）

yum list installed | grep zlib

yum list installed | grep pcre

yum list installed | grep openssl

（CentOS最小版都装上了 没有则安装yum -y install zlib pcre openssl）

#### 2.新增yum源

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

#### 3.执行yum安装
yum install nginx

### 2.下载安装包方式

#### 1.安装依赖包

yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel

#### 2.下载并解压安装包

```
//创建一个文件夹
cd /usr/local
mkdir nginx
cd nginx
//下载tar包 在[nginx官网](http://nginx.org/)查找最新版本 如[ nginx-1.17.8](http://nginx.org/download/nginx-1.17.8.tar.gz)
wget http://nginx.org/download/nginx-1.17.8.tar.gz
tar -xvf nginx-1.17.8.tar.gz
//编译安装nginx
//进入nginx目录
cd /usr/local/nginx/nginx-1.17.8
//执行命令
./configure
//执行make编译命令
make
//执行make install命令 安装编译出来的文件
make install
```
./configure可能会提示报错 
1.错误提示：./configure: error: the HTTP rewrite module requires the PCRE library.
解决：yum -y install pcre-devel
2.错误提示：./configure: error: the HTTP cache module requires md5 functions from OpenSSL library. 
解决：yum -y install openssl openssl-devel

## 2.配置和常用命令

命令
```
//服务（yum安装）
//开机自启动
systemctl enable nginx.service
// 启动服务器
nginx
nginx -c /etc/nginx/nginx.conf
// 关闭服务器
nginx -s quit
nginx -s stop
// 检测配置文件
nginx -t
// 重启服务器/重新加载配置文件
./nginx -s reload
```
配置文件
```
//yum安装配置文件在etc中 安装包编译安装配置文件在编译安装上级目录的conf中
vim /etc/nginx/conf.d/default.conf

#全局块开始（主要影响Nginx全局）

# 指定可以运行Nginx服务器的用户或组 默认由nobody账户运行
#user  nobody;
# 要开启的最大进程数 实现并发处理服务的关键 
# 每个Nginx进程平均耗费10M~12M内存。建议指定和CPU的数量一致即可。（或设为auto）
worker_processes  1;

# 错误日志的存放路径 日志输出级别有debug（详细），info，notice，warn，error，erit（最少）可供选择
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

# 指定进程pid的存储文件位置和名称 默认logs/nginx.pid
#pid        logs/nginx.pid;

#全局块结束

#events块开始
# 设定Nginx的工作模式（use）及连接数上限
events {
	# 定义Nginx每个进程的最大连接数，默认是1024
    worker_connections  1024;
}

#events块结束

#http块开始

http {
	# 主要用于将其他的Nginx配置或第三方模块的配置引用到当前的主配文件中，减少主配置文件的复杂度
    include       mime.types;
    # 当文件类型未定义时使用这种方式 默认类型为二进制流
    default_type  application/octet-stream;

	#自定义服务日志
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

	# 允许开启高效文件方式传输模式
    sendfile        on;
    # 防止网络阻塞 tcp_nopush和tcp_nodelay都为on
    #tcp_nopush     on;

	# 连接超时时间（秒） 客户端连接保持活动的超时时间。在超过这个时间之后，服务器会关闭该连接
    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
	
	# 继承全局块
    server {
    	# 指定虚拟主机的服务端口
        listen       80;
        # 用来指定IP地址或域名，多个域名之间用空格分开
        server_name  localhost;

		# 网页的默认编码格式
        #charset koi8-r;

		# 访问日志存放路径
        #access_log  logs/host.access.log  main;
        

		# 继承server块
		# URL地址匹配支持正则表达式匹配，也支持条件判断匹配
        location / {
        	# 指定虚拟主机的网页根目录（相对路径或绝对路径）
            root   html;
            # 默认首页地址
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        # 50x错误时重定向到错误页面
        error_page   500 502 503 504  /50x.html;
        # = 精确匹配
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        # php脚本文件访问127.0.0.1
        # ~正则表达式模式匹配，区分大小写
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        # 
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    # 
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    # 主机名 名称 别名 另一名称.别名
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

#http块结束
```

## 3.反向代理

启动两个 Tomcat

tomcat7  8005 8009 8080
tomcat9  8006 8010 8081

修改配置文件

```
#定义两个虚拟主机 监听80端口 指向8080和8081
server {
	listen       80;
	server_name  *.test1.com;
	location / {
		proxy_pass  http://127.0.0.1:8080;
		index  index.html index.htm;
	}
}
server {
	listen       80;
	server_name  *.test2.com;
	location / {
		proxy_pass  http://127.0.0.1:8081;
		index  index.html index.htm;
	}
}
```

修改客户机hosts文件
反向代理服务器IP a.test1.com
反向代理服务器IP a.test2.com

./nginx -s reload

## 4.负载均衡

启动两个 Tomcat

tomcat7  8005 8009 8080
tomcat9  8006 8010 8081

修改配置文件

```
#定义一个虚拟主机 监听80端口 访问由upstream定义的地址列表
server {
	listen       80;
	server_name  127.0.0.1;
	location / {
		# backserver是由upstream定义的访问列表
		proxy_pass  http://backserver;
		index  index.html index.htm;
	}
}
```
./nginx -s reload

upstream模块支持6种方式的负载均衡策略（算法）：
	轮询（默认方式）
	weight（权重方式）
	ip_hash（依据ip分配方式）
	least_conn（最少连接方式）
	fair（第三方提供的响应时间方式）
	url_hash（第三方通过的依据URL分配方式）

### 1.轮询（默认）

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。 
upstream backserver { 
	server 192.168.1.112:8080; 
	server 192.168.1.112:8081; 
} 

### 2.指定权重

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。 （weight默认为1）
upstream backserver { 
	server 192.168.1.112:8080 weight=10; 
	server 192.168.1.112:8081 weight=10; 
}

### 3.ip_hash解决session丢失

每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。 当有服务器需要剔除，必须手动down掉。
upstream backserver { 
ip_hash; 
	server 192.168.1.112:8080; 
	server 192.168.1.112:8081; 
} 