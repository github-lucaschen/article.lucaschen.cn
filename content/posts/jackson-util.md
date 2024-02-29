---
author: Lucas Chen
title: 'Jackson Util'
date: 2023-12-17
description: >-
  Jackson 封装 JsonUtil
tags:
  - jackson
  - json
categories:
  - java
  - json
---

Jackson 是 SpringMVC 默认的 JSON 解析器。为保证项目使用统一的结构和相同的返回值，将相关操作封装为工具类。

<!--more-->

## 工具类代码

```java
package com.**.**.utils;


import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.databind.json.JsonMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import com.fasterxml.jackson.datatype.jsr310.deser.LocalDateTimeDeserializer;
import com.fasterxml.jackson.datatype.jsr310.ser.LocalDateTimeSerializer;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;

import java.text.SimpleDateFormat;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.List;
import java.util.Map;
import java.util.Objects;
import java.util.Optional;

@Slf4j
public final class JsonUtil {

    private JsonUtil() {
        throw new IllegalStateException("cannot create instance of static util class");
    }

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
            // java.util.Date(new SimpleDateFormat("yyyy-MM-dd").parse("2024-01-01"))
            // --> String("2024-01-01 00:00:00")
            .defaultDateFormat(new SimpleDateFormat(LOCAL_DATE_TIME_FORMAT))
            .build();


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
    }


    public static Optional<String> toJson(final Object object) {
        if (Objects.nonNull(object)) {
            try {
                return Optional.of(jsonMapper.writeValueAsString(object));
            } catch (JsonProcessingException e) {
                log.error("JsonUtil toJson : {}", e);
            }
        }
        return Optional.empty();
    }

    public static <T> Optional<T> toObject(final String json, Class<T> clazz) {
        if (StringUtils.isNoneBlank(json) && Objects.nonNull(clazz)) {
            try {
                return Optional.of(jsonMapper.readValue(json, clazz));
            } catch (JsonProcessingException e) {
                log.error("JsonUtil toObject : {}", e);
            }
        }
        return Optional.empty();
    }

    public static <T> Optional<T> toObject(final String json,
                                           TypeReference<T> typeReference) {
        if (StringUtils.isNoneBlank(json) && Objects.nonNull(typeReference)) {
            try {
                return Optional.of(jsonMapper.readValue(json, typeReference));
            } catch (JsonProcessingException e) {
                log.error("JsonUtil toObject : {}", e);
            }
        }
        return Optional.empty();
    }

    public static <T> Optional<T> toObject(final Map map, Class<T> clazz) {
        if (MapUtils.isNotEmpty(map) && Objects.nonNull(clazz)) {
            return Optional.of(jsonMapper.convertValue(map, clazz));
        }
        return Optional.empty();
    }

    @SuppressWarnings("rawtypes")
    public static Optional<Map> toMap(final String json) {
        if (StringUtils.isNoneBlank(json)) {
            try {
                return Optional.ofNullable(jsonMapper.readValue(json,
                        new TypeReference<Map<String, Object>>() {
                        }));
            } catch (JsonProcessingException e) {
                log.error("JsonUtil toMap : {}", e);
            }
        }
        return Optional.empty();
    }

    @SuppressWarnings("rawtypes")
    public static Optional<Map> toMap(final Object object) {
        if (Objects.nonNull(object)) {
            return Optional.of(jsonMapper.convertValue(object,
                    new TypeReference<Map<String, Object>>() {
                    }));
        }
        return Optional.empty();
    }

    public static <T> Optional<List<T>> toObjects(final String json, TypeReference<List<T>> typeReference) {
        if (StringUtils.isNoneBlank(json) && Objects.nonNull(typeReference)) {
            try {
                return Optional.of(jsonMapper.readValue(json, typeReference));
            } catch (JsonProcessingException e) {
                log.error("JsonUtil toObjects : {}", e);
            }
        }
        return Optional.empty();
    }

    public static Optional<JsonNode> toTree(final String json) {
        if (StringUtils.isNoneBlank(json)) {
            try {
                return Optional.of(jsonMapper.readTree(json));
            } catch (JsonProcessingException e) {
                log.error("JsonUtil toTree : {}", e);
            }
        }
        return Optional.empty();
    }
}
```

### 为什么使用 Optional 作为返回值

