---
title: '面试问题记录'
description: '郑州 10+ 面试问题记录'
keywords: 'interview,questions'

date: 2024-06-05T17:25:26+08:00

categories:
  - Doc
tags:
  - Questions
---

记录郑州后端 Java 开发, 面试遇到的问题记录总结。

<!--more-->

## 数据结构

### 红黑树

红黑树是一种自平衡二叉搜索树，其特点是通过在插入和删除节点时进行适当的旋转和重新着色操作，确保树的高度保持在对数级别，从而保证插入、删除和查找操作的时间复杂度都是 _O_(log _n_)。

红黑树有以下五个基本性质：

1. **节点是红色或黑色**：
   - 每个节点都是红色或黑色。
2. **根节点是黑色**：
   - 根节点总是黑色的。
3. **所有叶子节点（NIL 节点）是黑色**：
   - 这里的叶子节点是指树的外部节点，即树中没有子节点的那些终端节点，被认为是黑色的空节点。
4. **红色节点的两个子节点都是黑色**：
   - 不能有两个连续的红色节点（即红色节点不能有红色的父节点或子节点）。
5. **从任一节点到其每个叶子的所有路径包含相同数目的黑色节点**：
   - 这个属性确保了没有一条路径会比其他路径长两倍，从而保证了树的平衡。

红黑树的插入和删除操作较复杂，需要在普通二叉搜索树的插入和删除操作基础上，进行额外的调整（旋转和重新着色）来保持红黑树的性质。以下是这些操作的简要说明：

- 插入操作
  1. 插入新节点：
     - 插入新节点时，按照二叉搜索树的规则插入新节点，并将新节点标记为红色。
  2. 调整树：
     - 插入后可能会违反红黑树的性质，尤其是插入节点是红色的，会导致两个连续的红色节点。
     - 通过重新着色和旋转操作来恢复红黑树的平衡和性质。
- 删除操作
  1. 删除节点：
     - 删除节点时，按照二叉搜索树的规则删除节点。
  2. 调整树：
     - 删除操作可能会导致红黑树的性质被破坏，特别是会影响黑色节点数量一致性的性质。
     - 通过重新着色和旋转操作来恢复红黑树的平衡和性质。

红黑树由于其平衡性和高效的操作性能，被广泛应用于计算机科学中的许多领域，包括：

- 实现关联数组和集合：如 C++中的`std::map`和`std::set`，Java 中的`TreeMap`和`TreeSet`。
- 作为系统底层数据结构：如 Linux 内核中的完全公平调度器（CFS）和许多数据库系统中的索引结构。

例子：

```txt
      10(B)
     /    \
   5(R)  20(B)
  /  \      \
3(B)  7(B)  30(R)
```

在这个例子中：

- 根节点 10 是黑色的。
- 所有叶子节点（如 3, 7, 30）到根节点的路径上有相同数目的黑色节点。
- 红色节点 5 的子节点 3 和 7 都是黑色的。

总结

> 红黑树通过额外的颜色属性和旋转操作，确保树在插入和删除节点后仍保持平衡，从而保证了高效的查找、插入和删除操作。它是许多系统和应用程序中重要的数据结构，提供了良好的性能和可靠性。

### LVS 调度算法

LVS（Linux Virtual Server）是一种用于负载均衡的技术，可以将网络流量分发到多台服务器，以提高系统的可扩展性和可靠性。在 LVS 中，有几种常用的调度算法，用于决定如何分配客户端请求到后端服务器。以下是几种主要的 LVS 调度算法及其特点：

#### 轮询调度（Round Robin, RR）

**特点**：

- 按照简单的轮询方式将请求依次分配给每台服务器。
- 不考虑服务器的当前负载。

**优点**：

- 简单易实现。
- 适合负载均衡比较均匀的场景。

**缺点**：

- 不考虑服务器性能和负载，可能导致性能不均衡。

#### 加权轮询调度（Weighted Round Robin, WRR）

**特点**：

- 在轮询调度的基础上，考虑服务器的性能，为每台服务器分配一个权重。
- 权重高的服务器分配到的请求更多。

**优点**：

- 可以根据服务器性能进行合理分配，提高整体性能。
- 比 RR 更灵活。

**缺点**：

- 权重设置不当可能导致负载不均衡。

#### 最少连接调度（Least Connections, LC）

**特点**：

- 优先将新请求分配给当前连接数最少的服务器。
- 动态调整负载，适合长连接服务。

