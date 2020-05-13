[TOC]

# Web on Servlet Stack

Version 5.1.3.RELEASE

This part of the documentation covers support for Servlet-stack web applications built on the Servlet API and deployed to Servlet containers. Individual chapters include [Spring MVC](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#mvc), [View Technologies](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#mvc-view), [CORS Support](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#mvc-cors), and [WebSocket Support](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#websocket). For reactive-stack web applications, see [Web on Reactive Stack](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web-reactive.html#spring-web-reactive).

* `this part`: 这部分

这部分文档包括 对构建在 Servlet API 上并部署到Servlet容器中的 Servlet-stack web 应用程序的支持。单独的章节包括 [Spring MVC]()、[VIEW Technologies]()、[CORS支持] 和 [WebSocket 支持]()。对于 `reactive-stack web` 应用程序，请参见 [Web on Reactive Stack]()。

# 1. Spring Web MVC

Spring Web MVC is the original web framework built on the Servlet API and has been included in the Spring Framework from the very beginning. The formal name, “Spring Web MVC,” comes from the name of its source module ([`spring-webmvc`](https://github.com/spring-projects/spring-framework/tree/master/spring-webmvc)), but it is more commonly known as “Spring MVC”.

- `original ` ： 起初的
- `formal `: 正式的

`Spring Web MVC`是构建在 `Servlet API` 上的原始Web框架，从一开始就包含在`spring framework`中，它的正式名称是 `Spring Web MVC`,这个名称来自项目源模块的名称`spring-webmvc`。但是它通常被称为 `Spring MVC`

Parallel to Spring Web MVC, Spring Framework 5.0 introduced a reactive-stack web framework whose name, “Spring WebFlux,” is also based on its source module ([`spring-webflux`](https://github.com/spring-projects/spring-framework/tree/master/spring-webflux)). This section covers Spring Web MVC. The [next section](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web-reactive.html#spring-web-reactive) covers Spring WebFlux.

- `introduce`: 介绍 ， 引入
- `section` ： 部分

spring framework 5.0 引入了一个和 `Spring Web MVC`同级的`reactive-stack web framework`叫做 `Spring WebFlux`,他也是基于他的项目模块名`spring-webflux`命名的。 这一部分介绍的是（`covers` : 涵盖）`Spring Web MVC`,下一部分会介绍 `Spring WebFlux`。

For baseline information and compatibility with Servlet container and Java EE version ranges, see the Spring Framework [Wiki](https://github.com/spring-projects/spring-framework/wiki/Spring-Framework-Versions).

* `compatibility`：通用性，兼容性

对于版本与servlet容器和JavaEE版本范围的兼容性，请参阅Spring框架[Wiki](https://github.com/spring-projects/spring-framework/wiki/Spring-Framework-Versions).。

## 1.1 DispatcherServlet

[Same as in Spring WebFlux](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web-reactive.html#webflux-dispatcher-handler)

Spring MVC, as many other web frameworks, is designed around the front controller pattern where a central `Servlet`, the `DispatcherServlet`, provides a shared algorithm for request processing, while actual work is performed by configurable delegate components. This model is flexible and supports diverse workflows.

* `designed `:设计
* `actual`: 实际的，真实的
* `performe `: 执行
* `flexible`: 灵活的
* `diverse`: 不同的

`Spring MVC`，和许多其他的web框架一样，都是围绕一个中心`Servlet`的前端控制器模式设计。`Spring MVC`中的这个中心`Servlet`就是 `DispatcherServlet`,它提供了一个用于处理请求的共享算法，在实际工作执行中通过配置的组件来完成工作。这是一个灵活的模型，他支持不同的工作流程

The `DispatcherServlet`, as any `Servlet`, needs to be declared and mapped according to the Servlet specification by using Java configuration or in `web.xml`. In turn, the `DispatcherServlet` uses Spring configuration to discover the delegate components it needs for request mapping, view resolution, exception handling, [and more](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/web.html#mvc-servlet-special-bean-types).

