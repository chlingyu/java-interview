# 08-Java并发编程

---

## 一、线程基础
### 1. 进程和线程的区别？

一句话：**进程是资源分配的最小单位，线程是 CPU 调度的最小单位。** 一个进程里可以有多个线程，线程共享进程的资源。

| 对比项 | 进程 | 线程 |
|--------|------|------|
| 定义 | 操作系统分配资源的最小单位 | CPU 调度的最小单位 |
| 资源 | 有独立的内存空间、文件描述符等 | 共享进程的堆、方法区；**私有**栈、PC、本地方法栈 |
| 切换开销 | 大（要切换页表、刷 TLB 缓存等） | 小（只需切换栈和寄存器） |
| 通信方式 | 管道、Socket、共享内存、信号量等（IPC） | 直接读写共享变量（但要注意线程安全） |
| 崩溃影响 | 一个进程崩溃不影响其他进程 | 一个线程崩溃**整个进程都会挂** |

**Java 线程和操作系统线程的关系：**

在 HotSpot JVM 中，Java 线程是**一对一映射到操作系统内核线程**的（1:1 模型）。每个 `new Thread().start()` 底层都会调用操作系统的 `pthread_create()`（Linux）创建一个内核线程。

**背诵口诀：** 进程管资源，线程跑代码。进程隔离安全，线程共享高效。

> 面试话术：「进程是资源分配的最小单位，线程是 CPU 调度的最小单位。线程共享进程的堆和方法区，但各自有私有的栈和程序计数器。HotSpot 的 Java 线程与操作系统线程是一对一的关系。」

### 2. 创建线程有哪几种方式？各有什么优缺点？

**严格来说只有一种：`new Thread()`。** 但从使用方式上分 **4 种**：

| 方式 | 核心做法 | 有返回值 | 可复用 |
|------|---------|---------|--------|
| 继承 `Thread` | 重写 `run()` | ❌ | ❌ |
| 实现 `Runnable` | `new Thread(runnable)` | ❌ | ✅ |
| 实现 `Callable` + `FutureTask` | `new Thread(futureTask)` | ✅ | ✅ |
| **线程池** `ExecutorService` | `pool.submit(runnable/callable)` | 可选 | ✅ |

```java
// 方式1：继承 Thread
class MyThread extends Thread {
    @Override public void run() { /* ... */ }
}

// 方式2：实现 Runnable（推荐）
new Thread(() -> { /* ... */ }).start();

// 方式3：Callable + FutureTask（有返回值）
FutureTask<String> task = new FutureTask<>(() -> "result");
new Thread(task).start();
String result = task.get(); // 阻塞获取结果

// 方式4：线程池（实际开发首选）
ExecutorService pool = Executors.newFixedThreadPool(10);
pool.submit(() -> { /* ... */ });
```

**为什么更推荐 Runnable/Callable + 线程池？**
- Java 是单继承，继承了 Thread 就不能再继承其他类
- Runnable/Callable 实现了**任务和线程的解耦**
- 线程池**复用线程**，避免频繁创建销毁的开销

**背诵口诀：** 继承 Thread 太死板，实现接口更灵活，线程池才是生产标配。

> 面试话术：「严格来说 Java 创建线程只有 new Thread 一种方式，但任务定义有继承 Thread、实现 Runnable、实现 Callable 三种。实际开发用线程池提交任务，因为线程池能复用线程、控制并发数。Callable 比 Runnable 多了返回值和异常抛出能力。」

### 3. 线程的生命周期和状态转换？

**Java 线程有 6 种状态**，定义在 `Thread.State` 枚举中：

| 状态 | 含义 | 触发条件 |
|------|------|---------|
| `NEW` | 新建，还没 start | `new Thread()` 之后 |
| `RUNNABLE` | 可运行（包含就绪和运行中） | 调用 `start()` 后 |
| `BLOCKED` | 阻塞，等待获取 synchronized 锁 | 进入 `synchronized` 块但锁被其他线程持有 |
| `WAITING` | 无限期等待 | `Object.wait()`、`Thread.join()`、`LockSupport.park()` |
| `TIMED_WAITING` | 有超时的等待 | `Thread.sleep(ms)`、`wait(ms)`、`join(ms)` |
| `TERMINATED` | 终止 | run() 执行完毕或抛出异常 |

```
NEW → start() → RUNNABLE ←→ BLOCKED（等锁）
                    ↕
              WAITING / TIMED_WAITING（等通知/超时）
                    ↓
               TERMINATED
```

