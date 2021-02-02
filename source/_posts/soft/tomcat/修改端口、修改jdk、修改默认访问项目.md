---
title: 修改端口、修改jdk、修改默认访问项目
date: 2019-09-21 00:00:00
tags: [tomcat]
categories: tomcat
---

## 修改端口

## 修改jdk

## 修改默认访问项目
<Host appBase="webapps" autoDeploy="true" name="localhost" unpackWARs="true">
    <!-- SingleSignOn valve, share authentication between web applications
         Documentation at: /docs/config/valve.html -->
    <!--
    <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
    -->
    <!-- Access log processes all example.
         Documentation at: /docs/config/valve.html
         Note: The pattern used is equivalent to using pattern="common" -->
    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" pattern="%h %l %u %t &quot;%r&quot; %s %b" prefix="localhost_access_log." suffix=".txt"/>
</Host>
中间加一行
<Context path="" docBase="default"/>