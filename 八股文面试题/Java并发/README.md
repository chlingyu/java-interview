# Java 并发

---

## 一、线程基础

### 1. 创建线程有几种方式？

**一句话总结**：严格来说只有 **1 种**（实现 `Runnable` 传给 `Thread`），但面试通常答 **4 种**：继承 Thread、实现 Runnable、实现 Callable、线程池。

| 方式 | 特点 | 有返回值？ | 推荐？ |
|------|------|-----------|--------|
| 继承 `Thread` | 重写 `run()` 方法 | ❌ | ❌ Java 单继承，不灵活 |
| 实现 `Runnable` | 实现 `run()` 方法 | ❌ | ✅ 接口可多实现 |
| 实现 `Callable` | 实现 `call()` 方法，配合 `FutureTask` 使用 | ✅ | ✅ 需要返回值时用 |
| 线程池 `ExecutorService` | 提交任务给线程池 | ✅/❌ 都行 | ✅ **实际开发首选** |

```java
// 方式1：继承 Thread
new Thread() { public void run() { /* 任务 */ } }.start();

// 方式2：实现 Runnable（推荐）
new Thread(() -> { /* 任务 */ }).start();

// 方式3：实现 Callable（有返回值）
FutureTask<String> task = new FutureTask<>(() -> "结果");
new Thread(task).start();
String result = task.get();  // 阻塞等待结果
```

> **面试话术**：创建线程本质上只有一种方式，就是构造 Thread 对象。但从写法上通常说 4 种：继承 Thread 类、实现 Runnable 接口、实现 Callable 接口配合 FutureTask 获取返回值、以及通过线程池提交任务。实际开发中推荐用线程池，因为可以复用线程，避免频繁创建销毁的开销。

---

### 2. 线程有哪几种状态？

**一句话总结**：Java 线程有 **6 种状态**，定义在 `Thread.State` 枚举中。

```
NEW（新建）→ start() → RUNNABLE（可运行）
                         ↓ ↑
               ┌── BLOCKED（阻塞，等锁）
               ├── WAITING（无限等待）
               └── TIMED_WAITING（限时等待）
                         ↓
                   TERMINATED（终止）
```

| 状态 | 含义 | 触发方式 |
|------|------|---------|
| **NEW** | 线程刚创建，还没启动 | `new Thread()` |
| **RUNNABLE** | 可运行（包括正在运行和等待 CPU 调度） | `start()` |
| **BLOCKED** | 等待获取 `synchronized` 锁 | 另一个线程持有锁 |
| **WAITING** | 无限期等待，直到被唤醒 | `wait()`、`join()`、`LockSupport.park()` |
| **TIMED_WAITING** | 限时等待 | `sleep(ms)`、`wait(ms)`、`join(ms)` |
| **TERMINATED** | 线程执行完毕或异常退出 | `run()` 执行完 |

**背诵口诀**：「**新可阻等时终**」（新建、可运行、阻塞、等待、限时等待、终止）

---

### 3. sleep() 和 wait() 有什么区别？

**一句话总结**：`sleep()` 是 Thread 的方法，**不释放锁**，到时间自动醒；`wait()` 是 Object 的方法，**会释放锁**，必须被 `notify()` 唤醒。

| 对比项 | `sleep()` | `wait()` |
|--------|-----------|----------|
| 所属类 | `Thread` | `Object` |
| 释放锁 | ❌ **不释放** | ✅ **释放** |
| 使用条件 | 任何地方都能用 | 必须在 `synchronized` 块中用 |
| 唤醒方式 | 时间到了自动醒 | 被 `notify()`/`notifyAll()` 唤醒 |
| 用途 | 让线程暂停一段时间 | 线程间通信（等待某个条件满足） |

**背诵口诀**：「**sleep 不放锁自己醒，wait 放锁等人叫**」

**⚠️ 面试挖坑提醒**：「wait() 为什么必须在 synchronized 块中调用？」—— 因为 `wait()` 需要先持有锁才能释放锁，不在同步块中调用会抛 `IllegalMonitorStateException`。

---

## 二、锁与同步

### 4. 什么是 Java 内存模型（JMM）？

**一句话总结**：JMM（Java Memory Model）定义了多线程环境下，线程之间如何**共享变量**的规则。核心概念是**主内存**（所有线程共享）和**工作内存**（每个线程私有的副本）。

**JMM 模型图示**：

```
线程A 的工作内存（变量副本）  ←→  主内存（共享变量）  ←→  线程B 的工作内存（变量副本）
```