**优点**：

- 能够较好地平衡负载，适应变化的请求模式。

**缺点**：

- 需要实时统计每台服务器的连接数，开销较大。

#### 加权最少连接调度（Weighted Least Connections, WLC）

**特点**：

- 在最少连接调度的基础上，考虑服务器的权重。
- 权重高的服务器在相同连接数下分配更多请求。

**优点**：

- 结合了 WLC 和 LC 的优点，能够更好地平衡负载。

**缺点**：

- 实现和维护复杂度较高。

#### 最短期望延迟调度（Shortest Expected Delay, SED）

**特点**：

- 计算每台服务器的期望延迟，将请求分配给期望延迟最短的服务器。
- 考虑连接数和服务器权重。

**优点**：

- 可以较好地预测和减少请求的延迟。

**缺点**：

- 需要计算期望延迟，增加了复杂度。

#### 最少队列调度（Never Queue, NQ）

**特点**：

- 优先将请求分配给当前没有排队请求的服务器。
- 如果所有服务器都在排队，则按照轮询方式分配。

**优点**：

- 减少请求排队等待时间，提高响应速度。

**缺点**：

- 实现和维护较为复杂。

LVS 的这些调度算法提供了多种方式来优化负载分配，可以根据具体的应用场景和需求选择最合适的算法。例如，对于需要快速响应的 Web 服务，可以选择 NQ 或 SED；对于计算密集型任务，可以选择 WRR 或 WLC 以更好地利用服务器资源。

### Nginx 负载均衡算法

Nginx 是一款功能强大的 HTTP 和反向代理服务器，广泛用于负载均衡和高性能网站。Nginx 支持多种负载均衡方式，能够根据不同的需求和场景选择合适的调度算法。以下是 Nginx 支持的主要负载均衡方式及其特点：

#### 轮询（Round Robin）

**特点**：

- 请求依次分配给每个后端服务器。
- 是默认的负载均衡方式。

**优点**：

- 实现简单。
- 适合负载较均匀的场景。

**缺点**：

- 不考虑后端服务器的性能和当前负载。

**配置示例**：

```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
}
```

#### 加权轮询（Weighted Round Robin）

**特点**：

- 在轮询的基础上，分配权重更高的服务器更多请求。
- 适用于服务器性能不一致的场景。

**优点**：

- 更灵活，可以根据服务器性能分配负载。

**缺点**：

- 权重设置不当可能导致负载不均衡。

**配置示例**：

```nginx
upstream backend {
    server backend1.example.com weight=3;
    server backend2.example.com weight=1;
}
```

#### 最少连接（Least Connections）

**特点**：

- 优先将请求分配给当前连接数最少的服务器。
- 适合长连接场景，如 WebSocket 和数据库连接。

**优点**：

- 动态平衡负载，适应变化的请求模式。

**缺点**：

- 实现和计算连接数有一定的开销。

**配置示例**：

```nginx
upstream backend {
    least_conn;
    server backend1.example.com;
    server backend2.example.com;
}
```

#### IP 哈希（IP Hash）

**特点**：

- 根据客户端 IP 地址的哈希值分配请求到固定的服务器。
- 适用于需要会话保持的场景。

**优点**：

- 同一客户端 IP 的请求总是分配给同一台服务器。

**缺点**：

- 负载分布可能不均匀。

**配置示例**：

```nginx
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
}
```

#### Hash（通用哈希）

**特点**：

- 基于指定的变量（如 URI、cookie 等）的哈希值进行分配。
- 提供了更多定制化选项。

**优点**：

- 可以根据具体需求实现复杂的负载均衡策略。

**缺点**：

- 配置和管理复杂度较高。

**配置示例**：

```nginx
upstream backend {
    hash $request_uri;
    server backend1.example.com;
    server backend2.example.com;
}
```

#### 随机（Random with Two Choices）

**特点**：

- 随机选择两台服务器，然后选择负载较轻的一台。
- 提供了一种简单但有效的负载均衡方式。

**优点**：

- 能够在简单的情况下提供较好的负载分布。

**缺点**：

- 可能在某些情况下不如其他算法精确。

**配置示例**：

```nginx
upstream backend {
    random two least_conn;
    server backend1.example.com;
    server backend2.example.com;
}
```

## Java 相关

### Java8 中新增了什么特性

Java 8 是一个重大版本，引入了许多新特性和改进，其中一些主要的新特性包括：

