---
title: 'Synchronized'
description: 'Synchronized'
keywords: 'java,thread'

date: 2024-03-18T09:52:48+08:00

categories:
  - Java
tags:
  - Synchronized
---

Synchronized 关键字相关。

<!--more-->

> 作者：pdai\
> 出处：<https://pdai.tech/md/java/thread/java-thread-x-key-synchronized.html>\

**本文参考以上文章进行学习和记录。**

1. 一把锁只能同时被一个线程获取，没有获得锁的线程只能等待
2. 每个实例都对应有自己的一把锁(`this`),不同实例之间互不影响；例外：锁对象是`\*.class` 以及 `synchronized` 修饰的是 `static` 方法的时候，所有对象公用同一把锁
3. `synchronized` 修饰的方法，无论方法正常执行完毕还是抛出异常，都会释放锁

## 对象锁

包括方法锁(默认锁对象为`this`,当前实例对象)和同步代码块锁(自己指定锁对象)

### 代码块形式：手动指定锁定对象，也可是是 this,也可以是自定义的锁

```java
import java.util.concurrent.TimeUnit;

public class Clazz {
    static Clazz instance = new Clazz();

    public void lockThis() {
        synchronized (this) {
            System.out.println("Thread: " + Thread.currentThread().getName() + " start. ");
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("Thread: " + Thread.currentThread().getName() + " end.");
        }
    }


    public static void main(String[] args) {
        new Thread(() -> instance.lockThis()).start();
        new Thread(() -> instance.lockThis()).start();
    }
}
```

output:

```text
Thread: Thread-0 start.
Thread: Thread-0 end.
Thread: Thread-1 start.
Thread: Thread-1 end.
```

```java
import java.util.concurrent.TimeUnit;

public class Clazz {
    static Clazz instance = new Clazz();
    final Object block1 = new Object();
    final Object block2 = new Object();


    public void lockObject() {
        synchronized (block1) {
            System.out.println("Block1, Thread: " + Thread.currentThread().getName() + " start.");
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("Block1, Thread: " + Thread.currentThread().getName() + " end.");
        }

        synchronized (block2) {
            System.out.println("Block2, Thread: " + Thread.currentThread().getName() + " start.");
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("Block2, Thread: " + Thread.currentThread().getName() + " end.");
        }
    }

    public static void main(String[] args) {
        new Thread(() -> instance.lockObject()).start();
        new Thread(() -> instance.lockObject()).start();
    }
}
```

output:

```text
Block1, Thread: Thread-0 start.
Block1, Thread: Thread-0 end.
Block2, Thread: Thread-0 start.
Block1, Thread: Thread-1 start.
Block1, Thread: Thread-1 end.
Block2, Thread: Thread-0 end.
Block2, Thread: Thread-1 start.
Block2, Thread: Thread-1 end.
```

### 方法锁形式：synchronized 修饰普通方法，锁对象默认为 this

```java
import java.util.concurrent.TimeUnit;

public class Clazz {
    static Clazz instance = new Clazz();

    public synchronized void method(){
        System.out.println("Thread: " + Thread.currentThread().getName() + " start. ");
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Thread: " + Thread.currentThread().getName() + " end.");
    }

    public static void main(String[] args) {
        new Thread(() -> instance.method()).start();
        new Thread(() -> instance.method()).start();
    }
}
```

output:

```text
Thread: Thread-0 start.
Thread: Thread-0 end.
Thread: Thread-1 start.
Thread: Thread-1 end.
```

## 类锁

指 `synchronize` 修饰静态的方法或指定锁对象为 `Class` 对象

### synchronize 修饰静态方法

**非**静态方法加锁：

```java
import java.util.concurrent.TimeUnit;

public class Clazz {
    static Clazz instance1 = new Clazz();
    static Clazz instance2 = new Clazz();

    public synchronized void method(){
        System.out.println("Thread: " + Thread.currentThread().getName() + " start. ");
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Thread: " + Thread.currentThread().getName() + " end.");
    }

    public static void main(String[] args) {
        new Thread(() -> instance1.method()).start();
        new Thread(() -> instance2.method()).start();
    }
}
```

