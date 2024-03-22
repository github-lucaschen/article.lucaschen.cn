---
title: 'Arthas'
description: 'arthas'
keywords: 'arthas'

date: 2024-03-22T14:29:11+08:00

categories:
  - arthas
tags:
  - arthas

draft: true
---

Alibaba 开源的 Java 诊断工具 Arthas 使用记录。

<!--more-->

## 开始

```shell
## 下载启动测试程序
curl -O https://arthas.aliyun.com/math-game.jar
java -jar math-game.jar

# 下载启动 arthas
curl -O https://arthas.aliyun.com/arthas-boot.jar
java -jar arthas-boot.jar
```

## 基础命令

### help

查看命令帮助信息，可以查看当前 arthas 版本支持的指令，或者查看具体指令的使用说明。

| 参数名称 | 参数说明                                   |
| -------- | ------------------------------------------ |
| 不接参数 | 查询当前 arthas 版本支持的指令以及指令描述 |
| [name:]  | 查询具体指令的使用说明                     |

```shell
$ help
 NAME         DESCRIPTION
 help         Display Arthas Help
 auth         Authenticates the current session
 keymap       Display all the available keymap for the specified connection.
 sc           Search all the classes loaded by JVM
 sm           Search the method of classes loaded by JVM
 classloader  Show classloader info
 jad          Decompile class
 getstatic    Show the static field of a class
 monitor      Monitor method execution statistics, e.g. total/success/failure count, average rt, fail rate, etc.
 stack        Display the stack trace for the specified class and method
 thread       Display thread info, thread stack
 trace        Trace the execution time of specified method invocation.
 watch        Display the input/output parameter, return object, and thrown exception of specified method invocation
 tt           Time Tunnel
 jvm          Display the target JVM information
 memory       Display jvm memory info.
 perfcounter  Display the perf counter information.
 ognl         Execute ognl expression.
 mc           Memory compiler, compiles java files into bytecode and class files in memory.
 redefine     Redefine classes. @see Instrumentation#redefineClasses(ClassDefinition...)
 retransform  Retransform classes. @see Instrumentation#retransformClasses(Class...)
 dashboard    Overview of target jvm's thread, memory, gc, vm, tomcat info.
 dump         Dump class byte array from JVM
 heapdump     Heap dump
 options      View and change various Arthas options
 cls          Clear the screen
 reset        Reset all the enhanced classes
 version      Display Arthas version
 session      Display current session information
 sysprop      Display and change the system properties.
 sysenv       Display the system env.
 vmoption     Display, and update the vm diagnostic options.
 logger       Print logger info, and update the logger level
 history      Display command history
 cat          Concatenate and print files
 base64       Encode and decode using Base64 representation
 echo         write arguments to the standard output
 pwd          Return working directory name
 mbean        Display the mbean information
 grep         grep command for pipes.
 tee          tee command for pipes.
 profiler     Async Profiler. https://github.com/jvm-profiling-tools/async-profiler
 vmtool       jvm tool
 stop         Stop/Shutdown Arthas server and exit the console.
 jfr          Java Flight Recorder Command
```

```shell
$ help dashboard
 USAGE:
   dashboard [-h] [-i <value>] [-n <value>]

 SUMMARY:
   Overview of target jvm's thread, memory, gc, vm, tomcat info.

 EXAMPLES:
   dashboard
   dashboard -n 10
   dashboard -i 2000

 WIKI:
   https://arthas.aliyun.com/doc/dashboard

 OPTIONS:
 -h, --help                              this help
 -i, --interval <value>                  The interval (in ms) between two executions, default is 5000 ms.
 -n, --number-of-execution <value>       The number of times this command will be executed.
```

### cls

清空当前屏幕区域。

### session

查看当前会话的信息，显示当前绑定的 pid 以及会话 id。

> **提示**
>
> 如果配置了 tunnel server，会追加打印 代理 id、tunnel 服务器的 url 以及连接状态。
>
> 如果使用了 staturl 做统计，会追加显示 statUrl 地址。