**这题面试经常挖坑：BLOCKED 和 WAITING 的区别？**

| 对比 | BLOCKED | WAITING |
|------|---------|---------|
| 等什么 | 等 **synchronized 锁** | 等**其他线程主动唤醒**（notify/unpark） |
| 是否主动 | **被动**，想进 synchronized 但锁被占了 | **主动**，调用 wait()/park() 让自己挂起 |
| 如何恢复 | 锁释放后自动获取 | 需要被 `notify()` / `notifyAll()` / `unpark()` 唤醒 |

**`sleep()` 和 `wait()` 的区别：**

| 对比项 | `sleep()` | `wait()` |
|--------|-----------|----------|
| 所属类 | `Thread`（静态方法） | `Object` |
| 是否释放锁 | **不释放** | **释放锁** |
| 唤醒方式 | 时间到自动醒 | 需要被 `notify()` / `notifyAll()` 唤醒 |
| 使用条件 | 任何地方 | 必须在 `synchronized` 块中 |

**背诵口诀：** 新就绪运行阻塞等待终——六个状态。sleep 不放锁，wait 放锁等通知。

> 面试话术：「Java 线程有 6 种状态：NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED。BLOCKED 是等 synchronized 锁，WAITING 是主动等待被唤醒。sleep 不释放锁，wait 释放锁。」

## 二、并发原理
### 4. Java 内存模型（JMM）是什么？happens-before 原则？

**JMM（Java Memory Model）是 Java 定义的一套规范，规定了多线程如何访问共享变量。** 核心思想是：每个线程有自己的**工作内存**（CPU 缓存），共享变量保存在**主内存**（堆），线程操作变量时先从主内存拷贝到工作内存，修改后再写回主内存。

```
线程1 工作内存 ←→ 主内存（共享变量 x=0） ←→ 线程2 工作内存
```

**JMM 要解决的三个问题：**

| 问题 | 含义 | Java 的解决方案 |
|------|------|----------------|
| **可见性** | 一个线程修改了变量，其他线程能不能**立即看到** | `volatile`、`synchronized`、`final` |
| **原子性** | 一个操作是**不可被中断的** | `synchronized`、`Lock`、`Atomic` 类 |
| **有序性** | 代码的执行顺序是否**按编写顺序** | `volatile`（禁止指令重排）、`happens-before` 规则 |

**什么是指令重排？**

编译器和 CPU 为了优化性能，可能会调整指令的执行顺序。单线程下结果不变（as-if-serial），但多线程下可能出问题。

**happens-before 原则（8 条）：**

如果操作 A happens-before 操作 B，那么 A 的结果**对 B 可见**，且 A 的执行顺序**在 B 之前**。

| 规则 | 含义 |
|------|------|
| **程序顺序** | 同一线程中，前面的操作 happens-before 后面的 |
| **锁规则** | `unlock` happens-before 后续的 `lock`（释放锁对获取锁可见） |
| **volatile 规则** | `volatile` 写 happens-before 后续的 `volatile` 读 |
| **线程启动** | `start()` happens-before 新线程的每个操作 |
| **线程终止** | 线程中的所有操作 happens-before `join()` 返回 |
| **中断** | `interrupt()` happens-before 被中断线程检测到中断 |
| **构造函数** | 对象构造函数的结束 happens-before `finalize()` |
| **传递性** | A happens-before B，B happens-before C → A happens-before C |

**背诵口诀：** JMM 管三件事：可见性、原子性、有序性。happens-before 保证跨线程的可见性和顺序。

> 面试话术：「JMM 是 Java 定义的多线程访问共享变量的规范。每个线程有工作内存，共享变量在主内存中。JMM 要解决可见性、原子性和有序性问题。happens-before 规则保证了多线程间操作的可见性，比如 volatile 的写 happens-before 后续的读，unlock happens-before 后续的 lock。」

### 5. CAS 是什么？有什么问题？

**CAS（Compare And Swap）是一种无锁的原子操作，比较内存中的值和预期值，相同则更新为新值，否则不操作。** 它是 `java.util.concurrent` 包的基石。

**CAS 的三个操作数：**
- **V**：要修改的内存地址的值
- **A**：预期的旧值（Expected）
- **B**：要设置的新值（New）

只有当 V == A 时，才把 V 更新为 B，否则不操作。整个**比较+更新是一条 CPU 原子指令**（x86 的 `cmpxchg`），不需要加锁。

```java
// AtomicInteger 的 incrementAndGet() 底层
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    // 底层是 do-while 循环 CAS：读旧值 → 算新值 → CAS 更新，失败就重试
}
```