output:

```text
Thread: Thread-0 start.
Thread: Thread-1 start.
Thread: Thread-0 end.
Thread: Thread-1 end.
```

静态方法加锁：

```java
import java.util.concurrent.TimeUnit;

public class Clazz {
    static Clazz instance1 = new Clazz();
    static Clazz instance2 = new Clazz();

    public static synchronized void method(){
        System.out.println("Thread: " + Thread.currentThread().getName() + " start. ");
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Thread: " + Thread.currentThread().getName() + " end.");
    }

    public static void main(String[] args) {
        new Thread(() -> instance1.method()).start();
        new Thread(() -> instance2.method()).start();
    }
}
```

output:

```text
Thread: Thread-0 start.
Thread: Thread-0 end.
Thread: Thread-1 start.
Thread: Thread-1 end.
```

### synchronized 指定锁对象为 Class 对象

```java
import java.util.concurrent.TimeUnit;

public class Clazz {
    static Clazz instance1 = new Clazz();
    static Clazz instance2 = new Clazz();

    public void method() {
        synchronized (Clazz.class) {
            System.out.println("Thread: " + Thread.currentThread().getName() + " start. ");
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("Thread: " + Thread.currentThread().getName() + " end.");
        }
    }

    public static void main(String[] args) {
        new Thread(() -> instance1.method()).start();
        new Thread(() -> instance2.method()).start();
    }
}
```

output:

```text
Thread: Thread-0 start.
Thread: Thread-0 end.
Thread: Thread-1 start.
Thread: Thread-1 end.
```

## 原理分析

### 加锁和释放锁的原理

![synchronized-principle](/imgs/posts/java-thread-synchronized/synchronized-principle.png)

查看反编译后的文件，关注框选的 `monitorenter` 和 `monitorexit` 即可。

`monitorenter` 和 `monitorexit` 指令，会让对象在执行，使其锁计数器 +1 或者 -1。

每一个对象在同一时间只与一个 monitor(锁)相关联，而一个 monitor 在同一时间只能被一个线程获得，一个对象在尝试获得与这个对象相关联的 monitor 锁的所有权的时候，`monitorenter` 指令会发生如下 3 中情况之一：

1. monitor 计数器为 0，意味着目前还没有被获得，那这个线程就会立刻获得然后把锁计数器 +1，一旦 +1，别的线程再想获取，就需要等待
2. 如果这个 monitor 已经拿到了这个锁的所有权，又重入了这把锁，那锁计数器就会累加，变成 2，并且随着重入的次数，会一直累加
3. 这把锁已经被别的线程获取了，等待锁释放

`monitorexit` 指令：释放对于 monitor 的所有权，释放过程很简单，就是讲 monitor 的计数器 -1，如果减完以后，计数器不是 0，则代表刚才是重入进来的，当前线程还继续持有这把锁的所有权，如果计数器变成 0，则代表当前线程不再拥有该 monitor 的所有权，即释放锁。

下图表现了对象，对象监视器，同步队列以及执行线程状态之间的关系：

![synchronized-principle-1](/imgs/posts/java-thread-synchronized/synchronized-principle-1.png)

该图可以看出，任意线程对 Object 的访问，首先要获得 Object 的监视器，如果获取失败，该线程就进入同步状态，线程状态变为 BLOCKED，当 Object 的监视器占有者释放后，在同步队列中得线程就会有机会重新获取该监视器。

### 可重入原理：加锁次数计数器

可重入：若一个程序或子程序可以“在任意时刻被中断然后操作系统调度执行另外一段代码，这段代码又调用了该子程序不会出错”，则称其为可重入（reentrant 或 re-entrant）的。即当该子程序正在运行时，执行线程可以再次进入并执行它，仍然获得符合设计时预期的结果。与多线程并发执行的线程安全不同，可重入强调对单个线程执行时重新进入同一个子程序仍然是安全的。