1. **Lambda 表达式和函数式接口**： Lambda 表达式是 Java 8 中最显著的新特性之一，它使得代码更加简洁和易读。同时，Java 8 还引入了函数式接口来支持 Lambda 表达式，如 java.util.function 包中的接口。
2. **流（Stream）API**： 流 API 提供了一种处理集合数据的新方式，它可以让你更容易地对集合进行筛选、映射、排序等操作，以函数式编程的方式来处理数据。
3. **默认方法（Default Methods）**： 接口中可以包含默认方法，这些方法可以在接口中进行实现，而不需要实现类来实现它们。这样做的目的是为了向已存在的接口添加新的方法，而不会破坏已有的实现。
4. **方法引用（Method References）**： 方法引用是一种更简洁的 Lambda 表达式的写法，它可以直接引用现有方法作为 Lambda 表达式的实现。
5. **Optional 类**： Optional 类是一个容器对象，用来表示一个值存在或不存在。它可以帮助开发者更容易地处理空指针异常。
6. **新的日期和时间 API（Date and Time API）**： Java 8 引入了全新的日期和时间 API，包括 java.time 包中的新类，以及用于处理日期、时间、时区等的新方法和工具类。
7. **CompletableFuture 类**： CompletableFuture 类是 Java 8 中新增的一个用于异步编程的类，它提供了更便捷的方式来处理异步任务和并发编程。
8. **并行数组（Parallel Arrays）**： Java 8 中新增了一系列支持并行操作的数组方法，如 parallelSort()方法等，可以更高效地处理大规模数据集。

这些新特性使得 Java 8 更加现代化和强大，提高了开发者的生产力和代码的质量。

### Java 中接口（Interface）和抽象类（Abstract Class）都是用来实现抽象化的机制，但它们之间有那些关键的区别

1. **方法实现**：
   - **接口**： 接口中的方法都是抽象的，即只有方法的声明，没有方法的实现。在 Java 8 之后，接口还可以包含默认方法（default method）和静态方法（static method），但它们必须提供实现。
   - **抽象类**： 抽象类中可以包含抽象方法（abstract method），也可以包含具体方法的实现。抽象类中的抽象方法必须由子类来实现或者由子类继续声明为抽象方法。
2. **多继承**：
   - **接口**： Java 中的类可以实现多个接口，从而具备多态的特性。一个类可以实现多个接口，但只能继承一个类。
   - **抽象类**： Java 中的类只能继承一个抽象类，不能继承多个抽象类。因此，抽象类的多态性相对受限。
3. **成员变量**：
   - **接口**： 接口中只能包含常量（static final）类型的成员变量，不能包含实例变量。
   - **抽象类**： 抽象类中可以包含常量、实例变量以及其他类型的成员变量。
4. **构造方法**：
   - **接口**： 接口中不能包含构造方法。
   - **抽象类**： 抽象类可以包含构造方法，用于子类实例化时调用。

适用场景：

- **接口**： 当多个无关的类需要遵循同一套行为规范时，使用接口是更好的选择。例如，比如定义可飞行（Flyable）接口，让鸟类（Bird）和飞机（Airplane）类实现该接口。
- **抽象类**： 当存在一组类具有共同的行为，但其中部分行为需要子类来实现时，可以考虑使用抽象类。抽象类提供了一种将公共行为放在一起的方式，并且可以为子类提供一些通用的方法实现。

### 解释死锁产生的条件，并提供一段简单的 Java 代码模拟死锁场景

死锁是指两个或多个进程在竞争有限的资源时，由于彼此持有对方所需资源的锁而相互等待的状态。死锁通常需要满足以下四个条件：

1. **互斥条件（Mutual Exclusion）**： 至少有一个资源是被排他性使用的，即一次只能被一个进程占用。如果一个进程在使用该资源时，其他进程无法访问，那么就说这个资源具有互斥性。
2. **占有且等待条件（Hold and Wait）**： 进程至少已经占有一个资源，并且在等待获取另一个资源，但是这时不释放已经占有的资源。
3. **非抢占条件（No Preemption）**： 资源不能被强行从一个进程中夺取，它只能被持有它的进程主动释放。
4. **循环等待条件（Circular Wait）**： 存在一个进程等待队列 {P1, P2, ..., Pn}，其中 P1 等待 P2 占有的资源，P2 等待 P3 占有的资源，...，Pn 等待 P1 占有的资源，形成了一个循环等待的环路。

