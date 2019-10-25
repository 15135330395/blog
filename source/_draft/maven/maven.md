---
title: maven
date: 2019-09-27 00:01:00
tags: [maven]
categories: maven
---

## 修改maven的配置文件

	E:\maven\apache-maven-3.5.4\conf\settings.xml
	3.jdk版本
	<profiles>
	<profile>
		<id>jdk1.8</id>
		<activation>
			<activeByDefault>true</activeByDefault>
			<jdk>1.8</jdk>
		</activation>
		<properties>
			<maven.compiler.source>1.8</maven.compiler.source>
			<maven.compiler.target>1.8</maven.compiler.target>
			<maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
		</properties>
	</profile>
	<profiles>
	1.本地仓库
	<settings>
		<localRepository>
			E:/maven/repository
		</localRepository>
	</settings>

 
	2.国内私服仓库（阿里云镜像）
	<mirrors>
		<mirror>
			<id>alimaven</id>
			<mirrorOf>central</mirrorOf>
			<name>aliyun maven</name>
			<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
		</mirror>
	</mirrors>	

## 添加maven环境变量
	MAVEN_HOME
	E:\maven\apache-maven-3.5.4
	%MAVEN_HOME%\bin

	cmd下
		mvn -V查看版本号

将maven的配置文件保存在 C:\Users\Administrator\.m2 中
	配置文件默认位置：C:\Users\Administrator\.m2
	本地库默认位置：C:\Users\Administrator\.m2\repository


## 开发工具中修改maven配置
idea中 setting配置 maven安装路径

## 新建maven项目
### pom文件
	GroupID：包名
	ArtifactID：项目名

maven项目中的配置文件pom.xml
	导包时：
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.15</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.56</version>
        </dependency>
    </dependencies>

	


## 聚合项目

Maven的常见打包方式：jar、war、pom

Pom工程一般都是父工程，管理jar包的版本、maven插件的版本、统一的依赖管理。聚合工程。



e3-parent：父工程，打包方式pom，管理jar包的版本号。

​	|           项目中所有工程都应该继承父工程

​	|--e3-common：通用的工具类通用的pojo。打包方式jar

​	|--e3-manager：服务层工程。聚合工程。Pom工程

​		|--e3-manager-dao：打包方式jar

​		|--e3-manager-pojo：打包方式jar

​		|--e3-manager-interface：打包方式jar

​		|--e3-manager-service：打包方式：jar

​		|--e3-manager-web：表现层工程。打包方式war



parent中pom.xml 配置需要导入的jar包

manager中的pom.xml 配置manager内model需要依赖的jar包







## 分布式项目





archetype 原型：

web项目

java项目



idea中的tomcat7插件





![1556005727925](maven/1556005727925.png)