**JMM 解决的三大问题**：

| 问题 | 含义 | 解决方式 |
|------|------|---------|
| **可见性** | 一个线程修改了变量，其他线程能不能马上看到 | `volatile`、`synchronized`、`final` |
| **原子性** | 一个操作能不能被中途打断 | `synchronized`、`Lock`、原子类（`AtomicInteger`） |
| **有序性** | 代码执行顺序是不是和写的一样 | `volatile`、`synchronized`、Happens-Before 规则 |

> **面试话术**：JMM 是 Java 定义的一套规范，描述多线程环境下变量的访问规则。每个线程都有自己的工作内存，存放主内存中变量的副本。线程对变量的操作都在工作内存中进行，然后再同步回主内存。JMM 主要解决可见性、原子性和有序性三大问题，通过 volatile、synchronized 等机制来保证。

---

### 5. 什么是 CAS？有什么问题？

**一句话总结**：CAS（Compare And Swap，比较并交换）是一种**无锁**的原子操作。它比较内存中的值和期望值，如果相同就更新为新值，否则不操作。是乐观锁的核心实现方式。

**CAS 三个操作数**：

- **V**（Value）：内存中的当前值
- **E**（Expected）：期望值（我以为它应该是多少）
- **N**（New）：要设置的新值

**执行逻辑**：如果 V == E，则把 V 更新为 N；如果 V ≠ E，说明被别人改过了，操作失败，重试。

```java
// AtomicInteger 的 CAS 使用示例
AtomicInteger count = new AtomicInteger(0);
count.compareAndSet(0, 1);  // 如果当前是 0，就改成 1
count.incrementAndGet();     // 原子性的 count++
```

**CAS 的三大问题**：

| 问题 | 说明 | 解决方案 |
|------|------|---------|
| **ABA 问题** | 值从 A 改成 B 又改回 A，CAS 以为没变过 | 加版本号/时间戳，用 `AtomicStampedReference` |
| **自旋开销** | 失败后反复重试，CPU 空转浪费资源 | 限制重试次数，或者改用锁 |
| **只能保证一个变量的原子性** | 无法同时对多个变量做原子操作 | 把多个变量封装成一个对象，用 `AtomicReference` |

**背诵口诀**：「**CAS 三问题：ABA、自旋、单变量**」

---

### 6. synchronized 关键字的原理是什么？

**一句话总结**：`synchronized` 是 Java 的内置锁，可以修饰**方法**或**代码块**，保证同一时刻只有一个线程执行被锁住的代码。底层通过 **Monitor（监视器锁）** 实现。

**三种用法**：

| 用法 | 锁的对象 | 示例 |
|------|---------|------|
| 修饰普通方法 | **当前对象实例**（this） | `synchronized void method()` |
| 修饰静态方法 | **当前类的 Class 对象** | `static synchronized void method()` |
| 修饰代码块 | **括号中指定的对象** | `synchronized(obj) { ... }` |

**底层原理**：
- **代码块**：编译后生成 `monitorenter`（获取锁）和 `monitorexit`（释放锁）指令
- **方法**：在方法标志中加上 `ACC_SYNCHRONIZED` 标记

**JDK 6 的锁优化**（锁升级过程，以 JDK 6~14 的 HotSpot 为主；JDK 15 默认关闭偏向锁，JDK 18+ 已移除偏向锁）：

```
无锁 → 偏向锁 → 轻量级锁 → 重量级锁（只能升级不能降级）
```

| 锁级别 | 适用场景 | 原理 |
|--------|---------|------|
| **偏向锁** | 只有一个线程访问 | 在对象头记录线程 ID，下次同一线程来就不用加锁了 |
| **轻量级锁** | 多个线程**交替**访问（无竞争） | 用 CAS（详见上一题）尝试获取锁，失败就升级 |
| **重量级锁** | 多个线程**同时**竞争 | 调用操作系统的互斥锁，线程会阻塞挂起 |

> **面试话术**：synchronized 可以修饰方法和代码块。修饰普通方法锁的是 this 对象，修饰静态方法锁的是 Class 对象。底层通过 Monitor 实现，代码块用 monitorenter 和 monitorexit 指令。JDK 6 之后做了很大的优化，引入了偏向锁、轻量级锁，锁可以从偏向锁升级到轻量级锁再到重量级锁，减少了不必要的性能开销。注意 JDK 15 默认关闭了偏向锁，面试时先确认聊的是哪个版本。

---

### 7. volatile 关键字有什么作用？

