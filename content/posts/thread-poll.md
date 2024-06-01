---
title: '线程池配置'
description: 'Java 线程池配置'
keywords: 'thread'

date: 2024-03-08T14:22:18+08:00

categories:
  - Java
tags:
  - Thread
  - Async
---

介绍如何配置和使用线程池。

<!--more-->

## 如何配置

### 配置类

> ThreadPoolConfig.java

```java
import lombok.Getter;
import lombok.Setter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.task.TaskExecutor;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.ThreadPoolExecutor;

@Getter
@Setter
@EnableAsync
@Configuration
@ConfigurationProperties("system.thread.pool")
public class ThreadPoolConfig {

    /**
     * 如果提交任务后线程还在运行，当线程数小于 corePoolSize 值时，
     * 无论线程池中的线程是否忙碌，都会创建线程，
     * 并把任务交给此新创建的线程进行处理，
     * 如果线程数少于等于 corePoolSize，那么这些线程不会回收，除非将 allowCoreThreadTimeOut 设置为 true，
     * 但一般不这么干，因为频繁地创建销毁线程会极大地增加系统调用的开销
     */
    private int corePoolSize = 100;

    /**
     * 如果线程数大于核心数（corePoolSize）且小于最大线程数（maximumPoolSize），
     * 则会将任务先丢到阻塞队列里，然后线程自己去阻塞队列中拉取任务执行
     */
    private int queueCapacity = 1000;

    /**
     * 线程池中最大可创建的线程数，如果提交任务时队列满了且线程数未到达这个设定值，则会创建线程并执行此次提交的任务，
     * 如果提交任务时队列满了但线池数已经到达了这个值，此时说明已经超出了线池程的负载能力，就会执行拒绝策略，
     * 不能让源源不断地任务进来把线程池给压垮了吧，首先要保证线程池能正常工作
     */
    private int maxPoolSize = 500;

    /**
     * 线程名称前缀
     */
    private String threadNamePrefix = "thread-pool-executor-";

    /**
     * 线程存活时间，如果在此时间内超出 corePoolSize 大小的线程处于 idle 状态，这些线程会被回收
     */
    private int keepAliveSeconds = 60;

    @Bean
    public TaskExecutor taskExecutor() {
        // 通过 Runtime 方法来获取当前服务器 CPU 内核数量，根据 CPU 内核数量来创建核心线程数和最大线程数
        int threadCount = Runtime.getRuntime().availableProcessors();
        final ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
        threadPoolTaskExecutor.setCorePoolSize(Math.min(threadCount, corePoolSize));
        threadPoolTaskExecutor.setMaxPoolSize(Math.min(threadCount, maxPoolSize));
        threadPoolTaskExecutor.setQueueCapacity(queueCapacity);
        threadPoolTaskExecutor.setThreadNamePrefix(threadNamePrefix);
        threadPoolTaskExecutor.setKeepAliveSeconds(keepAliveSeconds);
        /**
         * 此处的拒绝策略总共可以有 4 种，分别为：
         *
         * 1. AbortPolicy：丢弃任务并抛出异常，这也是默认策略
         * 2. CallerRunsPolicy：用调用者所在的线程来执行任务
         * 如果用的是 CallerRunsPolicy 策略，提交任务的线程（比如主线程）提交任务后并不能保证马上就返回，
         * 当触发了这个 reject 策略不得不亲自来处理这个任务
         * 3. DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务
         * 4. DiscardPolicy：直接丢弃任务，不抛出任何异常，这种策略只适用于不重要的任务
         */
        threadPoolTaskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        threadPoolTaskExecutor.setWaitForTasksToCompleteOnShutdown(true);
        threadPoolTaskExecutor.setAwaitTerminationSeconds(60);
        return threadPoolTaskExecutor;
    }
}

```

### 配置文件

> application.yaml

配置类中已经设置了相关默认值，如果配置文件中无相关配置项，则会使用默认配置。

```yaml
system:
  thread:
    pool:
      core-pool-size: 100
      max-pool-size: 500
      queue-capacity: 1000
      thread-name-prefix: thread-pool-
      keep-alive-seconds: 60
```

### 为什么不允许使用 Executors 快速创建线程池？

使用 Executors 快速生成的线程池没有完整的参数配置，`newCachedThreadPool` 方法的最大线程数设置成了 `Integer.MAX_VALUE`，而 `newSingleThreadExecutor` 方法创建 `workQueue` 时 `LinkedBlockingQueue` 未声明大小，相当于创建了无界队列，一不小心就会导致 OOM。

## 如何使用

### 手动调用线程池

```java
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

@Slf4j
@SpringBootTest
class ThreadTest {

    @Autowired
    private ThreadPoolTaskExecutor threadPoolTaskExecutor;

    @Test
    void test() {
        // without return value
        threadPoolTaskExecutor.execute(() -> log.info("task"));
        // with return value, will return a Future
        final Future<String> submit = threadPoolTaskExecutor.submit(() -> "task return");
    }
}
```

### 使用 @Async 注解

Step1: 在启动类上增加 `@EnableAsync` 注解开启异步方法调用

```java
@SpringBootApplication
@EnableAsync
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Step2: 定义和实现相关方法，在方法实现上增加 `@Async` 注解

```java
import java.util.concurrent.Future;

public interface ThreadService {
    Future<String> withReturn();

    void withoutReturn();
}
```

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.scheduling.annotation.Async;
import org.springframework.scheduling.annotation.AsyncResult;
import org.springframework.stereotype.Service;

import java.util.concurrent.Future;

@Slf4j
@Service
public class ThreadServiceImpl implements ThreadService {

    @Async
    @Override
    public Future<String> withReturn() {
        // If your method has a return value and needs to return a Future,
        // you can use AsyncResult to temporarily wrap it.
        // Otherwise null will be returned
        return new AsyncResult<>("with return value");
    }

    @Async
    @Override
    public void withoutReturn() {
        log.info("without return value");
    }
}
```

Step3: 测试调用

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;


@SpringBootTest
class ThreadServiceTest {

    @Autowired
    private ThreadService threadService;

    @Test
    void withReturn() throws ExecutionException, InterruptedException {
        final Future<String> stringFuture = threadService.withReturn();
    }

    @Test
    void withoutReturn() {
        threadService.withoutReturn();
    }
}
```