以下是一个简单的 Java 代码模拟死锁的场景：

```java
public class DeadlockExample {
    private static Object lock1 = new Object();
    private static Object lock2 = new Object();

    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            synchronized (lock1) {
                System.out.println("Thread 1: Holding lock 1...");

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                System.out.println("Thread 1: Waiting for lock 2...");
                synchronized (lock2) {
                    System.out.println("Thread 1: Holding lock 1 & 2...");
                }
            }
        });

        Thread thread2 = new Thread(() -> {
            synchronized (lock2) {
                System.out.println("Thread 2: Holding lock 2...");

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                System.out.println("Thread 2: Waiting for lock 1...");
                synchronized (lock1) {
                    System.out.println("Thread 2: Holding lock 1 & 2...");
                }
            }
        });

        thread1.start();
        thread2.start();
    }
}
```

在这个例子中，两个线程分别尝试获取锁 1 和锁 2，但它们的获取顺序是相反的。这样，当线程 1 获取了锁 1 并等待锁 2 时，线程 2 已经获取了锁 2 并等待锁 1，这就形成了循环等待的情况，导致了死锁。

### 描述 Java 内存模型（JVM）中主内存和工作内存的概念，以及他们是如何交互的

Java 内存模型（Java Memory Model，JMM）定义了 Java 程序中各种变量的访问规则和行为，以确保多线程环境下的可见性、有序性和原子性。JMM 中的主要概念包括主内存（Main Memory）和工作内存（Working Memory）。

1. **主内存（Main Memory）**： 主内存是 Java 线程共享的内存区域，所有线程都可以访问。主内存中存储了所有的共享变量，包括实例字段、静态字段和数组元素。主内存是所有线程的“源头”，当一个线程修改了主内存中的共享变量时，其他线程可以通过刷新工作内存或者重新加载来感知到这个修改。
2. **工作内存（Working Memory）**： 每个 Java 线程都拥有自己的工作内存，工作内存存储了主内存中的一部分变量副本。线程对变量的所有操作都必须在工作内存中进行，包括读取、赋值和锁定。线程之间无法直接访问对方的工作内存，线程间的数据交互必须通过主内存来完成。

工作内存和主内存之间的交互遵循以下规则：

1. **读取操作**： 当一个线程要读取一个变量的值时，它首先从主内存中读取该变量的值到自己的工作内存中，然后再进行操作。
2. **修改操作**： 当一个线程要修改一个变量的值时，它首先从主内存中读取该变量的值到自己的工作内存中，然后修改工作内存中的值，最后再将修改后的值刷新到主内存中。
3. **同步操作**： 同步操作包括锁定和解锁，当一个线程要锁定一个变量时，它会将这个变量的锁标志设置为自己的线程 ID，并清除其他线程的锁标志；当一个线程要解锁一个变量时，它会清除该变量的锁标志。同步操作保证了多个线程之间的可见性和原子性。

通过这些规则，Java 内存模型确保了多线程环境下的线程安全性和数据一致性。

### 单例模式会被垃圾回收吗？为什么

**单例模式**中创建的单例对象通常**不会被垃圾回收**，因为单例对象的生命周期与整个应用程序的生命周期相同，并且在整个应用程序的执行过程中始终保持活动状态。垃圾回收器（Garbage Collector）通常只会回收那些**没有被任何引用指向**的对象，而单例对象在应用程序中通常会**被持有引用**，以确保它们能够被访问和使用。

具体来说，单例模式通过**静态变量或者静态方法来创建单例对象**，这些静态变量或方法在整个应用程序的生命周期中始终存在于内存中，并且可以被其他部分的代码访问。因此，单例对象会一直被引用，垃圾回收器无法将其回收。

但是，在某些特殊情况下，如果单例对象的引用被释放，即使单例模式创建的对象也可以被垃圾回收。例如，如果单例对象是通过弱引用（Weak Reference）持有的，当没有强引用指向该对象时，垃圾回收器可以回收它。但是这种情况属于特例，不是单例模式的一般情况。

### JVM 有哪些组成部分

Java 虚拟机（JVM）是 Java 程序运行的核心组件，它负责将 Java 字节码转换为特定平台上的机器码并执行。JVM 主要由以下几个组成部分组成：

