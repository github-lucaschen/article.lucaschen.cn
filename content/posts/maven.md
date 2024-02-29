---
author: Lucas Chen
title: "Maven"
date: 2023-01-06
description: >-
  Maven 相关知识总结。
tags:
  - reprint
  - unknown
  - java
  - maven
categories:
  - java
  - en-US
series:
  - maven
---

Maven 相关知识总结。

包含以下部分：

1. 配置
2. 核心概念
3. POM
4. 命令行操作
5. 依赖
6. 继承

<!--more-->


## 1. 配置

### 1.1 配置本地目录

```xml
<localRepository>~\Documents\Maven\apache-maven-3.6.3\repository</localRepository>
```

### 1.2 配置远程仓库

```xml
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*</mirrorOf>
  <name>Plulic</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

### 1.3 配置 JDK 版本

```xml
<profile>
  <id>jdk1.8</id>
  <activation>
    <activeByDefault>true</activeByDefault>
    <jdk>1.8</jdk>
  </activation>
  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
  </properties>
</profile>
```

## 2. Maven 核心概念

### 2.1 GAV

- groupId：公司或组织的 id
- artifactId：一个项目或者是项目中的一个模块的 id
- version：版本号

### 2.2 POM

1. 含义
   POM：**P**roject **O**bject **M**odel，项目对象模型。和 POM 类似的是：DOM（Document Object Model），文档对象模型。它们都是模型化思想的具体体现。

2. 模型化思想
   POM 表示将工程抽象为一个模型，再用程序中的对象来描述这个模型。这样我们就可以用程序来管理项目了。我们在开发过程中，最基本的做法就是将现实生活中的事物抽象为模型，然后封装模型相关的数据作为一个对象，这样就可以在程序中计算与现实事物相关的数据。

3. 对应的配置文件
   POM 理念集中体现在 Maven 工程根目录下 pom.xml 这个配置文件中。所以这个 pom.xml 配置文件就是 Maven 工程的核心配置文件。其实学习 Maven 就是学这个文件怎么配置，各个配置有什么用

### 2.3 目录结构

- 目录的作用

  ```txt
  project_name ---> 项目名
    | src ---> 源码目录
    | | main ---> 主程序目录
    | | | java ---> Java 源码目录
    | | | | com.xx.xx ---> Package 目录
    | | | resource ---> 配置文件
    | | test ---> 测试程序目录
    | | | java ---> Java 源码目录
    | | | | com.xx.xx ---> Package 目录
    | | | resource ---> 配置文件
    | pom.xml 配置文件位置
  ```

- 目录结构的意义
  Maven 为了让构建过程能够尽可能自动化完成，所以必须约定目录结构的作用。例如：Maven 执行编译操作，必须先去 Java 源程序目录读取 Java 源代码，然后执行编译，最后把编译结果存放在 target 目录。
- 约定大于配置
  Maven 对于目录结构这个问题，没有采用配置的方式，而是基于约定。这样会让我们在开发过程中非常方便。如果每次创建 Maven 工程后，还需要针对各个目录的位置进行详细的配置，那肯定非常麻烦。
  目前开发领域的技术发展趋势就是：约定大于配置，配置大于编码。

## 3. POM

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <!-- 当前 Maven 工程的坐标 -->
  <groupId>init.lucaschen.maven</groupId>
  <artifactId>maven-java</artifactId>
  <version>1.0-SNAPSHOT</version>


  <!-- 当前 Maven 工程的打包方式，可选值有下面三种： -->
  <!-- jar：表示这个工程是一个Java工程  -->
  <!-- war：表示这个工程是一个Web工程 -->
  <!-- pom：表示这个工程是“管理其他工程”的工程 -->
  <packaging>jar</packaging>

  <name>maven-java</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <!-- 工程构建过程中读取源码时使用的字符集 -->
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
  </properties>

  <!-- 当前工程所依赖的信息 -->
  <dependencies>
    <!-- 使用 dependency 配置一个具体的依赖 -->
    <dependency>
      <!-- 在 dependency 标签内使用具体的坐标依赖我们需要的一个jar包 -->
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <!-- scope 标签配置依赖的范围 -->
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>init.lucaschen.maven</groupId>
      <artifactId>maven-java</artifactId>
      <version>1.0-SNAPSHOT</version>
      <scope>compile</scope>
      <!-- 使用excludes标签配置依赖的排除  -->
      <exclusions>
        <!-- 在exclude标签中配置一个具体的排除 -->
        <exclusion>
          <!-- 指定要排除的依赖的坐标（不需要写version） -->
          <groupId>commons-logging</groupId>
          <artifactId>commons-logging</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
  </dependencies>
```

