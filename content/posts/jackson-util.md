---
title: 'Jackson 工具类'
description: 'Jackson 封装 JsonUtils'
keywords: 'jackson,json'

date: 2023-12-17T21:46:25+08:00

categories:
  - Java
tags:
  - Jackson
  - JSON
---

Jackson 是 SpringMVC 默认的 JSON 解析器。为保证项目使用统一的结构和相同的返回值，将相关操作封装为工具类。

<!--more-->

## 工具类代码

```java
package com.**.**.util;

import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.databind.json.JsonMapper;
import com.fasterxml.jackson.databind.ser.std.ToStringSerializer;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateTimeSerializer;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;

import java.text.SimpleDateFormat;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.Collections;
import java.util.List;
import java.util.Objects;

@Slf4j
public final class JsonUtils {

    private static final String LOCAL_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
    private static final JsonMapper jsonMapper = JsonMapper.builder()
            // disable -- fail on unknown properties
            // ({"id":null,"field":"value"} -> Object.builder.field(value).build())
            // ignore non-existent field
            .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
            // non null field
            // ({"id":null,"field":"value"} -> {"field":"value"})
            // result ignore fields with null values
            .serializationInclusion(JsonInclude.Include.NON_NULL)
            // enable -- allow single quotes
            // ({'field':"value"})
            .configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true)
            // enable -- allow unquoted field names
            // ({field:"value"})
            .configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true)
            // disable -- write dates as timestamps
            // enable ---- java.util.Date(new SimpleDateFormat("yyyy-MM-dd").parse("2024-01-01"))
            //              --> Number(1704038400000)
            // disable ---- java.util.Date(new SimpleDateFormat("yyyy-MM-dd").parse("2024-01-01"))
            //              --> String("2023-12-31T16:00:00.000+00:00")
            .configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false)
            // Set date formatting template
            // java.util.Date(new SimpleDateFormat("yyyy-MM-dd").parse("2024-01-01"))
            // --> String("2024-01-01 00:00:00")
            .defaultDateFormat(new SimpleDateFormat(LOCAL_DATE_TIME_FORMAT)).build();

    static {
        // add Module "com.fasterxml.jackson.datatype:jackson-datatype-jsr310"
        // to enable handling Java 8 date/time type
        // set custom serializer and deserializer
        // to replace datetime format template [yyyy-MM-ddTHH:mm:ss] to [yyyy-MM-dd HH:mm:ss]
        final JavaTimeModule javaTimeModule = new JavaTimeModule();
        final DateTimeFormatter dateTimeFormatter =
                DateTimeFormatter.ofPattern(LOCAL_DATE_TIME_FORMAT);
        javaTimeModule.addSerializer(LocalDateTime.class,
                new LocalDateTimeSerializer(dateTimeFormatter));
        javaTimeModule.addDeserializer(LocalDateTime.class, new
                LocalDateTimeDeserializer(dateTimeFormatter));

        // 序列换成 json 时,将所有的 long 变成 string。因为 js 中的数字类型不能包含所有的 java long 值
        javaTimeModule.addSerializer(Long.class, ToStringSerializer.instance);

        jsonMapper.registerModule(javaTimeModule);
    }


    private JsonUtils() {
    }

    public static <T> String to(final T object) {
        if (Objects.isNull(object)) {
            return null;
        }
        try {
            return object instanceof String ? (String) object : jsonMapper.writeValueAsString(object);
        } catch (JsonProcessingException e) {
            log.error("JsonUtils::to({})", object, e);
            return null;
        }
    }

    public static <T> T to(final String json, final Class<T> clazz) {
        if (StringUtils.isBlank(json) || Objects.isNull(clazz)) {
            return null;
        }
        try {
            return jsonMapper.readValue(json, clazz);
        } catch (JsonProcessingException e) {
            log.error("JsonUtils::to({}, {})", json, clazz, e);
            return null;
        }
    }

    public static <T> T to(final String json, final TypeReference<T> typeReference) {
        if (StringUtils.isBlank(json) || Objects.isNull(typeReference)) {
            return null;
        }
        try {
            return jsonMapper.readValue(json, typeReference);
        } catch (JsonProcessingException e) {
            log.error("JsonUtils::to({}, {})", json, typeReference, e);
            return null;
        }
    }

    public static boolean isJson(final String json) {
        if (StringUtils.isBlank(json)) {
            return false;
        }
        try {
            jsonMapper.readTree(json);
            return true;
        } catch (JsonProcessingException e) {
            log.error("JsonUtils::isJson({})", json);
            return false;
        }
    }

    public static <T> String toPrettyPrinter(final T object) {
        if (Objects.isNull(object)) {
            return null;
        }
        try {
            if (object instanceof String) {
                return jsonMapper.writerWithDefaultPrettyPrinter()
                        .writeValueAsString(jsonMapper.readTree(object.toString()));
            } else {
                return jsonMapper.writerWithDefaultPrettyPrinter().writeValueAsString(object);
            }
        } catch (JsonProcessingException e) {
            log.error("JsonUtils::to({})", object, e);
            return null;
        }
    }

    public static List<String> findText(final String json, final String fieldName) {
        if (StringUtils.isBlank(json) || StringUtils.isBlank(fieldName)) {
            return Collections.emptyList();
        }
        try {
            return jsonMapper.readTree(json).findValuesAsText(fieldName);
        } catch (JsonProcessingException e) {
            log.error("JsonUtils::findText({}, {})", json, fieldName, e);
            return Collections.emptyList();
        }
    }

    public static String find(final String json, final String fieldName) {
        return findText(json, fieldName).stream().findFirst().orElse(null);
    }

    public static JsonNode toTree(final String json) {
        if (StringUtils.isNoneBlank(json)) {
            try {
                return jsonMapper.readTree(json);
            } catch (JsonProcessingException e) {
                log.error("JsonUtils::toTree({})", json, e);
                return null;
            }
        }
        return null;
    }

    public static <T> T convert(final Object object, final Class<T> clazz) {
        if (Objects.isNull(object) || Objects.isNull(clazz)) {
            return null;
        }
        return jsonMapper.convertValue(object, clazz);
    }

    public static <T> T convert(final Object object, final TypeReference<T> typeReference) {
        if (Objects.isNull(object) || Objects.isNull(typeReference)) {
            return null;
        }
        return jsonMapper.convertValue(object, typeReference);
    }
}
```