**一句话总结**：`volatile` 保证变量的**可见性**（一个线程修改后其他线程立即看到，详见 JMM 第 4 题）和**有序性**（防止指令重排序），但**不保证原子性**。

**两大作用**：

| 作用 | 说明 | 举例 |
|------|------|------|
| **可见性** | 修改 volatile 变量后，立即刷新到主内存，其他线程读取时从主内存拿最新值 | 一个线程修改 flag = true，另一个线程立刻能看到 |
| **有序性** | 通过插入内存屏障，禁止 volatile 读写前后的指令被编译器或 CPU 重新排序 | 双重检查锁定单例模式必须用 volatile |

**⚠️ 面试挖坑提醒**（这题面试经常挖坑）：「volatile 能保证原子性吗？」—— **不能！** 比如 `count++` 这个操作实际上是"读-改-写"三步，volatile 只能保证每次读到最新值，但多个线程同时 `count++` 仍然会出错。要保证原子性应该用 `AtomicInteger` 或 `synchronized`。

**经典应用**：**双重检查锁定（DCL）单例模式**

```java
class Singleton {
    private static volatile Singleton instance;
    public static Singleton getInstance() {
        if (instance == null) {                // 第一次检查
            synchronized (Singleton.class) {
                if (instance == null) {        // 第二次检查
                    instance = new Singleton(); // volatile 防止这里指令重排序
                }
            }
        }
        return instance;
    }
}
```

---

### 8. 什么是乐观锁和悲观锁？

**一句话总结**：**悲观锁**假设一定会冲突，先加锁再操作（如 `synchronized`）；**乐观锁**假设不会冲突，先操作再检查是否冲突（如 CAS）。

| 对比项 | 悲观锁 | 乐观锁 |
|--------|--------|--------|
| 核心思想 | "总是有人跟我抢" → 先锁再操作 | "应该没人跟我抢" → 先操作再验证 |
| 实现方式 | `synchronized`、`ReentrantLock` | **CAS**、版本号机制 |
| 适用场景 | 写多读少（冲突频繁） | **读多写少**（冲突较少） |
| 性能特点 | 线程会阻塞等待，有上下文切换开销 | 不阻塞，但冲突多时反复重试也耗 CPU |
| 数据库中的体现 | `SELECT ... FOR UPDATE` | 版本号字段 `WHERE version = ?` |

> **面试话术**：悲观锁认为并发冲突一定会发生，所以每次操作前先加锁，比如 synchronized 就是典型的悲观锁。乐观锁认为冲突概率低，先不加锁直接操作，提交时检查有没有被别人改过，如果改过就重试。CAS 就是乐观锁的实现方式。读多写少的场景用乐观锁性能更好，写多的场景用悲观锁更合适。

---

### 9. ReentrantLock 和 synchronized 有什么区别？

**一句话总结**：两者都是**可重入的互斥锁**，但 `ReentrantLock`（可重入锁）更灵活——支持**公平锁**、**可中断**、**超时获取**、**多条件变量**，代价是需要手动释放锁。

| 对比项 | synchronized | ReentrantLock |
|--------|-------------|---------------|
| 实现方式 | JVM 内置（关键字） | JDK 层面（`java.util.concurrent` 包） |
| 加锁/释放锁 | **自动**（进出同步块自动加锁释放） | **手动**（必须在 `finally` 中 `unlock()`） |
| 公平锁 | ❌ 只支持非公平 | ✅ 可选公平/非公平 |
| 可中断 | ❌ | ✅ `lockInterruptibly()` |
| 超时获取 | ❌ | ✅ `tryLock(timeout)` |
| 条件变量 | 只有一个（`wait/notify`） | 可以有**多个** `Condition` |
| 性能 | JDK 6 后优化很好，差别不大 | 高并发下略好 |

**⚠️ 面试挖坑提醒**：「什么是公平锁和非公平锁？」—— **公平锁**按申请顺序排队获取锁（先来先得），**非公平锁**允许插队（性能更好但可能导致线程饥饿）。`ReentrantLock` 默认是非公平锁，`new ReentrantLock(true)` 创建公平锁。

---

## 三、并发工具

### 10. ThreadLocal 是什么？有什么问题？

**一句话总结**：`ThreadLocal` 为每个线程提供一份**独立的变量副本**，线程之间互不影响。常用于存储**用户信息**、**数据库连接**等线程私有数据。

**原理**：每个线程内部有一个 `ThreadLocalMap`，以 `ThreadLocal` 对象为 key，存储该线程的变量副本。

