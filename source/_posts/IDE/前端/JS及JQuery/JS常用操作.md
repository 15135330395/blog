---
title: 简单操作
date: 2019-09-21 00:00:00
tags: [简单操作]
categories: JS及Jquery
---
### js去掉换行
str=str.replace(/^\n+|\n+$/g,&quot;&quot;);

### 正则
##### 必填或不为空
不转义(查询数据库 直接对比 如excel导入模板中设置的正则格式)：
\S+
转义(代码中对比)：
/\S+/

##### 数字
不转义：
    正整数：^\d+$
    三位小数的浮点型正数：^\d+(\.\d{1,3})?$
    三位小数的浮点型数字：^(\-|\+)?\d+(\.\d{1,3})?$

##### 日期
转义：
yyyy-MM-dd	/^[1-2][0-9][0-9][0-9]-[0-1]{0,1}[0-9]-[0-3]{0,1}[0-9]$/
yyyy/M/dd	/^[1-2][0-9][0-9][0-9]\/[0-1]{0,1}[0-9]\/[0-3]{0,1}[0-9]$/
yyyy/M/d    /^[1-2][0-9][0-9][0-9]\/[0-1]{0,1}[0-9]\/[0-3]{0,1}[0-9]$/
yyyy-M-d HH:mm:ss（不限制年份）：^\d{4}\-\d{1,2}\-\d{1,2}\s(20|21|22|23|[0-1]\d):[0-5]\d:[0-5]\d$

excel导入验证规则会用到 
!!正则验证数据格式   !!复杂验证 验证内容
 （还可能用到复杂的验证  比如导入的数据 有效期时间必须大于导入时间   还可能支持同时多种验证 先验证时间格式 后比较时间）
科学计数法的数据（比较准确）
不转义：^((-?\\d+.\\d*)[Ee]{1}(-?\\d+))$  （0.e0的数据）

