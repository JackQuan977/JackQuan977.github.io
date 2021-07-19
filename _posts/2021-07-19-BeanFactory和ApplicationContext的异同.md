---
layout:     post
title:      BeanFactory和ApplicationContext的异同
subtitle:   概述
date:       2021-07-19
author:     QuanLi
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - Spring
---

# BeanFactory和ApplicationContext的异同

**相同：**

- Spring提供了两种不同的**IOC 容器**，一个是**BeanFactory**，另外一个是**ApplicationContext**，它们都是Java interface，**ApplicationContext继承于BeanFactory**(ApplicationContext继承ListableBeanFactory)。
- 它们都可以用来配置XML属性，也支持属性的自动注入。
- 而ListableBeanFactory继承BeanFactory，BeanFactory 和 ApplicationContext 都提供了一种方式，使用getBean("bean name")获取bean。

BeanFactory 获取bean注册信息

~~~java
public class HelloWorldApp{ 
   public static void main(String[] args) { 
      XmlBeanFactory factory = new XmlBeanFactory (new ClassPathResource("beans.xml")); 
      HelloWorld obj = (HelloWorld) factory.getBean("helloWorld");    
      obj.getMessage();    
   }
}
~~~

ApplicationContext 获取bean注册信息

~~~java
public class HelloWorldApp{ 
   public static void main(String[] args) { 
      ApplicationContext context=new ClassPathXmlApplicationContext("beans.xml"); 
      HelloWorld obj = (HelloWorld) context.getBean("helloWorld");    
      obj.getMessage();    
   }
}
~~~

但是他们在工作和特性上有一些不同：

- `BeanFactory` ：延迟注入(**使用到某个 bean 的时候才会注入**),相比于`ApplicationContext` 来说会占用更少的内存，程序启动速度更快。
- `ApplicationContext` ：容器启动的时候，**不管你用没用到，一次性创建所有 bean 。**`BeanFactory` 仅提供了最基本的依赖注入支持，` ApplicationContext` 扩展了 `BeanFactory` ,除了有`BeanFactory`的功能还有额外更多功能，所以一般开发人员使用` ApplicationContext`会更多。
- BeanFactory不支持国际化，即i18n，但ApplicationContext提供了对它的支持。
- BeanFactory与ApplicationContext之间的另一个区别是能够将事件发布到注册为监听器的bean。
- BeanFactory 的一个核心实现是XMLBeanFactory 而ApplicationContext 的一个核心实现是ClassPathXmlApplicationContext，Web容器的环境我们使用WebApplicationContext并且增加了getServletContext 方法。
- 如果使用自动注入并使用BeanFactory，则需要使用API注册AutoWiredBeanPostProcessor，如果使用ApplicationContext，则可以使用XML进行配置。
- 简而言之，BeanFactory提供基本的IOC和DI功能，而ApplicationContext提供高级功能，BeanFactory可用于测试和非生产使用，但ApplicationContext是功能更丰富的容器实现，应该优于BeanFactory

