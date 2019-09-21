---
title: 06go-fastDFS及FastDFS、HDFS
date: 2019-09-20 00:00:06
tags: [Linux,服务器,文件服务器]
categories: 搭建Linux服务器
---
# Go-fastDFS

github：https://github.com/sjqzhang/go-fastdfs

搭配nginx使用

1.linux安装

```shell
wget --no-check-certificate  https://github.com/sjqzhang/go-fastdfs/releases/download/v1.3.1/fileserver -O fileserver && chmod +x fileserver && ./fileserver
```

2.运行

```
./fileserver
```

3.配置

vim conf/cfg.json（一个配置文件 全部搞定）





# fastDFS

用c语言编写的一款开源的分布式文件系统，充分考虑了冗余备份、负载均衡、线性扩容等机制，并注重高可用、高性能等指标，适用于小文件存储

1.有两个角色，tracker server（追踪服务器 负载均衡和调度）和storage server（存储服务器 文件存储）
2.所有服务器都是对等的，不存在Master-Slave 关系 （master有可能单点故障问题，而且 client 与 master 之间可能会出现瓶颈）
3.存储服务器采用分组方式，同组内存储服务器上的文件完全相同（RAID 1） 
4.不同组的storage server之间不会相互通信，同组storage server之间会相互连接进行文件同步
5.由storage server主动向tracker server报告状态信息，tracker server之间通常不会相互通信
6.客户端请求Tracker server进行文件上传、下载，通过Tracker server调度最终由Storage server以HTTP方式完成文件上传和下载

7.客户端请求Tracker server采用轮询方式，如果请求的tracker无法提供服务则换另一个tracker

8.storage集群由一个或多个组构成，集群存储总容量为集群中所有组的存储容量之和，单个组的容量是组内容量最小的服务器



### 安装

1.FastDFS依赖libevent库，需要安装：

yum list installed |grep libevent

yum -y install libevent

2.下载libfastcommon源包 解压 shell脚本安装

https://github.com/happyfish100/libfastcommon.git

```
cd /usr/fastdfs/libfastcommon
tar -zxvf libfastcommon-1.0.39.tar.gz
cd libfastcommon-1.0.39
./make.sh
./make.sh install
安装完成后/usr/lib64有libfastcommon.so文件
```

3.安装源包 解压 shell脚本安装

https://github.com/happyfish100/fastdfs

```
cd /usr/fastdfs
tar -zxvf fastdfs-5.11.tar.gz
cd fastdfs-5.11
./make.sh
./make.sh install
```

### 配置

Client.conf    客户端上传配置文件
Storage.conf    文件存储服务器配置文件
Tracker.conf    负责均衡调度服务器配置文件
http.conf        http服务器配置文件

tracker服务器：

```
cd /etc/fdfs
cp tracker.conf.sample tracker.conf
修改tracker.conf
vim tracker.conf
base_path=/home/yuqing/fastdfs
改为：（22行）
base_path=/home/fastdfs

#配置是否生效 false是生效
disabled=false
#绑定IP
bind_addr=
#服务端口
port=22122
#上传文件的选组方式 0轮询 1指定组 2存储负载均衡（选择剩余空间最大的组）
store_lookup=2
#当store_lookup=1且应用层不指定时才生效
store_group=group2
#下载文件的端口
http.server_port=8080 #http端口（fastdfs5.05以后已经没用了）
（内容很多 根据需求改）

在home文件夹中创建fastdfs文件夹
mkdir /home/fastdfs
启动：
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
查看
netstat -unltp|grep fdfs
日志
tail -100f /home/fastdfs/logs/trackerd.log
将tracker的启动添加到服务器的开机启动中:
vim /etc/rc.d/rc.local
添加/usr/bin/fdfs trackerd /etc/fdfs/tracker.conf restart
保存
```

storage服务器：

```
cd /etc/fdfs
cp storage.conf.sample storage.conf
修改storage.conf
vim storage.conf

group_name=group1
port=23001 #服务端口 （同组的storage端口号必须一致）
base_path=/home/yuqing/FastDFS改为：base_path=/home/fastdfs
store_path_count=1 #存储路径个数 需要和store_path个数匹配
store_path0=/home/yuqing/FastDFS改为：store_path0=/home/fastdfs/fdfs_storage
#如果有多个挂载磁盘则定义多个store_path
#store_path1=.....
#store_path2=......
tracker_server=192.168.1.111:22122   #配置tracker服务器:IP
#如果有多个则配置多个tracker
tracker_server=192.168.1.222:22122
http.server_port=8080 #http端口 （fastdfs5.05以后已经没用了）

在home文件夹中创建fastdfs和fdfs_storage文件夹
mkdir -p /home/fastdfs/fdfs_storage
启动
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
查看
netstat -unltp|grep fdfs
日志
tail -100f /opt/fastdfs_storage_info/logs/storage.log
启动成功后，可以通过fdfs monito值看集群的情况，即storage是否已经注册到tracker服务器中
/usr/bin/fdfs_monitor /etc/fdfs/storage.conf
查看192.168.224.29:23001（storage服务器IP）是ACTIVE状态即可
如果启动没有问题，可以通过以下步骤，将storage的启动添加到服务器的开机启动中:
vim /etc/rc.d/rc.local
添加/usr/bin/fdfs_ storage /etc/fdfs/storage.conf restart
保存
```

