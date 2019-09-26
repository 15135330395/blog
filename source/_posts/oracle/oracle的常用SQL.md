---
title: oracle的常用SQL
date: 2019-09-21 00:00:00
tags: [SQL]
categories: oracle
---
## oracle分页
SELECT * FROM (
  SELECT
      rownum rn ,t1.*
  FROM
      OA_QY_KQGL_XJTJB_SUB t1 , ORG_ORGANIZATION t2 
  WHERE
      rownum <= (1+1) * 15 and t1.bmname=t2.orgname and t1.mid=461 order by t2.sortno
)     
WHERE rn > 1 * 15

## oracle查询 数据断层主键
SELECT ROWNUM FROM ALL_OBJECTS WHERE ROWNUM < = (SELECT MAX(t.wfhisid) FROM ERP_WORKFLOW_HIS t)
MINUS
SELECT t.wfhisid FROM ERP_WORKFLOW_HIS t

## oracle查询某个列名在哪些表中
select column_name,table_name from user_tab_columns where lower(column_name)= lower('列名')


## Oracle创建序列缓存
CREATE SEQUENCE --序列名称
INCREMENT BY 步长
START WITH  起始值
MAXVALUE  最大值 /NOMAXvalue  --无最大值
MINVALUE 最小值
CYCLE/NOCYCLE --是否循环 最大值用完 是否从start with开始 否则从MINVALUE开始（NOCYCLE和NOMAXvalue一起用）
CACHE 缓存大小 --会造成跳号
Order/NOORDER; --排不排序 一般不排序   sequence number是timestamp才排序

## Oracle使用序列缓存
CurrVal：返回 sequence的当前值 
NextVal：增加sequence的值，然后返回 增加后sequence值 
SELECT 序列名称.CurrVal FROM 表
insert into 表名(id,name)values(序列名称.Nextval,'sequence 插入测试');
   注：必须先NextVal才能CurrVal  ； 在同一个语句里面使用多个NEXTV，其值不一样（++i）

## 创建触发器做自增主键
create or replace trigger 触发器名称
before insert on 表  --before:执行DML等操作之前触发
for each row  --行级触发器
begin 
    select 序列名.nextval into :new.id from dual;
end;


## 常用SQL
to_char(LOANDATE,'yyyy-MM-dd') LOANDATE,
( SELECT e.dictname FROM eos_dict_entry e WHERE e.dicttypeid = 'IPM_ZHBG_SQDLX' AND e.dictid = a.APPLICATIONTYPE ) APPLICATIONTYPE

SELECT LOWER('IPM_CONTRACT_SALE_KXXX') FROM dual

SELECT '$F{'||UPPER('ms')||'}' FROM dual


## 查询重复数据
	select * from 重复记录字段 in ( select 重复记录字段 form  数据表 group by 重复记录字段 having count(重复记录字段)>1)

## Oracle查询操作日志
SELECT
    t.SQL_TEXT,
    t.FIRST_LOAD_TIME 
FROM
    v$sqlarea t 
WHERE
    t.FIRST_LOAD_TIME LIKE '2019-07-18%' 
    AND lower( t.SQL_TEXT ) LIKE '%prp_car_%' 
    AND lower( t.SQL_TEXT ) NOT LIKE '%select%' 
ORDER BY
    t.FIRST_LOAD_TIME DESC

## oracle闪回
alter table 表名 enable row movement
--恢复表数据
flashback table 表名 to timestamp to_timestamp(删除时间点','yyyy-mm-dd hh24:mi:ss')
--关闭行移动功能
alter table 表名 disable row movement

##oracle查询最近删除的数据
SELECT * FROM 表 AS OF TIMESTAMP sysdate-1/24 where 条件