## 解析代码

### 构造函数

```java
public final class JsonUtil {
    private JsonUtil() {
        throw new IllegalStateException("cannot create instance of static util class");
    }
}
```

重写工具类无参私有构造函数，避免被实例化调用。

### 变量

```java
public final class JsonUtil {
    private static final String LOCAL_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";

    private static final JsonMapper jsonMapper = JsonMapper.builder()
            // disable -- fail on unknown properties
            // ({"id":null,"field":"value"} -> Object.builder.field(value).build())
            // ignore non-existent field
            .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
            // non null field
            // ({"id":null,"field":"value"} -> {"field":"value"})
            // result ignore fields with null values
            .serializationInclusion(JsonInclude.Include.NON_NULL)
            // enable -- allow single quotes
            // ({'field':"value"})
            .configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true)
            // enable -- allow unquoted control chars
            // ({"field":"value\n"})
            .configure(JsonParser.Feature.ALLOW_UNQUOTED_CONTROL_CHARS, true)
            // enable -- allow unquoted field names
            // ({field:"value"})
            .configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true)
            // disable -- write dates as timestamps
            // enable ---- java.util.Date(new SimpleDateFormat("yyyy-MM-dd").parse("2024-01-01"))
            //              --> Number(1704038400000)
            // disable ---- java.util.Date(new SimpleDateFormat("yyyy-MM-dd").parse("2024-01-01"))
            //              --> String("2023-12-31T16:00:00.000+00:00")
            .configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false)
            // Set date formatting template
            // java.util.Date(new SimpleDateFormat("yyyy-MM-dd").parse("2024-01-01")) --> String("2024-01-01 00:00:00")
            .defaultDateFormat(new SimpleDateFormat(LOCAL_DATE_TIME_FORMAT))
            .build();
```

