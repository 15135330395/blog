---
title: IE控制台报错addEventListener
date: 2019-09-21 00:00:00
tags: [IE控制台]
categories: JS及Jquery
---

IE如果控制台报错addEventListener 说明IE版本过低（9以下 如果是IE11则关闭兼容模式  根本原因是jquery1.x）
兼容IE9以下的addEventListener事件绑定方法（所有调用addEventListener的方法调用自定义Event.addEvents方法）：
```
var Event = {};
Event.addEvents = function(target,eventType,handle){
    if(document.addEventListener){
        Event.addEvents = function(target,eventType,handle){
            target.addEventListener(eventType,handle,false);
        };
    }else{
        Event.addEvents = function(target,eventType,handle){
            target.attachEvent('on'+eventType,function(){
                handle.call(target,arguments);
            });
        };
    };
    Event.addEvents(target,eventType,handle);
};
```