1. **类加载器（ClassLoader）**： 类加载器负责加载 Java 类文件到 JVM 中，并将其转换为运行时数据结构。类加载器通常分为三个层次：启动类加载器（Bootstrap ClassLoader）、扩展类加载器（Extension ClassLoader）和应用程序类加载器（Application ClassLoader）。
2. **运行时数据区（Runtime Data Area）**： 运行时数据区是 JVM 内存管理的核心部分，它主要包括方法区（Method Area）、堆（Heap）、虚拟机栈（Virtual Machine Stack）、本地方法栈（Native Method Stack）和程序计数器（Program Counter）等。
   - **方法区（Method Area）**： 用于存储类信息、常量、静态变量等数据，是各个线程共享的内存区域。
   - **堆（Heap）**： 用于存储对象实例和数组对象，是 Java 内存管理中最主要的内存区域。
   - **虚拟机栈（Virtual Machine Stack）**： 用于存储方法调用的局部变量、操作数栈、动态链接和方法返回地址等信息。
   - **本地方法栈（Native Method Stack）**： 与虚拟机栈类似，但是用于执行本地方法（Native Method）的栈。
   - **程序计数器（Program Counter）**： 记录当前线程执行的字节码指令地址，是线程私有的内存区域。
3. **执行引擎（Execution Engine）**： 执行引擎负责执行 JVM 中的字节码指令，通常包括解释器（Interpreter）、即时编译器（Just-In-Time Compiler，JIT Compiler）和垃圾收集器（Garbage Collector）等组件。
   - **解释器（Interpreter）**： 将字节码指令解释为机器码并逐条执行。
   - **即时编译器（Just-In-Time Compiler，JIT Compiler）**： 将字节码编译为本地机器码，以提高执行效率。
   - **垃圾收集器（Garbage Collector）**： 负责管理和回收堆中的内存，以释放不再使用的对象所占用的内存空间。
4. **本地方法接口（Native Interface）**： 本地方法接口允许 Java 应用程序调用本地方法，即使用 C、C++或其他本地语言编写的方法。

这些组成部分共同构成了 Java 虚拟机（JVM），实现了 Java 程序的跨平台性和自动内存管理。

### 多线程下如何确保数据一致性

在多线程环境下确保数据一致性是非常重要的，可以通过以下几种方法来实现：

1. **使用同步机制**： 在共享数据的访问中使用同步机制（如 synchronized 关键字、ReentrantLock 等），确保每次只有一个线程能够访问共享数据，从而避免多个线程同时修改数据导致的不一致性。
2. **使用 volatile 关键字**： 如果共享数据只是进行读取操作而不涉及写入，可以使用 volatile 关键字来保证可见性。volatile 关键字可以确保线程之间对变量的修改对其他线程可见，从而避免了因为线程之间缓存数据不一致而导致的数据不一致性问题。
3. **使用线程安全的数据结构**： Java 提供了一些线程安全的数据结构，如 ConcurrentHashMap、CopyOnWriteArrayList 等，这些数据结构内部实现了同步机制，可以保证在多线程环境下的线程安全性。
4. **使用原子类**： Java 提供了一系列原子类（Atomic Classes），如 AtomicInteger、AtomicLong 等，它们提供了一系列原子操作，可以确保在多线程环境下对共享变量的操作是原子性的，从而避免了因为并发操作导致的数据不一致性问题。
5. **使用线程安全的设计模式**： 在程序设计中，可以使用一些线程安全的设计模式，如单例模式、享元模式等，来确保在多线程环境下数据的一致性和安全性。

综上所述，确保数据一致性在多线程环境下是非常重要的，可以通过合适的同步机制、原子操作、线程安全的数据结构以及设计模式等手段来实现。

### HashMap 底层实现原理

### Garbage Collection Roots

在 Java 中，GC Roots 是垃圾回收器的起点，用于标识哪些对象是活跃的，并且不应该被回收。GC Roots 是指一组特定类型的引用，垃圾回收器从这些引用开始遍历对象图，找出所有可达的对象。以下是 Java 中所有的 GC Roots 类型：

1. **虚拟机栈（栈帧中的本地变量表）中的引用**：
   - 所有活动线程的栈中引用的对象。
   - 这些引用通常是方法调用的局部变量和参数。
2. **(static) 方法区中的类静态属性引用的对象**：
   - 类的静态变量引用的对象。
   - 即使类没有实例化，静态变量仍然存在于方法区内。
3. **(final) 方法区中的常量引用的对象**：
   - 常量池中的常量引用的对象，如字符串常量池中的字符串。
