---
title: EL表达式
date: 2019-09-21 00:00:00
tags: [EL表达式]
categories: EL和JSTL
---

EL表达式
Expression Language 表达式语言
四种标志位:page(pageContext)、request, session, application
${作用域.属性名称}
    $(pageScope.变量名}
    ${requestScope.变是名}
    ${sessionScope.变星名}
    ${applicationScope.变且名}

${变量.属性}

$(3/2)结果为1.5

三目运算符
3${3>2?"大于":"小于"}2  ————> 3大于2