```java
ThreadLocal<String> userHolder = new ThreadLocal<>();
userHolder.set("张三");       // 当前线程设置值
String user = userHolder.get(); // 当前线程获取值（"张三"）
userHolder.remove();            // ⚠️ 用完必须 remove！
```

**⚠️ 面试挖坑提醒**（这题面试经常挖坑）：「ThreadLocal 会导致内存泄漏吗？」—— **会！** ThreadLocalMap 的 key 是**弱引用**（GC 时会被回收），但 value 是**强引用**。如果 ThreadLocal 对象被回收了，key 变成 null，但 value 还在，就造成了内存泄漏。**解决办法**：用完后**必须调用 `remove()`**。

---

### 11. 为什么要用线程池？

**一句话总结**：线程池可以**复用线程**，避免频繁创建销毁线程的开销，还能**控制并发数量**防止资源耗尽。

**线程池的三大好处**：

1. **降低资源消耗**：复用已创建的线程，减少线程创建和销毁的开销
2. **提高响应速度**：任务到达时不需要等待线程创建就能执行
3. **便于管理**：可以统一控制最大并发数、监控线程状态

**⚠️ 面试挖坑提醒**：「为什么不推荐用 Executors 快捷方法创建线程池？」—— 《阿里巴巴 Java 开发手册》明确禁止，原因：

| 方法 | 问题 |
|------|------|
| `newFixedThreadPool` | 任务队列是**无界的** `LinkedBlockingQueue`，任务堆积可能导致**内存溢出（OOM）** |
| `newCachedThreadPool` | 最大线程数是 `Integer.MAX_VALUE`，可能创建**大量线程**导致 OOM |

**推荐做法**：手动用 `ThreadPoolExecutor` 构造函数创建，明确设置每个参数。

---

### 12. 线程池的 7 大参数是什么？

**一句话总结**：线程池的核心是 `ThreadPoolExecutor`，构造函数有 **7 个参数**，控制线程池的大小、队列、拒绝策略等行为。

```java
new ThreadPoolExecutor(
    corePoolSize,      // 核心线程数
    maximumPoolSize,   // 最大线程数
    keepAliveTime,     // 非核心线程空闲存活时间
    TimeUnit.SECONDS,  // 时间单位
    workQueue,         // 任务队列
    threadFactory,     // 线程工厂
    handler            // 拒绝策略
);
```

| 参数 | 含义 | 比喻 |
|------|------|------|
| **corePoolSize** | 核心线程数，即使空闲也不会被销毁 | 公司的正式员工 |
| **maximumPoolSize** | 最大线程数（核心 + 临时） | 正式员工 + 临时工 |
| **keepAliveTime** | 临时线程空闲多久后被销毁 | 临时工没活干多久后被辞退 |
| **unit** | keepAliveTime 的时间单位 | 秒/毫秒/分钟 |
| **workQueue** | 等待执行的任务队列 | 公司的待办任务列表 |
| **threadFactory** | 创建线程的工厂（可自定义线程名称） | HR 部门 |
| **handler** | 队列满且线程数达到最大时的拒绝策略 | 任务太多处理不过来时怎么办 |

**线程池执行流程**：

```
提交任务 → 核心线程有空？ → 是 → 核心线程执行
                          → 否 → 队列满了吗？ → 否 → 放入队列等待
                                              → 是 → 线程数 < 最大？ → 是 → 创建临时线程
                                                                      → 否 → 执行拒绝策略
```

**4 种拒绝策略**：

| 策略 | 行为 |
|------|------|
| **AbortPolicy**（默认） | 直接抛异常 `RejectedExecutionException` |
| **CallerRunsPolicy** | 让提交任务的线程自己执行（不抛弃任务） |
| **DiscardPolicy** | 静默丢弃新任务 |
| **DiscardOldestPolicy** | 丢弃队列中最老的任务，重新提交当前任务 |

**背诵口诀**：「**核心→队列→临时→拒绝**」（先用核心线程，满了放队列，队列满了开临时线程，都满了就拒绝）

**高频追问：核心线程数（corePoolSize）应该怎么设置？**

> 这题面试出现频率极高，很多人只会背公式，但面试官追问「为什么」就卡了。关键是理解思路，而不是死记数值。

**经验起点**（先估一个值，再压测调优）：

