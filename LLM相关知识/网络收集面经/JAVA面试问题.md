# SPringBoot
1. 什么是 Spring Boot？它的核心优势是什么？
Spring Boot 是 Spring 生态的“脚手架”，基于 Spring 框架，通过自动配置、约定优于配置（Convention Over Configuration） 简化 Spring 应用的搭建与开发，核心目标是“让开发人员快速启动一个独立、生产级别的 Spring 应用”。

核心优势：

简化配置：无需手动编写大量 XML 配置，通过 application.properties/yaml 或注解即可完成配置。
自动配置：根据依赖的 Jar 包（如引入 spring-boot-starter-web 则自动配置 Tomcat、Spring MVC），自动初始化相关 Bean 到 Spring 容器。
嵌入式容器：内置 Tomcat、Jetty、Undertow 等 Servlet 容器，无需单独部署 WAR 包，直接运行 Jar 包即可。
starters 依赖：提供标准化的 starter 依赖（如 spring-boot-starter-data-jpa），一键引入某功能所需的所有依赖，避免依赖冲突。
生产级特性：内置健康检查（Actuator）、 metrics 监控、外部化配置等，简化生产环境部署与维护。
无代码生成/XML 配置：基于注解驱动，无需手动生成代码或编写 XML。

2. Spring Boot 与 Spring、Spring MVC 的关系？
三者是“包含与扩展”的关系，核心目标都是简化 Java 开发，但定位不同：

Spring：核心是“依赖注入（DI）”和“面向切面（AOP）”，是整个生态的基础，负责管理 Bean 的生命周期、解耦组件。
Spring MVC：是 Spring 生态的“Web 模块”，基于 MVC 架构（Model-View-Controller），负责处理 HTTP 请求、路由分发、视图渲染等 Web 开发功能。
Spring Boot：不是替换 Spring/Spring MVC，而是整合并简化两者的使用——通过自动配置将 Spring MVC 的 DispatcherServlet、Tomcat 等组件自动初始化，开发者无需手动配置即可快速开发 Web 应用。
简单总结：Spring Boot = Spring + Spring MVC + 自动配置 + 嵌入式容器 + Starter 机制。




# Mybatis-plus



# referemce
1. https://blog.csdn.net/weixin_63490470/article/details/152333303
2. https://zhuanlan.zhihu.com/p/683114744
3. https://www.nowcoder.com/discuss/353158541458481152
4. https://topjavaer.cn/java/java-basic.html