可重入锁：又名递归锁，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者 class），不会因为之前已经获取过还没释放而阻塞。

```java
public class Clazz {
    private synchronized void method1() {
        System.out.println(Thread.currentThread().getId() + ": method1()");
        method2();
    }

    private synchronized void method2() {
        System.out.println(Thread.currentThread().getId()+ ": method2()");
        method3();
    }

    private synchronized void method3() {
        System.out.println(Thread.currentThread().getId()+ ": method3()");
    }

    public static void main(String[] args) {
        new Thread(()->new  Clazz().method1()).start();
    }
}
```

1. 初始状态： monitor 计数器 = 0，可获取锁
2. 进入 method1() 方法，执行 monitorenter, monitor 计数器 +1 = 1 (获取到锁)
3. 进入 method2() 方法，执行 monitorenter, monitor 计数器 +1 = 2
4. 进入 method3() 方法，执行 monitorenter, monitor 计数器 +1 = 3
5. 结束 method3() 方法，执行 monitorexit, monitor 计数器 -1 = 2
6. 结束 method2() 方法，执行 monitorexit, monitor 计数器 -1 = 1
7. 结束 method1() 方法，执行 monitorexit, monitor 计数器 -1 = 0 (释放锁)
8. 结束状态：monitor 计数器 = 0，锁被释放

这就是 Synchronized 的重入性，即在同一锁程中，每个对象拥有一个 monitor 计数器，当线程获取该对象锁后，monitor 计数器就会加一，释放锁后就会将 monitor 计数器减一，线程不需要再次获取同一把锁。

### 保证可见性的原理：内存模型和 happens-before 规则

Synchronized 的 happens-before 规则，即监视器锁规则：对同一个监视器的解锁，happens-before 于对该监视器的加锁。

```java
public class MonitorTest {
    private int a = 0;

    public synchronized void writer() {     // 1
        a++;                                // 2
    }                                       // 3

    public synchronized void reader() {    // 4
        int i = a;                         // 5
    }                                      // 6
}
```

该代码的 happens-before 关系如图所示：

![synchronized-happens-before.png](/imgs/posts/java-thread-synchronized/synchronized-happens-before.png)

在图中每一个箭头连接的两个节点就代表之间的 happens-before 关系，黑色的是通过程序顺序规则推导出来，红色的为监视器锁规则推导而出：线程 A 释放锁 happens-before 线程 B 加锁，蓝色的则是通过程序顺序规则和监视器锁规则推测出来 happens-before 关系，通过传递性规则进一步推导的 happens-before 关系。现在我们来重点关注 2 happens-before 5，通过这个关系我们可以得出什么?

根据 happens-before 的定义中的一条:如果 A happens-before B，则 A 的执行结果对 B 可见，并且 A 的执行顺序先于 B。线程 A 先对共享变量 A 进行加一，由 2 happens-before 5 关系可知线程 A 的执行结果对线程 B 可见即线程 B 所读取到的 a 的值为 1。

## JVM 锁优化

简单来说在 JVM 中 monitorenter 和 monitorexit 字节码依赖于底层的操作系统的 Mutex Lock 来实现的，但是由于使用 Mutex Lock 需要将当前线程挂起并从用户态切换到内核态来执行，这种切换的代价是非常昂贵的；然而在现实中的大部分情况下，同步方法是运行在单线程环境(无锁竞争环境)如果每次都调用 Mutex Lock 那么将严重的影响程序的性能。**不过在 jdk1.6 中对锁的实现引入了大量的优化，如锁粗化(Lock Coarsening)、锁消除(Lock Elimination)、轻量级锁(Lightweight Locking)、偏向锁(Biased Locking)、适应性自旋(Adaptive Spinning)等技术来减少锁操作的开销。**