**CAS 的三个问题：**

| 问题 | 说明 | 解决方案 |
|------|------|---------|
| **ABA 问题** | 值从 A 变成 B 再变回 A，CAS 以为没变过 | `AtomicStampedReference`（加版本号/时间戳） |
| **自旋开销** | 高竞争时 CAS 一直失败，空转浪费 CPU | 限制自旋次数，竞争激烈时退化为锁 |
| **只能保证单个变量的原子性** | 多个变量的操作无法用一个 CAS 完成 | 用 `AtomicReference` 把多个变量封装成一个对象，或者用锁 |

**ABA 问题详解：**

线程 1 读到变量 = A，准备 CAS 更新为 C。期间线程 2 把 A 改成 B 再改回 A。线程 1 的 CAS 发现值还是 A，就成功更新了——但实际上数据已经被修改过了，在某些场景（如链表操作）可能导致错误。

**背诵口诀：** CAS 比较再交换，无锁高效但有三坑：ABA、自旋、单变量。

> 面试话术：「CAS 是一种无锁原子操作，通过比较内存值和预期值来决定是否更新，底层是 CPU 的 cmpxchg 指令。主要问题有 ABA（用版本号解决）、高竞争时自旋浪费 CPU、只能操作单个变量。Java 的 AtomicInteger、AtomicReference 底层都是 CAS。」

## 三、锁机制
### 6. synchronized 的实现原理？锁升级过程？

**synchronized 是 Java 内置的重量级锁关键字，底层基于 JVM 的 Monitor（管程）实现。** JDK 1.6 后做了大量优化，引入了偏向锁 → 轻量级锁 → 重量级锁的**锁升级**机制。

**synchronized 的两种用法和底层实现：**

| 用法 | 锁住什么 | 字节码层面 |
|------|---------|-----------|
| 修饰方法 | 实例方法锁 `this`，静态方法锁 `Class` 对象 | 方法标志位 `ACC_SYNCHRONIZED` |
| 修饰代码块 | 锁 `()` 中指定的对象 | `monitorenter` + `monitorexit` 指令 |

**锁升级过程（JDK 1.6+ HotSpot）：**

锁的信息存在对象头的 **Mark Word** 中，升级过程是**单向的，不能降级**：

```
无锁 → 偏向锁 → 轻量级锁 → 重量级锁
```

| 锁状态 | 适用场景 | 实现原理 | Mark Word 标志位 |
|--------|---------|---------|-----------------|
| **偏向锁** | **只有一个线程**反复进入同步块 | Mark Word 写入线程 ID，后续该线程进入时**只比较线程 ID**，不做任何同步操作 | 01 |
| **轻量级锁** | **两个线程交替**执行（无竞争） | 线程在栈帧中创建锁记录（Lock Record），用 **CAS** 把 Mark Word 复制到锁记录并替换为锁记录指针 | 00 |
| **重量级锁** | **多线程竞争**激烈 | 膨胀为 Monitor（操作系统的**互斥量 Mutex**），竞争失败的线程**阻塞挂起**（用户态→内核态切换，开销大） | 10 |

**升级触发条件：**
- 偏向锁 → 轻量级锁：**第二个线程**来竞争锁
- 轻量级锁 → 重量级锁：CAS 自旋**超过一定次数**（默认 10 次，`-XX:PreBlockSpin`）或**自适应自旋**判断不值得再自旋

**这题面试经常挖坑：JDK 15 废弃了偏向锁**

JDK 15 标记偏向锁为废弃（`-XX:-UseBiasedLocking`），因为现代应用中锁竞争普遍，偏向锁的撤销开销反而成为负担。

**synchronized 的其他优化（高频追问）：**
- **锁消除**：JIT 编译器通过逃逸分析发现锁对象不会被其他线程访问，就**直接去掉锁**（如方法内 `new StringBuffer()` 的 synchronized）
- **锁粗化**：连续多次对同一对象加锁解锁，编译器**合并为一次大范围的锁**（如循环内加锁 → 循环外加锁）

**背诵口诀：** 偏向一个人，轻量俩交替，重量多竞争。标志位 01、00、10。JDK 15 废弃偏向锁。

> 面试话术：「synchronized 底层用 Monitor 实现。JDK 1.6 引入锁升级优化：偏向锁只记录线程 ID，适合单线程；轻量级锁用 CAS 操作 Mark Word，适合两个线程交替执行；竞争激烈时膨胀为重量级锁，走操作系统的互斥量。锁升级是单向的，不能降级。注意 JDK 15 已经废弃了偏向锁。此外还有锁消除和锁粗化两种编译器优化。」

