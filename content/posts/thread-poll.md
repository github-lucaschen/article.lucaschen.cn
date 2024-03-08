---
title: 'Thread Poll'
description: 'thread-poll'
keywords: 'thread,poll'

date: 2024-03-08T14:22:18+08:00

categories:
  - thread
tags:
  - thread
  - async
---

Configure using thread pool

<!--more-->

## How to configure

> application.yaml

```yaml
thread:
  pool:
    core-pool-size: 8
    max-pool-size: 20
    queue-capacity: 100
    name-prefix: thread-pool-
    keep-alive-seconds: 60
```

> ThreadPoolConfig.java

```java
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.task.TaskExecutor;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.ThreadPoolExecutor;

@Data
@EnableAsync
@Configuration
@ConfigurationProperties("thread.pool")
public class ThreadPoolConfig {
    private int corePoolSize;
    private int maxPoolSize;
    private int queueCapacity;
    private String namePrefix;
    private int keepAliveSeconds;

    /**
     * when a task is added to the thread pool through execute(Runnable) method:
     * <p>
     * 1. if thread-pool's thread number is less than <strong>corePoolSize</strong> at this time,
     * even if all threads in the thread pool are idle,
     * new threads will be created to handle the added tasks.
     * 2. if thread-pool's thread number is equals than <strong>corePoolSize</strong> at this time,
     * but the buffer queue <strong>workQueue</strong> is not full,
     * the task is put into the buffer queue.
     * 3. if thread-pool's thread number is equals than <strong>corePoolSize</strong> at this time,
     * and the buffer queue <strong>workQueue</strong> is full,
     * but thread-pool's thread number is less than <strong>maxPoolSize</strong>,
     * new threads will be created to handle the added tasks.
     * 4. if thread-pool's thread number is equals than <strong>corePoolSize</strong> at this time,
     * and the buffer queue <strong>workQueue</strong> is full,
     * and thread-pool's thread number is equals than <strong>maxPoolSize</strong>,
     * then handle this task through the strategy specified by <strong>maxPoolSize</strong>
     * <p>
     * That is to say:
     * The priority of processing tasks is:
     * 1. corePoolSize
     * 2. workQueue
     * 3. maxPoolSize
     * <p>
     * If all three are full, use <strong>maxPoolSize</strong> to handle rejected tasks
     *
     * @return Custom TaskExecutor
     */
    @Bean
    public TaskExecutor taskExecutor() {
        final ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
        threadPoolTaskExecutor.setCorePoolSize(corePoolSize);
        threadPoolTaskExecutor.setMaxPoolSize(maxPoolSize);
        threadPoolTaskExecutor.setQueueCapacity(queueCapacity);
        threadPoolTaskExecutor.setThreadNamePrefix(namePrefix);
        threadPoolTaskExecutor.setKeepAliveSeconds(keepAliveSeconds);
        threadPoolTaskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        threadPoolTaskExecutor.setWaitForTasksToCompleteOnShutdown(true);
        threadPoolTaskExecutor.setAwaitTerminationSeconds(60);
        return threadPoolTaskExecutor;
    }
}

```

## How to use

### Use ThreadPollTaskExecutor

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

### Use @Async

Step1: Add the EnableAsync annotation to the startup class to enable the asynchronous calling function

```java
@SpringBootApplication
@EnableAsync
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Step2: Define and implement related methods

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

Step3: Call test

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
