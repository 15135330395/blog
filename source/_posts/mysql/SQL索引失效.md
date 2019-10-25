---
title: SQL索引失效
date: 2019-09-21 00:00:00
tags: [SQL,索引]
categories: mysql
---
1.like查询以%开头（like '%abc' 或者like '%abc%'，'abc%'不会失效）  
2.条件查询有 or
3.
<!-- in 放弃子表的索引 -->
<!-- exists 放弃主表的索引 -->
<!-- 用not exists比not in要快 -->

4.查询is null条件
使用 <> 、not in 、not exist、!=