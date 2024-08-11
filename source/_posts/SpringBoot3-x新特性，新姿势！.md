---
title: SpringBoot3.x新特性，新姿势！
date: 2024-08-11 11:58:12
tags: SpringBoot3.x新特性
categories: 教程
---



搞技术这么多年，说实话，我见过不少吹牛的，什么“这技术难度很高”、“你这活儿做不了”。春秋大梦！**哥今天就给你们来点干货，告诉你SpringBoot 3.x是怎么回事**。

别光看着流口水，拿好小本本，记好了！

#### 1\. 快速入门

先整明白Spring Boot 3.x咋用，这不难，就像初学骑自行车。你先了解下基础配置和依赖，写个简单的“Hello World”出来。就是在`application.properties`里配置几个关键参数，启动项目，你就能在浏览器里看到“Hello Spring Boot 3.x”了。

#### 2\. 最佳实践

要搞出高大上的东西，得有些规矩和套路。记住几个要点：模块化开发，配置文件分环境管理，使用注解减少样板代码。这样干，项目可维护性高，遇到锅甩起来也方便。

#### 3\. 构建系统&Starters

Starters就是一套速效救心丸，帮你快速集成各种功能。想用数据库？引入`spring-boot-starter-data-jpa`。要整Web项目？那就`spring-boot-starter-web`。这些玩意儿让你少写不少配置，省下时间去摸鱼。

#### 4\. DevTools(IDEA 热部署&远程调试&LiveReload)

IDEA热部署简直是神器，修改代码不重启服务器，效率杠杠的。远程调试也好使，能直接在服务器上断点调试。LiveReload能让前端页面实时刷新，前端小哥们可以边写边看效果，爱不释手。

#### 5\. 特性-Spring Application

Spring Boot的核心就是Spring Application，启动时各种组件自动装配，配置文件自动读取。你就当它是个大总管，管理着项目的方方面面，省心省力。

#### 6\. 特性-自定义FailureAnalyzer

出了问题怎么办？自定义FailureAnalyzer能帮你定位问题原因，还能自定义报错信息。相当于项目里的“老中医”，哪里不对劲，一把脉就知道。

#### 7\. 特性-事件与监听

事件监听机制就像项目里的“大喇叭”，谁有事喊一嗓子，相关组件都知道了。用起来也简单，定义事件、发布事件、处理事件，分分钟搞定。

#### 8\. 特性-配置与配置源

Spring Boot的配置文件支持多种格式，`properties`、`yaml`都行。多环境配置也很灵活，`application-dev.properties`、`application-prod.properties`，按环境加载。这样本地开发、线上部署互不干扰。

#### 9\. 特性-类型安全的配置属性

类型安全配置属性让你告别硬编码，把配置项变成强类型属性，写错了IDEA直接红线警告，妥妥的！

#### 10\. 特性-Profiles&多环境配置

Profiles帮你管理不同环境的配置，开发、测试、生产环境各自独立。切换环境只要改下启动参数，项目能快速适应不同环境，部署起来爽到飞起。

#### 11\. 特性-配置元数据

配置元数据能让IDEA智能提示配置项，写配置文件时有提示，再也不用翻文档了。幸福感爆棚！

#### 12\. 特性-自动配置和自定义Starter

Spring Boot的自动配置机制让你少写配置，自定义Starter更是让你可以根据项目需要，封装自己的依赖和配置，一次配置，多次使用，简直不要太爽。

#### 13\. 特性-日志

日志是项目的眼睛，Spring Boot默认用Logback，你可以根据需要切换到Log4j2。配置文件里写几行配置，日志级别、输出格式随心所欲。

#### 14\. 特性-国际化

项目要走向国际，国际化必须得搞。Spring Boot支持国际化配置，根据客户端语言自动切换，外语系的妹子都能看懂你的项目了。

#### 15\. 特性-JSON(gson, jackson, json-b, fastjson)

JSON处理用Jackson最顺手，配置简单，性能也不错。需要更高性能可以考虑Fastjson，但要注意安全问题，别把裤子都赔了。

#### 16\. Servlet Web应用程序开发(Spring MVC)

Spring MVC是Web开发的老朋友，Spring Boot把它集成得很好，写个Controller，配置好路由，前后端就能愉快地玩耍了。

#### 17\. Servlet Web应用程序开发(嵌入式容器)

Spring Boot默认用Tomcat，也支持Jetty、Undertow，配置文件里改下配置就行。嵌入式容器让你开发测试更方便，不用再去手动部署容器了。

#### 18\. SQL数据源配置

配置数据库连接很简单，`application.properties`里写上数据库连接信息就行。HikariCP是默认连接池，性能好，配置也简单。

#### 19\. SQL数据连接池(HikariCP, Tomcat pool, DBCP2, Druid)

连接池选哪个好？HikariCP性能最佳，Tomcat pool稳健，DBCP2老牌，Druid功能丰富。根据项目需求选吧，反正都挺好用。

#### 20\. JdbcTemplate详解

JdbcTemplate简化了JDBC操作，查询、更新、删除样样都行。你只要写SQL，别的交给JdbcTemplate，省时省力。

#### 21-23. Spring Data JPA详解

Spring Data JPA让你不写SQL也能操作数据库，接口+注解搞定CRUD。复杂查询用方法名定义，或者写`@Query`注解。数据源配置、初始化都简单，数据库搞错了改配置就行。

#### 24-25. Spring Data JDBC详解

Spring Data JDBC是Spring Data JPA的轻量级替代，性能更高，配置也简单。接口+注解风格一致，用起来很顺手。

#### 26-27. 多数据源配置

多数据源配置用`AbstractRoutingDataSource`，可以根据条件动态切换数据源。分包模式也很常用，每个数据源一个配置文件，互不干扰。

#### 28-29. 构建RESTful API

Spring Boot构建RESTful API简直是小菜一碟，写个Controller，注解`@RequestMapping`，前后端就能愉快互动了。集成Swagger-UI还能自动生成API文档，让前端小哥写起接口文档来也轻松。

#### 30\. MybatisPlus集成

MybatisPlus让你用Mybatis更爽，简化了CRUD操作，支持多数据源，性能也不错。整合Spring Boot后，配置文件里写几行配置，就能用上强大的MybatisPlus了。
