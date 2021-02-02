[TOC]

# 库级别

## oracle查询操作日志
```
SELECT
    t.SQL_TEXT,
    t.FIRST_LOAD_TIME
FROM
    v$sqlarea t
WHERE
    t.FIRST_LOAD_TIME LIKE '2020-01-01%'
    AND lower( t.SQL_TEXT ) LIKE 'select%'
ORDER BY
    t.FIRST_LOAD_TIME DESC
```
## oracle闪回（没试过）

alter table 表名 enable row movement
--恢复表数据
flashback table 表名 to timestamp to_timestamp(删除时间点','yyyy-mm-dd hh24:mi:ss')
--关闭行移动功能
alter table 表名 disable row movement

## oracle查询最近删除的数据（试过 可以还原三个小时内的数据 删除表会失效）
```
SELECT * FROM 表 AS OF TIMESTAMP sysdate-1/24 where 条件
select * from 表 as of timestamp to_timestamp('2020-01-01 10:00:00','YYYY-MM-DD HH24:MI:SS') where id=多少;
```

## oracle延长用户过期时间
```
SELECT username,PROFILE FROM dba_users;

SELECT * FROM dba_profiles WHERE profile='DEFAULT' AND resource_name='PASSWORD_LIFE_TIME';

ALTER profile DEFAULT limit PASSWORD_LIFE_TIME 360;（永不过期：ALTER profile DEFAULT limit PASSWORD_LIFE_TIME UNLIMITED;）

SELECT * FROM dba_users;
```

## oracle提示列说明无效
查看关键字
select * from v$reserved_words;

## oracle锁和解锁
[oracle锁和解锁](https://www.cnblogs.com/Dev0ps/p/9089947.html)

# 表级别

## 查询表结构
### mysql

### oracle
```
SELECT
	lower( temp.column_name ) AS column_name,
	lower( temp.data_type ) AS column_type,
	( CASE WHEN ( temp.nullable = 'N' AND temp.constraint_type != 'P' ) THEN 'true' ELSE NULL END ) AS is_required,
	( CASE WHEN temp.constraint_type = 'P' THEN 'true' ELSE 'false' END ) AS is_pk,
	temp.column_id AS sort,
	temp.comments AS column_comment 
FROM
	(
	SELECT
		col.column_id,
		col.column_name,
		col.nullable,
		col.data_type,
		colc.comments,
		uc.constraint_type,
		row_number () over ( partition BY col.column_name ORDER BY uc.constraint_type DESC ) AS row_flg 
	FROM
		user_tab_columns col
		LEFT JOIN user_col_comments colc ON colc.table_name = col.table_name AND colc.column_name = col.column_name
		LEFT JOIN user_cons_columns ucc ON ucc.table_name = col.table_name AND ucc.column_name = col.column_name
		LEFT JOIN user_constraints uc ON uc.constraint_name = ucc.constraint_name 
	WHERE
		col.table_name = upper( 'CAP_USER' ) 
	) temp 
WHERE
	temp.row_flg = 1 
ORDER BY
	temp.column_id
```

## 查询某个列名在哪些表中
### mysql
SELECT * FROM information_schema.columns WHERE column_name='列名';
### oracle
select column_name,table_name from user_tab_columns where lower(column_name)= lower('列名')

## 数据库系统表
### mysql
information_schema
### oracle
DBA_OBJECT
ALL_TABLES
USER_TABLES

## oracle创建、使用序列缓存
CREATE SEQUENCE --序列名称
INCREMENT BY 步长
START WITH  起始值
MAXVALUE  最大值 //NOMAXvalue 无最大值
MINVALUE 最小值
CYCLE/NOCYCLE --是否循环 最大值用完 是否从start with开始 否则从MINVALUE开始（NOCYCLE和NOMAXvalue一起用）
CACHE 缓存大小 --会造成跳号
Order/NOORDER; --排不排序 一般不排序   sequence number是timestamp才排序

CurrVal：返回 sequence的当前值 
NextVal：增加sequence的值，然后返回 增加后sequence值 
SELECT 序列名称.CurrVal FROM 表
insert into 表名(id,name)values(序列名称.Nextval,'sequence 插入测试');
   注：必须先NextVal才能CurrVal  ； 在同一个语句里面使用多个NEXTV，其值不一样（++i）

## oracle创建触发器做自增主键
create or replace trigger 触发器名称
before insert on 表  --before:执行DML等操作之前触发
for each row  --行级触发器
begin 
    select 序列名.nextval into :new.id from dual;
end;

# 数据级别

## 格式化
### 数字
#### mysql 
[MySQL数字的取整、四舍五入、保留n位小数](https://blog.csdn.net/pan_junbiao/article/details/86519389)
#### oracle
[Oracle中，利用sql语句中的函数实现保留两位小数和四舍五入保留两位小数](https://blog.csdn.net/u013456370/article/details/74923338)
### 字符串
#### mysql

#### oracle


### 日期、时间
#### 计算两个日期间隔的天数、月数和年数
##### mysql
[MySQL计算两个日期相差的天数、月数、年数](https://blog.csdn.net/qq_25580555/article/details/107907879)

[MYSQL 获取当前时间加上一个月](https://blog.csdn.net/linybo/article/details/36185357)
now() 当前时间
date_add() 增加
date_sub() 减少

##### oracle 
[Oracle 计算两个日期间隔的天数、月数和年数](https://blog.csdn.net/u013991521/article/details/79293846)

[TRUNC函数的用法](https://blog.csdn.net/haiross/article/details/12837033)
select sysdate from dual
select to_char(sysdate,'yyyy-mm-dd hh:mi:ss') from dual 
select trunc(sysdate,'yyyy') from dual --当年的第一天
select trunc(sysdate,'mm') from dual --当月的第一天
select trunc(sysdate,'dd') from dual --当前时间(精确到天)
select trunc(sysdate,'d') from dual --当前星期的第一天
select trunc(sysdate,'hh') from dual --当前时间（精确到小时）
select trunc(sysdate,'mi') from dual --当前时间（精确到分钟，没有精确到秒的）

trunc(number)的用法一般有以下几种：
trunc(55.5,-1) = 50; //-1(负数)表示从小数点左边第一位截取后面全置为零;
trunc(55.55,1) = 55.5; //1(正数)表示小数点后面保留一位;
trunc(55.55) = 55; //截取整数部分;

trunc(sysdate,'y')
统计周期（本年1月1日 到 本年12月31日）
select trunc(sysdate,'y'),last_day(add_months(trunc(SYSDATE,'y'),11)) from dual

[oracle中的add_months()函数总结](https://blog.csdn.net/baidu_36695217/article/details/79798531)
add_months(日期,正负数)

## 分页
### mysql
```
SELECT
	t.* 
FROM
	erp_workflow_his t
ORDER BY t.wfhisid DESC
LIMIT 第几条开始,每页条数
```
### oracle
```
SELECT
	* 
FROM
	(
		SELECT
			ROWNUM RN,
			t1.*
		FROM ERP_WORKFLOW_HIS t1
		WHERE
			ROWNUM <= ( 1+第几页 ) * 每页条数
		ORDER BY t1.WFHISID
	) 
WHERE
	RN > 第几页 * 每页条数
```

## 查询数据主键断层
### mysql

### oracle
```
SELECT
	ROWNUM
FROM
	ALL_OBJECTS
WHERE
	ROWNUM < = ( SELECT MAX( t.wfhisid ) FROM ERP_WORKFLOW_HIS t ) 
MINUS
SELECT
	t.wfhisid 
FROM
	ERP_WORKFLOW_HIS t
```

## like查询百分号、下划线
like '%\_%' escape '\'（\也可以换任意字符）
下划线“_”不是单纯的表示下划线的意思，而是表示匹配单一任何字符！

也可以使用instr
instr(列名,'_')!=0

## mysql 批量修改成连续ID
SELECT @t:=0;
update t_city set SKILLID=(@t:=@t+1) order by code;

## mysql 查询自然序号
SELECT
	LPAD( ( @i := @i + 1 ), 4, '0' ) num,
	a.user_id 
FROM
	sys_user a,
	( SELECT @i := 0 ) t2 
ORDER BY a.user_id ASC

## 汉字排序
### mysql
SELECT * FROM 表名 ORDER BY CONVERT (字段名 USING gbk ) [DESC]
### oracle
SELECT * FROM 表名 ORDER BY NLSSORT( 字段名, 'NLS_SORT = SCHINESE_PINYIN_M' ) [DESC]

## 查询重复数据
SELECT * FROM 数据表 WHERE 重复记录字段 IN ( SELECT 重复记录字段 FROM 数据表 GROUP BY 重复记录字段 HAVING count( 重复记录字段 ) > 1 )

## oracle VARCHAR2和NVARCHAR2的区别
区别一：
VARCHAR2（size type），size最大为4000，type可以是char也可以是byte，不标明type时默认是byte（如：name  VARCHAR2(60)）。
NVARCHAR2（size），size最大值为2000，单位是字符
区别二：
VARCHAR2最多存放4000字节的数据，最多可以可以存入4000个字母，或最多存入2000个汉字（数据库字符集编码是GBK时，varchar2最多能存放2000个汉字，数据库字符集编码是UTF-8时，那就最多只能存放1333个汉字，呵呵，以为最大2000个汉字的傻了吧！）

NVARCHAR2（size），size最大值为2000，单位是字符，而且不管是汉字还是字母，每个字符的长度都是2个字节。所以nvarchar2类型的数据最多能存放2000个汉字，也最多只能存放2000个字母。并且NVARCHAR2不受数据库字符集的影响。


## 整理eos角色用户菜单
with tab1 as (
    SELECT
        a.ROLE_ID 角色id,
        a.ROLE_NAME 角色名,
        WM_CONCAT(c.empname) 当前用户
    FROM
        CAP_ROLE a
        LEFT JOIN cap_partyauth b ON a.ROLE_ID = b.ROLE_ID ,
        ORG_EMPLOYEE c
    WHERE
        b.PARTY_ID = to_char(c.empid)
        group by a.ROLE_ID,
        a.ROLE_NAME
    ORDER BY
        a.ROLE_ID
),
tab2 as(
    select PARTY_ID,wmsys.wm_concat(funcname) 菜单名
      from (select a.PARTY_ID,
                   b.funcname,
                   sum(lengthb(b.funcname||',')) over(partition by PARTY_ID ) descr_length
              from CAP_RESAUTH a
        LEFT JOIN APP_FUNCTION b ON a.RES_ID = b.funccode 
    )
    where descr_length < 4000
    group by PARTY_ID
)
select tab1.角色ID,tab1.角色名,tab1.当前用户,tab2.菜单名 from tab1 LEFT JOIN tab2 on tab1.角色id=tab2.PARTY_ID

## Oracle数据行拆分多行
[Oracle数据行拆分多行方法示例](https://www.jb51.net/article/125507.htm)
单行拆分

如果表数据只有一行，则可以直接在原表上直接使用connect by+正则的方法,比如：
select regexp_substr('444.555.666', '[^.]+', 1, level) col
from dual
connect by level <= regexp_count('444.555.666', '\.') + 1 

多行拆分
如果数据表存在多行数据需要拆分，也可以在原表上使用connect+正则的方法：
### 方法一

with t as
(select '111.222.333' col
from dual
union all
select '444.555.666' col
from dual)
select regexp_substr(col, '[^.]+', 1, level)
from t
connect by level <= regexp_count(col, '\.\') + 1
and col = prior col
and prior dbms_random.value > 0

### 方法二（测试失败）
使用构造的最大行数值关联原表：
with t as
(select '111.222.333' col
from dual
union all
select '444.555.666' col
from dual)
select regexp_substr(col, '[^.]+', 1, lv)
from t, (select level lv from dual connect by level < 10) b
where b.lv <= regexp_count(t.col, '\.\') + 1 
这种方法设置第二个数据集的时候要小于可能的最大值，然后两数据集做关联，在做大数据量拆分的时候，这个数值设置得当，拆分行数相对一致的情况下，效率比方法一直接connect by要高。

### 方法三（测试通过）
使用table函数：

with t as
(select '111.222.333' col
from dual
union all
select '444.555.666' col
from dual)
select column_value
from t,
table(cast(multiset
(select regexp_substr(col, '[^.]+', 1, level) dd
from dual
connect by level <= regexp_count(t.col, '\.\') + 1) as
sys.odcivarchar2list)) a 

这个方法输出的列名是固定的，column_value依赖于sys.odcivarchar2list这个类型的输出，该方法对于大数据量的拆分效率比第二个方法好。

### 方法四（未测试）
with t as
(select '111.222.333' col
from dual
union all
select '444.555.666' col
from dual)
select regexp_substr(col, '[^.]+', 1, trim(column_value))
from t,
xmltable(concat('1 to ',regexp_count(t.col, '\.\') + 1)) a ;
注意：大数据量的拆分时，谨慎使用正则的方法去做，可以使用substr+instr的方式替换正则。
如果以上方法的效率仍然不理想，可考虑使用plsql块。