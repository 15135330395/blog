---
title: mysql测试
date: 2020-02-07 13:29:46
tags: [mysql测试]
categories: mysql
---
# 1.mysql单表数据量上限测试
## 1.服务器环境
win10系统
内存8G
mysql版本5.5.62

查询方式   navicat连接查询

## 2.准备数据
1.创建两张表，一张为内存表，一张为正式表,内存表主要放存储过程生成的随机数据，正式表再用查询插入从内存表中获取数据。

> mysql的utf8(最大3字节)不是真正的UTF-8 utf8mb4(最大4字节)才是 比如utf8不支持emoji符号

```mysql
#数据表
DROP TABLE IF EXISTS `max_data_test`;
CREATE TABLE `max_data_test` (
	`id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
	`column1` varchar(20) NOT NULL COMMENT '列1',
	`column2` varchar(20) NULL COMMENT '列2',
    `column3` varchar(20) NULL COMMENT '列3',
    `column4` varchar(20) NULL COMMENT '列4',
    `column5` varchar(20) NULL COMMENT '列5',
    `column6` varchar(20) NULL COMMENT '列6',
	`column7` varchar(20) NULL COMMENT '列7',
	`column8` int(11) NULL COMMENT '列8',
	`column9` int(11) NULL COMMENT '列9',
	`column10` datetime NULL COMMENT '列10',
	PRIMARY KEY (`id`),
	KEY `index_column1` (`column1`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8;
#内存表
DROP TABLE IF EXISTS `temp_table`;
CREATE TABLE `temp_table` (
	`id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
	`column1` varchar(20) NOT NULL COMMENT '列1',
	`column2` varchar(20) NULL COMMENT '列2',
    `column3` varchar(20) NULL COMMENT '列3',
    `column4` varchar(20) NULL COMMENT '列4',
    `column5` varchar(20) NULL COMMENT '列5',
    `column6` varchar(20) NULL COMMENT '列6',
	`column7` varchar(20) NULL COMMENT '列7',
	`column8` int(11) NULL COMMENT '列8',
	`column9` int(11) NULL COMMENT '列9',
	`column10` datetime NULL COMMENT '列10',
	PRIMARY KEY (`id`),
	KEY `index_column1` (`column1`)
) ENGINE = MEMORY DEFAULT CHARSET = utf8;
```
2.存储过程：

```mysql
CREATE DEFINER=`root`@`localhost` PROCEDURE `add_many_memory`(IN amount int)
BEGIN
    DECLARE i INT DEFAULT 1;
	WHILE (i <= amount) DO
        INSERT INTO temp_table (column1, column2, column3, column4, column5, column6, column7, column8, column9, column10)
        VALUES ('测试1', '测试2', '测试3', '测试4', '测试5', '测试6', '测试7', FLOOR(RAND() * 1000), FLOOR(RAND() * 100), NOW());
        SET i = i + 1;
    END WHILE;
END
```
3.调用存储过程：（一百万）

```mysql
call add_many_data(1000000)
```

问题：直接调用可能会出现 The table 'temp_table' is full
解决：修改临时表大小 安装路径中my.ini 新增或修改 tmp_table_size=1G max_heap_table_size=1G
			重启mysql
			查看当前情况 SHOW VARIABLES WHERE Variable_name LIKE '%tmp_table_size%'
			清空表 truncate temp_table

4.插入数据表中

```mysql
INSERT into max_data_test SELECT * from temp_table
```

## 3.测试

select * from max_data_test

一百万 3.5s左右
二百万 10s左右
四百万(4328100) 30s左右


select id from max_data_test/select column1 from max_data_test

一百万 2s左右
二百万 4s左右
四百万(4328100) 3s左右


select column2 from max_data_test

一百万 2s左右
二百万 4s左右
四百万(4328100) 8s左右

一百万、二百万单列查询时 多次执行 时间会大幅减少