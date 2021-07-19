---
layout:     post
title:      JustAuth 第三方登录开源库
subtitle:   概述
date:       2019-8-16
author:     Gavin
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - web
---

# JustAuth 第三方登录开源库

​	JustAuth是**史上最全的整合第三方登录的开源库**。目前已支持Github、Gitee、微博、钉钉、百度、Coding、腾讯云开发者平台、OSChina、支付宝、QQ、微信、淘宝、Google、Facebook、抖音、领英、小米、微软、今日头条、Teambition、StackOverflow、Pinterest、人人、华为、企业微信、酷家乐、Gitlab、美团、饿了么和推特等第三方平台的授权登录。 Login, so easy!

## 安装

JustAuth 使用 Java 开发，可以引入 JustAuth 依赖：

~~~xml
<dependency>
  <groupId>me.zhyd.oauth</groupId>
  <artifactId>JustAuth</artifactId>
  <version>${latest.version}</version>
</dependency>
~~~

## 示例

第三方登录使用的是 OAuth 2 模式，授权流程参与的角色包括：

- 资源所有者（Resource Owner）：**用户**
- 资源服务器（Resource Server）：托管受保护的用户账号信息
- 授权服务器（Authorization Server）：授权服务器，验证用户身份然后为客户端派发资源访问令牌
- 客户端（Client）：代表意图访问受限资源的第三方应用

**OAuth2流程**

​	对于JustAuth 而言，其核心就是每个平台所对应的一个个具体的 request 类，在进行授权之前，需要就具体的平台创建对应的 request 实例：

```javascript
// 创建授权request
AuthRequest authRequest = new AuthGiteeRequest(AuthConfig.builder()
        .clientId("clientId")
        .clientSecret("clientSecret")
        .redirectUri("redirectUri")
        .build());
```

在这个例子中，针对 Gitee 平台，使用 AuthGiteeReques 类，并配置在该平台的的开发者账号，包括 clientId，clientSecret，和 redirectUri。然后，就可以调用 authRequest 的 authorize 接口，获取对应的授权链接：

```javascript
String authorizeUrl = authRequest.authorize("state");
```

获取授权链接后，可以进行手动重定向。一个简单的重定向回调实例如下：

```javascript
/**
 * 
 * @param source 第三方授权平台，以本例为参考，该值为gitee（因为上面声明的AuthGiteeRequest）
 */
@RequestMapping("/render/{source}")
public void renderAuth(@PathVariable("source") String source, HttpServletResponse response) throws IOException {
    AuthRequest authRequest = getAuthRequest(source);
    String authorizeUrl = authRequest.authorize(AuthStateUtils.createState());
    response.sendRedirect(authorizeUrl);
}
```

从当前页面，跳转到第三方平台的授权验证页面，并携带当前状态信息。

当用户完成了第三方平台的授权后，授权登录会返回 code，并调用回调：

```javascript
AuthResponse response = authRequest.login(callback);
```

一个简单的回调接口可以这样实现：

```javascript
/**
 * 
 * @param source 第三方授权平台，以本例为参考，该值为gitee（因为上面声明的AuthGiteeRequest）
 */
@RequestMapping("/callback/{source}")
public Object login(@PathVariable("source") String source, AuthCallback callback) {
    AuthRequest authRequest = getAuthRequest(source);
    AuthResponse response = authRequest.login(callback);
    return response;
}
```

通过给 AuthRequest 的 login 接口提供一个 AuthCallback，实现授权码的回调。

另外，对于支持刷新 token 的平台，可以使用 refresh 接口进行 token 的刷新：

```javascript
/**
 * 
 * @param source 第三方授权平台，以本例为参考，该值为gitee（因为上面声明的AuthGiteeRequest）
 * @param token  login成功后返回的refreshToken
 */
@RequestMapping("/refresh/{source}")
public Object refreshAuth(@PathVariable("source") String source, String token){
    AuthRequest authRequest = getAuthRequest(source);
    return authRequest.refresh(AuthToken.builder().refreshToken(token).build());
}
```

而对于取消授权的平台，可以使用 revoke 实现：

```javascript
/**
 * 
 * @param source 第三方授权平台，以本例为参考，该值为gitee（因为上面声明的AuthGiteeRequest）
 * @param token  login成功后返回的accessToken
 */
@RequestMapping("/revoke/{source}/{token}")
public Object revokeAuth(@PathVariable("source") String source, @PathVariable("token") String token) throws IOException {
    AuthRequest authRequest = getAuthRequest(source);
    return authRequest.revoke(AuthToken.builder().accessToken(token).build());
}
```

JustAuth 对于各平台提供了统一的接口，使得多平台的授权登录的实现变得十分简单。同时，JustAuth 也在文档提供了不同平台的开发应用申请等具体流程，并提供了接口测试的平台，上手十分容易。

![image-20210715202311897](https://i.loli.net/2021/07/15/Dewxjoif1JaCOM5.png)

# 总结

​	JustAuth 作为一个第三方登录组件库，提供了极为全面的平台支持，基本满足了第三方登录的日常需求，省去了重复阅读不同平台的 SDK 文档，和对接实现的麻烦，使用统一的接口，隐去了不同平台接口令人头疼的差异，使得第三方授权变得十分清晰明了，很方便地就能整合到业务应用中，大大提升了开发效率。同时，JustAuth 也提供了对于自定义平台验证的支持，使得其具备极强的可扩展性，可以适用于各种各样的业务场景，十分值得尝试。