定义变量：
`LOCAL_DATE_TIME_FORMAT` 是本地常用日期时间编码。

`JsonMapper` 是自 2.10 版本起，给 `ObjectMapper` 提供了一个子类：`JsonMapper`，使得语义更加明确，专门用于处理 `JSON` 格式。

为了方便创建 `JsonMapper` ,因此 Jackson 也顺应潮流的提供了 `MapperBuilder` 构建器（2.10 版本起）,因此我们可以构建一个 `JsonMapper` 对象，并设置相关的值。

#### 未知字段是否忽略

```java
            // disable -- fail on unknown properties
            // ({"id":null,"field":"value"} -> Object.builder.field(value).build())
            // ignore non-existent field
            .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
```

即对 `JSON` 字符串作相应的转换的时候，如果有未知的字段和指定的类对应不上，`true` 失败，返回报错， `false` 忽略。

对应的测试代码：

```java
    @Test
    void failOnUnknownProperties() {
        final String json = "{\"ids\":null,\"username\":\"username\",\"password\":\"password\"}";
        final UserPO userPO = JsonUtil.toObject(json, UserPO.class).orElse(null);
        assertTrue(Objects.nonNull(userPO));
        assertNull(userPO.getId());
        assertTrue(StringUtils.equals("username", userPO.getUsername()));
        assertTrue(StringUtils.equals("password", userPO.getPassword()));
    }
```

#### 是否返回 null

```java
            // non null field
            // ({"id":null,"field":"value"} -> {"field":"value"})
            // result ignore fields with null values
            .serializationInclusion(JsonInclude.Include.NON_NULL)
```

如果对象的属性为空，转换为 `JSON` 的时候是否需要包含 `null` 字段。

对应的测试代码:

```java
    @Test
    void nonNullField() {
        final UserPO userPO = UserPO.builder()
                .id(null)
                .username("username")
                .password("password")
                .build();
        final String json = JsonUtil.toJson(userPO).orElse("");
        assertTrue(StringUtils.isNoneBlank(json));
        assertTrue(StringUtils.equals(json, "{\"username\":\"username\",\"password\":\"password\"}"));
    }
```

#### 是否允许使用单引号

```java
            // enable -- allow single quotes
            // ({'field':"value"})
            .configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true)
```

设置 `JSON` 为非标准的使用单引号的格式收否可以正常进行转换，`false` 报错抛出，`true` 进行转换。

对应的测试代码:

```java
    @Test
    void allowSingleQuotes() {
        final String json = "{'username':'username','password':'password'}";
        final UserPO userPO = JsonUtil.toObject(json, UserPO.class).orElse(null);
        assertTrue(Objects.nonNull(userPO));
        assertNull(userPO.getId());
        assertTrue(StringUtils.equals("username", userPO.getUsername()));
        assertTrue(StringUtils.equals("password", userPO.getPassword()));
    }
```

#### 是否允许转义字符出现

```java
            // enable -- allow unquoted control chars
            // ({"field":"value\n"})
            .configure(JsonParser.Feature.ALLOW_UNQUOTED_CONTROL_CHARS, true)
```

设置如果 `JSON` 中有转义字符时如何处理，`false` 报错抛出，`true` 进行转换。

对应的测试代码:

```java
    @Test
    void allowUnquotedControlChars() {
        final String json = "{\"id\":null,\"username\":\"username\t\",\"password\":\"password\n\"}";
        final UserPO userPO = JsonUtil.toObject(json, UserPO.class).orElse(null);
        assertTrue(Objects.nonNull(userPO));
        assertNull(userPO.getId());
        assertTrue(StringUtils.equals("username\t", userPO.getUsername()));
        assertTrue(StringUtils.equals("password\n", userPO.getPassword()));
    }
```

#### 是否允许无引号包裹

```java
            // enable -- allow unquoted field names
            // ({field:"value"})
            .configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true)
```

