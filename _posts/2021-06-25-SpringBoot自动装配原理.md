---
layout:     post
title:      SpringBoot自动装配原理
subtitle:   概述
date:       2021-06-25
author:     QuanLi
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - Spring
---

# SpringBoot自动装配原理

### 什么是自动装配

​	使用过 Spring 的小伙伴，一定有被 XML 配置统治的恐惧。即使 Spring 后面引入了基于注解的配置，我们在开启某些 Spring 特性或者引入第三方依赖的时候，还是需要用 XML 或 Java 进行显式配置。

举个例子。没有 Spring Boot 的时候，我们写一个 RestFul Web 服务，还首先需要进行如下配置。

~~~java
@Configuration
public class RESTConfiguration
{
    @Bean
    public View jsonTemplate() {
        MappingJackson2JsonView view = new MappingJackson2JsonView();
        view.setPrettyPrint(true);
        return view;
    }

    @Bean
    public ViewResolver viewResolver() {
        return new BeanNameViewResolver();
    }
}
~~~

spring-servlet.xml

~~~xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context/ http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/mvc/ http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="com.howtodoinjava.demo" />
    <mvc:annotation-driven />

    <!-- JSON Support -->
    <bean name="viewResolver" class="org.springframework.web.servlet.view.BeanNameViewResolver"/>
    <bean name="jsonTemplate" class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>

</beans>
~~~

但是，Spring Boot 项目，我们**只需要添加相关依赖**，无需配置，通过启动下面的 `main` 方法即可。

~~~java
@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
~~~

###  SPI

我们现在提到自动装配的时候，一般会和 Spring Boot 联系在一起。但是，实际上 Spring Framework 早就实现了这个功能。Spring Boot 只是在其基础上，通过 SPI 的方式，做了进一步优化。

- SpringBoot 定义了一套接口规范，这套规范规定：**SpringBoot 在启动时会扫描外部引用 jar 包中的`META-INF/spring.factories`文件，将文件中配置的类型信息加载到 Spring 容器（此处涉及到 JVM 类加载机制与 Spring 的容器知识），并执行类中定义的各种操作。对于外部 jar 来说，只需要按照 SpringBoot 定义的标准，就能将自己的功能装置进 SpringBoot。**

### @SpringBootApplication注解

