---
layout:     post
title:      Spring Boot 之 logback 配置
subtitle:   
author:     WeYunx
header-style: text
catalog: true
tags:
    - Java
    - Spring
    - Spring boot
    - Logback

---

LogBack 默认集成在 Spring Boot 中，是基于 `Slf4j` 的日志框架。默认情况下 Spring Boot 是以 INFO 级别输出到控制台。

它的日志级别是：

> ALL < TRACE < DEBUG < INFO < WARN < ERROR < OFF

##  配置

LogBack 可以直接在 `application.properties` 或 `application.yml` 中配置，但仅支持一些简单的配置，复杂的文件输出还是需要配置在 `xml` 配置文件中。配置文件可命名为 `logback.xml` ， LogBack 自动会在 `classpath` 的根目录下搜索配置文件，不过 Spring Boot 建议命名为 `logback-spring.xml`，这样会自动引入 Spring Boot 一些扩展功能。

如果需要引入自定义名称的配置文件，需要在 Spring Boot 的配置文件中指定，如：

```yml
logging:
  config: classpath:logback-spring.xml
```

同时 Spring Boot 提供了一个默认的 `base.xml`  配置，可以按照如下方式引入：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
	<include resource="org/springframework/boot/logging/logback/base.xml"/>
</configuration>
```

`base.xml` 提供了一些基本的默认配置以及在控制台输出时的关键字配色，具体文件内容可以看[这里](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/base.xml)，可以查看到一些常用的配置写法。

### 详细配置

#### 变量

可以使用 `<property>` 来定义变量：

```xml
<property name="log.path" value="/var/logs/application" />
```

同时可以引入 Spring 的环境变量：

```xml
<property resource="application.yml" />
<property resource="application.properties" />
```

所有的变量都可以通过 `${}` 来调用。

#### 输出到控制台

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%.-1level|%-40.40logger{0}|%msg%n</pattern>
    </encoder>
  </appender>
 
  <logger name="com.mycompany.myapp" level="debug" />
  <logger name="org.springframework" level="info" />
  <logger name="org.springframework.beans" level="debug" />
 
  <root level="warn">
    <appender-ref ref="console" />
  </root>
</configuration>
```



#### 输出到文件

```xml
<property name="LOG_FILE" value="LogFile" />
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${LOG_FILE}.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!-- 每日归档日志文件 -->
        <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.gz</fileNamePattern>
        <!-- 保留 30 天的归档日志文件 -->
        <maxHistory>30</maxHistory>
        <!-- 日志文件上限 3G，超过后会删除旧的归档日志文件 -->
        <totalSizeCap>3GB</totalSizeCap>
    </rollingPolicy>
    <encoder>
        <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
</appender> 
```



#### 多环境配置

LogBack 同样支持多环境配置，如  `dev` 、 `test` 、 `prod`

```xml
<springProfile name="dev">
    <logger name="com.mycompany.myapp" level="debug"/>
</springProfile>
```

启动的时候 `java -jar xxx.jar --spring.profiles.active=dev` 即可使配置生效。

如果要使用 Spring 扩展的 profile 支持，配置文件名必须命名为 `LogBack_Spring.xml`，此时当 `application.properties` 中指定为 `spring.profiles.active=dev` 时，上述配置才会生效。







*未完待续...*