## 4. 命令行操作

### 4.1 创建工程

```shell
mvn   archetype:generate
主命令 子命令
      插件:目标
```

### 4.2 清理操作

```shell
mvn clean
```

效果： 删除 target 目录

### 4.3 编译

```shell
mvn compile
```

主程序编译

```shell
mvn test-compile
```

测试程序编译：

主体程序编译结果存放的目录：target/classes

测试程序编译结果存放的目录：target/test-classes

### 4.4 测试操作

```shell
mvn test
```

测试的报告存放的目录：target/surefire-reports

### 4.5 打包操作

```shell
mvn package
```

打包的结果——jar 包，存放的目录：target

### 4.6 安装操作

```shell
mvn install
```

安装的效果是将本地构建过程中生成的 jar 包存入 Maven 本地仓库。这个 jar 包在 Maven 仓库中的路径是根据它的坐标生成的。

另外，安装操作还会将 pom.xml 文件转换为 XXX.pom 文件一起存入本地仓库。所以我们在 Maven 的本地仓库中想看一个 jar 包原始的 pom.xml 文件时，查看对应 XXX.pom 文件即可，它们是名字发生了改变，本质上是同一个文件。

### 4.7 查看当前工程所依赖包

列表形式

```shell
mvn dependency:list
```

树形式

```shell
mvn dependency:tree
```

## 5. 依赖

### 5.1 依赖范围

标签的位置：`dependencies -> dependency -> scope`
标签的可选值：`compile | test | provided | system | runtime | import`

|          | main 目录（空间） | test 目录（空间） | 开发过程（时间） | 部署到服务器（时间） |
| -------- | ----------------- | ----------------- | ---------------- | -------------------- |
| compile  | T                 | T                 | T                | T                    |
| test     | **F**             | T                 | T                | **F**                |
| provided | T                 | T                 | T                | **F**                |

- compile：通常使用的第三方框架的 jar 包这样在项目实际运行时真正要用到的 jar 包都是以 compile 范围进行依赖的。比如 SSM 框架所需 jar 包。

- test：测试过程中使用的 jar 包，以 test 范围依赖进来。比如 junit。

- provided：在开发过程中需要用到的“服务器上的 jar 包”通常以 provided 范围依赖进来。比如 servlet-api、jsp-api。而这个范围的 jar 包之所以不参与部署、不放进 war 包，就是避免和服务器上已有的同类 jar 包产生冲突，同时减轻服务器的负担。说白了就是：“**服务器上已经有了，你就别带啦！**”

### 5.2 依赖传递

A 依赖 B，B 依赖 C，那么在 A 没有配置对 C 的依赖的情况下，A 里面能不能直接使用 C？

在 A 依赖 B，B 依赖 C 的前提下，C 是否能够传递到 A，取决于 B 依赖 C 时使用的依赖范围。

- B 依赖 C 时使用 compile 范围：可以传递
- B 依赖 C 时使用 test 或 provided 范围：不能传递，所以需要这样的 jar 包时，就必须在需要的地方明确配置依赖才可以。

### 5.3 依赖排除