Java8 新增了 `Optional` 来处理空指针问题，这里作为工具类，在有可能返回 `null` 的情况下，不如直接返回 `Optional`。

这样可以明确告知调用方，此处可能会有 `null`, 请增加相应处理逻辑。

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
    }
```

为 `JsonMapper` 注册 `JavaTimeModule` 模块，处理 `LocalDateTime` 实现转换问题。

### 方法

#### toJson(final Object object)

```java
    public static Optional<String> toJson(final Object object) {
        if (Objects.nonNull(object)) {
            try {
                return Optional.of(jsonMapper.writeValueAsString(object));
            } catch (JsonProcessingException e) {
                log.error("JsonUtil toJson : {}", e);
            }
        }
        return Optional.empty();
    }
```

测试方法:

```java
    @Test
    void toJson() {
        final ImmutableMap<String, String> immutableMap = ImmutableMap.of("k1", "v1", "k2", "v2");
        final String immutableMapJson = JsonUtil.toJson(immutableMap).orElse("");
        assertTrue(StringUtils.equals(immutableMapJson, "{\"k1\":\"v1\",\"k2\":\"v2\"}"));
        final HashMap<String, String> hashMap = new HashMap<>();
        hashMap.put("k1", "v1");
        hashMap.put("k2", "v2");
        final String hashMapJson = JsonUtil.toJson(hashMap).orElse("");
        assertTrue(StringUtils.equals(hashMapJson, "{\"k1\":\"v1\",\"k2\":\"v2\"}"));
        final UserPO admin = UserPO.builder().id(null).username("admin").password("admin").build();
        final String json = JsonUtil.toJson(admin).orElse("");
        assertEquals(json, "{\"username\":\"admin\",\"password\":\"admin\"}");
    }
```

#### toObject(final String json, Class<T> clazz)

```java
    public static <T> Optional<T> toObject(final String json, Class<T> clazz) {
        if (StringUtils.isNoneBlank(json) && Objects.nonNull(clazz)) {
            try {
                return Optional.of(jsonMapper.readValue(json, clazz));
            } catch (JsonProcessingException e) {
                log.error("JsonUtil toObject : {}", e);
            }
        }
        return Optional.empty();
    }
```

测试方法:

```java
    @Test
    void toObject() {
        final String json = "{\"id\":null,\"username\":\"username\",\"password\":\"password\"}";
        final UserPO userPO = JsonUtil.toObject(json, UserPO.class).orElse(null);
        assertTrue(Objects.nonNull(userPO));
        assertNull(userPO.getId());
        assertEquals("username", userPO.getUsername());
        assertEquals("password", userPO.getPassword());
    }
```

#### toObject(final String json, TypeReference<T> typeReference)

```java
    public static <T> Optional<T> toObject(final String json,
                                           TypeReference<T> typeReference) {
        if (StringUtils.isNoneBlank(json) && Objects.nonNull(typeReference)) {
            try {
                return Optional.of(jsonMapper.readValue(json, typeReference));
            } catch (JsonProcessingException e) {
                log.error("JsonUtil toObject : {}", e);
            }
        }
        return Optional.empty();
    }
```

测试方法:

```java
    @Test
    void toObjectTypeReference() {
        final String json = "{\"id\":null,\"username\":\"username\",\"password\":\"password\"}";
        final UserPO userPO = JsonUtil.toObject(json, new TypeReference<UserPO>() {
        }).orElse(null);
        assertTrue(Objects.nonNull(userPO));
        assertNull(userPO.getId());
        assertEquals("username", userPO.getUsername());
        assertEquals("password", userPO.getPassword());
    }
```

#### toObject(final Map map, Class<T> clazz)

```java
    public static <T> Optional<T> toObject(final Map map, Class<T> clazz) {
        if (MapUtils.isNotEmpty(map) && Objects.nonNull(clazz)) {
            return Optional.of(jsonMapper.convertValue(map, clazz));
        }
        return Optional.empty();
    }
```

测试代码:

```java
    @Test
    void toObjectFromMap() {
        final ImmutableMap<String, String> immutableMap = ImmutableMap
                .of("k2", "v2", "username", "username", "password", "password");
        final UserPO userPO = JsonUtil.toObject(immutableMap, UserPO.class).orElse(null);
        assertTrue(Objects.nonNull(userPO));
        assertNull(userPO.getId());
        assertEquals("username", userPO.getUsername());
        assertEquals("password", userPO.getPassword());
    }
