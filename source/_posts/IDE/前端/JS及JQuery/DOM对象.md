---
title: DOM对象
date: 2019-09-21 00:00:00
tags: [DOM对象]
categories: JS及Jquery
---
# 获取DOM对象s

document.getElementById(&quot;ID的值&quot;);

document.getElementsByClassName(&quot;类名&quot;)

document.getElementsByTagName(&quot;标签名&quot;);

document.getElementsByName(&quot;name属性的值&quot;)



document.querySelector(&quot;选择器&quot;);//只能获取第一个

document.querySelectorAll(&quot;选择器&quot;)//获取所有的，返回的数组

# 创建DOM对象

var div = document.createElement(&quot;标签名&quot;);//返回的&lt;div&gt;&lt;/div&gt;

var text = document.createTextNode(&quot;aa&quot;)//创建了一个&quot;aa&quot;字符串

# 操作DOM对象的属性

## 第一种方式

弊端：只能操作W3C提供的标签的属性

1.先获取对象

2.对象名.属性名

## 第二种方式

获取属性值：dom对象.getAttribute(&quot;属性名&quot;)

设置属性值：dom对象.setAttribute(&quot;属性名&quot;,&quot;属性值&quot;);

删除属性值：dom对象.removeAttribute(&quot;属性名&quot;);

## 总结

如果只操作原有属性，用哪种方式都行，但是如果要操作自定义属性，那么必须使用第二种方式

# 删除,添加DOM对象

提示：按钮使用超链接完成

## 超链接当作按钮使用时候的注意事项

href=&quot;&quot;默认会刷新当前页面

href=&quot;#&quot;就会滚动到页面的最上面

所以我们的目标是点击超链接的时候什么都不做

```
<a href="javascript:void(0)" >添加</a>
```



添加: 父元素.appendChild(子元素)

删除 父元素.removeChild(子元素)

# 通过JS操作样式

1.对象名.style.样式名=&quot;值&quot;

2.对象名.className = &quot;类名&quot;;


# 总结
# 0.js组成

ESMAScript(基本语法)+BOM+DOM

# DEBUG

1.F12

2.点击console旁边的Source

3.找到JS源代码

4.添加断点

5.刷新页面(或触发事件)，目的是让JS执行

6.F10一行一行执行,F8进入下一个断点

7.选中要观察的变量，右键 xxxx watch

# 1.通过JS操作标签

## 1.1获取标签对象



```javascript
var dom对象 = document.getElementById("id值");
```



## 1.2操作标签对象的普通属性

```
//获取属性值
console.log(dom对象.属性名);
//设置属性值
dom对象.属性名 = "值";
```



## 1.3操作标签对象的特殊属性innerHTML

```
dom对象.innerHTML = "html代码";
```



# 2.通过JS给标签绑定事件

## 2.1绑定事件的第一种方式

```javascript
<button onclick="console.log('我被点击了');" type="button">孔老</button>
```

通过标签的onclick属性绑定事件,属性值是一段JS代码

## 2.2绑定事件的第一种方式变形

```javascript
		<script>
            //把代码抽取到方法中
			function aa(){
				for(var i=0; i<100; i++){
					console.log(i);
				}
			}
			
		</script>
		<button onclick="aa()" type="button">孔老</button>
```

因为这段代码可能非常长，比如:

<button onclick=&quot;for(var i=0;i<100; i++){console.log(i);}&quot; type=&quot;button&quot;>孔老</button>

如果以属性的形式编写代码，特别不方便阅读，所以把代码抽取到方法中，然后调用方法就好了

<button onclick=&quot;aa()&quot; type=&quot;button&quot;>孔老</button>

## 2.3绑定事件的第二种方式

通过DOM对象来绑定

```html
<button  id="btn" >孔老</button>
<script>
    var btn =  document.getElementById("btn");
    btn.onclick = function(){//必须点击按钮才能触发事件的执行
        console.log(1);
    }
</script>
```

## 2.5灯泡案例

1.页面应该存放img标签，并且设定默认的图片

2.给图片标签绑定单击事件

3.在单击事件中去修改img标签的src属性来实现切换图片

```html
<body>
	<img id="light"  src="img/off.gif">
	
	<script>
		var img = document.getElementById("light");
		
		var flag = false;//false关灯状态
		img.onclick = function(){
			if(!flag){//如果是关灯状态(flag==false)，就改为开灯,flag = true
				img.src = "img/on.gif";	
				flag = true;
			}else{//如果是开灯状态(flag==true)，就改为关灯flag = false;
				img.src = "img/off.gif";
				flag = false;
			}
		}
	</script>
</body>