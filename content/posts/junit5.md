---
author: Lucas Chen
title: 'Junit5 使用手册'
date: 2023-01-09
description: >-
  Junit5 使用手册
tags:
  - reprint
  - java
  - junit5
categories:
  - java
  - zh-CN
series:
  - junit5
---

Junit5 简单使用手册

<!--more-->

> 作者：Yudoge\
> 出处：<https://www.cnblogs.com/lilpig/p/15386258.html>\
> 遵循 [CC BY-NC-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接及本声明。\
> 本文对其内容和格式进行了整理。

## 概述

Junit5 的框架主要有三个部分组成分别是：JUnit Platform + JUnit Jupiter + JUnit Vintage3

1. JUnit Platform
   其主要作用是在 JVM 上启动测试框架。它定义了一个抽象的 TestEngine API 来定义运行在平台上的测试框架；也就是说其他的自动化测试引擎或开发人员⾃⼰定制的引擎都可以接入 Junit 实现对接和执行。同时还支持通过命令行、Gradle 和 Maven 来运行平台（这对于我们做自动化测试至关重要）
2. JUnit Jupiter
   这是 Junit5 的核心，可以看作是承载 Junit4 原有功能的演进，包含了 JUnit 5 最新的编程模型和扩展机制；很多丰富的新特性使 JUnit ⾃动化测试更加方便、功能更加丰富和强大。也是测试需要重点学习的地方；Jupiter 本身也是⼀一个基于 Junit Platform 的引擎实现，对 JUnit 5 而言，JUnit Jupiter API 只是另一个 API！。
3. JUnit Vintage3
   Junit 发展了 10 数年，Junit 3 和 Junit 4 都积累了大量的⽤用户，作为新一代框 架，这个模块是对 JUnit3，JUnit4 版本兼容的测试引擎，使旧版本 junit 的⾃动化测试脚本也可以顺畅运行在 Junit5 下，它也可以看作是基于 Junit Platform 实现的引擎范例。

大致意思就是 JUnit Platform 提供测试引擎，然后 JUnit Jupiter 用于运行 JUnit 5，JUnit Vintage 用于运行 JUnit3 和 4，它们最终都会在 JUnit Platform 提供的测试引擎上运行

## 依赖

```xml

<!--junit5-->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.8.2</version>
</dependency>
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-runner</artifactId>
    <version>1.8.2</version>
</dependency>
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-launcher</artifactId>
    <version>1.8.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-console-standalone</artifactId>
    <version>1.8.2</version>
    <scope>test</scope>
</dependency>
```

## 测试类和方法

- 测试类：有至少一个测试方法的任何顶层类，静态成员类或`@Nested`类，测试类必须不是抽象的必须有单一的构造器
- 测试方法：任何一个被`@Test`、`@RepeatedTest`、`@ParameterizedTest`、`@TestFactory`或`@TestTemplate`直接或间接标注的实例方法。
- 生命周期方法：任何直接或间接标注`@BeforeAll`, `@AfterAll`, `@BeforeEach`, or `@AfterEach`的任意方法。
  测试方法和生命周期方法必须是定义在本类中、从父类或接口继承的非抽象方法，而且除了`@TestFactory`必须返回值外，其它的都不能返回值。

> 测试类和方法不必须是 public，但不能是 private。
>
> 除非是有有技术原因，否则不推荐在测试方法上显式编写 public。比如需要跨包或模块。

一个使用了测试方法和生命周期方法的小例子

```java
package init.lucaschen.study.junit5;


import org.junit.jupiter.api.*;

@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class Junit5Test {
    @BeforeAll
    static void beforeAll() {
        System.out.println("@BeforeAll: 在测试类执行之前执行的方法 必须用 static 生命");
    }

    @BeforeEach
    void beforeEach() {
        System.out.println("@BeforeEach: 在每个测试用例前执行的方法");
    }

    @Test
    @Disabled
    void disable() {
        System.out.println("@Disabled 相当但 junit4 的 Ignore 跳过该测试用例");
    }

    @RepeatedTest(3)
    @Tag("tag1")
    void repeatedTest() {
        System.out.println("@RepeatedTest(3) 重复执行");
    }

    @Test
    @Tag("tag2")
    @DisplayName("displayName1")
    void displayName() {
        System.out.println("@DisplayName(\"name\") 展示用例名");
    }

    @AfterEach
    void afterEach() {
        System.out.println("@AfterEach: 在每个测试用例后执行的方法");
    }

    @AfterAll
    static void afterAll() {
        System.out.println("AfterAll: 在测试类执行之后执行的方法 必须用 static 声明");
    }

    @Test
    @Order(1)
    void divideTest() {
        System.out.println("------------------------------------------------------------------------");
        System.out.println("@Order: 定义执行顺序");
        System.out.println("需要先在类上打上@TestMethodOrder(MethodOrderer.OrderAnnotation.class)注解");
        System.out.println("------------------------------------------------------------------------");
    }

}

```

输出：

```shell
@BeforeAll: 在测试类执行之前执行的方法 必须用 static 生命
@BeforeEach: 在每个测试用例前执行的方法
------------------------------------------------------------------------
@Order: 定义执行顺序
需要先在类上打上@TestMethodOrder(MethodOrderer.OrderAnnotation.class)注解
------------------------------------------------------------------------
@AfterEach: 在每个测试用例后执行的方法
@BeforeEach: 在每个测试用例前执行的方法
@RepeatedTest(3) 重复执行
@AfterEach: 在每个测试用例后执行的方法
@BeforeEach: 在每个测试用例前执行的方法
@RepeatedTest(3) 重复执行
@AfterEach: 在每个测试用例后执行的方法
@BeforeEach: 在每个测试用例前执行的方法
@RepeatedTest(3) 重复执行
@AfterEach: 在每个测试用例后执行的方法

void init.lucaschen.study.junit5.Junit5Test.disable() is @Disabled
@BeforeEach: 在每个测试用例前执行的方法
@DisplayName("name") 展示用例名
@AfterEach: 在每个测试用例后执行的方法
AfterAll: 在测试类执行之后执行的方法 必须用 static 声明

Process finished with exit code 0
```

图片：

![junit5-common-notes](/imgs/posts/junit5/junit5-common-notes.png)

<!--
## 测试套件的常用注解

| 注解                                                              | 作用                   |
| :---------------------------------------------------------------- | :--------------------- |
| `@RunWith(JUnitPlatform.class)`                                   | 执行套件               |
| `@SelectPackages({ "com.packageA", com."packageB" })`             | 创建测试套件           |
| `@SelectClasses({ A.class, B.class})`                             | 创建测试套件           |
| `@IncludePackages({ "com.packageA", com."packageB" })`            | 过滤需要执行的测试包   |
| `@ExcludePackages({ "com.packageA", com."packageB" })`            | 过滤不需要执行的测试包 |
| `@IncludeClassNamePatterns({ "com.package.A", "com.package.B" })` | 过滤需要执行的测试包   |
| `@ExcludeClassNamePatterns({ "com.package.A", "com.package.B" })` | 过滤不需要执行的测试包 |
| `@IncludeTags({ "tagA" })`                                        | 过滤需要测试的方法     |
| `@ExcludeTags({ "tagA" })`                                        | 过滤不需要测试的方法   |

有些测试用例可能每次都要执行，有些可能只有特定版本才需要运行，测试套件可以帮助我们很好地将测试用例管理起来，指定需要运行的测试套件。
套件注解实例：
 -->

## Assertions

JUnit Jupiter 支持所有 JUnit 4 中的断言方法并添加了一些新的，适用于 Java8 Lambda 表达式的断言方法。所有断言方法都是`org.junit.jupiter.api.Assertions`的静态方法。

### 失败消息

```java
@Test
void standardAssertions() {
    Assertions.assertEquals(2, 1 + 1);
    Assertions.assertEquals(4, 2 * 2,
            "The optional failure message is now the last parameter");
    Assertions.assertTrue(1 < 2, () -> "Lazy evaluated message");
}
```

这个示例表明 assert 可以有一个失败时的消息，如果这个消息需要复杂的计算，那么可以通过一个 Lambda 表达式来进行惰性计算，就是只有当断言失败时才会计算这个值。

### 成组断言

```java
@Test
void groupedAssertions() {
    Assertions.assertAll("test group",
            () -> { Assertions.assertEquals(2, 1 + 1); },
            () -> { Assertions.assertEquals(4, 2 + 2); }
    );
}
```

### 依赖断言

```java
@Test
void dependentAssertions() {
    Assertions.assertAll(
            () -> {
                System.out.println("1");
                Assertions.assertEquals(3, 1 + 1); // Error
                System.out.println("2");
                Assertions.assertEquals(2, 6 - 4);
            },
            () -> {
                System.out.println("3");
                Assertions.assertEquals(1, 1 * 1);
                System.out.println("4");
            }
    );
}
```

输出 1 3 4，在一个断言块中，如果其中一个断言失败，那么块中随后出现的代码都会被跳过。

### 异常断言

```java
@Test
void exceptionAssertions() {
    Exception exception = Assertions.assertThrows(ArithmeticException.class,
            () -> divide(1, 0));
    Assertions.assertEquals("/ by zero", exception.getMessage());
}
```

### 超时断言

```java
@Test
void timeoutNotExceeded() {
    Assertions.assertTimeout(Duration.ofMinutes(2), () -> {});
}

@Test
void timeoutNotExceededWithResult() {
    String actualResult = Assertions.assertTimeout(Duration.ofMinutes(2), () -> "a result");
    Assertions.assertEquals("a result", actualResult);
}

@Test
void timeoutExceeded() {
    Assertions.assertTimeout(Duration.ofMillis(10), () -> Thread.sleep(100));
}

@Test
void timeoutExceededWithPreemptiveTermination() {
    // 在新线程执行，注意这个测试方法中不要有任何依赖ThreadLocal存储的东西，否则可能出现副作用。
    Assertions.assertTimeoutPreemptively(Duration.ofMillis(10), () -> new CountDownLatch(1).await());
}
```

## 条件测试

### @EnabledOnOs/@DisabledOnOs

```java
@Test
@EnabledOnOs(OS.WINDOWS)
void testOnWindows() {
    System.out.println("WINDOWS!!");
}

@Test
@EnabledOnOs(OS.MAC)
void testOnMac() {
    System.out.println("MAC!!");
}
```

### @EnabledOnJre/.etc

还有一些条件测试针对当前 Java 系统

```java
@Test
@EnabledOnJre(JRE.JAVA_8)
void testOnJava8() {
    System.out.println("JAVA8!!");
}

@Test
@EnabledOnJre(JRE.JAVA_11)
void testOnJava11() {
    System.out.println("JAVA11!!");
}

@Test
@DisabledForJreRange(max = JRE.JAVA_8)
void testOnAfterJava8() {
    System.out.println("After Java 8");
}

// java系统属性
@Test
@EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")
void testOnly64BitArchitecture() {
    System.out.println("64Bit Architecture...");
}

// 本机系统环境变量
@Test
@EnabledIfEnvironmentVariable(named = "username", matches = "lilpig")
void testOnlyLilpig() {
    System.out.println("LILPIG!!");
}
```

## 测试顺序

### 方法测试顺序

默认情况下 JUnit 使用一个明确但意义不明显的排序顺序，这保证了每次运行测试用例得到的结果是相同的。

编写单元测试时是不依赖测试执行顺序的，因为单元测试中的每一个测试用例时无关的，但当编写集成测试时，可能就需要指定顺序。

`@TestMethodOrder`可以指定测试方法的顺序，其中传入一个`MethodOrder`类型的类，用于进行排序，我们可以使用默认的几个也可以自定义。

```java
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class AssertionsDemo {
    // ...
}
```

如上代码，使用测试方法上`@Order`注解定义的顺序进行排序，所以我们可以给测试方法加上`@Order`注解

```java
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class AssertionsDemo {
    private Calculator calculator = new Calculator();

    @Test
    @Order(1)
    void standardAssertions() {
    }

    @Test
    @Order(2)
    void groupedAssertions() {
    }

    @Test
    @Order(3)
    void dependentAssertions() {
    }
}
```

### 类测试顺序

大部分情况下都不会有一个需求要改类测试顺序，但有时候我们可能需要一个随机顺序确保被测试代码在生产中不会因为隐含的依赖关系而出错，或者其他什么需求。

类测试顺序没法使用注解来定义了，需要用配置文件告诉 JUnit：`src/test/resource/junit-platform.properties`

```java
junit.jupiter.testclass.order.default = \
    org.junit.jupiter.api.ClassOrderer$OrderAnnotation
```

上面使用`@Order`注解进行排序。

### @Nested 类测试顺序

上面的顺序仅对于顶层类，对于`@Nested`测试类，它们的测试顺序相对于父类，可以在父类上加入`@TestClassOrder`来指定其顺序。

## @Nested 嵌套测试类

嵌套测试类可以让我们构建测试的层级结构，像如下示例

```java
@DisplayName("A stack")
public class StackTest {
    Stack<String> stack;

    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {
        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("isEmpty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {
            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }

        }
    }
}
```

## 依赖注入

JUnit5 有了依赖注入功能，可以在测试类构造器、生命周期方法和测试方法中加入参数，实现依赖注入。

JUnit5 的依赖注入由`ParameterResolver`来完成，它是一个接口，用于在运行时动态解析要注入的参数。如下是一些内建解析器。

### TestInfoParameterResolver

TestInfo 是当前运行的测试的一些信息，它作用在构造器上，就是当前测试类的信息，作用在生命周期方法上，就是触发生命周期方法的测试方法信息，作用在测试方法上，那么就是测试方法的信息。由`TestInfoParameterResolver`注入。

```java
@DisplayName("TestInfo Demo")
public class TestInfoTest {
    TestInfoTest(TestInfo testInfo) {
        assertEquals("TestInfo Demo", testInfo.getDisplayName());
    }

    @BeforeEach
    void beforeEach(TestInfo testInfo) {
        String displayName = testInfo.getDisplayName();
        assertTrue(displayName.equals("TEST 1") || displayName.equals("test2()"));
    }

    @Test
    @DisplayName("TEST 1")
    @Tag("my-tag")
    void test1(TestInfo testInfo) {
        assertEquals("TEST 1", testInfo.getDisplayName());
        assertTrue(testInfo.getTags().contains("my-tag"));
    }

    @Test
    void test2() {
    }
}
// DefaultTestInfo
//  [displayName = 'TEST 1',
//  tags = [my-tag],
//  testClass = class init.lucaschen.study.junit5.TestInfoTest,
//  testMethod = void init.lucaschen.study.junit5.TestInfoTest.test1(org.junit.jupiter.api.TestInfo)]
```

### RepetitionInfoParameterResolver

当一个@`RepeatedTest`、`@BeforeEach`、`@AfterEach`方法被调用并且参数中存在`RepetitionInfo`类型的参数时，这个实例会被注入。其中包含一些测试重复次数和总次数的信息。

### TestReporterParameterResolver

当方法参数中存在`TestReport`类型的参数时，会由`TestReporterParameterResolver`注入这个参数。

`TestReport`用来发布一些关于测试附加数据，用于生成测试报表。

```java
class TestReporterDemo {

    @Test
    void reportSingleValue(TestReporter testReporter) {
        testReporter.publishEntry("a status message");
    }

    @Test
    void reportKeyValuePair(TestReporter testReporter) {
        testReporter.publishEntry("a key", "a value");
    }

    @Test
    void reportMultipleKeyValuePairs(TestReporter testReporter) {
        Map<String, String> values = new HashMap<>();
        values.put("user name", "dk38");
        values.put("award year", "1974");
        testReporter.publishEntry(values);
    }
}
```

> 其它类型的`ParameterResolver`都需要通过`@ExtendWith`注解才能使用。

### RandomParametersExtension

复制 [RandomParametersExtension](https://github.com/junit-team/junit5-samples/blob/r5.8.1/junit5-jupiter-extensions/src/main/java/com/example/random/RandomParametersExtension.java) 的代码，然后使用这个 Resolver。

```java
@ExtendWith(RandomParametersExtension.class)
public class MyRandomParameterTest {
    Calculator calculator = new Calculator();
    @Test
    void testAdd(@RandomParametersExtension.Random int n) {
        assertEquals(n * n, calculator.multiply(n, n));
    }
}
```

## Repeated Tests

`@RepeatedTests`可以重复执行一个测试方法若干次，每次调用，测试方法都像一个普通`@Test`方法一样，它具有一样的生命周期回调和扩展支持。

```java
@RepeatedTest(10)
void repeatedTest() {
    // ...
}
```

`@RepeatedTest`标注的测试方法的 DisplayName 可能需要引用到目前的重复次数，所以可以从`@RepeatedTest`的 name 属性中设置当前测试方法的 DisplayName。

```java
@DisplayName("LALALA")
@RepeatedTest(value = 10, name = "{displayName} {currentRepetition} of {totalRepetitions}")
void repeatedTest() {
    assertEquals(1, 1/1);
}
```

占位符列表如下：

```java
String DISPLAY_NAME_PLACEHOLDER = "{displayName}";
String CURRENT_REPETITION_PLACEHOLDER = "{currentRepetition}";
String TOTAL_REPETITIONS_PLACEHOLDER = "{totalRepetitions}";
```

## @ParameterizedTest(参数化测试)

`@ParamterizedTest`用于给测试方法提供一些参数，这个注解会消费掉 Source 中配置的那些参数，第一种参数就是下标参数(indexed argument)，遵循 Source 中的变量和测试方法参数中的变量的一对一关系，第二种就是也可以通过一个聚合对象(aggregator)来从 Source 中获取数据，然而其它参数也可能通过`ParameterResolver`传递进来，比如`TestInfo`，它们之间的顺序关系如下：

1. 0 个或多个具有下标的 Source 参数需要定义在方法列表最前
2. 0 个或多个聚合对象 Source 参数要定义在第一条的后面
3. 0 个或多个 ParameterResolver 的参数定义在最后

`indexed argument`由`ArgumentsProvider`提供，而一个`aggregator`则需要通过一个`ArgumentsAccessor`类型的参数来接收，或者是任意一个打了`@AggregateWith`的参数。

> 继承自`java.lang.AutoCloseable`的参数(或`java.io.Cloaseable`，它继承自`AutoCloseable`)，将会自动再`@AfterEach`和所有`@AfterEachCallback`扩展被调用后被关闭。如果这个 AutoCloseable 资源横跨多个测试方法，那么你可能不希望它被关闭，可以设置`@ParameterizedTest(autoCloseArguments = false)`

### @ValueSource

ValueSource 中的参数会一个一个的被注入到测试方法中。

```java
@ParameterizedTest
@ValueSource(strings = {"racecar", "radar"})
void palindromes(String candidate) {
    assertFalse(StringUtils.isBlank(candidate));
}
```

ValueSource 提供一个原始数组，可以是如下类型

```java
short[] shorts() default {};
byte[] bytes() default {};
long[] longs() default {};
float[] floats() default {};
double[] doubles() default {};
char[] chars() default {};
boolean[] booleans() default {};
String[] strings() default {};
Class<?>[] classes() default {};
```

### @NullSource

提供一个单一 null 值

```java
@ParameterizedTest
@NullSource
void isNull(String arg1) {
    assertNull(arg1);
}
```

### @EmptySource

提供空值，可以是`java.lang.String`,`java.util.List`, `java.util.Set`, `java.util.Map`和原始数组，但不能是它们的子类型。

```java
@ParameterizedTest
@EmptySource
void isEmpty(int arr[]) {
    assertTrue(arr.length == 0);
}
```

### @NullAndEmptySource

只是 NullSource 和 EmptySource 的组合

像如下这种情况，想确定 null、空值和空白字符串对方法的影响

```java
@ParameterizedTest
@NullSource
@EmptySource
@ValueSource(strings = { " ", "   ", "\t", "\n" })
void nullEmptyAndBlankStrings(String text) {
    assertTrue(text == null || text.trim().isEmpty());
}
```

可以改写成

```java
@ParameterizedTest
@NullAndEmptySource
@ValueSource(strings = { " ", "   ", "\t", "\n" })
```

### @EnumSource

提供一个枚举参数。

```java
enum Gender {
    MALE, FEMALE
}

@ParameterizedTest
@EnumSource
void testEnumSource(Gender gender) {
    assertNotNull(gender);
}
```

这种情况下，会将对应枚举类中的所有枚举项都用来调用一次测试方法，可以通过 names 指定哪些枚举被用来测试。

```java
@ParameterizedTest
@EnumSource(names = "MALE")
void testEnumSource(Gender gender) {
    assertNotNull(gender);
}
```

- `names`还可以是正则表达式，匹配的项目名被用来测试
- `mode`用来指定粒度，是排除还是选择

![enum-source](/imgs/posts/junit5/enum-source.png)

### @MethodSource

使用一个工厂方法来生成测试参数

```java
@ParameterizedTest
@MethodSource("stringProvider")
void testWithExplicitLocalMethodSource(String argument) {
    assertNotNull(argument);
}

static Stream<String> stringProvider() {
    return Stream.of("apple", "banana");
}
```

1. 工厂方法必须是 static，除非测试类标注了`@TestInstance(Lifecycle.PER_CLASS)`
2. 返回值必须是一个 Stream 值，这个值的定义是所有 JUnit 可以转换成 Stream 的值，比如 Stream，DoubleStream，LongStream 等，还有对象或原始数组。

如果工厂方法具有和测试方法一样的名字，那么可以不用指定`@MethodSource`的参数。

如果测试方法具有多个参数，可以返回`Stream<Arguments>`对象。`Arguments`对象是 JUnit 定义的一个代表多个参数的类型，可以使用`arguments`方法生成。

```java
@ParameterizedTest
@MethodSource("stringIntAndListProvider")
void testWithMultipleArgMethodSource(String name, int count, List<String> list) {
    assertEquals(5, name.length());
    assertTrue(count >= 1 && count <= 10);
    assertEquals(2, list.size());
}

static Stream<Arguments> stringIntAndListProvider() {
    return Stream.of(
            Arguments.arguments("apple", 1, Arrays.asList("a","b")),
            Arguments.arguments("lemon", 5, Arrays.asList("x","y"))
    );
}
```

### @CsvSource/@CsvFileSource

通过 Csv 文件作为参数源

### @ArgumentsSource

自定义 ArgumentsProvier 来提供参数。前面说过测试方法中的参数都是`@ParameterizedTest`从`ArgumentsProvider`中拿的，其实前面使用的每一个`@XXXSoruce`其中都是通过`@ArgumentsProvider`提供了一个`ArgumentsProvider`

比如这个

```java
@ArgumentsSource(EnumArgumentsProvider.class)
public @interface EnumSource {}
```

自定义的`ArgumentsProvider`必须是一个顶级类或者一个嵌套静态类。

```java
public class MyArgumentsProvider implements ArgumentsProvider {
    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) throws Exception {
        return Stream.of("apple", "banana").map(Arguments::of);
    }
}
```

## 参数转换

像`@CsvSource`这种东西，只能传递 String 类型的参数，尽管有时候我们希望这个字段是整数或者小数。这时候需要有参数转换。

### 隐式转换

这个代码竟然能用。

```java
@ParameterizedTest
@CsvSource({
        "apple, 9.99",
        "banana, 2"
})
void testConverter(String fruit, double price) {
    Assertions.assertNotNull(fruit);
    Assertions.assertTrue(price > 0);
}
```

这证明 JUnit 提供了内置的隐转参数转换功能。String 类型的数据会被转换成各种类型，这依赖测试方法中定义的参数类型。

下面是具体的转换方式表。

![implicit-conversion-1](/imgs/posts/junit5/implicit-conversion-1.png)

![implicit-conversion-2](/imgs/posts/junit5/implicit-conversion-2.png)

### 隐式对象转换

除了上面定义的类型，JUnit Jupiter 允许将指定的 Source 中的数据转换成一个测试方法参数中的对象类型参数。

1. 使用工厂方法，在对应类中创建一个非 private 的静态方法，这个方法只有一个字符串参数，返回对应类的实例。
2. 使用构造器，需要一个非 private 构造器接受一个字符串参数，对应的类必须是顶层类或一个嵌套 static 类。

```java
@ParameterizedTest
@ValueSource(strings = {"MYSQL", "JAVA"})
void testConvertToObjectWithConstructor(Book book) {
    Assertions.assertNotNull(book);
}
```

```java
public class Book {
    private String title;
    public Book(String title) {
        this.title = title;
    }
}
```

### 显式转换

```java
@ParameterizedTest
@EnumSource(ChronoUnit.class)
void testConverter(
        @ConvertWith(ToStringArgumentConverter.class) String arg
) {
    Assertions.assertNotNull(ChronoUnit.valueOf(arg));
}
```

```java
public class ToStringArgumentConverter extends SimpleArgumentConverter {
    @Override
    protected Object convert(Object source, Class<?> targetType) throws ArgumentConversionException {
        assertEquals(String.class, targetType, "Can only convert to String");
        if (source instanceof Enum<?>)
            return ((Enum<?>) source).name();
        return String.valueOf(source);
    }
}
```

`TypedArgumentConverter`使用泛型进行特定类型到特定类型的转换。

显式转换可以使用注解进行简化。

```java
@Target({ ElementType.ANNOTATION_TYPE, ElementType.PARAMETER })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@ConvertWith(ToStringArgumentConverter.class)
public @interface ToStringPattern {

}


@ParameterizedTest
@EnumSource(ChronoUnit.class)
void testConverterWithAnnotation(
        @ToStringPattern String arg
) {
    Assertions.assertNotNull(ChronoUnit.valueOf(arg));
}
```

对于这种方式还想传参数的，参考 JUnit 的@JavaTimeArgumentConverter

## 参数聚合

由于 Source 中的参数只能和测试方法中的参数一对一对应，如果需要携带大量参数，那么测试方法将有巨大的签名，需要使用一个办法将其聚合

```java
@ParameterizedTest
@CsvSource({
        "Jane, Doe, FEMALE, 1990-05-20",
        "John, Doe, MALE, 1990-10-22"
})
void testWithArgumentsAccessor(ArgumentsAccessor accessor) {
    Person person = new Person(
            accessor.getString(0),
            accessor.getString(1),
            accessor.get(2, Gender.class),
            accessor.get(3, LocalDate.class)
    );

    assertNotNull(person);
}
```

这能用，但是这个`ArgumentsAccessor`和测试方法中为了将其转换成 Person 所写的代码，和这个测试用例又有什么关系呢？放这碍眼。

```java
@ParameterizedTest
@CsvSource({
        "Jane, Doe, FEMALE, 1990-05-20",
        "John, Doe, MALE, 1990-10-22"
})
void testWithArgumentsAccessor(
        @AggregateWith(PersonAggregation.class) Person person) {
    assertNotNull(person);
}
```

同样，如果你看`@AggregateWith(PersonAggregation.class)`不顺眼，也可以自己定义一个注解类。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.PARAMETER)
@AggregateWith(PersonAggregator.class)
public @interface CsvToPerson {
}
```

```java
@ParameterizedTest
@CsvSource({
    "Jane, Doe, F, 1990-05-20",
    "John, Doe, M, 1990-10-22"
})
void testWithCustomAggregatorAnnotation(@CsvToPerson Person person) {
    // perform assertions against person
}
```

## 动态测试

```java
@TestFactory
Stream<DynamicTest> randomNumberTest() {
    Iterator<Integer> inputGenerator = new Iterator<Integer>() {
        Random random = new Random();
        int cur = 0;
        @Override
        public boolean hasNext() {
            return cur < 10;
        }

        @Override
        public Integer next() {
            cur++;
            return random.nextInt(100);
        }
    };

    Function<Integer, String> displayNameGenerator = (input) -> "input: " + input;

    ThrowingConsumer<Integer> testExecutor = (input) -> assertEquals(input * 2, calculator.multiply(input, 2));

    return DynamicTest.stream(inputGenerator, displayNameGenerator, testExecutor);
}
```

更简单的写法，如果不需要迭代器生成测试参数时

```java
@TestFactory
Stream<DynamicTest> dynamicTestsFromStreamFactoryMethodWithNames() {
    Stream<Named<String>> inputStream = Stream.of(
            named("racecar is a palindrome", "racecar"),
            named("radar is also a palindrome", "radar")
    );

    return DynamicTest.stream(inputStream, text->assertNotNull(text));
}
```

DynamicNode 代表一个动态测试节点，这个节点可以是一个 DynamicTest 和一个 DynamicContainer，前者代表一个动态测试，后者代表一个测试容器，容器中可以包含其他容器和测试，这就构成了嵌套的测试。

```java
@TestFactory
Stream<DynamicNode> dynamicTestWithContainers() {
    return Stream.of("A", "B", "C")
            .map(input -> dynamicContainer("Container " + input, Stream.of(
                    dynamicTest("not null", () -> assertNotNull(input)),
                    dynamicContainer("properties", Stream.of(
                            dynamicTest("length > 0", () -> assertTrue(input.length() > 0)),
                            dynamicTest("not empty", () -> assertFalse(input.isEmpty()))
                    ))
            )));
}
```

## @Timeout

在测试方法，测试工厂，测试模板和生命周期方法上添加`@Timeout`，可以让该方法在指定时间内若未完成则自动放弃执行。默认的时间单位是秒，但是可配置。

这个和之前的`assertTimeoutPreemptively()`不同，这个注解标注的测试方法运行在主线程，当超时时，由其他线程来打断主线程。这能保证 JUnit 与类似 Spring 这种对线程敏感的框架一起工作。

如在类上添加`@Timeout`，那么类内部的所有测试方法和嵌套测试类中的测试方法都将应用同一个超时时间(除非在内部类和测试方法上也有`@Timeout`)。

**类层级上的`@Timeout`将不会作用到生命周期方法上**

当它作用于`@TestFactory` 上时，它对整个工厂中的测试执行奏效，但不会验证其中的每一个 `DynamicNode`，请用 `assertTimeout` 等方法代替。

## 并行测试

默认情况下 JUnit Jupiter 在单线程中顺序测试每个方法，可以通过`junit.jupiter.execution.parallel.enabled`字段配置

配置之后，默认情况下还是顺序执行，还需要设置 JUnit 的执行模式。`junit.jupiter.execution.parallel.mode.default`

- SAME_THREAD
  强制方法在父线程中执行。如果使用这个，所有测试方法会在与`@BeforeAll`和`@AfterAll`执行的线程中执行
- CONCURRENT
  并发执行除非被资源锁强制在单一线程执行

除此之外，也可以通过`@Execution` 注解指定单一类的执行模式。

通过如下配置，可以让类并行测试，而类中的方法顺序执行

```java
junit.jupiter.execution.parallel.enabled = true
junit.jupiter.execution.parallel.mode.default = same_thread
junit.jupiter.execution.parallel.mode.classes.default = concurrent
```

在并行测试模式下，一遇到共享资源便需要一些同步机制。`@ResourceLock` 注解为方法开启基于读写锁的同步机制。
