---
clayout: post
title: Layui数据表格缓存问题
categories:
 - Layui
tags:
 - Layui
---

Layui数据表格很好使用，减少了像我这样的程序员很多工作，有很多组件拿来就可使用，其中的数据表格使用频率极其高，但是我在使用中还是遇到不少问题，以下将自己使用中遇到的问题和采取的解决方案（如果有😄）做个记录。

> 我使用版本：2.2.3

<!-- more -->

### 数据表格重载问题

`table.reload()`操作具有继承性，使用`table.render()`进行初始化渲染时的参数，进行表格重载时都会带上。如下所示，动态渲染出表格。

```javascript
        table.render({
            elem: '#LAY_table_materials'
            , cols: [[
                {
                    field: 'typeName',
                    title: '材料类型',
                    align:'center',
                    width:'15%'
                }
                , {
                    field: 'materialName',
                    title: '材料名称',
                    align:'center',
                    width:'16%'
                }
                , {
                    field: 'unit',
                    title: '单位',
                    align:'center',
                    width:'10%'
                }
                , {
                    field: 'price',
                    title: '单价（元）',
                    align:'center',
                    edit: 'text',
                    event:"editPrice",
                    width:'15%'
                }
                , {
                    field: 'quantity',
                    title: '购买数量',
                    align:'center',
                    edit: "text",
                    event:"editQuantity",
                    width:'10%'
                }
                , {
                    field: 'totalPrice',
                    title: '总价（元）',
                    align:'center',
                    width:'15%'
                }
                , {
                    fixed: 'right',
                    title: '操作',
                    align:'center',
                    toolbar: '#barOption',
                    width:'20%'
                }
            ]]
            ,done: function(res, curr, count){
                ......
            }
            <c:if test="${not empty id}">
            , url: '../can_material_order/layUIPage'
            , method: 'post'
            ,where:{
                "purchaseId":id
            }
            , page: true
            , request: {
                limitName: "size"
            }
            , limits: [10, 20, 30, 40, 50]
            , limit: 10
            </c:if>
        });
```

后续表格重载时使用`table.reload(LAY_table_materials)`，等同于重新执行了初始化渲染，done里的函数同样会执行。

现在我有这样一个需求：从弹出框中的数据表格选择需要采购的原材料，点击确定后父页面获得所选择的原材料，然后动态添加到父页面的数据表格中（一开始只有表头），再对表中的单价或者数量单元格编辑后，执行重载，将总价和合计行计算显示出来，我使用以下方法进行才有效。

```javascript
table.reload("LAY_table_materials",{
	data:table.cache["LAY_table_materials"],
	url:''
});
```

如果去掉data参数，若进行过删除操作（数据表格的del()方法，未与后端交互），重载后之前删掉的数据又回来了。**原因是table会保存初次渲染时的数据，没有赋值data，同时URL又为空的话，表格重载的数据就是第一次渲染时的。**

### 数据表格删除问题

删除实例：

```javascript
if(layEvent === 'detail'){ //查看
    //do somehing
  } else if(layEvent === 'del'){ //删除
    layer.confirm('真的删除行么', function(index){
      obj.del(); //删除对应行（tr）的DOM结构，并更新缓存
      layer.close(index);
      //向服务端发送删除指令
    });
  } else if(layEvent === 'edit'){ //编辑
    //do something
    
```

再看源码：

![](http://jianger-upic.test.upcdn.net/uPic/%E6%88%AA%E5%B1%8F2020-09-24%20%E4%B8%8B%E5%8D%888.07.37.png)

这一删除操作不是直接删除元素，而是将元素内容置空，这样在使用`table.cache`时会出现问题，例如判断当前数据表格是否为空，还得遍历一下数组。可以更改源码中的`table.cache[that.key][index]=[]为table.cache[that.key][index].splice(index,1)`实现真正删除元素。

其他问题有待发现更新......



