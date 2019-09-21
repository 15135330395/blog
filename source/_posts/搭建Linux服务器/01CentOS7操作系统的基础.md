---
title: 01CentOS7操作系统的基础
date: 2019-09-20 00:00:01
tags: [Linux,服务器,CentOS7]
categories: 搭建Linux服务器
---
# 系统目录简介
	/ 根目录
	bin (binaries)存放二进制可执行文件
	sbin (super user binaries)存放二进制可执行文件，只有root才能访问
	etc (etcetera)存放系统置文件
	usr (unix shared resources)用于存放共享的系统资源home存放用户文件的根目录
	root 超级用户目录
	dev (devices)用于存放设备文件
	lib (library)存放跟文件系统中的程序运行所需要的共享库及内核模块
	mnt (mount)系统管理员安装临时文件系统的安装点
	boot 存放用于系统引|导时使用的各种文件
	tmp (tempor any)用于存放各种临时文件
	var (variable)用于存放运行时需要改变数据的文件

# 磁盘管理
## 1.查看当前文件目录
	pwd			显示工作目录
## 2.列出目录内容
	ls [参数][文件或目录] 缺省为ls -a
	ls -l		显示文件详细信息（简化命令为ll）
	ls -al 		显示文件详细信息（包括隐藏文件）
	ls -a/all   显示文件名字（包括隐藏文件）
	ls -t		更改时间排序
	ls -r		反向排序
## 3.切换目录
	cd [目录]	   缺省为cd ~
	cd 目录名	  进入指定文件夹（不存在则提示）
	cd ~		当前用户目录
	cd /		根目录
	cd -		上次访问目录
	cd ..		上一级目录
## 4.创建目录
	mkdir [参数] 目录名（支持多级）
	mkdir -p 目录名	父目录不存在先生成父目录（parents）
	mkdir -v 目录名	显示执行过程
## 5.删除目录
	rmdir 目录名	删除空目录
# 文件管理
## 文件浏览
### 1.查看文件内容
	cat 文件	显示文件所有内容
### 2.分页查看文件内容
	more 文件	分页显示文件内容
进入后
	回车	向下一行
	空格	向下滚动一屏（ctrl+f）
	b	 返回上一屏（ctrl+b）
	q	 退出
### 3.分页查看文件内容增强版
	less [参数] 文件
	less -m 文件名	百分比显示（相当于more）
	less -n 文件名	显示每行行数
进入后
	空格		前进一页（page down键）
	b		 返回上一屏（page up键）
	d		 前进半页
	u		 后退半页
	回车		前进一行（方向键向下）
	y		 后退一行（方向键向上）
	/字符串   向上搜索
	?字符串   向下搜索
	v		 进入vim编辑器
	左右方向键 水平滚动条
	q		 退出
### 3.查看文件末尾（可用于查看log文件）
	tail 必要参数 [选择参数][文件]
	-行数	显示函数
	-f	 循环读取（ctrl+c取消）
	&	 后台运行
## 文件操作
### 1.复制文件或目录
	cp [参数] 源文件或目录 目标文件或目录
	-r/--recursive	递归处理 指定目录的文件和子目录全部处理（复制目录必须带-r参数）
	-b				覆盖文件时备份旧文件
### 2.移动或更名 文件或目录

```
mv [源文件或目录] [目标文件或目录]
-f/--force	若目标文件或目录与现有文件或目录重复，则直接覆盖
```

### 3.删除 文件或目录

```
rm [-dfirv] [--help] [--version] [文件或目录]
-f/--force		强制删除文件或目录
-r/--recursive	递归处理
```

### 4.查找 文件或目录

```
find [目录] [参数]
-name	指定字符串作为寻找文件或目录的范本文件
```

## 文档编辑
### 1.基本操作

vi或vim命令

```
vim 文件名		  进入一般模式
i/a/o			进入插入模式
esc				从插入模式退到一般模式
:				从一般模式进入命令行/末行模式
:w/:wq/:q/:q!	末行模式命令：保存/保存并退出/退出/强制退出（还可以编辑环境 入寻找字符串 列出行号等）
```

