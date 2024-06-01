---
title: 'Logback 如何配置'
description: 'logback-configuration-file'
keywords: 'logback'

date: 2024-03-08T15:11:57+08:00

categories:
  - Java
tags:
  - Logback
---

给出基础的示例，讲述如何配置 SpringBoot 默认日志实现 Logback 应该如何进行配置文件的配置

<!--more-->

## 文件位置

`/resources/logback-spring.xml`

## 配置内容

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration scan="true">
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

    <springProperty scope="context" name="log.path" source="logging.file.path"
                    defaultValue="./logs"/>
    <springProperty scope="context" name="spring.application.name"
                    source="spring.application.name"/>
    <springProperty scope="context" name="spring.profiles.active" source="spring.profiles.active"/>
    <springProperty scope="context" name="log.level.console" source="logging.level.console"
                    defaultValue="INFO"/>
    <property name="FILE" value="${log.path}/${spring.application.name}"/>

    <!-- 控制台实时输出，采用高亮语法，用于开发环境 -->
    <appender name="CONSOLE_APPENDER" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>·
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset class="java.nio.charset.Charset">UTF·-8</charset>
        </encoder>
    </appender>


    <!-- 整个项目的所有日志 -->
    <appender name="ROOT_APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${FILE}/info.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- 每天一归档 -->·
            <fileNamePattern>
                ${log.path}/${spring.application.name}/%d{yyyy-MM}/root-%d{yyyy-MM-dd}-%i.log
            </fileNamePattern>
            <!-- 单个日志文件最多 100MB, 60 天的日志周期，最大不能超过 20GB -->
            <maxFileSize>128MB</maxFileSize>
            <maxHistory>60</maxHistory>
            <totalSizeCap>20GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset class="java.nio.charset.Charset">UTF-8</charset>
        </encoder>
    </appender>

    <!-- 共用异常包的日志 -->
    <appender name="ERROR_APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${FILE}/error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>
                ${log.path}/${spring.application.name}/%d{yyyy-MM}/error-%d{yyyy-MM-dd}-%i.log
            </fileNamePattern>
            <maxFileSize>128MB</maxFileSize>
            <maxHistory>60</maxHistory>
            <totalSizeCap>20GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset class="java.nio.charset.Charset">UTF-8</charset>
        </encoder>
        <!-- 此日志文档只记录 ERROR 级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- uat,prod -->
    <springProfile name="=uat,prod">
        <root level="${log.level.console}">
            <appender-ref ref="ROOT_APPENDER"/>
            <appender-ref ref="ERROR_APPENDER"/>
        </root>
    </springProfile>

    <!-- dev,sit -->
    <springProfile name="dev,sit">
        <root level="${log.level.console}">
            <appender-ref ref="CONSOLE_APPENDER"/>
            <appender-ref ref="ROOT_APPENDER"/>
            <appender-ref ref="ERROR_APPENDER"/>
        </root>
    </springProfile>

</configuration>
```
