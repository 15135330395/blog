---
title: 完整的url地址
date: 2019-09-21 00:00:00
tags: [url]
categories: tomcat
---

在编写接口代码时，尽量使用request中获取的完整地址
确保拓展性 如https协议 域名 修改端口
并注释上是请求哪的服务器
如果是测试数据，记得注明，测试完注释

### 项目名（webapp文件夹名）

    String path = request.getContextPath();
### URL地址

    String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
    <a href="<%=basePath%>" />