### 2.常用命令

插入

```

```

复制或粘贴

```

```

定位

```

```

删除

```

```

选择

```

```

### 3.管道命令

|	位于回车键上面（shift+\）

```
命令1|命令2		将命令1的输出内容作为命令2的输入内容 一般与grep使用
```

### 4.文本查找命令

```
全局正则表达式输出
grep [option] parrern [file]	搜索/过滤特定字符
-i/--ignore-case	忽略大小写
搭配管道命令使用 如：cat /root/install.log | grep -i control	在cat命令输出结果 过滤control
```

# 系统命令


查看端口情况

netstat -anp


### 1.查看进程

```
ps [参数]
-e/-a		显示所有程序
-f			显示UID、PPIP、C与STIME
```

### 2.杀死进程
```
kill [参数] [程序]	删除执行中的程序或工资
-l<信息编号>	  列出全部的信息名称
-9 pid号			强制终止
```

### 3.查看网络配置
```
ifconfig
```
### 4.测试网络
```
ping IP地址或域名
ctrl+c退出ping
```

# 二进制软件包管理及配置（针对红帽系列）
1.RPM包管理

RPM软件包的一个例子:

	sudo-1.7.2pl-5.el5.i386.rpm

	其中包括软件名(sudo),版本号(1.7.2pl),发行号(5.el5),和硬件版本(i386)

2.YUM包管理

应用yum的好处:

	自动解决软件包依赖关系;

	方便的软件包升级;


wget

## 1.相关命令
1.rpm包管理

```
rpm [参数] [软件]
-a			查询所有套件
-v			显示指令执行过程
-h/--hash	套件安装时列出标记
-q			询问模式
-i/--install 套件名	安装指定套件
-u/--upgrade 套件名	升级指定套件
-e/--erase 套件名		删除指定套件
--nodeps		不验证套件档的相互关联性

挂载光盘:(?)
	mkdir /mnt/cdrom
	mount /dev/cdrom /mnt/cdrom

常用：
rpm -ivh 软件.rpm	--安装
rpm -Uvh 软件	--更新
rpm -e --nodeps 软件	--强制删除
rpm -qa --查看
```

2.yum包管理

```
yum list 软件包名 --软件包查询
yum info 软件包名 --软件包信息
yum install 软件 --安装
yum remove 软件 --卸载
yum check-update 软件 --检测升级
yum update 软件包名 --升级
```
## 2.配置yum源

1.备份配置文件
```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```
2.下载阿里云的Centos-7.repo文件

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.rep
```
```
[serverid]
#serverid是用于区别各个不同的repository（仓库;贮藏室），必须有一个独一无二的名称
name=Some name for this server
#name，是对repository的描述

#$releasever 表示当前系统的发行版本，可以通过rpm -qi centos-release命令查看
#$basearch 系统硬件架构(CPU指令集),使用命令arch得到

baseurl=url://path/to/repository/
#可以跟多个url 但baseurl只能有一个
#url之后可以加上多个选项，如gpgcheck、exclude、failovermethod等
gpgcheck=1
#其中gpgcheck，exclude的含义和[main]部分相同，但只对此服务器起作用(main在/etc/yum.conf中)
failovermethode=priority
#failovermethode 有两个选项roundrobin和priority，意思分别是有多个url可供选择时，yum选择的次序，roundrobin是随机选择，如果连接失 败则使用下一个，依次循环，priority则根据url的次序从第一个开始。如果不指明，默认是roundrobin。
```
阿里巴巴

网易163

清华大学

```
yum list		#显示yum包
yum clean all	#清除yum缓存
yum makecache	#缓存本地yum仓库中的软件包信息
```

## 3.其他方式安装

源代码包安装

```
tar –xzvf 压缩包.tar.gz	(解压解包)
cd 解压后目录
./configure –prefix=/usr/local/指定的安装路径	(配置)
说明:--prefix参数用来指定安装目录,默认是安装在/usr/local/下
	rpm包是在打包的时候作者已经做好了安装的路径,所以不用设置
