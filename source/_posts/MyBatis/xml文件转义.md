---
title: xml文件转义
date: 2019-09-21 00:00:00
tags: [xml文件转义]
categories: MyBatis
---
xml写 大于＞  小于＜


1.特殊标签
<![CDATA[    ]]>

如：
    select * from table a where a.field <![CDATA[ >= ]]> 20
    <![CDATA[ select * from table a where a.field >= 20 ]]>
    <![CDATA[ select * from table a where a.field  ]]> >= #{field}
    select * from table a where <if 条件> a.field <![CDATA[ >= ]]> #{field} </if>
2.转义字符
	＞：&gt;
	＜：&lt;


　在XML中，需要转义的字符有： 
　　(1)&　　　&amp; 
　　(2)<　　　&lt; 
　　(3)>　　　&gt; 
　　(4)＂　　　&quot; 
　　(5)＇　　　&apos; 
　　但是严格来说，在XML中只有”<”和”&”是非法的，其它三个都是可以合法存在的，但是，把它们都进行转义是一个好的习惯。 
　　不管怎么样，转义前的字符也好，转义后的字符也好，都会被xml解析器解析，为了方便起见，使用<![CDATA[]]>来包含不被xml解析器解析的内容。但要注意的是： 
　　(1) 此部分不能再包含”]]>”； 
　　(2) 不允许嵌套使用； 
　　(3)”]]>”这部分不能包含空格或者换行。 

　　最后，说说<![CDATA[]]>和xml转移字符的关系，它们两个看起来是不是感觉功能重复了？ 
　　是的，它们的功能就是一样的，只是应用场景和需求有些不同： 
　　(1)<![CDATA[]]>不能适用所有情况，转义字符可以； 
　　(2) 对于短字符串<![CDATA[]]>写起来啰嗦，对于长字符串转义字符写起来可读性差； 
　　(3) <![CDATA[]]>表示xml解析器忽略解析，所以更快。