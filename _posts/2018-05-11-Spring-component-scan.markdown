---
layout:       post
title:        "Spring配置 <context:component-scan>使用说明"
subtitle:     ""
date:         2018-05-11 19:50:00
author:       "sunpwork"
header-mask:  0.3
catalog:      true
tags:
    - Java
    - Spring配置
---

转载自 [`CSDN<context:component-scan>使用说明`](https://blog.csdn.net/chunqiuwei/article/details/16115135)  [`Spring中<context:annotation-config/>的作用`](https://blog.csdn.net/chenlong220192/article/details/46723561)  部分修改

在xml配置了`<context:component-scan>`这个标签后，spring可以自动去扫描base-pack下面或者子包下面的java文件，如果扫描到有`@Component @Controller @Service`等这些注解的类，则把这些类注册为bean。

这里我们顺带讲一下`<context:annotation-config/>`这个标签的使用：

`<context:annotation-config/>`的作用是向Spring容器注册以下四个BeanPostProcessor：

* AutowiredAnnotationBeanPostProcessor
* CommonAnnotationBeanPostProcessor
* PersistenceAnnotationBeanPostProcessor
* RequiredAnnotationBeanPostProcessor

那么，为什么要注册这四个BeanPostProcessor呢？

是为了让系统能够识别相应的注解。

例如：

1. 如果想使用@Autowired注解，那么就必须事先在 Spring 容器中声明 AutowiredAnnotationBeanPostProcessor Bean。
传统声明方式如下：`<bean class=”org.springframework.beans.factory.annotation. AutowiredAnnotationBeanPostProcessor “/>`

2. 如果想使用`@ Resource 、@ PostConstruct、@ PreDestroy`等注解就必须声明CommonAnnotationBeanPostProcessor  Bean。

3. 如果想使用`@PersistenceContext`注解，就必须声明PersistenceAnnotationBeanPostProcessor的Bean。

4. 如果想使用`@Required`的注解，就必须声明RequiredAnnotationBeanPostProcessor的Bean。

以上这些注解是很常用的，如果按照传统的方式进行配置将会非常繁琐，所以Spring给我们提供了一个简便的方式：`<context:annotation-config/>`，使用该元素可以自动声明以上注解。

注：由于`<context:component-scan base-package=”xx.xx”/>`也包含了自动注入上述Bean的功能，所以`<context:annotation-config/>` 可以省略。如果两者都进行了配置，则只有前者有效。