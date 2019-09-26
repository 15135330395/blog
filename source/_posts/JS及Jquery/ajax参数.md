---
title: ajax参数
date: 2019-09-21 00:00:00
tags: [ajax]
categories: JS及Jquery
---
$.ajax({
		url: "com.fcb.erp.tools.util.runSql.biz.ext",
		type: 'POST',
		data: json,
		cache: false,
		//cache属性是true时：在第一次请求完成之后，如果地址和参数不变化，第二次去请求，
		//                                会默认获取缓存中的数据，不去读取服务器端的最新数据。
		//cache属性是false时：每次读取的是最新的数据。
		//ajax缓存只对GET方式的请求有效，因为浏览器认为POST请求提交的内容必定有变化，所以不走缓存。
		contentType:'text/json',
		async: true/false,//异步还是同步 默认是true
		success: function (data) {
	        		console.info(data);
		}
});