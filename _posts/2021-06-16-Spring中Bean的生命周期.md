---
layout:     post
title:      Spring中Bean的生命周期
subtitle:   概述
date:       2021-06-16
author:     QuanLi
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - Spring
---

# Spring中Bean的生命周期

​	对于普通的 Java 对象，当我们使用`new`关键字创建对象的时候，如果它没有任何引用，则其会被垃圾回收机制回收。而由 Spring IoC 容器托管的对象，它们的生命周期则是完全由容器控制。正确理解Bean 的生命周期非常重要，因为**Spring对Bean的管理可扩展性非常强**，下面展示了一个Bean的构造过程

![img](https://img2018.cnblogs.com/blog/1515111/201905/1515111-20190523200616898-700260604.png)

**或者**

![preview](https://pic1.zhimg.com/v2-baaf7d50702f6d0935820b9415ff364c_r.jpg?source=1940ef5c)

### 1、实例化Bean

​	**BeanFactory容器：**当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。

​	**ApplicationContext容器**：当容器启动结束后，便实例化所有的bean。

### 2、Bean的属性注入（依赖注入）

​	**按照Spring上下文对实例化的Bean进行配置，也就是IOC注入**

​	实例化后的对象被封装在BeanWrapper对象中，并且此时对象仍然是一个原生的状态，并没有进行依赖注入。
紧接着，Spring根据BeanDefinition中的信息进行依赖注入。
并且通过BeanWrapper提供的设置属性的接口完成依赖注入。

### 3、注入Aware接口

​	Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给bean。

- **如果Bean实现了BeanNameAware接口的话，Spring将Bean的Id传递给setBeanName()方法**
- **如果Bean实现了BeanFactoryAware接口的话，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入**
- **如果Bean实现了ApplicationContextAware接口的话，Spring将调用Bean的setApplicationContext()方法，将bean所在应用上下文引用传入进来**

### 4、BeanPostProcessor

​	当经过上述几个步骤后，bean对象已经被正确构造，但如果你**想要对象被使用前再进行一些自定义的处理**，就可以通过BeanPostProcessor接口实现。

该接口提供了两个函数：

- **postProcessBeforeInitialzation( Object bean, String beanName )** 
  当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  这个函数会先于InitialzationBean执行，因此称为前置处理。 
  所有Aware接口的注入就是在这一步完成的。
- **postProcessAfterInitialzation( Object bean, String beanName )** 
  当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  这个函数会在InitialzationBean完成后执行，因此称为后置处理。

### 5、InitializingBean与init-method

当BeanPostProcessor的前置处理完成后就会进入本阶段。 
InitializingBean接口只有一个函数：

- afterPropertiesSet()

这一阶段也可以在bean正式构造完成前增加我们自定义的逻辑，但它与前置处理不同，**由于该函数并不会把当前bean对象传进来，因此在这一步没办法处理对象本身，只能增加一些额外的逻辑。** 
若要使用它，我们需要让bean实现该接口，并把要增加的逻辑写在该函数中。然后Spring会在前置处理完成后检测当前bean是否实现了该接口，并执行afterPropertiesSet函数。

当然，Spring为了降低对客户代码的侵入性，给bean的配置提供了init-method属性，该属性指定了在这一阶段需要执行的函数名。Spring便会在初始化阶段执行我们设置的函数。init-method本质上仍然使用了InitializingBean接口。

### 6、Bean可以使用了

1. **此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。**

### 7、DisposableBean和destroy-method

​	**如果bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用。**



​	**当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean接口，会调用其实现的destroy方法**

​	**最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法**