### 去掉字符串中所有的空格或单引号
str=str.replace(/\s/g,&quot;&quot;);
b= b.replace(/\'/g, &quot;&quot;);

### 获得年龄
function getAge(str){
    var r = str.match(/^(\d{1,4})(-|\/)(\d{1,2})\2(\d{1,2})$/);
    if(r==null)return false;     
    var d= new Date(r[1],r[3]-1,r[4]);
    if(d.getFullYear()==r[1]&&(d.getMonth()+1)==r[3]&&d.getDate()==r[4]){   
        var date = new Date();
        var Y = date.getFullYear() - r[1];
        var M = date.getMonth() + 1 - r[3];
        var D = date.getDate();
        if(parseInt(D) - parseInt(r[4]) < 0){
            M--;
            if(M<0){
                Y--;
                M = 11;
            }
        }
        if(M<0){
            Y--;
            M = 11;
        }
        return M?(Y+"岁零"+M+"月"):(Y+"岁");
    }   
    return("日期格式错误！");
}

### 获得天数差
function getDays(date1 , date2){
	var date1Str = date1.split("-");//将日期字符串分隔为数组,数组元素分别为年.月.日
	//根据年 . 月 . 日的值创建Date对象
	var date1Obj = new Date(date1Str[0],(date1Str[1]-1),date1Str[2]);
	var date2Str = date2.split("-");
	var date2Obj = new Date(date2Str[0],(date2Str[1]-1),date2Str[2]);
	var t1 = date1Obj.getTime();
	var t2 = date2Obj.getTime();
	var dateTime = 1000*60*60*24; //每一天的毫秒数
	var minusDays = Math.floor(((t2-t1)/dateTime));//计算出两个日期的天数差
	var days = Math.abs(minusDays);//取绝对值
	return days+1;
}

### 字符串 数组相互转换
1. 数组转字符串：
	var a, b;
	a = new Array(0,1,2,3,4);
	b = a.join(&quot;,&quot;);
2. 字符串转数组：
	var s = &quot;abc,abcd,aaa&quot;;
	ss = s.split(&quot;,&quot;);// 在每个逗号处进行分解  [&quot;abc&quot;, &quot;abcd&quot;, &quot;aaa&quot;]

	var s1 = &quot;helloworld&quot;;
	ss1 = s1.split('');  // 每个字符分解 [&quot;h&quot;, &quot;e&quot;, &quot;l&quot;, &quot;l&quot;, &quot;o&quot;, &quot;w&quot;, &quot;o&quot;, &quot;r&quot;, &quot;l&quot;, &quot;d&quot;]

### 四舍五入
官方说法是：
toFixed它是一个四舍六入五成双的诡异的方法(也叫银行家算法)，"四舍六入五成双"含义：对于位数很多的近似数，当有效位数确定后，其后面多余的数字应该舍去，只保留有效数字最末一位，这种修约（舍入）规则是“四舍六入五成双”，也即“4舍6入5凑偶”这里“四”是指≤4 时舍去，"六"是指≥6时进上，"五"指的是根据5后面的数字来定，当5后有数时，舍5入1；当5后无有效数字时，需要分两种情况来讲：①5前为奇数，舍5入1；②5前为偶数，舍5不进。（0是偶数）

仅作四舍五入（toFixed()是四舍六入五成双的银行家舍入法 也可以直接改写toFixed方法(Number.prototype.toFixed = function (s){}) 实现方式不唯一）
function RoundNumber(num, pos)
{
    return Math.round(num * Math.pow(10, pos+1)/Math.pow(10,1)) / Math.pow(10, pos);
}

### JS数据的操作
#### JSON/对象
1.被花括号 {} 包围
2.以键/值对书写
3.键必须是字符串，值必须是有效的 JSON 数据类型（字符串、数字、对象、数组、布尔或 null）
4.键和值由冒号分隔
5.每个键/值对由逗号分隔
【6.对象可以有方法】
如：myObj =  { "name":"Bill Gates", "age":62, "car":null };
访问
    x = myObj.name;
    x = myObj["name"];
    for (x in myObj) {
       document.getElementById("demo").innerHTML += x;
    }
    for (x in myObj) {
       document.getElementById("demo").innerHTML  += myObj[x];
    }
新增/修改
    myObj.cars.car1 = "Mercedes Benz";
    myObj.cars["car2"] = "BMW";
删除
    delete 对象.属性
    delete myObj.cars.car1;
合并
    Object.assign(对象1,对象2,...);
    会合并到对象1 重复的数据会被覆盖
键名称
    var a = {key:123};
    Object.keys(a);//["key"]
    Object.keys(a)[0];//"key"


#### 数组
定义
    var fruits = new Array();
    var fruits = ["Banana", "Orange", "Apple", "Mango"];
在最后新增
    fruits.push("Kiwi");
    fruits[fruits.length] = "Kiwi";
在第一个新增
    var a = fruits.unshift("Lemon");//5
新增指定位置
    var fruits = ["Banana", "Orange", "Apple", "Mango"];
    fruits.splice(2, 0, "Lemon", "Kiwi");
    第一个参数（2）定义了应添加新元素的位置（拼接）。
    第二个参数（0）定义应删除多少元素。
    其余参数（“Lemon”，“Kiwi”）定义要添加的新元素。
    splice() 方法返回一个包含已删除项的数组：
修改
    fruits[1] = "1";
删除(弹出)最后一个
    var x = fruits.pop();//Kiwi
删除第一个 并把所有其他元素“位移”到更低的索引
    var y = fruits.shift();//Lemon
删除指定位置
    delete fruits[1];//变成undefined 会在数组留下未定义的空洞
    fruits.splice(0, 1);//删除fruits中的第一个元素 不留空洞
合并数组
    var myGirls = ["Cecilie", "Lone"];
    var myBoys = ["Emil", "Tobias", "Linus"];
    var myChildren = myGirls.concat(myBoys);
    不会更改现有数组 总是返回一个新数组

### 根据表名和编码字段获取最大编码
```javascript
function createOAMaxCode(module,tablename,codefiled){
        var yearAndMonth=nui.formatDate(new Date(),"yyyyMM");
        var maxCode  = "";
        var sqlStr =" select max(substr("+codefiled+",-3)) as maxCode from "+tablename+" where "+codefiled+" like '"+module+"-"+new Date().getFullYear()+"%'";
        console.log(sqlStr);
        var ret = ERP.runSql(sqlStr, "select", false);
        console.log(ret);
        var retNum = parseInt(ret.dataObject[0].MAXCODE); 
        console.log(retNum);
        var re = /^[0-9]+.?[0-9]*/;//判断字符串是否为数字//判断正整数/[1−9]+[0−9]∗]∗/ 
        if(ret.result == 'success' && isEmpty(ret.dataObject[0].MAXCODE)){//如果是新的一年则编码从1计数
            maxCode="001";
        }else if( !re.test(ret.dataObject[0].MAXCODE)){//查询出错
            maxCode="编码生成错误，请手动输入..";
        } else{
            maxCode = PrefixInteger(retNum+1,3);//最大编码数值
        }
        var modeCode = module+"-"+yearAndMonth+"-"+maxCode;
        return modeCode;
}
//补0
function PrefixInteger(num, length) {
    return (Array(length).join('0') + num).slice(-length);
}
//判断字符是否为空
function isEmpty(obj){
    if(typeof obj == "undefined" || obj == null || obj == ""){
        return true;
    }else{
        return false;
    }
}
```
### 页面上监听回车调用方法
可用于 直接回车登陆 以及 直接回车查询
document.onkeydown = function(){
    var event = window.event;
    if (event.keyCode == 13) {
        search();
    }
};
$(document).keydown(function(event){
    if(event.keyCode == 13){
        search();
    }
});

### JS中计算误差
1.限制精度结果四舍五入
2.引入专用的计算JS

### 寻找第三方接口 判断法定节假日、调休日、周末、工作日 方便考勤、请假功能统计日期