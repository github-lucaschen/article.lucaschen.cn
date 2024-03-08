---
title: 'Logback Configuration File'
description: 'logback-configuration-file'
keywords: 'logback,configuration,file'

date: 2024-03-08T15:11:57+08:00

categories:
  - logback
tags:
  - logback
  - configuration
---

logback configuration file

<!--more-->

## file location

`/resources/logback-spring.xml`

## file content

> **You should change APP_NAME property to your project name.**

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>

    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

    <property name="APP_NAME" value="app-name"/>
    <property name="LOG_PATH" value="logs"/>
    <property name="LOG_FILE" value="${LOG_PATH}/${APP_NAME}.log"/>
    <property name="WARN_LOG_FILE" value="${LOG_PATH}/${APP_NAME}.warn.log"/>
    <property name="ERROR_LOG_FILE" value="${LOG_PATH}/${APP_NAME}.error.log"/>

    <appender name="ROLLING_FILE"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}</file>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset class="java.nio.charset.Charset">UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.%i</fileNamePattern>
            <maxHistory>7</maxHistory>
            <maxFileSize>50MB</maxFileSize>
            <totalSizeCap>500MB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <appender name="WARN_ROLLING_FILE"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${WARN_LOG_FILE}</file>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset class="java.nio.charset.Charset">UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${WARN_LOG_FILE}.%d{yyyy-MM-dd}.%i</fileNamePattern>
            <maxHistory>7</maxHistory>
            <maxFileSize>50MB</maxFileSize>
            <totalSizeCap>500MB</totalSizeCap>
        </rollingPolicy>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>WARN</level>
        </filter>
    </appender>

    <appender name="ERROR_ROLLING_FILE"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${ERROR_LOG_FILE}</file>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset class="java.nio.charset.Charset">UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${ERROR_LOG_FILE}.%d{yyyy-MM-dd}.%i</fileNamePattern>
            <maxHistory>7</maxHistory>
            <maxFileSize>50MB</maxFileSize>
            <totalSizeCap>500MB</totalSizeCap>
        </rollingPolicy>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
    </appender>

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset class="java.nio.charset.Charset">UTF-8</charset>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="ROLLING_FILE"/>
        <appender-ref ref="WARN_ROLLING_FILE"/>
        <appender-ref ref="ERROR_ROLLING_FILE"/>
    </root>
</configuration>
```
