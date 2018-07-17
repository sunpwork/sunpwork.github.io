---
layout:       post
title:        "jQuery表格插件_Datatables 使用"
subtitle:     ""
date:         2018-07-15 19:50:00
author:       "sunpwork"
header-mask:  0.3
catalog:      true
tags:
    - 前端
    - 表格插件
    - 分页
    - dataTables
---

>之前一直在做基于微信公众号的开发，很少去写PC端后台的前端，这几天在做一个后台管理平台，一直在选择用啥前端模板好，偶然看到sb-admin这个模板功能特别丰富，样式也很好看，果断入坑。sb-admin的内容我就不去多讲了，大家到GitHub下载源码根据自己的需求去抄就可以了。sb-admin这个模板官方给的案例用要到了jQuery的表格插件Datatables，下面就简单记录一下我在学习这个模板过程中的一些经验。

首先先甩出[官方文档](https://datatables.net/manual/)，哈哈，这是纯英文的，其实对于英文文档我也是很抗拒的，但是谁让这些好用的东西大多数是老外写出来的呢。我看英文文档的基本就是直接看案例代码，然后根据目录来找我想要的东西。话不多说，首先我们现在官网找到下载页面，官方提供了如下5种下载方式。

![download](/img/in-post/dataTables/download.png)

后面三种我们暂且不看，使用第一个官方cdn或者下载源码到自己的项目中都可以，最后在HTML中引用这两个文件即可。

```javascript
<link rel="stylesheet" type="text/css" href="https://cdn.datatables.net/1.10.19/css/jquery.dataTables.css">

<script type="text/javascript" charset="utf8" src="https://cdn.datatables.net/1.10.19/js/jquery.dataTables.js"></script>
```

```javascript
<link rel="stylesheet" type="text/css" href="/DataTables/datatables.css">
<script type="text/javascript" charset="utf8" src="/DataTables/datatables.js"></script>
```

然后我们在我们的HTML中先手写一个普通的表格出来。

```html
<table id="table_example">
    <thead>
    <tr>
        <th>列1</th>
        <th>列2</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>数据1</td>
        <td>数据11</td>
    </tr>
    <tr>
        <td>数据2</td>
        <td>数据22</td>
    </tr>
    </tbody>
</table>
```
预览效果：

![example](/img/in-post/dataTables/example.png)

我们现在使用DataTables插件初始化这个表格：
```javascript
$(document).ready(function () {
    $('#table_example').DataTable();
});
```
预览效果：
![dataTableExample](/img/in-post/dataTables/dataTableExample.png)

我们可以看到，在使用DataTables初始化之后，包括 分页 搜索 排序 ，表格该有的功能都添加进来了。现在我们是将列表的数据写死的，下面我们去掉`<tbody>`标签中的数据，然后在初始化之前加入如下代码：
```javascript
var data = [
    [11,12],
    [21,22],
    [31,32],
    [41,42]
];

$(document).ready(function () {
    $('#table_example').DataTable({
        data:data
    });
});
```
这样我们就可以动态的去设定表格的数据了，除了Arrays，还支持Object，如下：
```javascript
var data = [
        {column1:11,column2:12},
        {column1:21,column2:22},
        {column1:31,column2:32},
        {column1:41,column2:42},
    ];

    $(document).ready(function () {
        $('#table_example').DataTable({
            columns:[
                {data:"column1"},
                {data:"column2"}
            ],
            data:data 
        })ksgjhfdh ;
    });
```
一般情况下用在初始化DataTables时使用data属性绑定表格数据都是先从服务器拿到所有的数据，然后绑定到表格上。因为后台传过来的大多数是序列化的对象，所有一般会用Objects来绑定的多，但是这种一开始就将所有数据都绑定到表格的方式我不是特别推荐，虽然这样做可以把表格的搜索，分页，排序全部交由前端控件进行处理，但是一旦数据量特别大的时候，比如说这张数据表中一共有1w条数据，这样访问一次这个页面，服务器会向前端把这1w条数据全部传过来，这样太占用服务器资源了。所以DataTables提供了serverSide模式，也就是服务器模式。

现在我们换一个案例，我们数据库中有如下这样一张表：

![databaseExample](/img/in-post/dataTables/databaseExample.jpg)

我们现在使用DataTables的serverSide模式从服务器获取数据：
```javascript
$(document).ready(function() {
    $('#dataTables-example').DataTable({
        responsive: true,
        serverSide: true,//开启服务器模式
        ajax:{
            url:"datasource",
            dataFilter:function (res) {
                var resData = JSON.parse(res).data;
                return JSON.stringify({
                    draw:resData.draw,
                    recordsTotal:resData.recordsTotal,
                    recordsFiltered: resData.recordsFiltered,
                    data: resData.list
                })
            }
        },
        paging:true,
        searching:false,
        columns:[
            {data:"id"},
            {data:"name"},
            {data:"idCardNumber"},
            {data:"tel"},
            {data:"studentType"}
        ]
    });
});
```
ajax中属性的内容表示获取数据库信息的URL，dataTables向服务器请求的数据大致如下：
```
draw: 1
columns[0][data]: id
columns[0][name]: 
columns[0][searchable]: true
columns[0][orderable]: true
columns[0][search][value]: 
columns[0][search][regex]: false
columns[1][data]: name
columns[1][name]: 
columns[1][searchable]: true
columns[1][orderable]: true
columns[1][search][value]: 
columns[1][search][regex]: false
columns[2][data]: idCardNumber
columns[2][name]: 
columns[2][searchable]: true
columns[2][orderable]: true
columns[2][search][value]: 
columns[2][search][regex]: false
columns[3][data]: tel
columns[3][name]: 
columns[3][searchable]: true
columns[3][orderable]: true
columns[3][search][value]: 
columns[3][search][regex]: false
columns[4][data]: studentType
columns[4][name]: 
columns[4][searchable]: true
columns[4][orderable]: true
columns[4][search][value]: 
columns[4][search][regex]: false
order[0][column]: 0
order[0][dir]: asc
start: 0
length: 10
search[value]: 
search[regex]: false
```

我们先简单的说明一下这些请求数据的含义，`draw`是dataTables绘制的次数，也就是ajax请求的次数，因为没请求一次，dataTables都会进行重绘，`columns[i][data]`对应的是我们在colums属性中设置的每列对应的data，`order[0][column]`  以及 `order[0][dir]`是排序依据的字段以及排序方式（升序/降序），`start`以及`length`是用来分页的，表示偏移量以及每页显示的长度。其他的字段我们暂且不管，下面是官方给的服务器返回json格式：
```json
{
    "draw": 1,
    "recordsTotal": 57,
    "recordsFiltered": 57,
    "data": [
        [
            "Angelica",
            "Ramos",
            "System Architect",
            "London",
            "9th Oct 09",
            "$2,875"
        ],
        [
            "Ashton",
            "Cox",
            "Technical Author",
            "San Francisco",
            "12th Jan 09",
            "$4,800"
        ],
        ...
    ]
}
```

我们逐一的解释一下以上每个数据的含义，`draw`就是dataTables发送给我们的draw，我们原样返回就可以了，recordsTotal表示数据库中的总数据量，`recordsFiltered`表示数据库筛选过的数据量，注意，这个并不是返回的数据数量，是在filter是过滤的意思，当我们使用搜索框的时候会去修改`recordsFiltered`这个值，一般情况下我们`recordsTotalrecordsFiltered`两个值是相等的。然后`data`就是返回的数据数组，和之前的用dataTables构造中的data格式以及用法是一样的。很多情况下，我们服务器返回的数据格式跟dataTables所要求的格式是不同的，所以dataTables提供了`dataFilter`这个属性，在上面代码中其实以及写过了：
```javascript
dataFilter:function (res) {
    var resData = JSON.parse(res).data;
    return JSON.stringify({
        draw:resData.draw,
        recordsTotal:resData.recordsTotal,
        recordsFiltered: resData.recordsFiltered,
        data: resData.list
    })
}
```
这样我们就可以灵活的去适应服务器的数据了。至于服务器端如何去处理并返回相应的数据，我会在下一章 mybatis与pageHelper中详细讲解。