```

#### toMap(final String json)

```java
    @SuppressWarnings("rawtypes")
    public static Optional<Map> toMap(final String json) {
        if (StringUtils.isNoneBlank(json)) {
            try {
                return Optional.ofNullable(jsonMapper.readValue(json,
                        new TypeReference<Map<String, Object>>() {
                        }));
            } catch (JsonProcessingException e) {
                log.error("JsonUtil toMap : {}", e);
            }
        }
        return Optional.empty();
    }
```

测试方法:

```java
    @Test
    void toMap() {
        final String json = "{\"id\":null,\"username\":\"username\",\"password\":\"password\"}";
        final Map map = JsonUtil.toMap(json).orElse(null);
        assertTrue(Objects.nonNull(map));
        assertNull(map.get("id"));
        assertEquals("username", map.get("username"));
        assertEquals("password", map.get("password"));
    }
```

#### toMap(final Object object)

```java
    @SuppressWarnings("rawtypes")
    public static Optional<Map> toMap(final Object object) {
        if (Objects.nonNull(object)) {
            return Optional.of(jsonMapper.convertValue(object,
                    new TypeReference<Map<String, Object>>() {
                    }));
        }
        return Optional.empty();
    }
```

测试代码:

```java
    @Test
    void toMapFromObject() {
        final UserPO admin1 = UserPO.builder().id(null).username("username").password("password").build();
        final Map map = JsonUtil.toMap(admin1).orElse(null);
        assertTrue(Objects.nonNull(map));
        assertNull(map.get("id"));
        assertEquals("username", map.get("username"));
        assertEquals("password", map.get("password"));
    }
```

#### toObjects(final String json, TypeReference<List<T>> typeReference)

```java
    public static <T> Optional<List<T>> toObjects(final String json, TypeReference<List<T>> typeReference) {
        if (StringUtils.isNoneBlank(json) && Objects.nonNull(typeReference)) {
            try {
                return Optional.of(jsonMapper.readValue(json, typeReference));
            } catch (JsonProcessingException e) {
                log.error("JsonUtil toObjects : {}", e);
            }
        }
        return Optional.empty();
    }
```

测试方法:

```java
   @Test
    void toObjects() {
        final UserPO admin1 = UserPO.builder().username("admin1").password("admin1").build();
        final UserPO admin2 = UserPO.builder().username("admin2").password("admin2").build();
        final UserPO admin3 = UserPO.builder().username("admin3").password("admin3").build();
        final ImmutableList<UserPO> userPOs = ImmutableList.of(admin1, admin2, admin3);
        final String json = JsonUtil.toJson(userPOs).orElse("");
        assertTrue(StringUtils.isNoneBlank(json));
        final List<UserPO> userPOList = JsonUtil.toObjects(json, new TypeReference<List<UserPO>>() {
        }).orElse(Collections.emptyList());
        assertTrue(CollectionUtils.isNotEmpty(userPOList));
        assertTrue(CollectionUtils.isEqualCollection(userPOs, userPOList));
    }
```

#### toTree(final String json)

```java
   public static Optional<JsonNode> toTree(final String json) {
        if (StringUtils.isNoneBlank(json)) {
            try {
                return Optional.of(jsonMapper.readTree(json));
            } catch (JsonProcessingException e) {
                log.error("JsonUtil toTree : {}", e);
            }
        }
        return Optional.empty();
    }
```

测试方法:

```java
    @Test
    void toTree() {
        final String json = "{\"id\":null,\"username\":\"username\",\"password\":\"password\"}";
        final JsonNode jsonNode = JsonUtil.toTree(json).orElse(null);
        assertTrue(Objects.nonNull(jsonNode));
        final JsonNode idNode = jsonNode.get("id");
        assertTrue(idNode.isNull());
        final JsonNode usernameNode = jsonNode.get("username");
        assertTrue(StringUtils.equals("username", usernameNode.textValue()));
        final JsonNode passwordNode = jsonNode.get("password");
        assertTrue(StringUtils.equals("password", passwordNode.textValue()));
    }
```
