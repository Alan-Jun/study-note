[TOC]

# Spring Boot Reference Guide

`reference`: v 引用，n 参考,参考书

`当前版本 2.1.0.RELEASE`

## Authors

Phillip Webb, Dave Syer, Josh Long, Stéphane Nicoll, Rob Winch, Andy Wilkinson, Marcel Overdijk, Christian Dupuis, Sébastien Deleuze, Michael Simons, VedranPavić, Jay Bryant, Madhura Bhave

# Part I. Spring Boot Documentation

This section provides a brief overview of Spring Boot reference documentation. It serves as a map for the rest of the document.

`brief `:简要的

`the rest of` : 其余的

翻译： 

这部分我们提供关于`Spring boot`的简要参考文档，它用作文档其余部分的映射(索引)。**注意这一部分主要就是做索引使用的**

## 1. About the Documentation

The Spring Boot reference guide is available as

- [HTML](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/html)
- [PDF](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/pdf/spring-boot-reference.pdf)
- [EPUB](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/epub/spring-boot-reference.epub)

The latest copy is available at [docs.spring.io/spring-boot/docs/current/reference](https://docs.spring.io/spring-boot/docs/current/reference).

`available `: 能找到的，可获得的

`available for `可供....使用的

Copies of this document may be made for your own use and for distribution to others, provided that you do not charge any fee for such copies and further provided that each copy contains this Copyright Notice, whether distributed in print or electronically.

`distribution` :  分配

翻译： 

`Spring boot `参考指导这里提供了下面几种可获取的文档

最新的副本可以在[docs.spring.io/spring-boot/docs/current/reference](https://docs.spring.io/spring-boot/docs/current/reference) 中获得

本文件的副本可以供您自己使用，也可以分配给他人使用，规定你不能为这些文件收取任何费用，并且进一步规定，每份副本都需要包含本版权通知，无论是以印刷形式分发还是以电子方式分发。

## 2. Getting Help

If you have trouble with Spring Boot, we would like to help.

- Try the [How-to documents](#IX. How-to guides). They provide solutions to the most common questions.
-  Learn the Spring basics. Spring Boot builds on many other Spring projects. Check the [spring.io](https://spring.io/) web-site for a wealth of reference documentation. If you are starting out with Spring, try one of the [guides](https://spring.io/guides).
- Ask a question. We monitor [stackoverflow.com](https://stackoverflow.com/) for questions tagged with [`spring-boot`](https://stackoverflow.com/tags/spring-boot).
- Report bugs with Spring Boot at [github.com/spring-projects/spring-boot/issues](https://github.com/spring-projects/spring-boot/issues).

`trouble`: 麻烦，问题，困扰

`Report `: 报告，指出，举报

翻译： 

如果你有任何关于spring boot 的问题，我们很愿意帮助到你，你可以通过下面这些方式，寻求帮助

* 你可以尝试查看 [How-to documents](#IX. How-to guides)，这里提供了大多数常见问题的解决方法
* `spring boot `建立在许多其他的spring 项目之上。如果你要学习spring 基础知识请查看[spring.io](https://spring.io/) 网站的大量参考文档。如果您刚开始使用Spring，不要瞎搞，请看看这个[指南](https://spring.io/guides)。
* 我们关注着`spring boot`在[stackoverflow.com](https://stackoverflow.com/) 的问题，（也就是说你可以在这个网站发布关于spring boot的问题）
* 你也可以在[github.com/spring-projects/spring-boot/issues](https://github.com/spring-projects/spring-boot/issues).提`Spring Boot`的`bug`。

## 3. First Steps

If you are getting started with Spring Boot or 'Spring' in general, start with [the following topics](#Part II. Getting Started):

- **From scratch:** [Overview](#8. Introducing Spring boot) | [Requirements](#9. System Requirements) | [Installation](#10. Installing  Spring Boot)
- **Tutorial:** [Part 1](#11. Developing Your First Spring Boot Application) | [Part 2](#11.3 Writing the Code)
- **Running your example:** [Part 1](#11.4 Running the Example) | [Part 2](11.5 Creating an Executable Jar)

翻译：

如果你开始使用`spring` 或 `Spring boot`,请从[以下主题](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#getting-started)开始：

* **从头开始** ： [概览](#8. Introducing Spring boot) | [必要条件](#9. System Requirements) |[安装](#10. Installing  Spring Boot)
* **教程** ：[第一部分](#11. Developing Your First Spring Boot Application) | [第二部分](#11.3 Writing the Code)
* **运行你得例子** ：[第一部分](#11.4 Running the Example) | [第二部分](#11.5 Creating an Executable Jar)

## 4. Working with Spring boot

Ready to actually start using Spring Boot? [We have you covered](#Part III. Using Spring Boot):

- **Build systems:** [Maven](#13.2 Maven)| [Gradle](#13.3 Gradle)) | [Ant](#13.4 Ant) | [Starters](#13.5 Starters)
- **Best practices:**[Code Structure](#14. Structuring Your Code) | [@Configuration](#15. Configuration Classes) | [@EnableAutoConfiguration](#16. Auto-configuration) | [Beans and Dependency Injection](#17. Spring Beans and Dependency Injection)
- **Running your code**[IDE](#19.1 Running from an IDE) | [Packaged](#19.2 Running as a Packaged Application) | [Maven](#19.3 Using the Maven Plugin) | [Gradle](#19.4 Using the Gradle Plugin)
- **Packaging your app:** [Production jars](#21. Packaging Your Application for Production)
- **Spring Boot CLI:**  [Using the CLI](# Part VII. Spring Boot CLI)

`actually `: 实际上，确实地

翻译： 

准备好正式开始使用`spring boot`了吗？[我们为您提供了](#Part III. Using Spring Boot)：

* **构建系统**：[Maven](#13.2 Maven)| [Gradle](#13.3 Gradle)) | [Ant](#13.4 Ant) | [Starters](#13.5 Starters)
* **最佳做法**：[Code Structure](#14. Structuring Your Code) | [@Configuration](#15. Configuration Classes) | [@EnableAutoConfiguration](#16. Auto-configuration) | [Beans and Dependency Injection](#17. Spring Beans and Dependency Injection)
* **运行代码**：[IDE](#19.1 Running from an IDE) | [Packaged](#19.2 Running as a Packaged Application) | [Maven](#19.3 Using the Maven Plugin) | [Gradle](#19.4 Using the Gradle Plugin)
* **打包应用**:[Production jars](#21. Packaging Your Application for Production)
* **Spring Boot CLI:** [Using the CLI](# Part VII. Spring Boot CLI)

## 5. Learning about Spring boot features

Need more details about Spring Boot’s core features? [The following content is for you](#Part IV. Spring Boot features):

- **Core Features:** [SpringApplication](#23. SpringApplication) | [External Configuration](#24. Externalized Configuration) | [Profiles](#25. Profiles) | [Logging](#26. Logging)
- **Web Applications:** [MVC](#28.1 The Spring Web MVC Framework) | [Embedded Containers](# 28.4 Embedded Servlet Container Support)
- **Working with data:**[SQL](#30. Working with SQL Databases) | [NO-SQL](#31. Working with NoSQL Technologies)
- **Messaging:** [Overview](#33. Messaging) | [JMS](#33.1 JMS)
- **Testing:** [Overview](#45. Testing) | [Boot Applications](#45.3 Testing Spring Boot Applications) | [Utils](#45.4 Test Utilities)
- **Extending:** [Auto-configuration](#49. Creating Your Own Auto-configuration) |[@Conditions](#49.3 Condition Annotations)

翻译：

你需要了解有关Spring Boot核心功能的更多详细信息吗？ [The following content is for you](#Part IV. Spring Boot features):

- **核心特性:** [SpringApplication](#23. SpringApplication) | [外部配置](#24. Externalized Configuration) | [Profiles](#25. Profiles) | [日志](#26. Logging)
- **Web 应用程序:** [MVC](#28.1 The Spring Web MVC Framework) | [Embedded Containers](# 28.4 Embedded Servlet Container Support)
- **数据使用+存储:** [SQL](#30. Working with SQL Databases) | [NO-SQL](#31. Working with NoSQL Technologies)
- **Messaging:** [Overview](#33. Messaging) | [JMS](#33.1 JMS)
- **Testing:** [Overview](#45. Testing) | [Boot Applications](#45.3 Testing Spring Boot Applications) | [Utils](#45.4 Test Utilities)
- **扩展:** [Auto-configuration](#49. Creating Your Own Auto-configuration) |[@Conditions](#49.3 Condition Annotations)

## 6. Moving to Production

When you are ready to push your Spring Boot application to production, we have [some tricks](#Part V. Spring Boot Actuator: Production-ready features) that you might like:

- **Management endpoints:** [Overview](#53. Endpoints) | [Customization](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#)
- **Connection options:** [HTTP](#54. Monitoring and Management over HTTP) | [JMX](#55. Monitoring and Management over JMX)
- **Monitoring:** [Metrics](#57. Metrics) | [Auditing](#58. Auditing) | [Tracing](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#) | [Process](#60. Process Monitoring)

翻译：

当你准备push你的spring boot 应用到生产环境的时候，我们有一些你可能喜欢的[技巧](#Part V. Spring Boot Actuator: Production-ready features)：

**上面那部分我们做开发一看就懂的我就不翻译了**

## 7. Advanced Topics

Finally, we have a few topics for more advanced users:

- **Spring Boot Applications Deployment:** [Cloud Deployment](#63. Deploying to the Cloud) | [OS Service](#64.2 Unix/Linux Services)
- **Build tool plugins:** [Maven](#71. Spring Boot Maven Plugin) | [Gradle](#72. Spring Boot Gradle Plugin)
- **Appendix:** [Application Properties](#Appendix A. Common application properties) | [Auto-configuration classes](#Appendix C. Auto-configuration classes) | [Executable Jars](#Appendix E. The Executable Jar Format)

翻译：

最后我们为少部分更高级（指的是有更高使用要求的用户）的用户提供了一部分主题：

# Part II. Getting Started

If you are getting started with Spring Boot, or “Spring” in general, start by reading this section. It answers the basic “what?”, “how?” and “why?” questions. It includes an introduction to Spring Boot, along with installation instructions. We then walk you through building your first Spring Boot application, discussing some core principles as we go.

翻译：

如果你开始使用`spring boot`或`spring`,请先阅读本节，这一节包括Spring Boot简介以及安装说明。 然后，我们将引导您构建您的第一个Spring Boot应用程序，并在我们讨论时讨论一些核心原则。

## 8. Introducing Spring boot

Spring Boot makes it easy to create stand-alone, production-grade Spring-based Applications that you can run. We take an opinionated view of the Spring platform and third-party libraries, so that you can get started with minimum fuss. Most Spring Boot applications need very little Spring configuration.

You can use Spring Boot to create Java applications that can be started by using `java -jar` or more traditional war deployments. We also provide a command line tool that runs “spring scripts”.

Our primary goals are:

- Provide a radically faster and widely accessible getting-started experience for all Spring development.
- Be opinionated out of the box but get out of the way quickly as requirements start to diverge from the defaults.
- Provide a range of non-functional features that are common to large classes of projects (such as embedded servers, security, metrics, health checks, and externalized configuration).
- Absolutely no code generation and no requirement for XML configuration.

`goal`:目标

翻译：

Spring Boot可以轻松创建可以运行的独立的，生产级的基于Spring的应用程序。 我们对Spring平台和第三方库基于我们的思想进行了一种包装，这样您就可以轻松上手了。 大多数Spring Boot应用程序只需要很少的Spring配置。

您可以使用Spring Boot创建可以使用`java -jar`或更传统的war部署启动的Java应用程序。 我们还提供了一个运行`“spring scripts ”`的命令行工具。

我们的主要目标是：

* 为所有使用Spring的开发开发者提供从根本上更快且可广泛访问的入门体验。
* 提供大型项目（例如嵌入式服务器，安全性，度量标准，运行状况检查和外部化配置）通用的一系列非功能性功能。
* 绝对没有代码生成，也不需要XML配置。

## 9. System Requirements

Spring Boot 2.1.1.RELEASE requires [Java 8](https://www.java.com/) and is compatible up to Java 11 (included). [Spring Framework 5.1.3.RELEASE](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/) or above is also required.

Explicit build support is provided for the following build tools:

| Build Tool | Version |
| ---------- | ------- |
| Maven      | 3.3+    |
| Gradle     | 4.4+    |

`Spring Boot 2.1.1.RELEASE`依赖java 8 （兼容java11），同时 [Spring Framework 5.1.3.RELEASE](https://docs.spring.io/spring/docs/5.1.3.RELEASE/spring-framework-reference/) 也是同样的依赖情况

以下构建工具提供了显式构建支持

| Build Tool | Version |
| ---------- | ------- |
| Maven      | 3.3+    |
| Gradle     | 4.4+    |

### 9.1 Servlets Containers

Spring Boot supports the following embedded servlet containers:

| Name         | Servlet Version |
| ------------ | --------------- |
| Tomcat 9.0   | 4.0             |
| Jetty 9.4    | 3.1             |
| Undertow 2.0 | 4.0             |

You can also deploy Spring Boot applications to any **Servlet 3.1+ compatible container**.

这里就不做翻译了

## 10. Installing  Spring Boot

Spring Boot can be used with “classic” Java development tools or installed as a command line tool. Either way, you need [Java SDK v1.8](https://www.java.com/) or higher. Before you begin, you should check your current Java installation by using the following command:

```
$ java -version
```

If you are new to Java development or if you want to experiment with Spring Boot, you might want to try the [Spring Boot CLI](#10.2.1 Manual Installation) (Command Line Interface) first. Otherwise, read on for “classic” installation instructions.

翻译：

Spring Boot可以与“经典”Java开发工具一起使用，也可以作为命令行工具安装。 无论哪种方式，您都需要Java SDK v1.8或更高版本。 在开始之前，您应该使用以下命令检查当前的Java安装：

```
$ java -version
```

### 10.1 Installation Instructions for the Java Developer

java开发人员安装说明

You can use Spring Boot in the same way as any standard Java library. To do so, include the appropriate `spring-boot-*.jar` files on your classpath. Spring Boot does not require any special tools integration, so you can use any IDE or text editor. Also, there is nothing special about a Spring Boot application, so you can run and debug a Spring Boot application as you would any other Java program.

Although you *could* copy Spring Boot jars, we generally recommend that you use a build tool that supports dependency management (such as Maven or Gradle).

翻译：

您可以像使用任何标准Java库一样使用Spring Boot。 为此，请在类路径中包含相应的`spring-boot-*.jar`文件。 Spring Boot不需要任何特殊工具集成，因此您可以使用任何IDE或文本编辑器。 此外，Spring Boot应用程序没有什么特别之处，因此您可以像运行任何其他Java程序一样运行和调试Spring Boot应用程序。

虽然您可以复制Spring Boot jar，但我们通常建议您使用支持依赖关系管理的构建工具（例如Maven或Gradle）。

#### 10.1.1 Maven Installation

Spring Boot is compatible with Apache Maven 3.3 or above. If you do not already have Maven installed, you can follow the instructions at [maven.apache.org](https://maven.apache.org/).

Spring Boot dependencies use the `org.springframework.boot` `groupId`. Typically, your Maven POM file inherits from the `spring-boot-starter-parent` project and declares dependencies to one or more [“Starters”](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#using-boot-starter). Spring Boot also provides an optional [Maven plugin](https://docs.spring.io/spring-boot/docs/2.1.1.RELEASE/reference/htmlsingle/#build-tool-plugins-maven-plugin) to create executable jars.

The following listing shows a typical `pom.xml` file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>myproject</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<!-- Inherit defaults from Spring Boot -->
    <!-- 继承默认的 Spring Boot parent -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.1.RELEASE</version>
	</parent>

	<!-- Add typical dependencies for a web application -->
    <!-- typical 典型的 -->
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>

	<!-- Package as an executable jar -->
    <!-- 打包为可执行 jar -->
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

#### 10.1.2 Gradle Installation

### 10.2 Installing the Spring Boot CLI

#### 10.2.1 Manual Installation

#### 10.2.2 Installation with SDKMAN!

#### 10.2.3 OSX Homebrew Installation

#### 10.2.4 MacPorts Installation

#### 10.2.5 Command-line Completion

#### 10.2.6 Windows Scoop Installation

#### 10.2.7 Quick-start Spring CLI Example

### 10.3 Upgrading from an Earlier Version of Spring Boot

`Upgrade ` 升级

`Earlier `:早期的

If you are upgrading from an earlier release of Spring Boot, check the [“migration guide” on the project wiki](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide) that provides detailed upgrade instructions. Check also the[“release notes”](https://github.com/spring-projects/spring-boot/wiki) for a list of “new and noteworthy” features for each release.

When upgrading to a new feature release, some properties may have been renamed or removed. Spring Boot provides a way to analyze your application’s environment and print diagnostics at startup, but also temporarily migrate properties at runtime for you. To enable that feature, add the following dependency to your project:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-properties-migrator</artifactId>
	<scope>runtime</scope>
</dependency>
```

翻译：

如果要从早期版本的Spring Boot升级，请查看项目Wiki上的“迁移指南”，其中提供了详细的升级说明。 另请查看“发行说明”，了解每个版本的“新的和值得注意的”功能列表。

升级到新功能版本时，某些属性可能已重命名或删除。 Spring Boot提供了一种在启动时分析应用程序环境和打印诊断的方法，还可以在运行时临时迁移属性。 要启用该功能，请将以下依赖项添加到项目中：

## 11. Developing Your First Spring Boot Application

This section describes how to develop a simple “Hello World!” web application that highlights some of Spring Boot’s key features. We use Maven to build this project, since most IDEs support it.

本节介绍如何开发一个简单的“Hello World！”Web应用程序，该应用程序突出了Spring Boot的一些主要功能。 我们使用Maven来构建这个项目，因为大多数IDE都支持它。

Before we begin, open a terminal and run the following commands to ensure that you have valid versions of Java and Maven installed:

```
$ java -version
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)
```

```
$ mvn -v
Apache Maven 3.5.4 (1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-17T14:33:14-04:00)
Maven home: /usr/local/Cellar/maven/3.3.9/libexec
Java version: 1.8.0_102, vendor: Oracle Corporation
```

### 11.1 Creating the POM



## 12. What to Read Next

# Part III. Using Spring Boot

## 13. Build Systems

## 14. Structuring Your Code

## 15. Configuration Classese

## 16. Auto-configuration

## 17. Spring Beans and Dependency Injection

## 18. Using The @SpringBootApplication Annotation

## 19. Running Your Application

## 20. Developer Tools

## 21. Packaging Your Application For Production

## 22. What to read next

# Part IV. Spring Boot features

## 23. SpringApplication

## 24. Externalized Configuration

## 25. Profiles

## 26. Logging

## 27. JSON

## 28. Developing Web Application

## 29. Security

## 30. Working with SQL Databases

## 31. Working with NoSQL Technologies

## 32. Caching 

## 33. Messaging

## 34. Calling REST Services with `RestTemplate`

## 35. Calling REST Services with `WebClient`

## 36. Validation

## 37. Sending Email

## 38. Distributed Transactions with JTA

## 39. Hazelcast

## 40. Quartz Schedule

## 41. Task Execution and Scheduling

## 42. Spring Integration

## 43. Spring Session

## 44. Monitoring and Management over JMX

## 45. Testing

## 46. WebSockets

## 47. Web Service

## 48. Calling Web Service with `webserviceTemplate`

## 49. Creating Your Own Auto-configration

## 50. Kotlin support

## 51. What to Read Next

# Part V. Spring Boot Actuator: Production-ready features

## 52. Enabling Production-ready features

## 53. Endpoints

## 54. Monitoring and Management over HTTP

## 55. Monitoring and Management over JMX

## 56. Loggers

## 57. Metrics

## 58. Auditing

## 59. HTTP Tracing

## 60. Process Monitoring

## 61. Cloud Foundry Support

## 62. What to Read Next

# Part VI. Deploying Spring Boot Applications

## 63. Deploying to the Cloud

## 64. Installing Spring Boot Application

## 65. What to Read Next

# Part VII. Spring Boot CLI

## 66. Installing the CLI

## 67. Using the CLI

## 68. Developing Applications with the Groovy Beans DSL

## 69. Configuring the CLI with `settings.xml`

## 70. What to Read Next

# Part VIII. Build tool plugins

## 71. Spring Boot maven Plugin

## 72. Spring Boot Gradle Plugin

## 73. Spring Boot AntLib Module

## 74. Supprting Other Build Systems

## 75. What to Read Next

# Part IX. How-to guides

## 76. Spring Boot Application

## 77. Properties and Configoration

## 78. Embedded Web Servers

## 79. Spring MVC

## 80. Testing With Spring Security 

## 81. jersey

## 82. HTTP Clients

## 83. Logging

## 84. Data Access

## 85. Database Initalization

## 86. Messaging

## 87. Batch Applications

## 88. Actuator

## 89. Security

## 90. Hot Swapping

## 91. Build

## 92. Traditional Deployment

# Part X. Appendices

## A. Common application properties

## B. Configuration Metadata

## C. Auto-configuration classes

## D. Test auto-configuration annotations

## E. The Executable Jar Format

## F. Dependency versions