4. **(native) 本地方法栈中的 JNI 引用**：
   - 通过 JNI（Java Native Interface）创建的引用对象。
5. **(Thread) 活动线程**：
   - 所有活动线程本身也是 GC Roots。
6. **(ClassLoader) 系统类加载器**：
   - 加载系统类的类加载器（如 Bootstrap ClassLoader）。
7. **(synchronized) 被同步锁持有的对象**：
   - 持有 synchronized 锁的对象也是 GC Roots。

示例代码：

1. 栈上的引用

   ```java
   public class GCRootExample {
      public static void main(String[] args) {
         MyObject myObject = new MyObject();
         // myObject 是 GC Root
      }
   }
   ```

2. 静态变量引用

   ```java
   public class GCRootExample {
      private static MyObject staticObject = new MyObject();
      public static void main(String[] args) {
         // staticObject 是 GC Root
      }
   }
   ```

3. 常量引用

   ```java
   public class GCRootExample {
      public static final MyObject constantObject = new MyObject();
      public static void main(String[] args) {
         // constantObject 是 GC Root
      }
   }
   ```

4. 本地方法引用

   ```java
   public class GCRootExample {
      public static void main(String[] args) {
         MyObject myObject = getObjectFromNative();
         // myObject 是 GC Root
      }

      public static native MyObject getObjectFromNative();
   }
   ```

5. 活动线程

   ```java
   public class GCRootExample {
      public static void main(String[] args) {
         Thread thread = new Thread(() -> {
               // 线程对象本身是 GC Root
         });
         thread.start();
      }
   }
   ```

6. 系统类加载器

   ```java
   public class GCRootExample {
      public static void main(String[] args) {
         ClassLoader classLoader = ClassLoader.getSystemClassLoader();
         // classLoader 是 GC Root
      }
   }
   ```

7. 同步锁持有的对象

   ```java
   public class GCRootExample {
      public static void main(String[] args) {
         MyObject myObject = new MyObject();
         synchronized (myObject) {
               // 持有同步锁的 myObject 是 GC Root
         }
      }
   }
   ```

## 数据库相关

### 编写一个 SQL 查询，用于统计每个部门薪资最高的员工信息，包括员工姓名、部门名称和最高薪资

```mysql
SELECT
    e.name AS employee_name,
    d.department_name,
    MAX(e.salary) AS max_salary
FROM
    employees e
INNER JOIN
    departments d ON e.department_id = d.department_id
WHERE
    (e.department_id, e.salary) IN
    (SELECT
         department_id, MAX(salary)
     FROM
         employees
     GROUP BY
         department_id)
GROUP BY
    e.department_id, d.department_name;
```

这个查询首先通过内连接将员工表（employees）和部门表（departments）连接起来，然后使用子查询来找到每个部门的最高薪资。最后，通过再次连接部门表，并且在子查询的结果中查找匹配的部门 ID 和薪资，来得到每个部门薪资最高的员工信息。

### MySQL 事务, 隔离级别