```shell
$ session
 Name        Value
--------------------------------------------------
 JAVA_PID    96956
 SESSION_ID  14219aff-0255-4e7d-aaa8-244061bd8277
```

### reset

> **提示**
>
> 重置增强类，将被 Arthas 增强过的类全部还原，Arthas 服务端 stop 时会重置所有增强过的类

```shell
# 还原指定类
reset Test
# 还原所有类
reset
```

### version

输出当前目标 Java 进程所加载的 Arthas 版本号

```shell
$ version
3.7.2
```

### history

打印命令历史。

> **提示**
>
> 历史指令会通过一个名叫 history 的文件持久化，所以 history 指令可以查看当前 arthas 服务器的所有历史命令，而不仅只是当前次会话使用过的命令。

| 参数名称 | 参数说明                |
| -------- | ----------------------- |
| [c:]     | 清空历史指令            |
| [n:]     | 显示最近执行的 n 条指令 |

### quit-stop

退出当前 Arthas 客户端，其他 Arthas 客户端不受影响。等同于 exit、logout、q 三个指令。

> **提示**
>
> 只是退出当前 Arthas 客户端，Arthas 的服务器端并没有关闭，所做的修改也不会被重置。

### keymap

`keymap` 命令输出当前的快捷键映射表：

| 快捷键      | 快捷键说明       | 命令名称             | 命令说明                         |
| ----------- | ---------------- | -------------------- | -------------------------------- |
| "\C-a"      | ctrl + a         | beginning-of-line    | 跳到行首                         |
| "\C-e"      | ctrl + e         | end-of-line          | 跳到行尾                         |
| "\C-f"      | ctrl + f         | forward-word         | 向前移动一个单词                 |
| "\C-b"      | ctrl + b         | backward-word        | 向后移动一个单词                 |
| "\e[D"      | 键盘左方向键     | backward-char        | 光标向前移动一个字符             |
| "\e[C"      | 键盘右方向键     | forward-char         | 光标向后移动一个字符             |
| "\e[B"      | 键盘下方向键     | next-history         | 下翻显示下一个命令               |
| "\e[A"      | 键盘上方向键     | previous-history     | 上翻显示上一个命令               |
| "\C-h"      | ctrl + h         | backward-delete-char | 向后删除一个字符                 |
| "\C-?"      | ctrl + shift + / | backward-delete-char | 向后删除一个字符                 |
| "\C-u"      | ctrl + u         | undo                 | 撤销上一个命令，相当于清空当前行 |
| "\C-d"      | ctrl + d         | delete-char          | 删除当前光标所在字符             |
| "\C-k"      | ctrl + k         | kill-line            | 删除当前光标到行尾的所有字符     |
| "\C-i"      | ctrl + i         | complete             | 自动补全，相当于敲 TAB           |
| "\C-j"      | ctrl + j         | accept-line          | 结束当前行，相当于敲回车         |
| "\C-m"      | ctrl + m         | accept-line          | 结束当前行，相当于敲回车         |
| "\C-w"      | -                | backward-delete-word | -                                |
| "\C-x\e[3~" | -                | backward-kill-line   | -                                |
| "\e\C-?"    | -                | backward-kill-word   | -                                |

- 任何时候 tab 键，会根据当前的输入给出提示
- 命令后敲 - 或 -- ，然后按 tab 键，可以展示出此命令具体的选项

#### 自定义快捷键

在当前用户目录下新建 `$USER_HOME/.arthas/conf/inputrc` 文件，加入自定义配置。

#### 后台异步命令相关快捷键

ctrl + c: 终止当前命令
ctrl + z: 挂起当前命令，后续可以 bg/fg 重新支持此命令，或 kill 掉
ctrl + a: 回到行首
ctrl + e: 回到行尾

### cat

> **提示**
>
> 打印文件内容，和 linux 里的 cat 命令类似。

### echo

> 提示
>
> 打印参数，和 linux 里的 echo 命令类似。

### grep

> **提示**
>
> 类似传统的 grep 命令。

### tee

> **提示**
>
> 类似传统的 tee 命令, 用于读取标准输入的数据，并将其内容输出成文件。
>
> tee 指令会从标准输入设备读取数据，将其内容输出到标准输出设备，同时保存成文件。