make				(编译)
说明:把文件生成可用执行和使用的文件
make install		(安装)
说明:如果在安装过程中出现了错误,那么就把安装目录和解压缩后的文件夹删除就可用了.但是如果安装的是MYSQL这样的包,就需要把MYSQL的进程停掉,然后删除安装目录和解压缩后的文件夹就可以了
```

shell脚本安装

```
tar –xzvf 压缩包.tar.gz
cd 解压后目录
cat README
./setup.sh
```

# 压缩和解压

```
tar 命令		压缩或解压
-c		  压缩（create 建立压缩文件的参数指令）
-x		  解压（extract 解开压缩文件的参数指令）
-z		  是否使用gzip压缩
-v		  是否显示压缩过程文件
-f 文件名	使用档名
常用压缩指令：tar zcvf 文件名
常用解压指令：tar zxvf 文件名
```

# 关机和重启
	init 3  图形界面切换命令界面
	init 5  命令界面切换图形界面
	init 6 或 reboot系统重启
	shutdown 关机 -h now（init0）  halt


# 文件权限

```
chmod		变更文件或目录的权限
chmod [参数] [<权限范围><符号><权限代号>] 文件或目录
-r/--recursive		递归处理

权限范围:
u:user，拥有者
g:group，所属群组
o:other，其他用户
a:all，全部的用户

符号
+添加权限
-取消权限

权限代号
r:读取，数字代号"4"
w:写入，数字代号"2"
x:执行文件或进入目录，数字代号"1"
-:不具任何权限，数字代号"0"

eg:
chmod u-rwx 文件或目录（拥有者所有权限）
chmod o-r-- 文件或目录（其他用户只读）
chmod 777 文件或目录（全部用户所有权限）
chmod 755 文件或目录（拥有者所有权限 所属群组不可写入 其他用户不可写入）

linux文件权限格式
<类型><用户><组><其他用户>
-rwxrw-r--
-/rwx/rw-/r--
1:文件类型 d目录 -普通文件 l链接文件
2-4:拥有者权限
5-7:所属组权限
8-10:其他用户权限
```

# 网络配置

## 1.VIM命令配置
输入ip查询命名 ip addr  也可以输入 ifconfig（如果没有ifconfig命令 则需要安装net-tools）查看ip，但此命令会出现3个条目，centos的ip地址是ens33条目中的inet值

```
vim /etc/sysconfig/network-scripts/ifcfg-ens33

DEVICE=eth33								#网卡名称
TYPE=Ethernet								#网卡类型
NM_CONTROLLED=yes							#
BOOTPROTO=dhcp								#dhcp获得id 还有设为静态static 则需要设置以下
#IPADDR=192.168.1.111						#静态ip地址
#GATEWAY=192.168.1.2						#网关
#PREFIX=24									#子网掩码
#DNS1=192.168.1.2
DEFROUTE=yes								#
IPV4_FAILURE_FATAL=yes						#
IPV6INIT=no									#
NAME=eth33									#
PEERDNS=yes									#
PEERROUTES=yes								#
LAST_CONNECT=1528693814						#
ONBOOT=yes									#是否开始启动网卡
```

重启网卡服务

```
service network restart
（CentOS7 网络服务器版 配置静态可能需要重启电脑 不好说）
```

测试确认ping www.baidu.com 确认网络恢复正常
## 2.setup命令配置（CentOS7之前）

```
setup
防火墙配置
键盘配置
网络配置——设备配置——第一块网卡——设置IP地址等信息
系统服务（设置开机自启动服务）
验证配置
上下键选择 tab键切换焦点 回车确定
```

## 3.nmtui命令配置（CentOS7之后  用了这个不能设置静态IP 关了吧）



# 修改主机名

```
vim /etc/sysconfig/network（重启生效）
hostnamectl set-hostname 主机名（立即生效）
```

# 用户切换
	su -       su - root    切换root用户
	su 普通用户名
	exit 退出到原先的用户
# 查看当前用户

当前登录系统的用户信息

	whoami
	或
	who am  i

# CentOS7查看和关闭防火墙

iptables通过控制端口来控制服务，而firewall则是通过控制协议来控制端口 （只能开一个）

ConterOS7.0以上默认使用的是firewall，ConterOS7.0以下默认使用的是iptables

## firewall（很麻烦）

查看防火墙状态

```
firewall-cmd --state（notrunning 未启动 / running 启动）
或
systemctl status firewalld（更详细）
```

停止firewall

```
systemctl stop firewalld.service
```

禁止firewall开机启动

```
systemctl disable firewalld.service
```

重启防火墙

```
firewall-cmd --reload
```

开放端口（修改后需要重启防火墙方可生效）

```
firewall-cmd --zone=public --add-port=8080/tcp --permanent
```

查看开放的端口

```
firewall-cmd --list-ports
```

关闭端口

```
firewall-cmd --zone=public --remove-port=8080/tcp --permanent
```
```
firewall-cmd --state 查看状态（若关闭，则先开启systemctl start firewalld）