### 测试

```
FastDFS安装成功可通过/usr/bin/fdfs_test测试上传、下载等操作。

修改/etc/fdfs/client.conf

base_path=/home/fastdfs
tracker_server=192.168.101.3:22122

使用格式：
/usr/bin/fdfs_test 客户端配置文件地址  upload  上传文件

比如将/home下的图片上传到FastDFS中：
/usr/bin/fdfs_test /etc/fdfs/client.conf upload /home/tomcat.png
（需要Nginx服务器提供http访问服务 同时可以解决group中storage服务器的同步延迟问题）
http://192.168.101.3/group1/M00/00/00/wKhlBVVY2M-AM_9DAAAT7-0xdqM485_big.png就是文件的下载路径。
对应storage服务器上的
/home/fastdfs/fdfs_storage/data/00/00/wKhlBVVY2M-AM_9DAAAT7-0xdqM485_big.png文件。

```

### JAVA测试

```
public class FastdfsClientTest {
	
	//客户端配置文件
	public String conf_filename = "F:fastdfsClient\\fdfs_client.conf"; 
    //本地文件，要上传的文件
	public String local_filename = "F:\\upload\\test.xlsx";

	//上传文件
    @Test 
    public void testUpload() { 
    	
    	for(int i=0;i<100;i++){

        try { 
            ClientGlobal.init(conf_filename); 

            TrackerClient tracker = new TrackerClient(); 
            TrackerServer trackerServer = tracker.getConnection(); 
            StorageServer storageServer = null; 

            StorageClient storageClient = new StorageClient(trackerServer, 
                    storageServer); 
            NameValuePair nvp [] = new NameValuePair[]{ 
                    new NameValuePair("item_id", "100010"), 
                    new NameValuePair("width", "80"),
                    new NameValuePair("height", "90")
            }; 
            String fileIds[] = storageClient.upload_file(local_filename, null, 
                    nvp); 

            System.out.println(fileIds.length); 
            System.out.println("组名：" + fileIds[0]); 
            System.out.println("路径: " + fileIds[1]); 

        } catch (FileNotFoundException e) { 
            e.printStackTrace(); 
        } catch (IOException e) { 
            e.printStackTrace(); 
        } catch (Exception e) {
			e.printStackTrace();
		} 
    	}
    }

}

```

### Nginx整合

参考资料：
http://blog.csdn.net/lynnlovemin/article/details/39398043 fastdfs集群的配置教程
http://blog.csdn.net/poechant/article/details/6977407 fastdfs系列教程
http://m.blog.csdn.net/blog/hfty290/42030339 tracker-leader的选举

为fastDFS提供http访问服务 同时可以解决group中storage服务器的同步延迟问题

#### 在tracker上安装nginx

在每个tracker上安装nginx，主要目的是做负载均衡及实现高可用。如果只有一台tracker服务器可以不配置nginx

#### 在Storage上安装nginx

下载FastDFS-nginx-module 解压

cd /usr/local

tar -zxvf FastDFS-nginx-module_v1.16.tar.gz

cd FastDFS-nginx-module/src

修改config文件将/usr/local/路径改为/usr/

vim conf

cp mod_FastDFS.conf /etc/fdfs/

修改mod_FastDFS.conf的内容

vim /etc/fdfs/mod_FastDFS.conf

base_path=/home/FastDFS

tracker_server=192.168.101.3:22122

\#tracker_server=192.168.101.4:22122（多个tracker配置多行）

url_have_group_name=true           #url中包含group名称

store_path0=/home/FastDFS/fdfs_storage   #指定文件存储路径

 

将libfdfsclient.so拷贝至/usr/lib下

cp /usr/lib64/libfdfsclient.so /usr/lib/

创建nginx/client目录

mkdir -p /var/temp/nginx/client



新建一个nginx配置文件nginx-fdfs.conf.

添加server:

 server {

​        listen       80;

​        server_name  192.168.101.3;

​         location /group1/M00/{

​                \#root /home/FastDFS/fdfs_storage/data;

​                ngx_fastdfs_module;

​        }

}

说明：

server_name指定本机ip

location /group1/M00/：group1为nginx 服务FastDFS的分组名称，M00是FastDFS自动生成编号，对应store_path0=/home/FastDFS/fdfs_storage，如果FastDFS定义store_path1，这里就是M01

# HDFS

一个hdfs集群，由一台运行了namenode的服务器，和N台运行了datanode的服务器组成

## 优缺点

### HDFS优点：

- 高容错性
  - 数据自动保存多个副本
  - 副本丢失后，自动恢复
- 适合批处理
  - 移动计算而非数据
  - 数据位置暴露给计算框架