- 锁粗化(Lock Coarsening)：也就是减少不必要的紧连在一起的 unlock，lock 操作，将多个连续的锁扩展成一个范围更大的锁。
- 锁消除(Lock Elimination)：通过运行时 JIT 编译器的逃逸分析来消除一些没有在当前同步块以外被其他线程共享的数据的锁保护，通过逃逸分析也可以在线程本的 Stack 上进行对象空间的分配(同时还可以减少 Heap 上的垃圾收集开销)。
- 轻量级锁(Lightweight Locking)：这种锁实现的背后基于这样一种假设，即在真实的情况下我们程序中的大部分同步代码一般都处于无锁竞争状态(即单线程执行环境)，在无锁竞争的情况下完全可以避免调用操作系统层面的重量级互斥锁，取而代之的是在 monitorenter 和 monitorexit 中只需要依靠一条 CAS 原子指令就可以完成锁的获取及释放。当存在锁竞争的情况下，执行 CAS 指令失败的线程将调用操作系统互斥锁进入到阻塞状态，当锁被释放的时候被唤醒(具体处理步骤下面详细讨论)。
- 偏向锁(Biased Locking)：是为了在无锁竞争的情况下避免在锁获取过程中执行不必要的 CAS 原子指令，因为 CAS 原子指令虽然相对于重量级锁来说开销比较小但还是存在非常可观的本地延迟。
- 适应性自旋(Adaptive Spinning)：当线程在获取轻量级锁的过程中执行 CAS 操作失败时，在进入与 monitor 相关联的操作系统重量级锁(mutex semaphore)前会进入忙等待(Spinning)然后再次尝试，当尝试一定的次数后如果仍然没有成功则调用与该 monitor 关联的 semaphore(即互斥锁)进入到阻塞状态。

### 锁类型

在 Java SE 1.6 里 Synchronized 同步锁，一共有四种状态：`无锁`、`偏向锁`、`轻量级锁`、`重量级锁`，它会随着竞争情况逐渐升级。锁可以升级但是不可以降级，目的是为了提供获取锁和释放锁的效率。

> 锁膨胀方向： 无锁 → 偏向锁 → 轻量级锁 → 重量级锁 (此过程是不可逆的)

### 自旋锁与自适应自旋锁

> 背景：在没有加入锁优化时，Synchronized 是一个非常“胖大”的家伙。在多线程竞争锁时，当一个线程获取锁时，它会阻塞所有正在竞争的线程，这样对性能带来了极大的影响。在挂起线程和恢复线程的操作都需要转入内核态中完成，这些操作对系统的并发性能带来了很大的压力。同时 HotSpot 团队注意到在很多情况下，共享数据的锁定状态只会持续很短的一段时间，为了这段时间去挂起和回复阻塞线程并不值得。在如今多处理器环境下，完全可以让另一个没有获取到锁的线程在门外等待一会(自旋)，但不放弃 CPU 的执行时间。等待持有锁的线程是否很快就会释放锁。为了让线程等待，我们只需要让线程执行一个忙循环(自旋)，这便是自旋锁由来的原因

自旋锁早在 JDK1.4 中就引入了，只是当时默认时关闭的。在 JDK 1.6 后默认为开启状态。自旋锁本质上与阻塞并不相同，先不考虑其对多处理器的要求，如果锁占用的时间非常的短，那么自旋锁的性能会非常的好，相反，其会带来更多的性能开销(因为在线程自旋时，始终会占用 CPU 的时间片，如果锁占用的时间太长，那么自旋的线程会白白消耗掉 CPU 资源)。因此自旋等待的时间必须要有一定的限度，如果自旋超过了限定的次数仍然没有成功获取到锁，就应该使用传统的方式去挂起线程了，在 JDK 定义中，自旋锁默认的自旋次数为 10 次，用户可以使用参数 `-XX:PreBlockSpin` 来更改。可是现在又出现了一个问题：如果线程锁在线程自旋刚结束就释放掉了锁，那么是不是有点得不偿失。所以这时候我们需要更加聪明的锁来实现更加灵活的自旋。来提高并发的性能。(这里则需要自适应自旋锁！)

