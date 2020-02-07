---
layout: post
title: 网站添加暗色模式的几种方式
categories:
 - 前端
 - 建站
tags:
 - Javascript
---

给网站添加暗色模式方式多种多样，下面提供一些思路方法，以供参考。

### DarkMode.js  -  [官网文档](https://darkmodejs.learn.uno/)

**方式方法**：最简单的方式，在body里添加一个script标签导入一个js文件即可 

**原理**：添加一个深色div层，覆盖整个页面，相当于给整个页面加了一个反色效果。

<!-- more -->

**优缺点**：
优点：

- 简单易上手，部署时间短；
- 可以部分定制化，还有酷炫动画效果；
- 不用更改原来样式代码。

缺点：

- 定制化程度不高，整个背景色都一样了，不能根据自己想法定制；
- 照片可能会显示异常； 
- 复杂点的页面布局可能无效。



### 动态增删CSS文件📄

**方式方法**：

- 根据前端的开关对暗色模式的css文件进行增删，起到样式覆盖的效果
- 利用客户端cookie保存主题设置，实现跨页面保存主题状态

**关键代码（JS）**：

```javascript
function getCookie(name) {           
    var arr = document.cookie.split(';');           
    for (var i = 0; i < arr.length; i++) {
        var arr2 = arr[i].split('=');
        var arrTest = arr2[0].trim();
        if (arrTest == name) {         
            return arr2[1];
        }
    }

}
function setCookie(name, value) {
    var date = new Date();
    var expires = 10;
    date.setTime(date.getTime() + expires * 24 * 60 * 60 * 1000)
    document.cookie = name + "=" + value + ";expires=" + date.toGMTString() + ";path=" + "/";
}	
function adddarkcss(){
		var creatHead = $('head');
		creatHead.append('<link rel="stylesheet" href="css样式表地址">')
	}
function removedarkcss(){
    var allsuspects=document.getElementsByTagName('link');
    for (var i=allsuspects.length; i>=0;i--){
        if (allsuspects[i] &&allsuspects[i].getAttribute('href')!=null && allsuspects[i].getAttribute('href').indexOf('css样式表地址')!=-1)
            allsuspects[i].parentNode.removeChild(allsuspects[i]);
    }
}
function changemode(){
    var status=getCookie("status");
    if (status=="dark"){
        removedarkcss();
        setCookie("status", "light");
    }

    else{
        adddarkcss();
        setCookie("status", "dark");
    }				
}
```

**注意：**  js文件的执行顺序是从上往下的，因此最好将以上js代码放在head标签内





### 动态增删标签的选择器

方式方法：使用暗色模式的css选择器（例如.dark）的样式配置暗色主题，然后根据前端的开关对所有标签的选择器进行增删（例如动态增删元素的类选择器".dark"），与第二种方式大同小异，不赘述。



### 总结

- 如果网站页面结构比较简单，可以直接引用darkmode.js的文件即可，方便快捷
- 网站页面结构复杂点或者需要高定制的还是需要另外写一个css样式表的
- 覆盖的样式表记得加上`!important`，否则有些样式无法改变，当然这也是样式覆盖的缺点之一