当 A 依赖 B，B 依赖 C 而且 C 可以传递到 A 的时候，A 不想要 C，需要在 A 里面把 C 排除掉。而往往这种情况都是为了避免 jar 包之间的冲突。

所以配置依赖的排除其实就是阻止某些 jar 包的传递。因为这样的 jar 包传递过来会和其他 jar 包冲突。

```xml
<dependency>
  <groupId>init.lucaschen.maven</groupId>
  <artifactId>maven-java</artifactId>
  <version>1.0-SNAPSHOT</version>
  <scope>compile</scope>
  <!-- 使用excludes标签配置依赖的排除  -->
  <exclusions>
    <!-- 在exclude标签中配置一个具体的排除 -->
    <exclusion>
      <!-- 指定要排除的依赖的坐标（不需要写version） -->
      <groupId>commons-logging</groupId>
      <artifactId>commons-logging</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

## 6. 继承

Maven 工程之间，A 工程继承 B 工程

- B 工程：父工程
- A 工程：子工程

本质上是 A 工程的 pom.xml 中的配置继承了 B 工程中 pom.xml 的配置。

### 6.1 作用

在父工程中统一管理项目中的依赖信息，具体来说是管理依赖信息的版本。

它的背景是：

- 对一个比较大型的项目进行了模块拆分。
- 一个 project 下面，创建了很多个 module。
- 每一个 module 都需要配置自己的依赖信息。

它背后的需求是：

- 在每一个 module 中各自维护各自的依赖信息很容易发生出入，不易统一管理。
- 使用同一个框架内的不同 jar 包，它们应该是同一个版本，所以整个项目中使用的框架版本需要统一。
- 使用框架时所需要的 jar 包组合（或者说依赖信息组合）需要经过长期摸索和反复调试，最终确定一个可用组合。这个耗费很大精力总结出来的方案不应该在新的项目中重新摸索。

通过在父工程中为整个项目维护依赖信息的组合既**保证了整个项目使用规范、准确的 jar 包**；又能够**将以往的经验沉淀下来**，节约时间和精力。

### 6.2 父工程 POM

```xml
<!-- 当前工程作为父工程，它要去管理子工程，所以打包方式必须是 pom -->
<!-- 只有打包方式为 pom 的 Maven 工程能够管理其他 Maven 工程。打包方式为 pom 的 Maven 工程中不写业务代码，它是专门管理其他 Maven 工程的工程。-->
<packaging>pom</packaging>

<modules>
  <module>maven-module-01</module>
  <module>maven-module-02</module>
  <module>maven-module-03</module>
</modules>

<!-- 通过自定义属性，统一指定 Spring 的版本 -->
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

  <!-- 自定义标签，维护 Spring 版本数据 -->
  <spring.version>4.3.6.RELEASE</spring.version>
</properties>

<!-- 使用 dependencyManagement 标签配置对依赖的管理 -->
<!-- 被管理的依赖并没有真正被引入到工程 -->
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-expression</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>${spring.version}</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

### 6.3 子工程 POM

```xml
<!-- 使用parent标签指定当前工程的父工程 -->
<parent>
  <!-- 父工程的坐标 -->
  <groupId>init.lucas.maven</groupId>
  <artifactId>maven-parent</artifactId>
  <version>1.0-SNAPSHOT</version>
</parent>

<!-- 子工程的坐标 -->
<!-- 如果子工程坐标中的groupId和version与父工程一致，那么可以省略 -->
<!-- <groupId>init.lucaschen.maven</groupId> -->
<artifactId>maven-module-01</artifactId>
<!-- <version>1.0-SNAPSHOT</version> -->


<!-- 子工程引用父工程中的依赖信息时，可以把版本号去掉。  -->
<!-- 把版本号去掉就表示子工程中这个依赖的版本由父工程决定。 -->
<!-- 具体来说是由父工程的dependencyManagement来决定。 -->
<dependencies>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-expression</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
  </dependency>
</dependencies>

```