| 任务类型 | 经验起点值 | 为什么 |
|---------|----------|-------|
| **CPU 密集型**（加密、压缩、复杂计算） | **CPU 核数** 或 **CPU 核数 + 1** | 线程太多只会抢 CPU 时间片，反而增加上下文切换开销。+1 是为了在线程偶尔被系统短暂卡一下时，CPU 也不会闲着 |
| **输入输出（IO）密集型**（网络调用、数据库查询、文件读写） | **CPU 核数 × 2** 或更高 | 线程大部分时间在等 IO 响应，CPU 是空闲的，多开些线程可以让 CPU 不闲着 |

**更精确的估算公式**（IO 密集型）：

```
线程数 = CPU 核数 × (1 + 平均等待时间 / 平均计算时间)
```

比如 4 核 CPU，一个请求平均等 IO 80ms、计算 20ms → 线程数 = 4 × (1 + 80/20) = **20**。简单说：如果等 IO 的时间大约是算 CPU 的 4 倍，线程数就可以大致开到核数的 5 倍。等待占比越高，线程数可以越多。

**⚠️ 面试挖坑提醒**：公式只是**起步参考值**，实际生产中还受这些因素影响：

- **队列类型**：如果用了**无界队列**（如 `LinkedBlockingQueue`），任务会一直往队列堆，`maximumPoolSize` 基本不起作用，很多人背了参数却不知道这一层
- **下游瓶颈**：数据库连接池只有 20 个连接，线程开 200 个也没用，大部分在等连接
- **接口超时**：下游响应慢，线程全阻塞在等待上，需要更多线程
- **混合任务**：既有 CPU 密集又有 IO 密集时，最好**拆成不同线程池**分别配置

**背诵口诀**：「**CPU 看核数，IO 看等待，先估后测再定值**」

**结论**：先按公式给一个起步值，然后**压测 + 监控**调到最优。面试中答出「公式 + 为什么 + 现实约束 + 压测调优」就是满分答案。

> **面试话术**：核心线程数的设置要看任务类型。CPU 密集型任务，线程数设为 CPU 核数或核数加 1 就够了，因为线程太多只会增加上下文切换。IO 密集型任务，线程大部分时间在等 IO，可以设为核数乘 2，更精确的做法是按等待时间和计算时间的比例估算。但公式只是起步值，实际还要考虑队列类型、下游连接数、超时等因素，最终通过压测确定。特别要注意，如果用了无界队列，最大线程数参数其实不起作用，任务会无限堆在队列里。

---

### 13. 什么是 AQS？

**一句话总结**：AQS（AbstractQueuedSynchronizer，抽象队列同步器）是 Java 并发包中**构建锁和同步器的底层框架**。`ReentrantLock`、`Semaphore`、`CountDownLatch` 等都是基于 AQS 实现的。

**核心原理**：
- 维护一个 **`state` 变量**（`volatile int`），表示锁的状态（0 表示未锁定，> 0 表示已锁定）
- 通过 **CAS** 修改 state 来获取/释放锁
- 获取锁失败的线程会被放入一个 **FIFO 等待队列**（先进先出，类似排队）

```
AQS 等待队列：
HEAD ←→ Node(线程A) ←→ Node(线程B) ←→ Node(线程C) ←→ TAIL
```

> **面试话术**：AQS 是 Java 并发包的基础框架，ReentrantLock、Semaphore 等都基于它实现。核心思想是用一个 volatile 的 state 变量表示同步状态，通过 CAS 来竞争修改这个状态。竞争失败的线程会被封装成节点放入一个先进先出的等待队列中排队。

---

### 14. 什么是死锁？怎么避免？

**一句话总结**：**死锁**是指两个或多个线程互相持有对方需要的锁，导致所有线程都**永远等待**下去，程序卡死。

**死锁示例**：

```java
// 线程A：先锁 lockA，再锁 lockB
synchronized(lockA) {
    synchronized(lockB) { /* ... */ }
}
// 线程B：先锁 lockB，再锁 lockA
synchronized(lockB) {
    synchronized(lockA) { /* ... */ }  // 💀 死锁！
}
```

**死锁的 4 个必要条件**（同时满足才会死锁）：

| 条件 | 含义 |
|------|------|
| **互斥** | 资源同一时刻只能被一个线程持有 |
| **持有并等待** | 线程持有一个资源的同时等待另一个资源 |
| **不可抢占** | 已持有的资源不能被其他线程强行夺走 |
| **循环等待** | 线程之间形成首尾相接的等待环路 |

**避免死锁的方法**：
- **按固定顺序加锁**：所有线程都先锁 A 再锁 B（打破循环等待）
- **设置超时**：用 `tryLock(timeout)` 代替 `lock()`（打破持有并等待）
- **减少锁的持有时间**：缩小同步代码块范围