### 7. synchronized 和 ReentrantLock 的区别？

一句话：**synchronized 是 JVM 层面的锁，自动释放；ReentrantLock 是 API 层面的锁，手动释放，功能更灵活。**

| 对比项 | synchronized | ReentrantLock |
|--------|--------------|---------------|
| 层级 | **JVM 关键字** | **JDK 类**（`java.util.concurrent.locks`） |
| 释放锁 | **自动**（代码块结束或异常自动释放） | **手动**（必须在 `finally` 中 `unlock()`，否则死锁） |
| 可中断 | 不可中断（线程在等锁时不能被 `interrupt`） | ✅ `lockInterruptibly()` 支持中断 |
| 超时等待 | 不支持 | ✅ `tryLock(timeout)` 支持 |
| 公平锁 | 非公平 | ✅ 构造传 `true` 可设为**公平锁** |
| 条件变量 | 只有 `wait/notify`（一个条件队列） | ✅ `Condition`（可以有**多个**条件队列，更灵活） |
| 可重入 | ✅ 可重入 | ✅ 可重入 |
| 性能 | JDK 1.6 后优化很大，**差距不大** | 差距不大 |

```java
// synchronized
synchronized (lock) {
    // 自动获取和释放锁
}

// ReentrantLock
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // 业务逻辑
} finally {
    lock.unlock(); // 必须手动释放！
}
```

**什么时候用 ReentrantLock？**
- 需要可中断的锁等待
- 需要超时获取锁
- 需要公平锁
- 需要多个条件变量（如生产者-消费者模型的 notFull/notEmpty）

**背诵口诀：** synchronized 简单自动够用，ReentrantLock 灵活手动高级。

> 面试话术：「synchronized 是 JVM 层面的关键字，自动释放锁；ReentrantLock 是 JDK 的类，需要手动 unlock。ReentrantLock 支持可中断锁、超时获取、公平锁和多条件变量，功能更灵活。性能方面 JDK 1.6 后两者差距不大。一般场景用 synchronized 就够，需要高级功能时用 ReentrantLock。」

### 8. volatile 的作用？能保证原子性吗？

**volatile 保证「可见性」和「有序性」，但不保证「原子性」。**

| 特性 | volatile | synchronized |
|------|----------|--------------|
| 可见性 | ✅ 写入后立即刷到主内存，其他线程立即可见 | ✅ |
| 有序性 | ✅ 禁止指令重排 | ✅ |
| 原子性 | ❌ **不保证** | ✅ |

**可见性：** volatile 变量写入时会把工作内存的值**强制刷回主内存**，读取时**强制从主内存重新加载**，保证所有线程看到的都是最新值。

**有序性：** volatile 变量的读写操作前后会**插入内存屏障**，禁止屏障两侧的指令重排。

**为什么不能保证原子性？** 看经典的 `i++` 例子：

```java
volatile int i = 0;
// i++ 实际上是三步操作：① 读取 i ② 计算 i+1 ③ 写回 i
// volatile 只保证每一步读写的可见性，不保证三步合在一起是原子的
```

两个线程同时 `i++`，可能同时读到 `i=0`，各自算出 1，各自写回 1，结果是 1 而不是 2。

**volatile 的典型应用场景：**
- **状态标志位**：`volatile boolean running = true;` 一个线程改，其他线程读
- **双重检查锁（DCL）的单例模式**：防止指令重排导致获取到未初始化的对象

```java
// DCL 单例必须加 volatile
private static volatile Singleton instance;
public static Singleton getInstance() {
    if (instance == null) {
        synchronized (Singleton.class) {
            if (instance == null) {
                instance = new Singleton(); // 没有 volatile，可能重排为：分配内存→赋引用→初始化
            }
        }
    }
    return instance;
}
```

**背诵口诀：** volatile 管可见和有序，不管原子。i++ 不原子，标志位和 DCL 要用它。

> 面试话术：「volatile 保证可见性和有序性，但不保证原子性。可见性通过强制刷主内存实现，有序性通过内存屏障禁止重排实现。典型应用是状态标志位和 DCL 单例模式。i++ 不是原子操作，volatile 解决不了，需要用 AtomicInteger 或 synchronized。」

### 9. volatile 的实现原理？什么是内存屏障？

**volatile 底层通过两种机制实现：① 写入时加 Lock 前缀指令（x86）；② 编译器在字节码中插入内存屏障。**

