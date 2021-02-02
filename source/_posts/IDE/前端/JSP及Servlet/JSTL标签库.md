---
title: JSTL标签库
date: 2019-09-21 00:00:00
tags: [JSTL标签库]
categories: EL和JSTL
---

JSTL标签库
JavaServer Pages Standard Tag Library JSP标准标签库

使用标签库必须导入
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
常用:
    <c:if test="条件 ${a==b}">
    </c:if>
    <c:forEach items="${list集合}" var="元素">
    </c:forEach>
    <c:forEach items="${list集合}" var="元素" varStatus="i"> //计数变量
    </c:forEach>

函数标签 调用函数必须导入函数库
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions"%>
    字符串截取：${fn:substring(info,0,4)}

格式化标签
<%@taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt"%>
    格式化日期:<fmt:formatDate value="${日期}" pattern="yyyy-MM-dd"/>