设置如果是非标准的无引号包裹字段如何处理，`false` 报错抛出，`true` 进行转换.

对应的测试代码:

```java
    @Test
    void allowUnquotedFieldNames() {
        final String json = "{id:null,username:\"username\",password:\"password\"}";
        final UserPO userPO = JsonUtil.toObject(json, UserPO.class).orElse(null);
        assertTrue(Objects.nonNull(userPO));
        assertNull(userPO.getId());
        assertTrue(StringUtils.equals("username", userPO.getUsername()));
        assertTrue(StringUtils.equals("password", userPO.getPassword()));
    }
```

#### Date 转换规则

```java
            // disable -- write dates as timestamps
            // enable ---- java.util.Date(new SimpleDateFormat("yyyy-MM-dd").parse("2024-01-01"))
            //              --> Number(1704038400000)
            // disable ---- java.util.Date(new SimpleDateFormat("yyyy-MM-dd").parse("2024-01-01"))
            //              --> String("2023-12-31T16:00:00.000+00:00")
            .configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false)
                        // Set date formatting template
            // java.util.Date(new SimpleDateFormat("yyyy-MM-dd").parse("2024-01-01"))
            // --> String("2024-01-01 00:00:00")
            .defaultDateFormat(new SimpleDateFormat(LOCAL_DATE_TIME_FORMAT))
```

设置是否将 `Date` 转换为时间戳

- 如果为 `true`,则会将 `java.util.Date(new SimpleDateFormat("yyyy-MM-dd").parse("2024-01-01"))` 转换为 `1704038400000`
- 如果设置为 `false`，则会转换为 `2023-12-31T16:00:00.000+00:00`
- 如果设置为 `false`，同时如果设置了转换模版，则按照设置的模版转换为 `2024-01-01 00:00:00`

对应的测试代码:

```java
    @Test
    void datetimeTest() throws ParseException {
        final Date date = new SimpleDateFormat("yyyy-MM-dd").parse("2024-01-01");
        final LocalDateTime localDateTime = LocalDateTime.of(2024, 1, 1, 0, 0, 0);
        final LocalDate localDate = LocalDate.of(2024, 1, 1);
        final LocalTime localTime = LocalTime.of(0, 0, 0);
        final DateTimeDTO buildDatTimeDTO = DateTimeDTO.builder()
                .date(date)
                .localDateTime(localDateTime)
                .localDate(localDate)
                .localTime(localTime)
                .build();
        final String json = JsonUtil.toJson(buildDatTimeDTO).orElse("");
        assertTrue(StringUtils.isNoneBlank(json));
        final DateTimeDTO dateTimeDTO = JsonUtil.toObject(json, DateTimeDTO.class).orElse(null);
        assertEquals(buildDatTimeDTO, dateTimeDTO);
    }
```

### 静态代码块

```java
    static {
        // add Module "com.fasterxml.jackson.datatype:jackson-datatype-jsr310"
        // to enable handling Java 8 date/time type
        // set custom serializer and deserializer
        // to replace datetime format template [yyyy-MM-ddTHH:mm:ss] to [yyyy-MM-dd HH:mm:ss]
        final JavaTimeModule javaTimeModule = new JavaTimeModule();
        final DateTimeFormatter dateTimeFormatter = DateTimeFormatter
                .ofPattern(LOCAL_DATE_TIME_FORMAT);
        javaTimeModule.addSerializer(LocalDateTime.class,
                new LocalDateTimeSerializer(dateTimeFormatter));
        javaTimeModule.addDeserializer(LocalDateTime.class,
                new LocalDateTimeDeserializer(dateTimeFormatter));
        jsonMapper.registerModule(javaTimeModule);

        // 序列换成 json 时,将所有的 long 变成 string。因为 js 中的数字类型不能包含所有的 java long 值
        javaTimeModule.addSerializer(Long.class, ToStringSerializer.instance);
    }
```

为 `JsonMapper` 注册 `JavaTimeModule` 模块，处理 `LocalDateTime` 实现转换问题。