### pwd

> 提示
>
> 返回当前的工作目录，和 linux 命令类似

### plaintext

### wc

### options

> 提示
>
> 全局开关

| 名称                   | 默认值 | 描述                                                                                                                                                     |
| ---------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| unsafe                 | false  | 是否支持对系统级别的类进行增强，打开该开关可能导致把 JVM 搞挂，请慎重选择！                                                                              |
| dump                   | false  | 是否支持被增强了的类 dump 到外部文件中，如果打开开关，class 文件会被 dump 到/${application working dir}/arthas-class-dump/目录下，具体位置详见控制台输出 |
| batch-re-transform     | true   | 是否支持批量对匹配到的类执行 retransform 操作                                                                                                            |
| json-format            | false  | 是否支持 json 化的输出                                                                                                                                   |
| disable-sub-class      | false  | 是否禁用子类匹配，默认在匹配目标类的时候会默认匹配到其子类，如果想精确匹配，可以关闭此开关                                                               |
| support-default-method | true   | 是否支持匹配到 default method， 默认会查找 interface，匹配里面的 default method。参考 #1105 在新窗口打开                                                 |
| save-result            | false  | 是否打开执行结果存日志功能，打开之后所有命令的运行结果都将保存到~/logs/arthas-cache/result.log 中                                                        |
| job-timeout            | 1d     | 异步后台任务的默认超时时间，超过这个时间，任务自动停止；比如设置 1d, 2h, 3m, 25s，分别代表天、小时、分、秒                                               |
| print-parent-fields    | true   | 是否打印在 parent class 里的 filed                                                                                                                       |
| verbose                | false  | 是否打印更多详细信息                                                                                                                                     |
| strict                 | true   | 是否启用 strict 模式                                                                                                                                     |

```shell
# 查看所有的 options
options
# 获取 option 的值
options json-format
# 设置指定的 option
options save-result true
```

> **提示**
>
> 默认情况下 `json-format` 为 false，如果希望 `watch`/`tt` 等命令结果以 json 格式输出，则可以设置 `json-format` 为 true。

#### 打开 unsafe 开关，支持 jdk package 下的类

默认情况下，`watch`/`trace`/`tt`/`trace`/`monitor` 等命令不支持 `java.\*` package 下的类。可以设置 `unsafe` 为 true，则可以增强。

```shell
options unsafe true
```

#### 关闭 strict 模式，允许在 ognl 表达式里设置对象属性

> 提示
>
> since 3.6.0

对于新用户，在编写 ognl 表达式时，可能会出现误用。

比如对于 `Student`，判断年龄等于 18 时，可能条件表达式会误写为 `target.age=18`，这个表达式实际上是把当前对象的 `age` 设置为 18 了。正确的写法是 `target.age==18`。

为了防止出现类似上面的误用，Arthas 默认启用 `strict` 模式，在 `ognl` 表达式里，禁止更新对象的 Property 或者调用 `setter` 函数。

以 `MathGame` 为例，会出现以下的错误提示。

```shell
$ watch demo.MathGame primeFactors 'target' 'target.illegalArgumentCount=1'
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 20 ms, listenerId: 6
watch failed, condition is: target.illegalArgumentCount=1, express is: target, By default, strict mode is true, not allowed to set object properties. Want to set object properties, execute `options strict false`, visit /Users/lucaschen/logs/arthas/arthas.log for more details.
```

用户如果确定要在 `ognl` 表达式里更新对象，可以执行 `options strict false`，关闭 strict 模式。

## 系统命令

### dashboard

> **提示**

当前系统的实时数据面板，按 ctrl+c 退出。

当运行在 Ali-tomcat 时，会显示当前 tomcat 的实时信息，如 HTTP 请求的 qps, rt, 错误数, 线程池信息等等。

| 参数名称 | 参数说明                                 |
| -------- | ---------------------------------------- |
| [i:]     | 刷新实时数据的时间间隔 (ms)，默认 5000ms |
| [n:]     | 刷新实时数据的次数                       |