在 JDK 1.6 中引入了**自适应自旋锁**。这就意味着自旋的时间不再固定了，而是由前一次在同一个锁上的自旋 时间及锁的拥有者的状态来决定的。如果在同一个锁对象上，自旋等待刚刚成功获取过锁，并且持有锁的线程正在运行中，那么 JVM 会认为该锁自旋获取到锁的可能性很大，会自动增加等待时间。比如增加到 100 此循环。相反，如果对于某个锁，自旋很少成功获取锁。那再以后要获取这个锁时将可能省略掉自旋过程，以避免浪费处理器资源。有了自适应自旋，JVM 对程序的锁的状态预测会越来越准确，JVM 也会越来越聪明。

### 锁消除

锁消除是指虚拟机即时编译器再运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行消除。锁消除的主要判定依据来源于逃逸分析的数据支持。意思就是：JVM 会判断在一段程序中的同步明显不会逃逸出去从而被其他线程访问到，那 JVM 就把它们当作栈上数据对待，认为这些数据是线程独有的，不需要加同步。此时就会进行锁消除。​ 当然在实际开发中，我们很清楚的知道哪些是线程独有的，不需要加同步锁，但是在 Java API 中有很多方法都是加了同步的，那么此时 JVM 会判断这段代码是否需要加锁。如果数据并不会逃逸，则会进行锁消除。比如如下操作：在操作 String 类型数据时，由于 String 是一个不可变类，对字符串的连接操作总是通过生成的新的 String 对象来进行的。因此 Javac 编译器会对 String 连接做自动优化。在 JDK 1.5 之前会使用 StringBuffer 对象的连续 append()操作，在 JDK 1.5 及以后的版本中，会转化为 StringBuilder 对象的连续 append()操作。

![synchronized-remove-lock.png](/imgs/posts/java-thread-synchronized/synchronized-remove-lock.png)

众所周知，StringBuilder 不是安全同步的，但是在上述代码中，JVM 判断该段代码并不会逃逸，则将该代码带默认为线程独有的资源，并不需要同步，所以执行了锁消除操作。(还有 Vector 中的各种操作也可实现锁消除。在没有逃逸出数据安全防卫内)

### 锁粗化

​ 原则上，我们都知道在加同步锁时，尽可能的将同步块的作用范围限制到尽量小的范围(只在共享数据的实际作用域中才进行同步，这样是为了使得需要同步的操作数量尽可能变小。在存在锁同步竞争中，也可以使得等待锁的线程尽早的拿到锁)。

​ 大部分上述情况是完美正确的，但是如果存在连串的一系列操作都对同一个对象反复加锁和解锁，甚至加锁操作时出现在循环体中的，那即使没有线程竞争，频繁的进行互斥同步操作也会导致不必要的性能操作。

这里贴上根据上述 Javap 编译的情况编写的实例 Java 类

```java
    public static String stringTest(String s1, String s2, String s3) {
        StringBuffer sb = new StringBuffer();
        sb.append(s1);
        sb.append(s2);
        sb.append(s3);
        return sb.toString();
    }
```

​ 在上述的连续 append()操作中就属于这类情况。JVM 会检测到这样一连串的操作都是对同一个对象加锁，那么 JVM 会将加锁同步的范围扩展(粗化)到整个一系列操作的 外部，使整个一连串的 append()操作只需要加锁一次就可以了。

## 理解

synchronized 是通过软件(JVM)实现的，简单易用，即使在 JDK5 之后有了 Lock，仍然被广泛的使用。

### 使用 Synchronized 有哪些要注意的

1. 锁对象不能为空，因为锁的信息都保存在对象头里
2. 作用域不宜过大，影响程序执行的速度，控制范围过大，编写代码也容易出错
3. 避免死锁
4. 在能选择的情况下，既不要用 Lock 也不要用 synchronized 关键字，用 java.util.concurrent 包中的各种各样的类，如果不用该包下的类，在满足业务的情况下，可以使用 synchronized 关键，因为代码量少，避免出错

### synchronized 是公平锁吗

synchronized 实际上是非公平的，新来的线程有可能立即获得监视器，而在等待区中等候已久的线程可能再次等待，这样有利于提高性能，但是也可能会导致饥饿现象
