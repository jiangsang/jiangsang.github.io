---
layout: post
title: laydate使用问题总结
categories:
 - Layui
tags:
 - laydate
---

在使用laydate的时候，遇到一些问题和框架的bug，在这里做个记录和提高解决方法（如果有的话😄），由于自己不是专业前端，可能提出的方法也不是很好，但有总比没有好。
### 我的环境
- 系统：macOS Catalina10.15.6
- 浏览器：Chrome 87.0.4280.88
- Layui版本：2.3.0

  <!-- more -->

### 使用时遇到的问题  

#### 日期选择框不弹出

日期控件绑定在输入框元素时,默认的触发事件为：focus,会有一些触发BUG,渲染时改为`trigger:'click'`即可.

如果绑定的元素非输入框，则默认事件为：click

#### type=“month”时的问题  
type=“month”已经碰到几个问题了，主要是有的地方跟默认type=“date”的逻辑不一样。
##### 开启范围选择时change事件无效  
```javascript
laydate.render({
  elem: '#test1' ,//指定元素
  type:'month',
  range:'～',
  change:function(value,date,endDate){
    console.log(value)
  }
});
```
经测试，把`type:'month'`注释时，change事件生效，把`range:'～'`注释时change事件生效。  
奇怪的是，当两个都注释时，change不生效，选择日期后会直接自动填充选定的日期并关闭弹窗，但是`type='month'`时选择月份后不会自动填充和关闭，需要点击确定按钮  
**现实需求**
![image-20210127103959820](https://image.jianger.space/uPic/image-20210127103959820.png)

这里有个需求是两条折线数据，横坐标是月份，为了避免出现两个相同月份和月份顺序错乱的问题，我采取了限制跨年的方法。  
但change事件无效就无法限制日期范围了，虽然可以在done事件中根据日期是否符合条件渲染数据，但是一点击确定按钮新的日期就自动填充了，done里再重新赋值也不行。同时也试过自己写点击事件来限制月份不跨年，但是不生效，即使使用了如`$(document).on("click",selector,funcation(){})`这样针对动态生成元素绑定事件的方法

**解决方案**

1. 第一种方法是不使用一个日期选择框来选择日期范围，将范围选择分为两个日期选择，然后根据开始日期动态更改结束日期的选择范围，通过修改min、max值来实现。
    
    ```javascript
    var startTime=laydate.render({
        elem: '#hqsjStart'
        ,trigger:'click'
        ,done:function (value,date) {
            endTime.config.min={
                year:date.year,
                month:date.month-1,//关键
                date:date.date
            };
        }
    });
    var endTime=laydate.render({
        elem: '#hqsjEnd'
        ,trigger:'click'
        ,done:function (value,date) {
            startTime.config.max={
                year:date.year,
                month:date.month-1,//关键
                date:date.date
            };
        }
    });
    ```
    
      
    
2. 这种方法我没有采用，因为这样选择框太多了，在这个需求里得变成四个选择框，并且在操作上连续性没那么强，如果只有一个范围选择的话可以使用。  

3. 第二种方法就是修改laydate.js的源码了，思路如下
    日期切换后就判断是否是同一年，不是的话就禁用确定按钮，并给出提示信息，修改代码如下，大概在源码694行（源码格式化之后）的位置添加  

    ```javascript
    }, T.prototype.setBtnStatus = function(e, t, n) {
        var a, i = this,
            r = i.config,
            o = w(i.footer).find(g),
            d = r.range && "date" !== r.type && "time" !== r.type;
        d && (t = t || i.startDate, n = n || i.endDate, a = i.newDate(t).getTime() > i.newDate(n).getTime(), i.limit(null, t) ||
        i.limit(null, n) ? o.addClass(s) : o[a ? "addClass" : "removeClass"](s), e && a && i.hint("string" == typeof e ? l
            .replace(/日期/g, e) : l))
        //此处为添加的代码,为了避免影响其他页面加了路径判断,只针对特定页面生效
        if (location.pathname == "/smsly/mgt/ticketAnalysis" && r.range && r.type == "month") {
            if (i.startDate.year != i.endDate.year) {
                o.addClass(s)
                i.hint("请选择同一年的月份")
                l="请选择同一年的月份"
            } else {
                o.removeClass(s)
            }
        }
    ```

效果如下：
![image-20210127110856726](https://image.jianger.space/uPic/image-20210127110856726.png)

直接改源码是不太推荐的,我这个是试了多种方法实在没能解决才在源码上改的,后续遇到问题继续更新本篇