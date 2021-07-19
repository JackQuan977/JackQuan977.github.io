---
layout:     post
title:      初识SpringSecurity
subtitle:   概述
date:       2021-07-15
author:     QuanLi
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - Spring
---

# 初识SpringSecurity

## 权限管理概念

​	权限管理，一般指根据系统设置的安全规则或者安全策略，用户可以访问而且只能访问自己被授权的资源。权限管 理几乎出现在任何系统里面，前提是需要有用户和密码认证的系统。

> 在权限管理的概念中，有两个非常重要的名词：
>
> - 认证：通过用户名和密码成功登陆系统后，让系统得到当前用户的角色身份。
> - 授权：系统根据当前用户的角色，给其授予对应可以操作的权限资源。

## 完成权限管理需要三个对象

- 用户：主要包含用户名，密码和当前用户的角色信息，可实现认证操作。
- 角色：主要包含角色名称，角色描述和当前角色拥有的权限信息，可实现授权操作。
- 权限：权限也可以称为菜单，主要包含当前权限名称，url地址等信息，可实现动态展示菜单。

> 注：这三个对象中，用户与角色是多对多的关系，角色与权限是多对多的关系，用户与权限没有直接关系， 二者是通过角色来建立关联关系的。

### 初识SpringSecurity

​	Spring Security是spring采用**AOP思想**，**基于servlet过滤器**实现的安全框架。它提供了完善的认证机制和方法级的 授权功能。是一款非常优秀的**权限管理框架**。

## SpringSecurity常用过滤器介绍

**过滤器是一种典型的AOP思想**

### SecurityContextPersistenceFilter

​	首当其冲的一个过滤器，作用之重要，自不必多言。

​	SecurityContextPersistenceFilter主要是使用SecurityContextRepository在session中保存或更新一SecurityContext，并将SecurityContext给以后的过滤器使用，来为后续filter建立所需的上下文。SecurityContext中存储了当前用户的认证以及权限信息。

### WebAsyncManagerIntegrationFilter

此过滤器用于集成SecurityContext到Spring异步执行机制中的WebAsyncManager

### HeaderWriterFilter

向请求的Header中添加相应的信息,可在http标签内部使用security:headers来控制（仅限于JSP页面）

### CsrfFilter

csrf又称跨域请求伪造，SpringSecurity会对所有post请求验证是否包含系统生成的csrf的token信息， 如果不包含，则报错。起到防止csrf攻击的效果。

### LogoutFilter

匹配URL为/logout的请求，实现用户退出,清除认证信息。

### UsernamePasswordAuthenticationFilter

认证操作全靠这个过滤器，默认匹配URL为/login且必须为POST请求。

### DefaultLoginPageGeneratingFilter

如果没有在配置文件中指定认证页面，则由该过滤器生成一个默认认证页面。

### DefaultLogoutPageGeneratingFilter

由此过滤器可以生产一个默认的退出登录页面

### BasicAuthenticationFilter

此过滤器会自动解析HTTP请求中头部名字为Authentication，且以Basic开头的头信息。

### RequestCacheAwareFilter

通过HttpSessionRequestCache内部维护了一个RequestCache，用于缓存HttpServletRequest

### SecurityContextHolderAwareRequestFilter

针对ServletRequest进行了一次包装，使得request具有更加丰富的API

### AnonymousAuthenticationFilter

当SecurityContextHolder中认证信息为空,则会创建一个匿名用户存入到SecurityContextHolder中。 spring security为了兼容未登录的访问，也走了一套认证流程，只不过是一个匿名的身份。

> 当用户以游客身份登录的时候，也就是可以通过设置某些接口可以匿名访问

### SessionManagementFilter

SecurityContextRepository限制同一用户开启多个会话的数量

### ExceptionTranslationFilter

异常转换过滤器位于整个springSecurityFilterChain的后方，用来转换整个链路中出现的异常

### FilterSecurityInterceptor

获取所配置资源访问的授权信息，根据SecurityContextHolder中存储的用户信息来决定其是否有权 限

> 该过滤器限制哪些资源可以访问，哪些不能够访问





