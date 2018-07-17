---
layout:       post
title:        "Spring 事务管理详解"
subtitle:     ""
date:         2018-06-14 19:50:00
author:       "sunpwork"
header-mask:  0.3
catalog:      true
tags:
    - Java
    - Spring
    - 事务管理
---
>这里先放上一位[大佬的博客](http://snailclimb.top/2018/05/19/%E5%8F%AF%E8%83%BD%E6%98%AF%E6%9C%80%E6%BC%82%E4%BA%AE%E7%9A%84Spring%E4%BA%8B%E5%8A%A1%E7%AE%A1%E7%90%86%E8%AF%A6%E8%A7%A3/)，本文所有内容都是我详细研读这篇博客加上我自己的一些见解所写。

# 什么是事务
我觉得用一句通俗易懂的话来讲，事务就是数据库逻辑上的一组操作，要么都执行，要么都不执行。

# 事务的特性
事务有四大特性，分别为：原子性、一致性、隔离性、持久性。

* 原子性：事务是最小的执行单位，不可分割。原子性确保所有操作要么全部完成，要么完全不起作用；
* 一致性：执行事务前后，数据保持一致；
* 隔离性：并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的；
* 持久性：一个事务被提交后，他对数据库中数据的改变是持久的，即使数据库发生故障也不应该对其有任何影响。

# Spring事务管理接口
## PlatformTransactionManager：事务管理器（接口）
Spring并不直接管理事务，而是提供了多种事务管理器，他只提供了事务管理接口org.springframework.transaction.PlatformTransactionManager，将事务管理的职责委托给Hibernate或者JTA(Java Transaction API)等持久化机制所提供的相关平台框架的事务来实现。
```java
public interface PlatformTransactionManager {

   TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;

   void commit(TransactionStatus status) throws TransactionException;
   
   void rollback(TransactionStatus status) throws TransactionException;

}
```
常用持久层框架所对应的PlatformTransactionManager接口实现类如下所示：

SpringJDBC、MyBatis -> org.springframework.jdbc.datasource.DataSourceTransactionManager

Hibernate -> org.springframework.orm.hibernate5.HibernateTransactionManager

我们在使用JDBC或者MyBatis进行数据持久化操作时，我们xml配置通常如下：
```xml
<bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="dataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
```

## TransactionDefinition
事务管理器接口PlatformTransactionManager 通过getTransaction(TransactionDefinition definition)方法来得到一个事务，这个方法里面的参数是TransactionDefinition类，这个类定义了一些基本的事务属性的常量，该接口的常量信息会在后面依次介绍到。
```java
public interface TransactionDefinition {
    // 返回事务的传播行为
    int getPropagationBehavior(); 
    // 返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据
    int getIsolationLevel(); 
    // 返回事务必须在多少秒内完成
    //返回事务的名字
    String getName()；
    int getTimeout();  
    // 返回是否优化为只读事务。
    boolean isReadOnly();
}
```

## 事务隔离级别
在典型的应用中，多个事务并发运行，经常会操作相同的数据来完成各自的任务（多个任务对统一数据进行操作），并发虽然是必须的，但是可能会导致如下问题：

1. 脏读：当一个事务正在访问数据并且对数据进行修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据。因为这个数据并没有提交，那么另外一个事务读到的这个数据就是“脏数据”，依据“脏数据”所做的操作可能是不正确的。
2. 不可重复读：指在一个事务内多次读同一数据，在这个事务还没有结束时，另一个事务也访问该数据并进行修改。那么，在第一个事务中的两次读数据之间，由于第二次事务的修改导致第一个事务两次读取的数据可能不太一样。
3. 幻读：幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。

为了解决如上的问题，TransactionDefinition接口中定义了五个表示隔离级别的常量：

1. TransactionDefinition.ISOLATION_DEFAULT: 使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.
2. TransactionDefinition.ISOLATION_READ_UNCOMMITTED: 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读
3. TransactionDefinition.ISOLATION_READ_COMMITTED: 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
4. TransactionDefinition.ISOLATION_REPEATABLE_READ: 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
5. TransactionDefinition.ISOLATION_SERIALIZABLE: 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

## 事务传播行为
当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：

支持当前事务的情况：

1. TransactionDefinition.PROPAGATION_REQUIRED： 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
2. TransactionDefinition.PROPAGATION_SUPPORTS： 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
3. TransactionDefinition.PROPAGATION_MANDATORY： 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。

不支持当前事务的情况：

1. TransactionDefinition.PROPAGATION_REQUIRES_NEW： 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
2. TransactionDefinition.PROPAGATION_NOT_SUPPORTED： 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
3. TransactionDefinition.PROPAGATION_NEVER： 以非事务方式运行，如果当前存在事务，则抛出异常。

其他情况：

* TransactionDefinition.PROPAGATION_NESTED： 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

这里需要指出的是，前面的六种事务传播行为是 Spring 从 EJB 中引入的，他们共享相同的概念。而 PROPAGATION_NESTED 是 Spring 所特有的。以 PROPAGATION_NESTED 启动的事务内嵌于外部事务中（如果存在外部事务的话），此时，内嵌事务并不是一个独立的事务，它依赖于外部事务的存在，只有通过外部的事务提交，才能引起内部事务的提交，嵌套的子事务不能单独提交。如果熟悉 JDBC 中的保存点（SavePoint）的概念，那嵌套事务就很容易理解了，其实嵌套的子事务就是保存点的一个应用，一个事务中可以包括多个保存点，每一个嵌套子事务。另外，外部事务的回滚也会导致嵌套子事务的回滚。

* 事务超时属性

事务超时属性说简单点就是一个事务允许执行的最长时间，如果超过该时间限制事务还没有完成，则自动回滚事务。在TransactionDefinition中以int的值来表示超时时间，其单位是秒。

* 事务只读属性

事务的只读属性是指，对事务性资源进行只读操作或者是读写操作。所谓事务性资源就是指那些被事务管理的资源，比如数据源、 JMS 资源，以及自定义的事务性资源等等。如果确定只对事务性资源进行只读操作，那么我们可以将事务标志为只读的，以提高事务处理的性能。在 TransactionDefinition 中以 boolean 类型来表示该事务是否只读。

* 回滚规则

这些规则定义了哪些异常会导致事务回滚而哪些不会。默认情况下，事务只有遇到运行期异常时才会回滚，而在遇到检查型异常时不会回滚。但是你可以声明事务在遇到特定的检查型异常时像遇到运行期异常那样回滚。同样，你还可以声明事务遇到特定的异常不回滚，即使这些异常是运行期异常。

## TransactionStatus接口介绍
TransactionStatus接口用来记录事务的状态 该接口定义了一组方法,用来获取或判断事务的相应状态信息.

PlatformTransactionManager.getTransaction(…) 方法返回一个 TransactionStatus 对象。返回的TransactionStatus 对象可能代表一个新的或已经存在的事务（如果在当前调用堆栈有一个符合条件的事务）。
```java
public interface TransactionStatus{
    boolean isNewTransaction(); // 是否是新的事物
    boolean hasSavepoint(); // 是否有恢复点
    void setRollbackOnly();  // 设置为只回滚
    boolean isRollbackOnly(); // 是否为只回滚
    boolean isCompleted; // 是否已完成
}
```