**Lock 前缀指令（硬件层面）：**

在 x86 架构中，对 volatile 变量的写操作会生成一条带 **Lock 前缀**的汇编指令。Lock 前缀的作用：
1. 将当前 CPU 缓存行的数据**立即写回主内存**
2. 通过**缓存一致性协议（MESI）**使其他 CPU 中该缓存行**失效**，强制从主内存重新读取

**内存屏障（Memory Barrier）：**

内存屏障是一种 CPU 指令，**阻止屏障两侧的指令跨越屏障重排序**。

| 屏障类型 | 作用 |
|---------|------|
| **LoadLoad** | 屏障前的 Load 先于屏障后的 Load 执行 |
| **StoreStore** | 屏障前的 Store 先于屏障后的 Store 执行 |
| **LoadStore** | 屏障前的 Load 先于屏障后的 Store 执行 |
| **StoreLoad** | 屏障前的 Store 先于屏障后的 Load 执行（**开销最大，全能屏障**） |

**JMM 对 volatile 的内存屏障策略：**
- volatile **写**之前插入 StoreStore，之后插入 **StoreLoad**
- volatile **读**之后插入 LoadLoad + LoadStore

简单理解：**写之后加屏障保证刷到主内存且后续操作不会重排到写之前；读之前加屏障保证从主内存取最新值。**

**背诵口诀：** volatile 靠 Lock 指令刷缓存，内存屏障禁重排。MESI 协议保一致。

> 面试话术：「volatile 底层在 x86 上通过 Lock 前缀指令实现，写入时立即刷回主内存，并通过 MESI 缓存一致性协议使其他 CPU 的缓存行失效。在 JMM 层面，编译器在 volatile 读写前后插入内存屏障，禁止指令重排。」

## 四、线程工具
### 10. ThreadLocal 是什么？有什么应用场景？会导致内存泄漏吗？

**ThreadLocal 是线程本地变量，每个线程都有自己独立的一份副本，互不干扰。** 不是用来做「线程同步」的，而是做「线程隔离」。

**底层原理：** 每个 `Thread` 对象内部有一个 `ThreadLocalMap`，key 是 `ThreadLocal` 对象（**弱引用**），value 是存的值。调用 `threadLocal.get()` 时，从当前线程的 `ThreadLocalMap` 中取值。

```java
// 简化理解
class Thread {
    ThreadLocal.ThreadLocalMap threadLocals; // 每个线程自己的 map
}
// threadLocal.get() → Thread.currentThread().threadLocals.get(this)
```

**应用场景：**
- **数据库连接管理**：每个线程保持自己的 Connection，避免传参
- **Session / 用户信息传递**：Web 请求中存当前用户信息，下游方法直接取
- **日期格式化**：`SimpleDateFormat` 线程不安全，用 ThreadLocal 每线程一个实例
- **Spring 的事务管理**：`@Transactional` 通过 ThreadLocal 保存当前线程的数据库连接

**这题面试经常挖坑：ThreadLocal 的内存泄漏问题**

`ThreadLocalMap` 的 key 是 `ThreadLocal` 的**弱引用**，GC 时可能被回收，变成 `null`。但 value 是**强引用**，不会被回收。如果线程是线程池中的长期存活线程，value 就一直占着内存，无法回收。

**如何避免？** 用完后必须调用 `threadLocal.remove()`。

```java
ThreadLocal<User> userHolder = new ThreadLocal<>();
try {
    userHolder.set(currentUser);
    // 业务逻辑...
} finally {
    userHolder.remove(); // 必须 remove！
}
```

**关联知识：`InheritableThreadLocal`**

普通 `ThreadLocal` 父线程的值**不会传递给子线程**。如果需要子线程继承父线程的值，用 `InheritableThreadLocal`。但在线程池场景下 `InheritableThreadLocal` 也有问题（线程复用导致值过期），可以用阿里的 `TransmittableThreadLocal`（TTL）解决。

**背诵口诀：** ThreadLocal 线程隔离不共享，key 弱引用会泄漏，用完必须 remove。子线程要继承用 InheritableThreadLocal。

> 面试话术：「ThreadLocal 给每个线程提供独立的变量副本，底层是每个 Thread 对象内部的 ThreadLocalMap。常用于存数据库连接、用户信息等需要线程隔离的数据。内存泄漏的原因是 key 是弱引用可能被 GC，但 value 是强引用不会被回收，线程池场景下必须在 finally 中调用 remove()。如果需要子线程继承父线程的值，可以用 InheritableThreadLocal。」

