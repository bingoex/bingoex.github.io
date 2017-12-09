---
layout: post
title: 浅析Spring Boot
categories: Java
description: 
keywords: 
---

本文涉及spring boot出现的背景及好处，最好会简单涉及源码和原理相关内容。

# 纯spring的问题
1. 需要做大量的配置：声明大量的bean，并把他们组装到一起
2. 需要解决很多依赖版本冲突的问题。特别是当应用依赖到很多组件的时候（Spring MVC, Mysql, Redis等），要解决这些依赖，需要花很多的功夫手工去配置，解决jar包版本冲突。
3. 如果是web应用，则额外需要一个servlet容器将程序跑起来。

# Spring Boot简介
Spring Boot在Spring4.0基础上长出来，Boot是引导的意思，意为帮助开发者快速搭建Spring应用

# Spring Boot的好处
- 让 编码 变得更简单：更多Annotation让编码更简洁，纯Java Config告别眼花缭乱的XML。
- 让 测试 变得更简单：支持大多数流行测试框架，本地就能起应用，做集成测试都相当快。
- 让 配置 变得更简单：最大化的自动配置，开箱即用，零配置情况下就可以启动应用（功能依赖，而不是jar包依赖）。
- 让 监控 变得更简单：actuator运行中的应用健康检查（支持http/ssh/telnet），中间件/线程/JVM等运行状态一览无余。
- 让 部署 变得更简单：内置Servlet容器（Tomcat/Jetty/Undertow），无需打WAR包（Fat Jar）， 本地一键启动部署DUBUG。


# Demo
```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```


# 源码流程

## 1、创建SpringApplication实例
- SpringApplication设置sources属性
- 推断断是否是web程序并设置到webEnvironment属性中
- 找出并设置所有的初始化器（ApplicationContextInitializer）
- 找出并设置所有的监听器（ApplicationListener）
- 找出并设置运行的main主类

## 2、调用SpringApplication实例run方法

- 加载SpringApplicationRunListeners，为后续事件机制做准备
```XML 
#Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener
```
- 配置应用环境信息（environment）

- 创建spring容器（AnnotationConfigEmbeddedWebApplicationContext或AnnotationConfigApplicationContext）

- 容器刷新的前置动作，执行初始化器

- spring容器刷新

- Servlet容器的启动

- 容器刷新的后置动作，执行runner

## 3、通过注解自动初始化功能模块（详情请看原理）



# 原理
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class))
public @interface SpringBootApplication {
    ...
}
```

@SpringBootApplication注解是一个组合注解,它主要包含@SpringBootConfiguration和@EnableAutoConfiguration（spring 3出现）。

@EnableAutoConfiguration注解，可以让springboot利用SPI的机制找到相应的类然后进行自动化配置的，会扫描class path, 根据class path当中的类来判断当前应用依赖于哪些组件，进而自动地创建这些组件所需的Bean。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({ EnableAutoConfigurationImportSelector.class,
    AutoConfigurationPackages.Registrar.class })
public @interface EnableAutoConfiguration {

    /**
    * Exclude specific auto-configuration classes such that they will never be applied.
    */
    Class<?>[] exclude() default {};
}
```

@EnableAutoConfiguration中利用@import引入了一个核心类EnableAutoConfigurationImportSelector，这个类会读取所有classpath下面的META-INF/spring.factories中key为org.springframework.boot.autoconfigure.EnableAutoConfiguration的值，这些值都是springboot中自动配置的相关类，也就是实现starter相关的自动配置的核心部分。

```XML
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer


# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.data.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.DeviceResolverAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.reactor.ReactorAutoConfiguration,\
org.springframework.boot.autoconfigure.security.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.FallbackWebSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.web.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketAutoConfiguration
```

在这些类中每个Configuation都定义了相关bean的实例化配置。都说明了哪些bean可以被自动配置，什么条件下可以自动配置，并把这些bean实例化出来。
```java
@Configuration
@ConditionalOnClass(Mongo.class)
@EnableConfigurationProperties(MongoProperties.class)
    public class MongoAutoConfiguration {

        @Autowired
        private MongoProperties properties;

        private Mongo mongo;

        @PreDestroy
        public void close() throws UnknownHostException {
            if (this.mongo != null) {
                this.mongo.close();
            }
        }

        @Bean
        @ConditionalOnMissingBean
        public Mongo mongo() throws UnknownHostException {
            this.mongo = this.properties.createMongoClient();
            return this.mongo;
        }

}
```

[更多spring注解请看](http://blog.csdn.net/b2222505/article/details/78613353)