我们可以把 `@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。

#### **`@ComponentScan`** **注解**

~~~java
ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
~~~

这个注解咱们都是比较熟悉的，无非就是**自动扫描并加载符合条件的Bean到容器中**，这个注解会默认扫描声明类所在的包开始扫描

这个注解一共包含以下几个属性：

~~~java
basePackages：指定多个包名进行扫描
basePackageClasses：对指定的类和接口所属的包进行扫
excludeFilters：指定不扫描的过滤器
includeFilters：指定扫描的过滤器
lazyInit：是否对注册扫描的bean设置为懒加载
nameGenerator：为扫描到的bean自动命名
resourcePattern：控制可用于扫描的类文件
scopedProxy：指定代理是否应该被扫描
scopeResolver：指定扫描bean的范围
useDefaultFilters：是否开启对@Component，@Repository，@Service，@Controller的类进行检测
~~~

#### **`@SpringBootConfiguration`注解**

这个注解更简单了，它只是对`Configuration`注解的一个封装而已

~~~java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
}
~~~

#### **`EnableAutoConfiguration`注解**

`EnableAutoConfiguration` 只是一个简单地注解，**自动装配核心功能的实现实际是通过 `AutoConfigurationImportSelector`类。**

~~~java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage //作用：将main包下的所欲组件注册到容器中
@Import({AutoConfigurationImportSelector.class}) //加载自动装配类 xxxAutoconfiguration
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
~~~

**利用`@Import`注解，将所有符合自动装配条件的bean注入到IOC容器中**

### AutoConfigurationImportSelector:加载自动装配类

进入类`AutoConfigurationImportSelector`，观察其`selectImports`方法，**这个方法执行完毕后，Spring会把这个方法返回的类的全限定名数组里的所有的类都注入到IOC容器中**

~~~java
private static final String[] NO_IMPORTS = new String[0];

public String[] selectImports(AnnotationMetadata annotationMetadata) {
        // <1>.判断自动装配开关是否打开
        if (!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
          //<2>.获取所有需要装配的bean
AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
            
AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(autoConfigurationMetadata, annotationMetadata);
            return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
        }
    }
~~~

这里我们需要重点关注一下**`getAutoConfigurationEntry()`方法**，这个方法主要负责加载自动配置类的。

该方法调用链如下：

![image-20210626165509391](C:\Users\16227\AppData\Roaming\Typora\typora-user-images\image-20210626165509391.png)

现在我们结合`getAutoConfigurationEntry()`的源码来详细分析一下：

~~~java
private static final AutoConfigurationEntry EMPTY_ENTRY = new AutoConfigurationEntry();

AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata, AnnotationMetadata annotationMetadata) {
        //<1>.
        if (!this.isEnabled(annotationMetadata)) {
            return EMPTY_ENTRY;
        } else {
            //<2>.
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
            //<3>.
            List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
            //<4>.
            configurations = this.removeDuplicates(configurations);
            Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.filter(configurations, autoConfigurationMetadata);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
        }
    }
~~~

**第 1 步**:

判断自动装配开关是否打开。默认`spring.boot.enableautoconfiguration=true`，可在 `application.properties` 或 `application.yml` 中设置

**第 2 步** ：

用于获取`EnableAutoConfiguration`注解中的 `exclude` 和 `excludeName`。

**第 3 步**

获取需要自动装配的所有配置类，读取`META-INF/spring.factories`

![image-20210626165643730](C:\Users\16227\AppData\Roaming\Typora\typora-user-images\image-20210626165643730.png)

从下图可以看到这个文件的配置内容都被我们读取到了。`XXXAutoConfiguration`的作用就是按需加载组件。

![image-20210626165716415](C:\Users\16227\AppData\Roaming\Typora\typora-user-images\image-20210626165716415.png)

不光是这个依赖下的`META-INF/spring.factories`被读取到，所有 Spring Boot Starter 下的`META-INF/spring.factories`都会被读取到。![image-20210626165818189](C:\Users\16227\AppData\Roaming\Typora\typora-user-images\image-20210626165818189.png)

如果，我们自己要创建一个 Spring Boot Starter，这一步是必不可少的。

**第 4 步** ：

到这里可能面![image-20210626165959538](C:\Users\16227\AppData\Roaming\Typora\typora-user-images\image-20210626165959538.png)试官会问你:“`spring.factories`中这么多配置，每次启动都要全部加载么？”。

很明显，这是不现实的。我们 debug 到后面你会发现，`configurations` 的值变小了。



**因为，这一步有经历了一遍筛选，`@ConditionalOnXXX` 中的所有条件都满足，该类才会生效。**

~~~java
@Configuration
// 检查相关的类：RabbitTemplate 和 Channel是否存在
// 存在才会加载
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {
}
~~~

有兴趣的童鞋可以详细了解下 Spring Boot 提供的条件注解

- `@ConditionalOnBean`：当容器里有指定 Bean 的条件下
- `@ConditionalOnMissingBean`：当容器里没有指定 Bean 的情况下
- `@ConditionalOnSingleCandidate`：当指定 Bean 在容器中只有一个，或者虽然有多个但是指定首选 Bean
- `@ConditionalOnClass`：当类路径下有指定类的条件下
- `@ConditionalOnMissingClass`：当类路径下没有指定类的条件下
- `@ConditionalOnProperty`：指定的属性是否有指定的值
- `@ConditionalOnResource`：类路径是否有指定的值
- `@ConditionalOnExpression`：基于 SpEL 表达式作为判断条件
- `@ConditionalOnJava`：基于 Java 版本作为判断条件
- `@ConditionalOnJndi`：在 JNDI 存在的条件下差在指定的位置
- `@ConditionalOnNotWebApplication`：当前项目不是 Web 项目的条件下
- `@ConditionalOnWebApplication`：当前项目是 Web 项 目的条件下

## 总结

Spring Boot 通过`@EnableAutoConfiguration`开启自动装配，通过 SpringFactoriesLoader 最终加载`META-INF/spring.factories`中的自动配置类实现自动装配，自动配置类其实就是通过`@Conditional`按需加载的配置类，想要其生效必须引入`spring-boot-starter-xxx`包实现起步依赖

- Spring Boot 通过`@EnableAutoConfiguration`开启自动装配
- `EnableAutoConfiguration` 只是一个简单地注解，**自动装配核心功能的实现实际是通过 `AutoConfigurationImportSelector`类。**
- 进入类`AutoConfigurationImportSelector`，观察其`selectImports`方法，**这个方法执行完毕后，Spring会把这个方法返回的类的全限定名数组里的所有的类都注入到IOC容器中**，**利用`@Import`注解，将所有符合自动装配条件的bean注入到IOC容器中**
- `selectImports`方法中的getAutoConfigurationEntry方法负责加载自动配置类，加载`META-INF/spring.factories`中的自动配置类实现自动装配
- 通过`@Conditional`按需加载的配置类