### 11. 线程池的核心参数？拒绝策略有哪些？

**线程池的核心构造方法有 7 个参数：**

```java
public ThreadPoolExecutor(
    int corePoolSize,        // 核心线程数
    int maximumPoolSize,     // 最大线程数
    long keepAliveTime,      // 非核心线程的空闲存活时间
    TimeUnit unit,           // 时间单位
    BlockingQueue<Runnable> workQueue,  // 任务队列
    ThreadFactory threadFactory,        // 线程工厂
    RejectedExecutionHandler handler   // 拒绝策略
)
```

| 参数 | 含义 |
|------|------|
| `corePoolSize` | **核心线程数**，即使空闲也不会被回收（除非设置 `allowCoreThreadTimeOut`） |
| `maximumPoolSize` | **最大线程数**，核心线程 + 非核心线程的上限 |
| `keepAliveTime` | **非核心线程**空闲超过这个时间就被回收 |
| `workQueue` | 任务队列，核心线程满了之后新任务先进队列 |
| `handler` | 队列也满了、线程数也到 max 时，执行拒绝策略 |

**4 种内置拒绝策略：**

| 策略 | 行为 |
|------|------|
| `AbortPolicy`（**默认**） | 直接抛 `RejectedExecutionException` |
| `CallerRunsPolicy` | 由**提交任务的线程**自己执行（降速但不丢任务） |
| `DiscardPolicy` | **默默丢弃**任务，不抛异常 |
| `DiscardOldestPolicy` | 丢弃队列中**最早的**任务，然后重新提交当前任务 |

**常用的任务队列：**

| 队列 | 特点 |
|------|------|
| `LinkedBlockingQueue` | **无界队列**（默认 Integer.MAX_VALUE），任务不会被拒绝，但可能 OOM |
| `ArrayBlockingQueue` | **有界队列**，需要指定容量 |
| `SynchronousQueue` | **不存储**任务，每个 put 必须等一个 take，适合快速消费的场景 |
| `PriorityBlockingQueue` | **优先级队列**，按优先级出队 |

**背诵口诀：** 核心最大存活队列工厂拒绝——七个参数。拒绝四策略：抛异常、调用者跑、悄悄丢、丢最老。

> 面试话术：「线程池核心参数有 7 个：核心线程数、最大线程数、空闲存活时间、时间单位、任务队列、线程工厂和拒绝策略。拒绝策略有 4 种：AbortPolicy 抛异常（默认）、CallerRunsPolicy 由调用者执行、DiscardPolicy 静默丢弃、DiscardOldestPolicy 丢弃最早的任务。」

### 12. 线程池的工作流程？为什么不建议用 Executors 创建线程池？

**线程池提交任务的处理流程：**

```
提交任务
  ↓
核心线程数未满？ ──是──→ 创建核心线程执行
  ↓ 否
任务队列未满？ ──是──→ 放入队列等待
  ↓ 否
最大线程数未满？ ──是──→ 创建非核心线程执行
  ↓ 否
执行拒绝策略
```

关键点：**先填核心线程 → 再放队列 → 再开非核心线程 → 最后拒绝。** 注意是**先队列后线程**，不是先开线程。

**为什么不建议用 Executors 创建线程池？**

《阿里巴巴 Java 开发手册》明确禁止，原因是 **Executors 的快捷方法会导致 OOM**：

| 方法 | 问题 |
|------|------|
| `newFixedThreadPool(n)` | 用的是 `LinkedBlockingQueue`（**无界队列**），任务堆积可能 OOM |
| `newSingleThreadExecutor()` | 同上，无界队列 |
| `newCachedThreadPool()` | `maximumPoolSize` 是 `Integer.MAX_VALUE`，**线程数无上限**，可能创建大量线程导致 OOM |
| `newScheduledThreadPool(n)` | 同 CachedThreadPool，最大线程数无上限 |

**正确做法：用 `ThreadPoolExecutor` 手动指定所有参数**

```java
ThreadPoolExecutor pool = new ThreadPoolExecutor(
    5,                                    // 核心线程数
    10,                                   // 最大线程数
    60, TimeUnit.SECONDS,                 // 非核心线程存活时间
    new ArrayBlockingQueue<>(100),        // 有界队列，容量 100
    new ThreadPoolExecutor.CallerRunsPolicy() // 拒绝策略
);
```

**线程池参数怎么配？**
- **CPU 密集型**任务（计算多）：核心线程数 = **CPU 核数 + 1**
- **IO 密集型**任务（等 IO 多）：核心线程数 = **CPU 核数 × 2**（或更多）
- 实际生产中最好**压测**确定最优值

