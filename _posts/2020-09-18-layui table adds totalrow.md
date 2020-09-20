---
clayout: post
title: Layui旧版本数据表格添加合计行
categories:
 - Layui
tags:
 - Layui
---

从Layui 2.4.0版本开始，Layui Table开始支持数据表格添加合计行的功能，表格渲染时添加一个属性参数即可。但是最近做项目，公司不使用新版本，又需要实现该功能，于是便只有自己实现了。

<!-- more -->

废话不多说，直接上代码实现，样式是Layui官方的样子。

> 我的Layui版本：2.2.3

```javascript
<style type="text/css">
    .layui-table-box .layui-table{
        width: 100%;
    }
    .layui-table-total .layui-table{
        width: 100%;
        background: #f2f2f2;
    }
    .layui-table-total .layui-table tr{
        text-align: center;
    }
</style>
...//省略其他代码
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
      width:'15%'
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
  //以下是关键，done的作用参见官方文档
  ,done: function(res, curr, count){
    $(".layui-table-total").remove();
    var totalDiv='<div class="layui-table-total">' +
        '<table cellspacing="0" cellpadding="0" border="0" class="layui-table">' +
        '<tbody><tr>' +
        '<td style="width: 15%;"><div class="layui-table-cell">合计</div></td>' +
        '<td style="width: 15%;"></td><td style="width: 10%;"></td><td style="width: 15%;"></td>' +
        '<td style="width: 10%;"><div class="layui-table-cell sumQuantity" ></div></td>' +
        '<td style="width: 15%;"><div class="layui-table-cell sumPrice" ></div></td>' +
        '<td style="width: 20%;"></td></tr></tbody></table></div>'
    if (count>0){
      $(".layui-table-box").after(totalDiv);
      countSum();
    }
  }
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
});

//合计行计算
function countSum() {
  var detailDatas= [];
  var sumPrice=0;
  var sumQuantity=0;
  detailDatas = table.cache["LAY_table_materials"];
  for(var i = 0; i < detailDatas.length; i++){
    //此处的判断是由于Layui的删除del()方法之后只是清空了内容，并不是删掉了元素
    if (JSON.stringify(detailDatas[i]).length<=2){
      continue;
    }
    sumPrice += parseFloat(detailDatas[i].totalPrice)>0?parseFloat(detailDatas[i].totalPrice):0;
    sumQuantity+=parseInt(detailDatas[i].quantity)>0?parseInt(detailDatas[i].quantity):0;
  }
  $(".layui-table-total .layui-table .sumQuantity").text(sumQuantity);
  $(".layui-table-total .layui-table .sumPrice").text(sumPrice);
}

//监听单元格编辑操作
table.on('edit(LAY_table_materials)', function(obj){ //注：edit是固定事件名，test是table原始容器的属性 lay-filter="对应的值"
  ......
  //发生编辑操作就重新计算合计
  countSum();
});
```

Layui使用起来还是有很多不舒服的地方的，有时间慢慢写下来自己遇到的问题和自己怎么解决的。