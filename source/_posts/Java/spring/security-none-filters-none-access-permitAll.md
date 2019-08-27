---
date: 2019-03-28 00:00:00
title: "[译文] Spring Security – security none, filters none, access permitAll"
author: Joden_He
tags: 
  - Java
  - Spring
categories: 
  - Java
  - Spring
description: "Spring Security 提供了几种将请求模式配置为不安全或者允许所有访问的机制。根据这些机制的不同，这可能意味着根本不在这个路径上运行安全过滤器链，或者运行了安全过滤器链但允许访问"

---

> 版权说明，文章翻译至 [Spring Security – security none, filters none, access permitAll](https://www.baeldung.com/security-none-filters-none-access-permitAll)

### 概述

Spring Security 提供了几种将请求模式配置为不安全或者允许所有访问的机制。根据这些机制的不同，这可能意味着根本不在这个路径上运行安全过滤器链，或者运行了安全过滤器链但允许访问。

### *access="permitAll”*

给 *<intercept-url\>* 标签设置 *access="permitAll"* 元素，将会给指定路径**允许**所有请求授权:

```xml
<intercept-url pattern="/login*" access="permitAll" />
```

或者通过 java 配置:

```java
http.authorizeRequests().antMatchers("/login*").permitAll()
```

这是在**不禁用安全过滤器**的情况下实现的，这些过滤器仍在运行，因此任何与 Spring Security 相关的功能仍然可以使用。

### *filters="none"*

这是在 Spring 3.1 前的功能，**已在 Spring 3.1 中弃用和被替代了**。

这个 filter 属性完全禁用指定的请求路径上的 Spring Security 过滤器链:

```xml
<intercept-url pattern="/login*" filters="none" />
```

当处理请求需要 Spring Security 的某些功能时，这可能会有问题。

由于在 Spring 3.0+ 版本这是一个被弃用的功能，在 Spring 3.1 中使用它将导致启动时出现运行时异常:

```shell
SEVERE: Context initialization failed
org.springframework.beans.factory.parsing.BeanDefinitionParsingException: 
Configuration problem: The use of "filters='none'" is no longer supported. 
Please define a separate <http> element for the pattern you want to exclude 
and use the attribute "security='none'".
Offending resource: class path resource [webSecurityConfig.xml]
    at o.s.b.f.p.FailFastProblemReporter.error(FailFastProblemReporter.java:68)
```

### *security=”none”*

正如我们在上面的错误消息中看到的那样，Spring 3.1 用一个新的表达式替换 *filters =“none”*  - *security =“none”*。

范围也发生了变化 —— 这不再在 *<intercept-url\>* 元素级别指定。相反，Spring 3.1 允许定义多个 *<http\>* 元素，每个元素都有自己的安全过滤链配置。因此，新的安全属性现在属于 *<http\>* 元素级别。

在实际使用中，它长这样:

```xml
<http pattern="/resources/**" security="none"/>
```

或者通过 java 配置:

```java
web.ignoring().antMatchers("/resources/**");
```

而不是旧的:

```xml
<intercept-url pattern="/resources/**" filters="none"/>
```

与 *filters =“none”* 类似，这也将完全禁用该请求路径的安全过滤器链 - 因此，当在应用程序中处理请求时，Spring Security 功能将不可用。

对于上面的示例而言，这不是问题，其主要涉及**提供静态资源** - 在实际中不需要做特殊处理。但是，如果以某种方式以编程方式处理请求 - 那么安全功能（例如，*requires-channel*，访问当前用户或调用安全方法）将不可用。

出于同样的原因，在已经配置了 *security="none"* 的 *<http\>* 元素上指定附加属性是没有意义的，因为该请求路径是不安全的，这些属性将被忽略。

或者，*access=’IS_AUTHENTICATED_ANONYMOUSLY’*  可用于允许匿名访问。

### *Caveats for security=”none”*

当使用多个 *<http\>* 元素时，有些配置了 *security =“none”*，请记住，定义这些元素的顺序很重要。我们应该首先使用特定的 *<http\>* 路径，最后再使用通用模式。

另请注意，如果 *<http\>* 元素**未指定模式**，则默认情况下，映射到通用匹配模式 - “/\*\* ” - 因此，此元素必须是最后一个。如果元素的顺序不正确，**则安全过滤器链**的**创建将失败**：

```shell
Caused by: java.lang.IllegalArgumentException: A universal match pattern ('/**') 
is defined  before other patterns in the filter chain, causing them to be ignored. 
Please check the ordering in your <security:http> namespace or FilterChainProxy bean configuration
    at o.s.s.c.h.DefaultFilterChainValidator.checkPathOrder(DefaultFilterChainValidator.java:49)
    at o.s.s.c.h.DefaultFilterChainValidator.validate(DefaultFilterChainValidator.java:39)
```

### 结论

本文讨论了允许使用 Spring Security 访问路径的选项 - 重点关注 **filters =“none”，security =“none” 和 access =“permitAll”** 之间的差异。

像往常一样，这些例子可以在 [GitHub](https://github.com/eugenp/tutorials/tree/master/spring-security-mvc-ldap) 上找到。

### 参考资料

- [Spring Security – security none, filters none, access permitAll](https://www.baeldung.com/security-none-filters-none-access-permitAll)