**背诵口诀：** 先核心后队列再扩容最后拒绝。Executors 不要用，手动建线程池才安全。

> 面试话术：「线程池工作流程是先用核心线程执行，满了放队列，队列满了创建非核心线程，都满了执行拒绝策略。不建议用 Executors 因为 FixedThreadPool 和 SingleThread 用无界队列可能 OOM，CachedThreadPool 线程数无上限可能 OOM。应该手动用 ThreadPoolExecutor 指定有界队列和合理的最大线程数。」

## 五、并发工具类
### 13. AQS 是什么？实现原理？

**AQS（AbstractQueuedSynchronizer，抽象队列同步器）是 JUC 并发包的核心框架，`ReentrantLock`、`Semaphore`、`CountDownLatch` 底层都靠它实现。**

**核心思想：**
- 用一个 `volatile int state` 表示**同步状态**（如锁是否被持有、信号量剩余数）
- 线程获取资源失败时，进入一个 **CLH 双向队列**排队等待
- 获取和释放资源通过 **CAS** 操作 state

```
AQS 结构：
state = 0 (未加锁) / 1 (已加锁)
CLH 队列：head → Node(Thread1) → Node(Thread2) → tail
```

**以 ReentrantLock 加锁为例：**
1. 线程 A 调用 `lock()`，CAS 把 state 从 0 改为 1，成功 → 获得锁
2. 线程 B 调用 `lock()`，CAS 失败（state=1），**封装成 Node 加入 CLH 队列尾部**，然后 `park()` 挂起
3. 线程 A 调用 `unlock()`，state 改为 0，唤醒队列头部的线程 B
4. 线程 B 被 `unpark()` 唤醒，再次 CAS 争抢锁

**AQS 的两种模式：**

| 模式 | 说明 | 典型实现 |
|------|------|---------|
| **独占模式** | 同一时刻只有一个线程能获取资源 | `ReentrantLock` |
| **共享模式** | 同一时刻多个线程能获取资源 | `Semaphore`、`CountDownLatch`、`ReadWriteLock` 的读锁 |

**state 在不同实现中的含义：**

| 实现 | state 含义 |
|------|-----------|
| `ReentrantLock` | 0 表示未锁定，≥1 表示加锁次数（可重入） |
| `Semaphore` | 表示可用的信号量个数 |
| `CountDownLatch` | 表示还需要倒数的次数 |

**背诵口诀：** AQS = state + CLH 队列 + CAS。独占靠 ReentrantLock，共享靠 Semaphore。

> 面试话术：「AQS 是 JUC 的核心同步框架，用 volatile int state 表示同步状态，用 CLH 双向队列管理等待线程。线程获取资源失败时入队等待，释放时唤醒队首线程。ReentrantLock 的 state 表示加锁次数，Semaphore 的 state 表示信号量个数。AQS 支持独占和共享两种模式。」

### 14. CountDownLatch、CyclicBarrier、Semaphore 的区别和使用场景？

一句话：**CountDownLatch 等别人做完，CyclicBarrier 等大家到齐，Semaphore 控制同时进去几个。**

| 对比项 | CountDownLatch | CyclicBarrier | Semaphore |
|--------|---------------|---------------|-----------|
| 作用 | 一个线程等待**其他 N 个线程**完成 | **N 个线程互相等待**，都到齐后一起继续 | 控制**同时访问资源**的线程数 |
| 计数方向 | **倒数**到 0 | 线程到达屏障点**累加**到 N | 获取/释放**许可** |
| 可复用 | ❌ 一次性，归零后不能重置 | ✅ **可重复使用**，一轮结束自动重置 |  ✅ 可重复使用 |
| 底层 | AQS（共享模式） | ReentrantLock + Condition | AQS（共享模式） |

```java
// CountDownLatch：主线程等 3 个子线程完成
CountDownLatch latch = new CountDownLatch(3);
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        doWork();
        latch.countDown(); // 完成一个，计数 -1
    }).start();
}
latch.await(); // 主线程阻塞，直到计数为 0

// CyclicBarrier：3 个线程互相等，都到了再一起走
CyclicBarrier barrier = new CyclicBarrier(3);
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        prepare();
        barrier.await(); // 每个线程到这里等待，3 个都到了才继续
        go();
    }).start();
}

// Semaphore：最多 3 个线程同时访问
Semaphore sem = new Semaphore(3);
sem.acquire(); // 获取许可（-1），没有许可就阻塞
try { accessResource(); }
finally { sem.release(); } // 释放许可（+1）
```

