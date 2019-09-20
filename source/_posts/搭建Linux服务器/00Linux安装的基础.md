---
title: 00Linux安装的基础
date: 2019-09-20 00:00:00
tags: [Linux,服务器]
categories: 搭建Linux服务器
---
## 0.检验文件完整性

CentOS-7-x86_64-DVD-1810.iso

| sha256                                                       | 文件                                |
| :----------------------------------------------------------- | :---------------------------------- |
| 3213b2c34cecbb3bb817030c7f025396b658634c0cf9c4435fc0b52ec9644667 | CentOS-7-x86_64-LiveGNOME-1810.iso  |
| 38d5d51d9d100fd73df031ffd6bd8b1297ce24660dc8c13a3b8b4534a4bd291c | CentOS-7-x86_64-Minimal-1810.iso    |
| 6d44331cc4f6c506c7bbe9feb8468fad6c51a88ca1393ca6b8b486ea04bec3c1 | CentOS-7-x86_64-DVD-1810.iso        |
| 87623c8ab590ad0866c5f5d86a2d7ed631c61d69f38acc42ce2c8ddec65ecea2 | CentOS-7-x86_64-LiveKDE-1810.iso    |
| 918975cdf947e858c9a0c77d6b90a9a56d9977f3a4496a56437f46f46200cf71 | CentOS-7-x86_64-Everything-1810.iso |
| 19d94274ef856c4dfcacb2e7cfe4be73e442a71dd65cc3fb6e46db826040b56e | CentOS-7-x86_64-NetInstall-1810.iso |

windows的cmd命令：

```
certutil -hashfile .\CentOS-7-x86_64-DVD-1810.iso SHA256
SHA256 的 .\CentOS-7-x86_64-DVD-1810.iso 哈希:
6d44331cc4f6c506c7bbe9feb8468fad6c51a88ca1393ca6b8b486ea04bec3c1
CertUtil: -hashfile 命令成功完成。
```

文本比对：https://tool.lu/diff/

## 1.选择语言

中文——简体中文(中国)

## 2.安装信息摘要

网络和主机名——打开以太网、修改主机名

时间和日期——亚洲、上海、修改为当前系统时间并打开网络时间（一般减八个小时）

安装源——自动检测的安装介质

软件选择——基础网页服务器——直接点完成

（开发学习选择开发及生成工作站，网页服务器选择基础网页服务器，普通用选择gnome界面版本即可，专家水平选择最小安装 特别干净 常用软件都没有）

安装位置——本地标准磁盘

## 3.配置root超级用户密码

 ## 4.等待安装完成

