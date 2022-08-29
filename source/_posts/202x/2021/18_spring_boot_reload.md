---
title: Springboot热部署
date: 2021-08-26 13:40:00
categories:
  - Java
tags:
  - Java
  - Springboot
  - Maven
---

通过设置实现 Springboot 项目在修改代码的时候，不进行冷启动，直接使用 springboot 提供的热部署功能

<!-- more -->

### 1. Maven 项目添加 pom 依赖

```
<!--SpringBoot热部署配置 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>

<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <fork>true</fork>
    </configuration>
</plugin>
```

热部署生效后，日志中的线程名称是[restartedMain]，会加载两次，但比冷启动的时间要短很多

### 2. 修改 IDEA 设置

如果想要自动编译，需要在 Idea 上修改如下两个配置：

- Settings -> Build -> Compiler 中勾选 Build project automatically 选项
- Settings -> Advanced Settings 中勾选 Allow auto-make to start even if developed application is currently running 选项
  > 注：在修改代码后，需要稍微等待一会儿，才会生效

如果不想要每次修改代码后都自动进行加载，可以进行手动编译：

- Build -> Build Project