- 适合大数据处理
  - GB 、TB 、甚至PB 级数据
  - 百万规模以上的文件数量
  - 10K+ 节点
- 可构建在廉价机器上
  - 通过多副本提高可靠性
  - 提供了容错和恢复机制

### HDFS缺点：

- 低延迟数据访问
  - 比如毫秒级- 低延迟与高吞吐率
- 小文件存取
  - 占用NameNode 大量内存
  - 寻道时间超过读取时间
- 并发写入、文件随机修改
  - 一个文件只能有一个写者
  - 仅支持append

## 角色

### NameNode（NN）

主要功能：

- 接受客户端的读写服务
- NameNode保存metadate信息包括
  - 文件owership和permissions
  - 文件包含哪些块
  - Block保存在哪个DataNode（由DataNode启动时上报）
- NameNode的metadate信息在启动后会加载到内存
  - metadata存储到磁盘文件名为”fsimage”
  - Block的位置信息不会保存到fsimage
  - edits记录对metadata的操作日志

### SecondaryNameNode（SNN）

HA  SNN 就没有了

- 它不是NN的备份（但可以做备份），它的主要工作是帮助NN合并edits log，减少NN启动时间。

- SNN执行合并时机

  - 根据配置文件设置的时间间隔fs.checkpoint.period 默认3600秒
- 根据配置文件设置edits log大小 fs.checkpoint.size 规定edits文件的最大值默 认是64MB

###  DataNode（DN）
- 存储数据（Block）
- 启动DN线程的时候会向NN汇报block信息
- 通过向NN发送心跳保持与其联系（3秒一次），如果NN 10分钟没有收到DN的心跳，则认为其已经lost，并copy其上的block到其它DN

## Block的副本放置策略

- 第一个副本：放置在上传文件的DN；如果是集群外提交，则随机挑选一台磁盘不太满，CPU不太忙的节点。
- 第二个副本：放置在于第一个副本不同的机架的节点上。
- 第三个副本：与第二个副本相同机架的节点。
- 更多副本：随机节点

## 安装和配置
1. 下载

   上传hadoop安装包到node1

2. 解压

   

3. ssh的免密码登陆

   

4. 修改hadoop-env.sh

   vim hadoop-env.sh

   修改JAVA_HOME

5. 修改core-site.xml

   vim core-site.xml

   #配置默认采用的文件系统 + 

   #（由于存储层和运算层松耦合，要为它们指定使用hadoop原生的分布式文件系统hdfs。value填入的是uri，参数是  分布式集群中主节点的地址 : 指定端口号）
   <property>
   <name>fs.defaultFS</name>
   <value>hdfs://node01:9000/</value>
   </property>

   #配置hadoop的公共目录
   #（指定hadoop进程运行中产生的数据存放的工作目录，NameNode、DataNode等就在本地工作目录下建子目录存放数据。但事实上在生产系统里，NameNode、DataNode等进程都应单独配置目录，而且配置的应该是磁盘挂载点，以方便挂载更多的磁盘扩展容量）
   <property>
   <name>hadoop.tmp.dir</name>
   <value>/home/thousfeet/app/hadoop-3.0.0/data/</value>
   </property>

6. 修改hdfs-site.xml

   vim hdfs-site.xml

   #配置hdfs的副本数
   #（客户端将文件存到hdfs的时候，会存放在多个副本。value一般指定3，但如果搭建的是伪分布式就只有一台机器，所以只写1）

   <property>
   <name>dfs.replication</name>
   <value>1</value>
   </property>

#指定namenode软件存放文件块的本地目录+ 指定datanode软件存放文件块的本地目录

   <configuration>

   <property>

   <name>dfs.namenode.name.dir</name>

   <value>/root/dfs/name</value>

   </property>

​    

   <property>

   <name>dfs.datanode.data.dir</name>

   <value>/root/dfs/data</value>

   </property>

​    

   </configuration>

7. 修改masters文件和slaves文件

   

8. 格式化namenode

   ```
   hadoop namenode -format
   ```

9. Start-hdfs.sh启动

要运行hadoop的命令，需要在linux环境中配置HADOOP_HOME和PATH环境变量

vim /etc/profile

export JAVA_HOME=/usr/java/jdk1.8.
export HADOOP_HOME=/安装目录
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

node1上启动 hadoop-daemon.sh start namenode

在windows上访问http://hdp-01:50070

启动众datanode们（在任意地方）hadoop-daemon.sh start datanode

## JAVA开发

Configuration conf = new Configuration();

FileSystem fs = FileSystem.get(new URI("hdfs://hdp-01:9000"),conf,"root");

上传文件—— fs.copyFromLocalFile(new Path("本地路径"),new Path("hdfs的路径"));

下载文件——fs.copyToLocalFile(new Path("hdfs的路径"),new Path("本地路径"))



参考资料：

调用JAVA API 对 HDFS 进行文件的读取、写入、上传、下载、删除等操作
https://blog.csdn.net/DF_XIAO/article/details/50601727java

实现对hdfs上文件的上传与下载
https://blog.csdn.net/CSDN_Hzx/article/details/87861096