**使用场景：**
- **CountDownLatch**：主线程等所有子任务完成后汇总结果、多个服务同时初始化
- **CyclicBarrier**：多线程分段计算后汇总（如 MapReduce 思想）、游戏等所有玩家就绪
- **Semaphore**：限流（如数据库连接池最多 10 个连接）、限制并发访问数

**背诵口诀：** Latch 等人做完，Barrier 等人到齐，Semaphore 限人数。Latch 一次性，Barrier 可复用。

> 面试话术：「CountDownLatch 是一个线程等其他线程完成，调 countDown 减计数，await 等到零；CyclicBarrier 是多个线程互相等待到齐后一起执行，可以重复使用；Semaphore 控制同时访问资源的线程数量，常用于限流。三者底层都基于 AQS。」

### 15. 什么是死锁？如何避免死锁？

**死锁是两个或多个线程互相持有对方需要的锁，导致所有线程都永远阻塞，谁也动不了。**

```java
// 经典死锁示例
Object lockA = new Object(), lockB = new Object();
// 线程1：先拿 A 再拿 B
new Thread(() -> {
    synchronized (lockA) {
        sleep(100);
        synchronized (lockB) { /* ... */ }  // 等线程2释放 B
    }
}).start();
// 线程2：先拿 B 再拿 A
new Thread(() -> {
    synchronized (lockB) {
        sleep(100);
        synchronized (lockA) { /* ... */ }  // 等线程1释放 A
    }
}).start();
// 结果：线程1拿着A等B，线程2拿着B等A → 死锁！
```

**死锁的四个必要条件（破坏任何一个就能避免）：**

| 条件 | 含义 | 破坏方法 |
|------|------|---------|
| **互斥** | 资源同一时刻只能被一个线程持有 | 一般无法破坏（锁本身就是互斥的） |
| **持有并等待** | 持有一个资源的同时等待另一个 | **一次性申请所有资源** |
| **不可剥夺** | 已获取的资源不能被其他线程强行抢走 | 用 `tryLock()` 带超时，获取不到就**主动释放已持有的锁** |
| **循环等待** | 线程间形成环形等待链 | **按固定顺序**加锁（如都先拿 A 再拿 B） |

**实际开发中如何避免死锁？**
1. **固定加锁顺序**：所有线程按相同的顺序获取锁（最有效）
2. **用 `tryLock()` 超时机制**：获取不到就放弃，避免永久等待
3. **减少锁的持有时间**：缩小 synchronized 代码块范围
4. **避免嵌套锁**：尽量不要在持有一把锁的情况下再去获取另一把

**如何排查死锁？**
- `jstack <pid>`：打印线程栈，能自动检测并报告死锁
- VisualVM / Arthas：图形化工具，一键检测死锁

**背诵口诀：** 死锁四条件：互斥、持有等待、不可剥夺、循环等待。破坏一个就行，最常用「固定顺序加锁」。

> 面试话术：「死锁是多个线程互相持有对方需要的锁，导致永久阻塞。必须满足四个条件：互斥、持有并等待、不可剥夺、循环等待。最有效的避免方式是固定加锁顺序，也可以用 tryLock 带超时避免永久等待。排查可以用 jstack 命令查看线程栈。」

## 复习优先级（3~5 年）
| 优先级 | 题目 |
|--------|------|
| P0 | 4. Java 内存模型（JMM）是什么？happens-before 原则？ |
| P0 | 6. synchronized 的实现原理？锁升级过程？ |
| P0 | 7. synchronized 和 ReentrantLock 的区别？ |
| P0 | 8. volatile 的作用？能保证原子性吗？ |
| P0 | 10. ThreadLocal 是什么？有什么应用场景？会导致内存泄漏吗？ |
| P0 | 11. 线程池的核心参数？拒绝策略有哪些？ |
| P0 | 12. 线程池的工作流程？为什么不建议用 Executors 创建线程池？ |
| P0 | 13. AQS 是什么？实现原理？ |
| P1 | 1. 进程和线程的区别？ |
| P1 | 3. 线程的生命周期和状态转换？ |
| P1 | 5. CAS 是什么？有什么问题？ |
| P1 | 9. volatile 的实现原理？什么是内存屏障？ |
| P1 | 14. CountDownLatch、CyclicBarrier、Semaphore 的区别和使用场景？ |
| P1 | 15. 什么是死锁？如何避免死锁？ |
| P2 | 2. 创建线程有哪几种方式？各有什么优缺点？ |