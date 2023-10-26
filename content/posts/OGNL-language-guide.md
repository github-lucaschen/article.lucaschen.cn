---
author: Lucas Chen
title: 'OGNL 语言指南(OGNL Language Guide)'
date: 2023-10-25T22:30:26+08:00
description: >-
  OGNL 语言指南
tags:
  - language
  - OGNL
categories:
  - language
  - en-US
series:
  - OGNL
---

翻译自[Apache 官网 OGNL](https://commons.apache.org/dormant/commons-ognl/language-guide.html)的语言指南。

<!--more-->

> Object-Graph Navigation Language (OGNL) is an open-source Expression Language (EL) for Java, which, while using simpler expressions than the full range of those supported by the Java language, allows getting and setting properties (through defined setProperty and getProperty methods, found in JavaBeans), and execution of methods of Java classes. It also allows for simpler array manipulation.
>
> -- [Wikipedia OGNL](https://en.wikipedia.org/wiki/OGNL)

对象图导航语言(OGNL)是 Java 的开源表达式语言(EL)，它是所有 Java 语言支持范围中最简单的表达式，它允许获取和设置属性(通过 JavaBeans 中定义的 setProperty 和 getProperty 方法)以及执行 Java 类的方法。它还允许更简单的数组操作。

## Syntax(语法)

> Basic OGNL expressions are very simple. The language has become quite rich with features, but you don't generally need to worry about the more complicated parts of the language: the simple cases have remained that way. For example, to get at the name property of an object, the OGNL expression is simply name. To get at the text property of the object returned by the headline property, the OGNL expression is headline.text.

基础的 OGNL 表达式非常简单，虽然该语言现在的特性已经变得非常丰富，但是你通常不需要担心语言中更复杂的部分：简单的情况下跟之前是一样的。例如：要获取对象的 name 属性，OGNL 的表达式就是简单的 name。要获取 headline 属性中的文本对象，OGNL 的表达式就是 headline.text。

> What is a property? Roughly, an OGNL property is the same as a bean property, which means that a pair of get/set methods, or alternatively a field, defines a property (the full story is a bit more complicated, since properties differ for different kinds of objects; see below for a full explanation).

什么是属性呢？大致来讲，OGNL 的属性跟 bean 对象的属性是完全相同的，这意味着一对 get/set 方法或者一个字段定义了一个属性(完整的情况会更加复杂，因为对象有不同类型的属性，详见下文的解释)。

> The fundamental unit of an OGNL expression is the navigation chain, usually just called "chain." The simplest chains consist of the following parts:

OGNL 表达式的基本单元是导航链，简称：“链”，最简单的链由下面几部分组成：

| Expression Element Part | Example                                                                    |
| :---------------------- | :------------------------------------------------------------------------- |
| Property names          | like the name and headline.text examples above                             |
| Method Calls            | hashCode() to return the current object's hash code                        |
| Array Indices           | listeners[0] to return the first of the current object's list of listeners |

| 表达式元素部分 | 示例                                           |
| :------------- | :--------------------------------------------- |
| 属性名称       | 上方举例的 name 和 headline.text               |
| 方法调用       | hashCode() 返回当前对象的哈希值                |
| 数组索引       | listeners[0] 返回 listeners 列表中的第一个对象 |

> All OGNL expressions are evaluated in the context of a current object, and a chain simply uses the result of the previous link in the chain as the current object for the next one. You can extend a chain as long as you like. For example, this chain:

所有 OGNL 表达式都在当前对象的上下文中求值,链中使用前一个链的返回结果作为当前链的当前对象，也就是支持链式调用，你想把链拉多长都可以，例如，下面这条链：

```java
name.toCharArray()[0].numericValue.toString()
```

> This expression follows these steps to evaluate:

这个表达式按照下面的步骤进行求值：

> - extracts the name property of the initial, or root, object (which the user provides to OGNL through the OGNL context);
> - calls the toCharArray() method on the resulting String;
> - extracts the first character (the one at index 0) from the resulting array;
> - gets the numericValue property from that character (the character is represented as a Character object, and the Character class has a method called getNumericValue());
> - calls toString() on the resulting Integer object. The final result of this expression is the String returned by the last toString() call.

---

- 提取初始对象或根对象(root)（用户通过 OGNL 上下文传递给 OGNL）的 name 属性
- 在返回的结果字符串上调用 toCharArray() 方法
- 从返回的结果数组中提取第一个字符（索引为 0 的字符）
- 从这个字符中获取 numericValue 属性(字符代表 Character 对象，Character 类有一个方法叫 getNumericValue())。
- 在返回结果的 Integer 对象上调用 toString() 方法。这个表达式最终的返回结果是 toString() 方法的返回的字符串。

Note that this example can only be used to get a value from an object, not to set a value. Passing the above expression to the Ognl.setValue() method would cause an InappropriateExpressionException to be thrown, because the last link in the chain is neither a property name nor an array index.

请注意，这个示例只能用于从对象中获取值，而不能用于设置值。将上述表达式传递给 Ognl.setValue() 方法将会抛出 InappropriateExpressionException 异常，因为链中的最后一个链接并不是属性名，也不是数组索引。

This is enough syntax to do the vast majority of what you ever need to do.

但是这种语法已经足够满足大多数工作所需。

## Expressions(表达式)

This section outlines the details the elements of OGNL's expressions.

本章将详细介绍 OGNL 表达式的各种细节。

### Constants(常量)

OGNL has the following kinds of constants:

OGNL 有以下几种常量：

- String literals, as in Java (with the addition of single quotes): delimited by single- or double-quotes, with the full set of character escapes;
- Character literals, also as in Java: delimited by single-quotes, also with the full set of escapes;
- Numeric literals, with a few more kinds than Java. In addition to Java's ints, longs, floats and doubles, OGNL lets you specify BigDecimals with a "b" or "B" suffix, and BigIntegers with an "h" or "H" suffix (think "huge"---we chose "h" for BigIntegers because it does not interfere with hexadecimal digits);
- Boolean (true and false) literals;
- The null literal.

---

- 字符串字面值，如在 Java (另增加单引号)中：由单引号或双引号包裹，并且带有完整的转义集；
- 字符字面量，也和 Java 中一样：被单引号分割，也带有全套的转义；
- 数字字面值，比 Java 中多了一些类型，除了 Java 中的整型、长型、浮点数和双精度浮点数外，OGNL 还允许您指定带有 "b" 或 "b" 后缀的 BigDecimals，以及带有 "h" 或 "h" 后缀的 BigIntegers(考虑 "huge" ——我们选择 "h" 表示 BigIntegers，因为它并不会干扰十六进制数字)；
- 布尔(真和假)字面值；
- 空文字。

### Referring to Properties(引用属性)

OGNL treats different kinds of objects differently in its handling of property references. Maps treat all property references as element lookups or storage, with the property name as the key. Lists and arrays treat numeric properties similarly, with the property name as the index, but string properties the same way ordinary objects do. Ordinary objects (that is, all other kinds) only can handle string properties and do so by using "get" and "set" methods (or "is" and "set"), if the object has them, or a field with the given name otherwise.

OGNL 在处理属性引用时以不同的方式对待不同类型的对象。使用属性名将所有属性映射为对象进行查找或存储。列表和数组使用同样的方式用索引(index)作为属性名处理数字属性。而处理字符串属性的方式与普通对象相同，普通对象(即所有其他类型的对象)如果有 "get" 和 "set" 方法(或 "is" 和 "set")，则只能通过使用处理字符串属性，如果没有则使用给定的字段名称处理字符串属性。

Note the new terminology here. Property "names" can be of any type, not just Strings. But to refer to non-String properties, you must use what we have been calling the "index" notation. For example, to get the length of an array, you can use this expression:

注意这里的新术语。属性 "name" 可以是任何类型，而不仅仅是字符串。但是要引用非字符串属性，必须使用我们一直称为 "index" 的符号。例如，要获取一个数组的长度，你可以使用这个表达式:

```java
array.length
```

But to get at element 0 of the array, you must use an expression like this:

但是要获取数组的第 0 个元素，你必须使用这样的表达式:

```java
array[0]
```

Note that Java collections have some special properties associated with them.

注意，Java 集合有一些与之关联的特殊属性。

### Indexing(索引)

As discussed above, the "indexing" notation is actually just property reference, though a computed form of property reference rather than a constant one.

如上所述，"indexing" 表示法实际上只是属性引用，尽管是一种计算形式的属性引用，而不是常数形式。

For example, OGNL internally treats the "array.length" expression exactly the same as this expression:

例如，OGNL 在内部处理 "array.length" 表达式与这个表达式完全相同:

```java
array["length"]
```

And this expression would have the same result (though not the same internal form):

而这个表达式会有相同的结果(尽管内部形式不同):

```java
array["len" + "gth"]
```

**Array and List Indexing**(数组和列表索引)

For Java arrays and Lists indexing is fairly simple, just like in Java. An integer index is given and that element is the referrent. If the index is out of bounds of the array or List and IndexOutOfBoundsException is thrown, just as in Java.

对于 Java 数组(arrays)和列表(Lists)来说索引(indexing)相当简单，和在 Java 中一样，给出一个整数索引就能的到元素的引用对象。如果索引超出数组或列表的边界，则抛出 IndexOutOfBoundsException 异常，也和 Java 中一样。

**JavaBeans Indexed Properties**(JavaBeans 索引属性)

JavaBeans supports the concept of Indexed properties. Specifically this means that an object has a set of methods that follow the following pattern:

JavaBeans 也支持索引属性的概念。具体来说，这意味着一个对象会有一组遵循以下模式的方法:

- `public PropertyType[] getPropertyName();`
- `public void setPropertyName(PropertyType[] anArray);`
- `public PropertyType getPropertyName(int index);`
- `public void setPropertyName(int index, PropertyType value);`

OGNL can interpret this and provide seamless access to the property through the indexing notation. References such as

OGNL 可以解释这一点，并通过索引符号提供对属性的无缝访问。

```java
someProperty[2]
```

are automatically routed through the correct indexed property accessor (in the above case through getSomeProperty(2) or setSomeProperty(2, value)). If there is no indexed property accessor a property is found with the name someProperty and the index is applied to that.

诸如此类的引用会自动通过正确的索引属性访问器进行路由（在上述情况下通过 getSomeProperty(2) 或 setSomeProperty(2, value)）。 如果没有索引属性访问器，则会找到名为 someProperty 的属性，并将索引应用于该属性。

**OGNL Object Indexed Properties**(OGNL 对象索引属性)

OGNL extends the concept of indexed properties to include indexing with arbitrary objects, not just integers as with JavaBeans Indexed Properties. When finding properties as candidates for object indexing, OGNL looks for patterns of methods with the following signature:

OGNL 扩展了索引属性的概念，包括对任意对象进行索引，而不仅仅是像 JavaBeans 索引属性那样对整数进行索引。 当查找属性作为对象索引的候选时，OGNL 会查找具有以下签名的方法模式：

- `public PropertyType getPropertyName(IndexType index);`
- `public void setPropertyName(IndexType index, PropertyType value);`

The PropertyType and IndexType must match each other in the corresponding set and get methods. An actual example of using Object Indexed Properties is with the Servlet API: the Session object has two methods for getting and setting arbitrary attributes:

PropertyType 和 IndexType 在相应的 set 和 get 方法中必须相互匹配。 使用对象索引属性的一个实际示例是 Servlet API：Session 对象有两种获取和设置任意属性的方法：

```java
public Object getAttribute(String name);
public void setAttribute(String name, Object value);
```

An OGNL expression that can both get and set one of these attributes is:

可以获取和设置这些属性之一的 OGNL 表达式是:

```java
session.attribute["foo"]
```

## Calling Methods(调用方法)

OGNL calls methods a little differently from the way Java does, because OGNL is interpreted and must choose the right method at run time, with no extra type information aside from the actual arguments supplied. OGNL always chooses the most specific method it can find whose types match the supplied arguments; if there are two or more methods that are equally specific and match the given arguments, one of them will be chosen arbitrarily.

OGNL 调用方法的方式与 Java 的方式略有不同，因为 OGNL 是解释性的，并且必须在运行时选择正确的方法，除了提供的实际参数之外，没有额外的类型信息。 OGNL 总是选择它能找到的最具体的方法，其类型与提供的参数相匹配； 如果有两个或多个同样具体且与给定参数匹配的方法，则将任意选择其中之一。

In particular, a null argument matches all non-primitive types, and so is most likely to result in an unexpected method being called.

特别是，空参数匹配所有非基本类型，因此很可能导致调用意外的方法。

Note that the arguments to a method are separated by commas, and so the comma operator cannot be used unless it is enclosed in parentheses. For example,

请注意，方法的参数是用逗号分隔的，因此不能使用逗号运算符，除非将其括在括号中。 例如，

```java
method( ensureLoaded(), name )
```

is a call to a 2-argument method, while

上面是对 2 参数方法的调用，而下面的

```java
method( (ensureLoaded(), name) )
```

is a call to a 1-argument method.

是对 1 参数方法的调用。

## Variable References(变量引用)

OGNL has a simple variable scheme, which lets you store intermediate results and use them again, or just name things to make an expression easier to understand. All variables in OGNL are global to the entire expression. You refer to a variable using a number sign in front of its name, like this:

OGNL 有一个简单的变量方案，它允许您存储中间结果并再次使用它们，或者只是命名事物以使表达式更易于理解。 OGNL 中的所有变量对于整个表达式都是全局的。 您可以在变量名称前面使用数字符号来引用变量，如下所示：

```java
#var
```

OGNL also stores the current object at every point in the evaluation of an expression in the this variable, where it can be referred to like any other variable. For example, the following expression operates on the number of listeners, returning twice the number if it is more than 100, or 20 more than the number otherwise:

OGNL 还将表达式求值中每个点的当前对象存储在 this 变量中，可以像任何其他变量一样引用它。 例如，以下表达式对侦听器的数量进行操作，如果大于 100，则返回该数字的两倍，否则返回该数字的 20：

```java
listeners.size().(#this > 100? 2*#this : 20+#this)
```

OGNL can be invoked with a map that defines initial values for variables. The standard way of invoking OGNL defines the variables root (which holds the initial, or root, object), and context (which holds the Map of variables itself).

可以使用定义变量初始值的映射来调用 OGNL。 调用 OGNL 的标准方法定义变量根(root)（保存初始或根对象(root)）和上下文（保存变量本身的映射）。

To assign a value to a variable explicitly, simply write an assignment statement with a variable reference on the left-hand side:

要显式地将值赋给变量，只需在左侧写一个赋值语句并引用变量:

```java
#var = 99
```

## Parenthetical Expressions(括号表达式)

As you would expect, an expression enclosed in parentheses is evaluated as a unit, separately from any surrounding operators. This can be used to force an evaluation order different from the one that would be implied by OGNL operator precedences. It is also the only way to use the comma operator in a method argument.

正如您所期望的那样，括在括号中的表达式作为一个单位计算，与周围的任何操作符分开。这可用于强制不同于 OGNL 操作符优先级所暗示的求值顺序。这也是在方法参数中使用逗号操作符的唯一方法。

## Chained Subexpressions(链接子表达式)

If you use a parenthetical expression after a dot, the object that is current at the dot is used as the current object throughout the parenthetical expression. For example,

如果在点后使用圆括号表达式，则圆点处的当前对象将用作整个圆括号表达式中的当前对象。例如,

```java
headline.parent.(ensureLoaded(), name)
```

traverses through the headline and parent properties, ensures that the parent is loaded and then returns (or sets) the parent's name.

遍历标题(headline)的父属性(parent)，确保父属性被加载(ensureLoaded())，然后返回(或设置)父属性的名称。

Top-level expressions can also be chained in this way. The result of the expression is the right-most expression element.

顶级表达式也可以通过这种方式链接。 表达式的结果是最右边的表达式元素。

```java
ensureLoaded(), name
```

This will call ensureLoaded() on the root object, then get the name property of the root object as the result of the expression.

这将在根对象(root)上调用 EnsureLoaded()，然后获取根对象(root)的 name 属性作为表达式的结果。

## Collection Construction(集合结构)

### Lists

To create a list of objects, enclose a list of expressions in curly braces. As with method arguments, these expressions cannot use the comma operator unless it is enclosed in parentheses. Here is an example:

```java
name in { null,"Untitled" }
```

This tests whether the name property is null or equal to "Untitled".

The syntax described above will create a instanceof the List interface. The exact subclass is not defined.

## Native Arrays

Sometimes you want to create Java native arrays, such as int[] or Integer[]. OGNL supports the creation of these similarly to the way that constructors are normally called, but allows initialization of the native array from either an existing list or a given size of the array.

```java
new int[] { 1, 2, 3 }
```

This creates a new int array consisting of three integers 1, 2 and 3.

To create an array with all null or 0 elements, use the alternative size constructor

```java
new int[5]
```

This creates an int array with 5 slots, all initialized to zero.

### Maps

Maps can also be created using a special syntax.

```java
#{ "foo" : "foo value", "bar" : "bar value" }
```

This creates a Map initialized with mappings for "foo" and "bar".

Advanced users who wish to select the specific Map class can specify that class before the opening curly brace

```java
#@java.util.LinkedHashMap@{ "foo" : "foo value", "bar" : "bar value" }
```

The above example will create an instance of the JDK 1.4 class LinkedHashMap, ensuring the the insertion order of the elements is preserved.

## Projecting Across Collections

OGNL provides a simple way to call the same method or extract the same property from each element in a collection and store the results in a new collection. We call this "projection," from the database term for choosing a subset of columns from a table. For example, this expression:

```java
listeners.{delegate}
```

returns a list of all the listeners' delegates. See the coercion section for how OGNL treats various kinds of objects as collections.

During a projection the #this variable refers to the current element of the iteration.

```java
objects.{ #this instanceof String ? #this : #this.toString()}
```

The above would produce a new list of elements from the objects list as string values.

## Selecting From Collections

OGNL provides a simple way to use an expression to choose some elements from a collection and save the results in a new collection. We call this "selection," from the database term for choosing a subset of rows from a table. For example, this expression:

```java
listeners.{? #this instanceof ActionListener}
```

returns a list of all those listeners that are instances of the ActionListener class.

### Selecting First Match

In order to get the first match from a list of matches, you could use indexing such as listeners.{? true }[0]. However, this is cumbersome because if the match does not return any results (or if the result list is empty) you will get an ArrayIndexOutOfBoundsException.

The selection syntax is also available to select only the first match and return it as a list. If the match does not succeed for any elements an empty list is the result.

```java
objects.{^ #this instanceof String }
```

Will return the first element contained in objects that is an instance of the String class.

### Selecting Last Match

Similar to getting the first match, sometimes you want to get the last element that matched.

```java
objects.{$ #this instanceof String }
```

This will return the last element contained in objects that is an instanceof the String class

## Calling Constructors

You can create new objects as in Java, with the new operator. One difference is that you must specify the fully qualified class name for classes other than those in the java.lang package.

This is only true with the default ClassResolver in place. With a custom class resolver packages can be mapped in such a way that more Java-like references to classes can be made. Refer to the OGNL Developer's Guide for details on using ClassResolver class (for example, new java.util.ArrayList(), rather than simply new ArrayList()).

OGNL chooses the right constructor to call using the same procedure it uses for overloaded method calls.

## Calling Static Methods

You can call a static method using the syntax @class@method(args). If you leave out class, it defaults to java.lang.Math, to make it easier to call min and max methods. If you specify the class, you must give the fully qualified name.

If you have an instance of a class whose static method you wish to call, you can call the method through the object as if it was an instance method.

If the method name is overloaded, OGNL chooses the right static method to call using the same procedure it uses for overloaded instance methods.

## Getting Static Fields

You can refer to a static field using the syntax @class@field. The class must be fully qualified.

## Expression Evaluation

If you follow an OGNL expression with a parenthesized expression, without a dot in front of the parentheses, OGNL will try to treat the result of the first expression as another expression to evaluate, and will use the result of the parenthesized expression as the root object for that evaluation. The result of the first expression may be any object; if it is an AST, OGNL assumes it is the parsed form of an expression and simply interprets it; otherwise, OGNL takes the string value of the object and parses that string to get the AST to interpret.

For example, this expression

```java
#fact(30H)
```

looks up the fact variable, and interprets the value of that variable as an OGNL expression using the BigInteger representation of 30 as the root object. See below for an example of setting the fact variable with an expression that returns the factorial of its argument. Note that there is an ambiguity in OGNL's syntax between this double evaluation operator and a method call. OGNL resolves this ambiguity by calling anything that looks like a method call, a method call. For example, if the current object had a fact property that held an OGNL factorial expression, you could not use this approach to call it

```java
fact(30H)
```

because OGNL would interpret this as a call to the fact method. You could force the interpretation you want by surrounding the property reference by parentheses:

```java
(fact)(30H)
```

## Pseudo-Lambda Expressions

OGNL has a simplified lambda-expression syntax, which lets you write simple functions. It is not a full-blown lambda calculus, because there are no closures---all variables in OGNL have global scope and extent.

For example, here is an OGNL expression that declares a recursive factorial function, and then calls it:

```java
#fact = :[#this<=1? 1 : #this*#fact(#this-1)], #fact(30H)
```

The lambda expression is everything inside the brackets. The #this variable holds the argument to the expression, which is initially 30H, and is then one less for each successive call to the expression.

OGNL treats lambda expressions as constants. The value of a lambda expression is the AST that OGNL uses as the parsed form of the contained expression.

## Pseudo-Properties for Collections

There are some special properties of collections that OGNL makes available. The reason for this is that the collections do not follow JavaBeans patterns for method naming; therefore the size(), length(), etc. methods must be called instead of more intuitively referring to these as properties. OGNL corrects this by exposing certain pseudo-properties as if they were built-in.

Special Collections Pseudo-Properties

| Collection                                | Special Properties                                                                                                                                                                                                                                                                                                                     |
| :---------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Collection (inherited by Map, List & Set) | - size: The size of the collection <br> - isEmpty: Evaluates to true if the collection is empty                                                                                                                                                                                                                                        |
| List                                      | - iterator: Evalutes to an Iterator over the List.                                                                                                                                                                                                                                                                                     |
| Map                                       | - keys: Evalutes to a Set of all keys in the Map <br> - values: Evaluates to a Collection of all values in the Map <br> **Note** These properties, plus size and isEmpty, are different than the indexed form of access for Maps (i.e. someMap["size"] gets the "size" key from the map, whereas someMap.size gets the size of the Map |
| Set                                       | - iterator: Evalutes to an Iterator over the Set                                                                                                                                                                                                                                                                                       |
| Iterator                                  | - next: Evalutes to the next object from the Iterator <br> - hasNext: Evaluates to true if there is a next object available from the Iterator                                                                                                                                                                                          |
| Enumeration                               | - next: Evalutes to the next object from the Enumeration <br>- hasNext: Evaluates to true if there is a next object available from the Enumeration<br> - nextElement: Synonym for next<br> - hasMoreElements: Synonym for hasNext                                                                                                      |

## Operators that differ from Java's operators

For the most part, OGNL's operators are borrowed from Java and work similarly to Java's operators. See the OGNL Reference for a complete discussion. Here we describe OGNL operators that are not in Java, or that are different from Java.

- The comma (,) or sequence operator. This operator is borrowed from C. The comma is used to separate two independent expressions. The value of the second of these expressions is the value of the comma expression. Here is an example:

  ```java
  ensureLoaded(), name
  ```

  When this expression is evaluated, the ensureLoaded method is called (presumably to make sure that all parts of the object are in memory), then the name property is retrieved (if getting the value) or replaced (if setting).

- List construction with curly braces ({}). You can create a list in-line by enclosing the values in curly braces, as in this example:

  ```java
  { null, true, false }
  ```

- The in operator (and not in, its negation). This is a containment test, to see if a value is in a collection. For example,

  ```java
  name in {null,"Untitled"} || name
  ```

## Setting values versus getting values

As stated before, some values that are gettable are not also settable because of the nature of the expression. For example,

```java
names[0].location
```

is a settable expression - the final component of the expression resolves to a settable property.

However, some expressions, such as

```java
names[0].length + 1
```

are not settable because they do not resolve to a settable property in an object. It is simply a computed value. If you try to evaluate this expression using any of the Ognl.setValue() methods it will fail with an InappropriateExpressionException.

It is also possible to set variables using get expressions that include the '=' operator. This is useful when a get expression needs to set a variable as a side effect of execution.

## Coercing Objects to Types

Here we describe how OGNL interprets objects as various types. See below for how OGNL coerces objects to booleans, numbers, integers, and collections.

### Interpreting Objects as Booleans

Any object can be used where a boolean is required. OGNL interprets objects as booleans like this:

- If the object is a Boolean, its value is extracted and returned;
- If the object is a Number, its double-precision floating-point value is compared with zero; non-zero is treated as true, zero as false;
- If the object is a Character, its boolean value is true if and only if its char value is non-zero;
- Otherwise, its boolean value is true if and only if it is non-null.

### Interpreting Objects as Numbers

Numerical operators try to treat their arguments as numbers. The basic primitive-type wrapper classes (Integer, Double, and so on, including Character and Boolean, which are treated as integers), and the "big" numeric classes from the java.math package (BigInteger and BigDecimal), are recognized as special numeric types. Given an object of some other class, OGNL tries to parse the object's string value as a number.

Numerical operators that take two arguments use the following algorithm to decide what type the result should be. The type of the actual result may be wider, if the result does not fit in the given type.

- If both arguments are of the same type, the result will be of the same type if possible;
- If either argument is not of a recognized numeric class, it will be treated as if it was a Double for the rest of this algorithm;
- If both arguments are approximations to real numbers (Float, Double, or BigDecimal), the result will be the wider type;
- If both arguments are integers (Boolean, Byte, Character, Short, Integer, Long, or BigInteger), the result will be the wider type;
- If one argument is a real type and the other an integer type, the result will be the real type if the integer is narrower than "int"; BigDecimal if the integer is BigInteger; or the wider of the real type and Double otherwise.

### Interpreting Objects as Integers

Operators that work only on integers, like the bit-shifting operators, treat their arguments as numbers, except that BigDecimals and BigIntegers are operated on as BigIntegers and all other kinds of numbers are operated on as Longs. For the BigInteger case, the result of these operators remains a BigInteger; for the Long case, the result is expressed as the same type of the arguments, if it fits, or as a Long otherwise.

### Interpreting Objects as Collections

The projection and selection operators (e1.{e2} and e1.{?e2}), and the in operator, all treat one of their arguments as a collection and walk it. This is done differently depending on the class of the argument:

- Java arrays are walked from front to back;
- Members of java.util.Collection are walked by walking their iterators;
- Members of java.util.Map are walked by walking iterators over their values;
- Members of java.util.Iterator and java.util.Enumeration are walked by iterating them;
- Members of java.lang.Number are "walked" by returning integers less than the given number starting with zero;
- All other objects are treated as singleton collections containing only themselves.

## Appendix: OGNL Language Reference

This section has a fairly detailed treatment of OGNL's syntax and implementation. See below for a complete table of OGNL's operators, a section on how OGNL coerces objects to various types, and a detailed description of OGNL's basic expressions.

### Operators

OGNL borrows most of Java's operators, and adds a few new ones. For the most part, OGNL's treatment of a given operator is the same as Java's, with the important caveat that OGNL is essentially a typeless language. What that means is that every value in OGNL is a Java object, and OGNL attempts to coerce from each object a meaning appropriate to the situation it is used in (see the section on coercion).

The following table lists OGNL operators in reverse precedence order. When more than one operator is listed in the same box, these operators have the same precedence and are evaluated in left-to-right order.