![dashboard](/imgs/posts/arthas/dashboard.png)

#### 数据说明

- ID: Java 级别的线程 ID，注意这个 ID 不能跟 jstack 中的 nativeID 一一对应。
- NAME: 线程名
- GROUP: 线程组名
- PRIORITY: 线程优先级, 1~10 之间的数字，越大表示优先级越高
- STATE: 线程的状态
- CPU%: 线程的 cpu 使用率。比如采样间隔 1000ms，某个线程的增量 cpu 时间为 100ms，则 cpu 使用率=100/1000=10%
- DELTA_TIME: 上次采样之后线程运行增量 CPU 时间，数据格式为`秒`
- TIME: 线程运行总 CPU 时间，数据格式为`分:秒`
- INTERRUPTED: 线程当前的中断位状态
- DAEMON: 是否是 daemon 线程

#### JVM 内部线程

Java 8 之后支持获取 JVM 内部线程 CPU 时间，这些线程只有名称和 CPU 时间，没有 ID 及状态等信息（显示 ID 为-1）。 通过内部线程可以观测到 JVM 活动，如 GC、JIT 编译等占用 CPU 情况，方便了解 JVM 整体运行状况。

- 当 JVM 堆(heap)/元数据(metaspace)空间不足或 OOM 时，可以看到 GC 线程的 CPU 占用率明显高于其他的线程。
- 当执行 `trace/watch/tt/redefine` 等命令后，可以看到 JIT 线程活动变得更频繁。因为 JVM 热更新 class 字节码时清除了此 class 相关的 JIT 编译结果，需要重新编译。

JVM 内部线程包括下面几种：

- JIT 编译线程: 如 `C1 CompilerThread0`, `C2 CompilerThread0`
- GC 线程: 如 `GC Thread0`, `G1 Young RemSet Sampling`
- 其它内部线程: 如 `VM Periodic Task Thread`, `VM Thread`, `Service Thread`

### thread

> 提示
>
> 查看当前线程信息，查看线程的堆栈

| 参数名称          | 参数说明                                                |
| ----------------- | ------------------------------------------------------- |
| id                | 线程 id                                                 |
| [n:]              | 指定最忙的前 N 个线程并打印堆栈                         |
| [b]               | 找出当前阻塞其他线程的线程                              |
| [i &lt;value&gt;] | 指定 cpu 使用率统计的采样间隔，单位为毫秒，默认值为 200 |
| [--all]           | 显示所有匹配的线程                                      |

这里的 cpu 使用率与 linux 命令 `top -H -p <pid>` 的线程 `%CPU` 类似，一段采样间隔时间内，当前 JVM 里各个线程的增量 cpu 时间与采样间隔时间的比例。

工作原理说明：

- 首先第一次采样，获取所有线程的 CPU 时间(调用的是 `java.lang.management.ThreadMXBean#getThreadCpuTime()` 及 `sun.management.HotspotThreadMBean.getInternalThreadCpuTimes()` 接口)
- 然后睡眠等待一个间隔时间（默认为 200ms，可以通过 `-i` 指定间隔时间）
- 再次第二次采样，获取所有线程的 CPU 时间，对比两次采样数据，计算出每个线程的增量 CPU 时间
- 线程 CPU 使用率 = 线程增量 CPU 时间 / 采样间隔时间 \* 100%

> **注意**
>
> 注意： 这个统计也会产生一定的开销（JDK 这个接口本身开销比较大），因此会看到 as 的线程占用一定的百分比，为了降低统计自身的开销带来的影响，可以把采样间隔拉长一些，比如 5000 毫秒。
>
> **提示**
>
> 另外一种查看 Java 进程的线程 cpu 使用率方法：可以使用 show-busy-java-threads 这个脚本

### jvm

### sysprop

### sysenv

### vmoption

### vmtool

### perfcounter

### logger

### getstatic

### ognl

### mbean

### heapdump

## 类命令

### sc

### sm

### jda

### mc-retransform

### mc-redefine

### dump

### classloader

## 增强命令

### monitor

### watch

### trace

### stack

### tt

### profiler

### jfr