具体内容请查看 [事务详解](../database-mysql/#事务)

### 联合索引单个索引的区别

- **多个单个索引**

  多个单个索引是指在多个不同的字段上分别创建单个索引。例如，如果有两个字段 `A` 和 `B`，分别创建在 `A` 和 `B` 上的单个索引：

  ```sql
   CREATE INDEX idx_A ON table_name (A);
   CREATE INDEX idx_B ON table_name (B);
  ```

- **联合索引**

  联合索引是指在多个字段上创建一个组合索引。例如，在字段 `A` 和 `B` 上创建联合索引：

  ```sql
  CREATE INDEX idx_AB ON table_name (A, B);
  ```

#### 主要区别

1. **查询优化**：
   - **多个单个索引**：只能独立地优化各自字段的查询。MySQL 可能会使用“索引合并”策略，但性能提升有限。
   - **联合索引**：可以优化基于多个字段的组合查询，特别是当查询条件包含这些字段时。
2. **覆盖索引**：
   - **多个单个索引**：无法成为覆盖索引。
   - **联合索引**：如果查询只涉及联合索引中的字段，可以成为覆盖索引，减少数据表访问。
3. **字段顺序**：
   - **多个单个索引**：没有顺序问题。
   - **联合索引**：字段顺序非常重要，MySQL 会按顺序使用索引，只有从最左字段开始的连续字段才能利用索引。

#### 索引生效的条件

1. 单个索引生效条件：
   - 查询中包含索引字段的过滤条件，例如 `WHERE A = value` 或 `WHERE B = value`。
   - 范围查询，如 `WHERE A > value` 或 `WHERE B < value`。
2. 联合索引生效条件：
   - 最左前缀匹配：必须从联合索引的最左字段开始使用。
     - 如 `CREATE INDEX idx_AB ON table_name (A, B)`，查询 `WHERE A = value` 和 `WHERE A = value AND B = value` 会使用索引，但 `WHERE B = value` 不会使用索引。
   - 匹配联合索引的前缀字段：如 `WHERE A = value AND B = value` 或 `WHERE A = value AND B > value`。
   - 排序优化：如果查询的 `ORDER BY` 子句匹配联合索引的字段顺序。
3. 索引合并：
   - MySQL 有时会尝试使用多个单个索引进行“索引合并”优化，使用多个索引的交集或并集来加速查询，但效果通常不如联合索引。

#### 索引示例

假设有一个包含 `first_name` 和 `last_name` 字段的用户表：

- 使用多个单个索引

  ```sql
  CREATE INDEX idx_first_name ON users (first_name);
  CREATE INDEX idx_last_name ON users (last_name);
  ```

  - 查询 `WHERE first_name = 'John'` 或 `WHERE last_name = 'Doe'` 会分别使用 `idx_first_name` 或 `idx_last_name`。
  - 查询 `WHERE first_name = 'John' AND last_name = 'Doe'` 可能会尝试索引合并，但性能不如联合索引。

- 使用联合索引

  ```sql
  CREATE INDEX idx_full_name ON users (first_name, last_name);
  ```

  - 查询 `WHERE first_name = 'John'` 会使用 `idx_full_name`。
  - 查询 `WHERE first_name = 'John' AND last_name = 'Doe'` 会充分利用 `idx_full_name`。
  - 查询 `WHERE last_name = 'Doe'` 不会使用 `idx_full_name`。

#### 索引生效的注意事项

- **选择性**：索引字段应具有高选择性（即字段值唯一性高），低选择性字段通常不适合作为索引。
- **查询模式**：设计索引时应考虑查询模式，确保索引能够覆盖最常用的查询。
- **表大小**：小表不需要过多索引，因为全表扫描的开销相对较低。
- **维护开销**：插入、更新、删除操作会增加索引维护的成本，过多的索引会影响写操作性能。

总结来说，多个单个索引和联合索引在查询优化和使用场景上有显著区别，合理选择和设计索引可以显著提升数据库性能。

### 如何判断有没有走索引

在 MySQL 中，可以使用 EXPLAIN 语句来查看查询的执行计划。例如：

```sql
EXPLAIN SELECT * FROM your_table WHERE your_column = 'some_value';
```

运行上述语句后，你会看到一个包含查询计划的表。重要的字段包括：

- **key**：显示查询使用的索引名称。如果该字段是 NULL，表示没有使用索引。
- **possible_keys**：显示可能使用的索引。
- **rows**：估计需要扫描的行数。使用索引时，这个值通常会比全表扫描要小。

## 服务治理

### 在分布式系统中，如何保证数据的一致性？请至少提出两种常用的解决方案并简要说明

在分布式系统中，保证数据的一致性是一个重要的挑战，因为数据分布在不同的节点上，并且可能会被多个并发操作影响。以下是一些常用的解决方案：

1. **ACID 事务**：在传统的关系型数据库系统中，ACID（原子性、一致性、隔离性、持久性）事务确保了数据的一致性。当多个操作被包含在同一个事务中时，它们要么全部成功执行，要么全部失败，并且在任何时刻数据库的状态都是一致的。
2. **基于日志的复制**：通过记录所有的数据操作，然后在不同的节点上重放这些操作来实现数据的复制。这种方法通常用于主从复制和多主复制的情况下，确保数据的一致性。
3. **分布式一致性协议**：例如 Paxos 和 Raft，这些协议提供了一种在分布式系统中达成一致性的方法。它们通过协调节点之间的通信和投票来确保数据的一致性。
4. **分布式事务**：像 2PC（两阶段提交）和 3PC（三阶段提交）这样的协议可以在分布式系统中实现事务的一致性。在这些协议中，所有节点都会就事务的提交达成一致意见，以确保数据在不同节点之间的一致性。
5. **版本向量**：在一些分布式数据库中，使用版本向量来跟踪不同节点上数据的变化，并通过比较版本向量来解决冲突，确保数据的一致性。

这些解决方案中的每一种都有其适用的场景和局限性，选择合适的方案取决于系统的需求和特性。

### Docker 的底层实现原理是什么

Docker 底层的实现主要依赖于几种关键技术和概念，包括容器化技术、文件系统、网络和命名空间。下面是 Docker 底层实现的主要组件和工作原理：

1. **容器化技术**
   Docker 使用容器来运行应用程序。容器是一个轻量级、独立、可执行的软件包，它包含了运行应用程序所需的所有环境。与虚拟机不同，容器共享宿主操作系统的内核，从而实现高效的资源利用。
2. **Namespaces（命名空间）**
   Namespaces 提供了一个隔离机制，使得容器看起来像是一个独立的操作系统实例。主要的命名空间包括：
   - **PID Namespace**：进程隔离，容器中的进程只看到自己进程树中的进程
   - **NET Namespace**：网络隔离，每个容器都有自己的网络设备、IP 地址和路由表
   - **IPC Namespace**：进程间通信隔离，隔离容器间的消息队列、信号量和共享内存
   - **MNT Namespace**：挂载点隔离，隔离容器之间的文件系统挂载点
   - **UTS Namespace**：主机名和域名隔离，容器可以拥有自己的主机名和域名
3. **Cgroups（Control Groups）**
   Cgroups 用于资源限制和监控。它允许限制容器的 CPU、内存、磁盘 I/O 和网络带宽等资源使用。这样可以确保一个容器的资源使用不会影响到其他容器或宿主系统。
4. **Union File Systems（联合文件系统）**
   Docker 使用联合文件系统（如 OverlayFS）来实现轻量级和高效的文件系统层。Docker 镜像由多个只读层组成，每一层代表了文件系统的一次变更。容器启动时，会在这些只读层之上添加一个可写层。这样的设计使得 Docker 镜像的存储和传输都非常高效。
5. **容器镜像**
   Docker 镜像是一个只读模板，用于创建 Docker 容器。每个镜像由一系列分层文件系统构成。Docker 使用 Copy-on-Write 策略来提高性能和资源利用效率。
6. **Docker Daemon（守护进程）**
   Docker Daemon 是 Docker 的后台服务，负责管理容器的生命周期。它处理来自 Docker CLI（命令行界面）的请求，创建、运行和监控容器。Docker Daemon 运行在宿主操作系统上，并通过 REST API 接受命令。
7. **网络**
   Docker 提供了几种网络模式来连接容器，包括桥接网络、主机网络和覆盖网络等。每种模式都有其特定的用途和实现方式：
   - **桥接网络**：默认网络模式，容器连接到一个桥接网络，可以互相通信。
   - **主机网络**：容器与宿主共享网络栈，性能较高但隔离性较差。
   - **覆盖网络**：用于多主机网络环境，通过 VXLAN 等技术实现跨主机的容器通信。
8. **存储**
   Docker 提供了多种数据存储选项：
   - **数据卷（Volumes）**：独立于容器的存储，可以在多个容器间共享。
   - **绑定挂载（Bind Mounts）**：将宿主机上的目录挂载到容器内。
   - **tmpfs 挂载**：在内存中创建的临时文件系统。
9. 安全
   Docker 通过多种机制提高容器的安全性，包括：
   - **内核功能剥夺（Capability Dropping）**：限制容器中的进程所能执行的内核功能。
   - **安全配置文件（Seccomp Profiles）**：使用 seccomp 过滤系统调用。
   - **AppArmor** 和 **SELinux**：强制访问控制。

通过这些底层技术的结合，Docker 实现了高效、灵活的容器化应用管理。Docker 的设计使得开发、测试和部署应用程序变得更加容易和高效，同时保持了良好的隔离性和资源利用率。

### k8s 中的 ConfigMap 是什么

在 Kubernetes 中，ConfigMap 是一种用于将非敏感数据配置分离出来的资源类型。它允许你将配置信息（比如环境变量、命令行参数、配置文件等）从应用程序中解耦，使得配置的管理更加灵活和可维护。

ConfigMap 可以存储键值对、文件或者文件的内容，并且可以被挂载到 Pod 中作为卷，或者直接注入到容器的环境变量中。这样，当需要更改配置时，你只需要更新 ConfigMap，而不需要重新构建镜像或者重新部署应用程序。

总的来说，ConfigMap 提供了一种机制，使得配置的管理更加方便，同时降低了应用程序与环境之间的耦合度。
