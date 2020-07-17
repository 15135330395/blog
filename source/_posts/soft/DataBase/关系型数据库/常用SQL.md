mysql oracle 数字格式化

https://blog.csdn.net/pan_junbiao/article/details/86519389
https://blog.csdn.net/u013456370/article/details/74923338

Oracle 计算两个日期间隔的天数、月数和年数
https://blog.csdn.net/u013991521/article/details/79293846

oracle
截取日期、时间 https://blog.csdn.net/haiross/article/details/12837033
trunc(sysdate,'y')
统计周期（本年1月1日 到 本年12月31日）
select trunc(sysdate,'y'),last_day(add_months(trunc(SYSDATE,'y'),11))   from dual
增加、减少月份 https://blog.csdn.net/baidu_36695217/article/details/79798531
add_months(日期,正负数)
