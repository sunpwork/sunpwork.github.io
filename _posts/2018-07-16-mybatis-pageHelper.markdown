---
layout:       post
title:        "mybatis pageHelper上手"
subtitle:     "dataTables分页后端"
date:         2018-07-16 20:52:00
author:       "sunpwork"
header-img:   "img/in-post/post-eleme-pwa/eleme-at-io.jpg"
header-mask:  0.3
catalog:      true
multilingual: true
tags:
    - 后端开发
    - Java
    - 分页
---

>在web开发过程中涉及到表格时，例如我们上一篇讲到的dataTables，就会涉及到分页的需求，通常我们将分页分为两种：前端分页和后端分页</br>
>前端分页：一次性请求数据表格中的所有记录(ajax)，然后在前端缓存并且计算count和分页逻辑，一般前端组件(例如dataTable)会提供分页动作。特点是：简单，很适合小规模的web平台；当数据量大的时候会产生性能问题，在查询和网络传输的时间会很长。</br>
>后端分页：在ajax请求中指定页码(pageNum)和每页的大小(pageSize)，后端查询出当页的数据返回，前端只负责渲染。特点是：复杂一些；性能瓶颈在MySQL的查询性能，这个当然可以调优解决。一般来说，web开发使用的是这种方式。我们说的也是后端分页。

# dataTables插件
## 数据格式
在上一个章节中，我们了解到dataTables开启serverSide（服务器模式）之后，前端发送给后台的数据入下图

``` json
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

# Mybatis分页插件PageHelper
在使用mybatis逆向工程生成的mapper中，没有生成分页相关的方法，但是mybatis提供了Mybatis分页插件PageHelper这个分页插件
## POM依赖
使用pageHepler需要在pom中加入如下依赖：
```xml
    <dependency>
      <groupId>com.github.pagehelper</groupId>
      <artifactId>pagehelper</artifactId>
      <version>${pageHelper.version}</version>
    </dependency>
```
## mybatis对pageHelper的配置
mybatis如何配置我这里就不多说了，如果我们使用`mybatis-config.xml`文件去配置mybatis，我们只需要在配置文件中加入`plugin`插件即可
```xml
<!--旧版本的pageHelper插件使用PageHepler类进行配置-->
<plugins>
    <plugin interceptor="com.github.pagehelper.PageHelper">
        <property name="dialect" value="mysql"/>
    </plugin>
</plugins>
```
如果没有加载`mybatis-config.xml`配置文件，我们可以在注入`SqlSessionFactoryBean`的时候去配置插件
```xml
<!--新版本的pageHelper插件使用PageInterceptor类进行配置-->
<bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="plugins">
            <array>
                <bean class="com.github.pagehelper.PageInterceptor">
                    <property name="properties">
                        <value>
                            helperDialect=mysql
                        </value>
                    </property>
                </bean>
            </array>
        </property>
    </bean>
```
这是最新版的配置方法，之前的版本是使用`com.github.pagehelper.PageHelper`这个类去添加这个插件的，最新版本的得使用`PageInterceptor`这个类去添加，然后旧版的是使用`dialect`这个属性去指定数据库类型，新版的则是`helperDialect`

## 如何使用
先上代码
```java
    PageHelper.startPage(pageNum,pageSize);
    List list = this.studentMapper.selectByExample(new StudentExample());
    return PageInfo.of(list);
```
在执行查询语句之前，我们先调用`PageHelper.startPage(pageNum,pageSize)`来进行分页约束，`pageNum`表示当前页数，`pageSize`表示每页显示的数量,然后我们使用逆向工程生成的方法查询所有的记录，最后我们使用`PageInfo.of()`来获取分页的结果，我们在监视窗口中看一下结果。
![](/img/in-post/post-pageHelper/pageHelper01.jpg)