firewall-cmd --list-ports 查看已开放的端口

开启8000端口：
firewall-cmd --zone=public(作用域) --add-port=8000/tcp(端口和访问类型) --permanent(永久生效)

firewall-cmd --zone=public --add-port=1521/tcp --permanent

firewall-cmd --zone=public --add-port=3306/tcp --permanent

firewall-cmd --zone=public --add-port=50070/tcp --permanent

firewall-cmd --zone=public --add-port=8088/tcp --permanent

firewall-cmd --zone=public --add-port=19888/tcp --permanent

firewall-cmd --zone=public --add-port=9000/tcp --permanent

firewall-cmd --zone=public --add-port=9001/tcp --permanent

firewall-cmd --reload -重启防火墙

firewall-cmd --list-ports 查看已开放的端口

systemctl stop firewalld.service停止防火墙

systemctl disable firewalld.service禁止防火墙开机启动



关闭端口：firewall-cmd --zone= public --remove-port=8000/tcp --permanent
```

## iptables（比较简单）

安装防火墙

```
yum install iptables-services
```

开启防火墙

```
systemctl start iptables.service
```

关闭防火墙

```
systemctl stop iptables.service
```

查看防火墙状态

```
systemctl status iptables.service
```

设置开机启动

```
systemctl enable iptables.service
```

编辑防火墙文件
```
vim /etc/sysconfig/iptables

:INPUT ACCEPT [0:0]
:FORWARD ACCEPT[0:0]
:OUTPUT ACCEPT[0:0]
-A INPUT -m state--state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -jACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 6379 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8080 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 8081 -j ACCEPT
-A INPUT -j REJECT--reject-with icmp-host-prohibited
-A FORWARD -jREJECT --reject-with icmp-host-prohibited
COMMIT

22		ssh(默认开启)
80		nginx
3306 	mysql
6379 	redis
8080	tomcat
8081	第二个tomcat
保存
```

如果没有该文件

```
控制台使用iptables命令随便写一条防火墙规则
iptables -A OUTPUT -j ACCEPT
iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
或者
service iptables save
```
重启防火墙使配置文件生效

```
systemctl restart iptables.service
```

# 关闭selinux
进入到/etc/selinux/config文件

```
vim /etc/selinux/config
将SELINUX=enforcing改为SELINUX=disabled
```

# 修改hosts文件

vim /etc/hosts

# 配制免密登录的命令（HDFS集群用到）

```
ssh-keygen -t rsa 或 ssh-keygen（客户机上）
三次回车

此时/root/.ssh目录下
id_rsa为私钥（客户机）
id_rsa.pub为公钥（服务器）

ssh-copy-id 192.168.1.111（ssh-copy-id root@主机名）
输入密码
```

# 帮助文档

man 命令名 更完整

help 命令名 简单叙述