**背诵口诀**：「**互斥持有不抢占，循环等待成死锁**」

---

### 15. CountDownLatch、CyclicBarrier、Semaphore 是什么？

**一句话总结**：三个并发协作工具——`CountDownLatch` 让一个线程**等待其他线程完成**，`CyclicBarrier` 让多个线程**在同一个点互相等待**，`Semaphore`（信号量）**控制同时访问的线程数量**。

| 工具 | 一句话 | 典型场景 |
|------|--------|---------|
| **CountDownLatch** | "等所有人做完我再做"（一次性） | 主线程等待多个子线程完成初始化 |
| **CyclicBarrier** | "所有人到齐了一起出发"（可重用） | 多线程分段计算，每段结束后汇总 |
| **Semaphore** | "停车场只有 N 个车位"（限流） | 控制数据库连接池的并发数 |

```java
// CountDownLatch 示例：主线程等 3 个子线程完成
CountDownLatch latch = new CountDownLatch(3);
// 每个子线程完成后：latch.countDown();
latch.await();  // 主线程阻塞，直到计数器变为 0

// Semaphore 示例：最多 3 个线程同时执行
Semaphore semaphore = new Semaphore(3);
semaphore.acquire();  // 获取许可（没有许可就阻塞）
// 执行任务...
semaphore.release();  // 释放许可
```

---

### 16. synchronized 和 volatile 有什么区别？

**一句话总结**：`volatile` 是**轻量级**同步，只保证可见性和有序性；`synchronized` 是**重量级**同步，保证可见性、有序性和原子性。

| 对比项 | volatile | synchronized |
|--------|----------|-------------|
| 修饰对象 | 只能修饰**变量** | 可以修饰**方法**和**代码块** |
| 原子性 | ❌ 不保证 | ✅ 保证 |
| 可见性 | ✅ 保证 | ✅ 保证 |
| 有序性 | ✅ 保证 | ✅ 保证 |
| 阻塞 | ❌ 不会阻塞 | ✅ 会阻塞 |
| 性能 | **更好** | 相对较差 |
| 使用场景 | 状态标志位（如 `boolean flag`） | 需要原子性操作的场景 |

**背诵口诀**：「**volatile 管可见不管原子，synchronized 三保险全管**」

---

### 17. 什么是 CompletableFuture？

**一句话总结**：`CompletableFuture` 是 Java 8 引入的**异步编程工具**，比传统的 `Future` 更强大——支持**链式调用、组合多个异步任务、异常处理**，解决了回调地狱的问题。

**常用方法**：

| 方法 | 作用 | 示例场景 |
|------|------|----------|
| `supplyAsync()` | 异步执行**有返回值**的任务 | 异步查询数据库 |
| `runAsync()` | 异步执行**无返回值**的任务 | 异步发送日志 |
| `thenApply()` | 上一步结果**转换**后返回 | 查到用户后转换为 DTO |
| `thenAccept()` | 上一步结果**消费**不返回 | 查到数据后打印日志 |
| `thenCompose()` | 上一步结果**作为下一步输入**（扁平化） | 查用户 → 查订单 |
| `allOf()` | 等待**所有**任务完成 | 并行调用多个接口后汇总 |
| `anyOf()` | 任意**一个**任务完成即返回 | 多个数据源取最快的 |
| `exceptionally()` | **异常处理** | 降级返回默认值 |

```java
// 示例：并行查询用户和订单，汇总结果
CompletableFuture<User> userFuture = CompletableFuture.supplyAsync(() -> getUser(id));
CompletableFuture<Order> orderFuture = CompletableFuture.supplyAsync(() -> getOrder(id));

CompletableFuture.allOf(userFuture, orderFuture).join(); // 等待两个都完成
User user = userFuture.get();
Order order = orderFuture.get();
```

---

## 复习优先级（3~5 年）

| 优先级 | 题目 |
|--------|------|
| **P0 高频必背** | 4. JMM、5. CAS、6. synchronized 原理、7. volatile、10. ThreadLocal、12. 线程池参数、14. 死锁 |
| **P1 中频建议掌握** | 1. 线程创建方式、3. sleep vs wait、8. 乐观悲观锁、9. ReentrantLock vs synchronized、11. 线程池好处、16. synchronized vs volatile、17. CompletableFuture |
| **P2 低频了解即可** | 2. 线程状态、13. AQS、15. CountDownLatch/CyclicBarrier/Semaphore |

