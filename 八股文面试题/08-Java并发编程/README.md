# 08 - Java 并发编程

> 覆盖并发编程核心知识：线程池、锁机制、volatile、CAS、AQS、ThreadLocal 等高频面试考点。线程池和锁是面试重灾区。

---

## 一、线程基础

### 1. 线程的创建方式有哪些？

> ⭐⭐⭐ 常问 | 难度：⭐⭐

**回答**：严格来说只有一种——`new Thread()`。但传入执行逻辑的方式有四种，面试一般答四种就行：

| 方式 | 核心写法 | 特点 |
|------|---------|------|
| 继承 Thread | `class MyThread extends Thread` | 简单，但 Java 单继承，不灵活 |
| 实现 Runnable | `new Thread(new Runnable(){...})` | ⚡**最常用**，解耦任务和线程 |
| 实现 Callable | `new FutureTask(new Callable(){...})` | 有返回值，能抛异常 |
| 线程池 | `executor.submit(task)` | ⚡**生产环境必须用这种**，复用线程 |

```java
// Callable + FutureTask：能拿到返回值
FutureTask<String> task = new FutureTask<>(() -> {
    return "执行结果";
});
new Thread(task).start();
String result = task.get(); // 阻塞等待结果
```

> **面试追问**：Runnable 和 Callable 的区别？Callable 有返回值（`call()` 返回泛型），能抛受检异常；Runnable 没有返回值（`run()` 返回 void），不能抛受检异常。

---

### 2. 线程有哪几种状态？

> ⭐⭐⭐ 常问 | 难度：⭐⭐

**回答**：Java 线程有 ⚡**6 种**状态，定义在 `Thread.State` 枚举中。你可以理解为一个员工的工作状态：

| 状态 | 说明 | 通俗理解 |
|------|------|---------|
| **NEW** | 线程创建了但还没调 `start()` | 签了合同还没入职 |
| **RUNNABLE** | 正在运行或等待 CPU 调度 | 在工位上干活（或排队等电脑） |
| **BLOCKED** | 等待获取 synchronized 锁 | 会议室被占了，在门口等 |
| **WAITING** | 调了 `wait()`/`join()`/`park()`，无限等待 | 等同事通知才能继续 |
| **TIMED_WAITING** | 调了 `sleep(n)`/`wait(n)`，限时等待 | 午休，定了闹钟 |
| **TERMINATED** | 执行完毕或异常退出 | 离职了 |

```
NEW ──start()──→ RUNNABLE ←──→ BLOCKED（等锁）
                    ↕
              WAITING / TIMED_WAITING
                    ↓
               TERMINATED
```

---

## 二、锁与同步

### 3. synchronized 的原理？锁升级过程？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐⭐

**一句话回答**：synchronized 底层依赖 Monitor（监视器锁），JDK 6 之后引入了锁升级机制：无锁 → 偏向锁 → 轻量级锁 → 重量级锁，逐步升级、不可降级。

**通俗理解**：

把 synchronized 想象成一间"自习室"的门锁：
- **无锁** = 自习室没人，门开着
- **偏向锁** = 只有你一个人来自习，门上贴个便利贴写"小王专用"，下次来直接进，不用开锁（⚡**零开销**）
- **轻量级锁** = 来了第二个人，便利贴撕掉，改成密码锁，两个人轮流试密码（**CAS 自旋**），谁试对了谁进去
- **重量级锁** = 试密码的人太多了，干脆换成保安看门，没轮到的人去旁边椅子上坐着等（**线程阻塞，交给操作系统调度**）

**回到技术**："便利贴"就是在对象头的 Mark Word 中记录线程 ID，下次同一个线程来直接比对，不需要 CAS。"试密码"就是 CAS 自旋尝试获取锁，线程不阻塞，适合锁竞争不激烈的场景。"保安看门"就是通过操作系统的 **mutex（互斥量）** 实现，涉及用户态到内核态的切换，开销很大。

**原理详解**：

```
锁升级过程（只升不降）

无锁 ──→ 偏向锁 ──→ 轻量级锁 ──→ 重量级锁
       同一线程    出现竞争     自旋超过阈值
       反复进入    CAS 自旋     或竞争激烈
```

| 锁状态 | Mark Word 存什么 | 适用场景 | 性能 |
|--------|-----------------|---------|------|
| **偏向锁** | 线程 ID | 只有一个线程访问 | ⚡最快，无 CAS |
| **轻量级锁** | 指向栈中锁记录的指针 | 两个线程交替访问，竞争不激烈 | CAS 自旋，不阻塞 |
| **重量级锁** | 指向 Monitor 的指针 | 多线程激烈竞争 | 最慢，线程阻塞挂起 |

> **注意**：JDK 15 之后默认禁用了偏向锁（`-XX:-UseBiasedLocking`），因为现代应用大多是多线程场景，偏向锁的撤销开销反而成了负担。

**🎤 面试这样答**：
> "synchronized 底层依赖对象头的 Mark Word 和 Monitor 实现。JDK 6 引入了锁升级优化：无锁状态下，第一个线程进入时升级为偏向锁，在 Mark Word 记录线程 ID，后续同一线程进入零开销。当出现第二个线程竞争时，升级为轻量级锁，通过 CAS 自旋获取锁，线程不阻塞。如果自旋超过一定次数或竞争激烈，就升级为重量级锁，依赖操作系统的互斥量，未获取锁的线程会被阻塞。锁只能升级不能降级。"

---

### 4. synchronized 和 ReentrantLock 的区别？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：synchronized 是 JVM 层面的关键字，自动加锁释放锁；ReentrantLock 是 API 层面的类，需要手动加锁释放锁，但功能更强大。

**通俗理解**：synchronized 像自动门，走进去自动关、走出来自动开，省心但功能单一。ReentrantLock 像密码门，需要自己输密码开门、出来手动锁门，麻烦一点但能设超时、能插队、能同时开多个门。

**回到技术**："自动门"就是 JVM 层面自动管理加锁和释放锁，进入 synchronized 代码块自动加锁，退出（包括异常）自动释放。"密码门"就是 ReentrantLock 需要手动调用 `lock()` 和 `unlock()`，忘了释放就会死锁。"能设超时"就是 `tryLock(timeout)`，"能插队"就是公平锁机制，"能同时开多个门"就是可以创建多个 Condition 实现精确唤醒。

| 对比项 | synchronized | ReentrantLock |
|--------|-------------|---------------|
| 层面 | JVM 关键字 | JUC 包的类 |
| 加锁/释放 | ⚡**自动**（进入代码块加锁，退出释放） | **手动**（`lock()` / `unlock()`，必须在 finally 中释放） |
| 可中断 | 不可中断，死等 | ⚡`lockInterruptibly()` 可中断等待 |
| 超时获取 | 不支持 | ⚡`tryLock(timeout)` 超时放弃 |
| 公平锁 | 只有非公平锁 | ⚡可选公平/非公平（构造参数 `new ReentrantLock(true)`） |
| 条件变量 | 只有一个 `wait/notify` | 可以创建⚡**多个 Condition**，精确唤醒 |

```java
// ReentrantLock 标准用法
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // 业务逻辑
} finally {
    lock.unlock(); // 必须在 finally 中释放，否则死锁
}
```

> **怎么选？** 优先用 synchronized（简单不易出错），需要超时、可中断、公平锁、多条件变量等高级功能时才用 ReentrantLock。

**🎤 面试这样答**：
> "synchronized 是 JVM 关键字，自动加锁释放锁，简单不易出错；ReentrantLock 是 JUC 包的类，需要手动加锁释放锁，但功能更强大——支持可中断等待、超时获取锁、公平锁、多个 Condition 精确唤醒。一般场景优先用 synchronized，需要高级功能时才用 ReentrantLock。"

---

### 5. volatile 关键字的作用？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：volatile 保证**可见性**和**有序性**，但不保证原子性。

**通俗理解**：

多线程就像几个人同时在各自的草稿纸（工作内存）上算题，算完再抄到黑板（主内存）上。问题是：你在草稿纸上改了答案，别人看的还是黑板上的旧答案。volatile 就是告诉大家："这个变量别抄到草稿纸上了，每次都直接看黑板、直接写黑板。"

**回到技术**："草稿纸"就是线程的**工作内存（CPU 缓存）**，"黑板"就是**主内存**。volatile 修饰的变量，每次读都从主内存读最新值，每次写都立刻刷回主内存，并通知其他线程缓存失效。

| 特性 | 说明 |
|------|------|
| **可见性** | 一个线程修改了 volatile 变量，其他线程⚡**立刻可见** |
| **有序性** | 禁止指令重排序（通过**内存屏障**实现） |
| **不保证原子性** | `i++` 这种"读-改-写"操作，volatile ⚡**管不了** |

```java
// 经典用法：双重检查锁的单例模式
public class Singleton {
    // 必须加 volatile，防止指令重排导致拿到未初始化的对象
    private static volatile Singleton instance;

    public static Singleton getInstance() {
        if (instance == null) {                // 第一次检查，避免不必要的加锁
            synchronized (Singleton.class) {
                if (instance == null) {        // 第二次检查，防止重复创建
                    instance = new Singleton(); // 非原子操作：1.分配内存 2.初始化 3.赋值引用
                }
            }
        }
        return instance;
    }
}
```

> **为什么 DCL 单例必须加 volatile？** `new Singleton()` 分三步：①分配内存 ②初始化对象 ③引用指向内存。JVM 可能把 ②③ 重排为 ③②，导致另一个线程拿到一个"分配了内存但还没初始化"的对象，用的时候就 NPE 了。volatile 禁止这个重排。

**🎤 面试这样答**：
> "volatile 有两个作用：一是保证可见性，修改后立刻刷回主内存并让其他线程的缓存失效；二是禁止指令重排序，通过内存屏障实现。但它不保证原子性，i++ 这种复合操作还是需要用 synchronized 或 AtomicInteger。最典型的应用是 DCL 双重检查锁单例，必须用 volatile 防止 new 对象时的指令重排。"

---

### 6. Java 内存模型（JMM）是什么？happens-before 规则？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐⭐

**一句话回答**：JMM（Java Memory Model）定义了多线程环境下变量的读写规则，核心是解决可见性和有序性问题。happens-before 规则是 JMM 提供的"承诺"——满足这些规则的操作，前一个操作的结果对后一个操作一定可见。

**通俗理解**：

多线程就像几个人同时在各自的草稿纸（工作内存）上算题，算完再抄到黑板（主内存）上。JMM 就是这间教室的"规章制度"，规定了什么时候必须看黑板、什么时候必须把草稿抄到黑板上。

happens-before 就是这套规章制度里的具体条款。比如"老师写完板书后，学生一定能看到"——这就是一条 happens-before 规则。

**回到技术**："草稿纸"就是线程的**工作内存（CPU 缓存/寄存器）**，"黑板"就是**主内存（堆内存）**。JMM 规定了线程何时从主内存读取、何时写回主内存。happens-before 是 JMM 对程序员的承诺：如果操作 A happens-before 操作 B，那么 A 的结果对 B 一定可见。

**核心 happens-before 规则**：

| 规则 | 含义 |
|------|------|
| **程序顺序规则** | 同一个线程内，前面的操作 happens-before 后面的操作 |
| **volatile 规则** | volatile 写 happens-before 后续的 volatile 读 |
| **锁规则** | unlock happens-before 后续的 lock（释放锁的操作对加锁的线程可见） |
| **线程启动规则** | `start()` happens-before 子线程的每个操作 |
| **线程终止规则** | 线程的所有操作 happens-before `join()` 返回 |
| **传递性** | A happens-before B，B happens-before C → A happens-before C |

```
JMM 内存交互示意：

线程 A 工作内存          主内存          线程 B 工作内存
┌──────────┐      ┌──────────┐      ┌──────────┐
│ 变量副本 x=1│ ──写回→ │   x=1    │ ←读取── │ 变量副本 x=0│
└──────────┘      └──────────┘      └──────────┘
                  （共享变量）

问题：线程 A 修改了 x，线程 B 可能还在用旧值
解决：volatile / synchronized / happens-before 规则保证可见性
```

**🎤 面试这样答**：
> "JMM 是 Java 内存模型，定义了多线程环境下共享变量的读写规则。每个线程有自己的工作内存，共享变量存在主内存中。JMM 通过 happens-before 规则来保证可见性和有序性：比如 volatile 写 happens-before 后续的 volatile 读，unlock happens-before 后续的 lock，还有传递性规则。满足 happens-before 关系的两个操作，前一个的结果对后一个一定可见。volatile 和 synchronized 的可见性保证本质上都是基于 JMM 的 happens-before 规则。"

---

### 7. CAS 是什么？有什么问题？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：CAS（Compare And Swap）是一种无锁的原子操作，比较内存值和预期值，相同则更新，不同则重试。

**通俗理解**：

你去抢最后一张电影票。你看到还剩 1 张（预期值），准备下单时再确认一下——如果还是 1 张（内存值 == 预期值），就改成 0 张，抢票成功。如果发现已经变成 0 张了（被别人抢了），就放弃或重试。整个过程不需要"锁住售票窗口"，大家都能同时尝试。

**回到技术**："看到还剩 1 张"就是读取当前值，"再确认一下"就是 Compare，"改成 0 张"就是 Swap。这三步由 CPU 指令（`cmpxchg`）保证是⚡**原子的**，不会被打断。

```java
// CAS 伪代码
boolean cas(内存地址, 预期值, 新值) {
    if (内存地址的当前值 == 预期值) {
        内存地址的当前值 = 新值;
        return true;  // 更新成功
    }
    return false;     // 更新失败，别人先改了
}

// Java 中的 CAS：AtomicInteger
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet(); // 底层就是 CAS 自旋：失败了就重试
```

**CAS 的三大问题**：

| 问题 | 说明 | 解决方案 |
|------|------|---------|
| **ABA 问题** | 值从 A 改成 B 又改回 A，CAS 以为没变过 | 加版本号：⚡`AtomicStampedReference` |
| **自旋开销** | 竞争激烈时一直重试，CPU 空转 | 自旋次数超限后升级为锁 |
| **只能保证一个变量的原子性** | 多个变量的复合操作管不了 | 用 `AtomicReference` 包装成一个对象，或用锁 |

**🎤 面试这样答**：
> "CAS 是 Compare And Swap，一种乐观锁思想的无锁算法。它比较内存中的值和预期值，相等就更新为新值，不相等就重试。Java 中 AtomicInteger 等原子类底层都是 CAS 实现的。CAS 有三个问题：ABA 问题可以用 AtomicStampedReference 加版本号解决；自旋开销在竞争激烈时会浪费 CPU；以及它只能保证单个变量的原子性。"

---

## 三、线程池

### 8. 线程池的核心参数有哪些？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：线程池有 7 个核心参数，面试必须全部说出来。

**通俗理解**：

把线程池想象成一家餐厅：

| 参数 | 餐厅比喻 | 技术含义 |
|------|---------|---------|
| **corePoolSize** | 正式员工数 | ⚡核心线程数，即使空闲也不裁 |
| **maximumPoolSize** | 最大员工数（正式 + 临时工） | ⚡最大线程数 |
| **keepAliveTime** | 临时工闲多久就辞退 | 非核心线程的空闲存活时间 |
| **unit** | 时间单位（小时/分钟） | keepAliveTime 的单位 |
| **workQueue** | 等候区的椅子 | ⚡阻塞队列，存放等待执行的任务 |
| **threadFactory** | 招聘标准（名字、优先级） | 创建线程的工厂 |
| **handler** | 等候区满了怎么办 | ⚡拒绝策略 |

**回到技术**："正式员工"就是核心线程，即使空闲也不会被回收。"临时工"就是非核心线程，空闲超过 keepAliveTime 就会被销毁。"等候区的椅子"就是阻塞队列（BlockingQueue），常用的有 LinkedBlockingQueue（有界/无界）和 ArrayBlockingQueue（有界）。"等候区满了怎么办"就是拒绝策略，决定队列满且线程数达到上限时如何处理新任务。

```java
// 线程池标准创建方式（生产环境推荐手动创建，不要用 Executors）
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5,                      // corePoolSize：核心线程数
    10,                     // maximumPoolSize：最大线程数
    60L,                    // keepAliveTime：临时线程空闲存活时间
    TimeUnit.SECONDS,       // unit：时间单位
    new LinkedBlockingQueue<>(100),  // workQueue：任务队列
    Executors.defaultThreadFactory(), // threadFactory：线程工厂
    new ThreadPoolExecutor.CallerRunsPolicy() // handler：拒绝策略
);
```

**四种拒绝策略**：

| 策略 | 行为 | 适用场景 |
|------|------|---------|
| **AbortPolicy** | ⚡**默认**，直接抛 `RejectedExecutionException` | 关键业务，不能丢任务 |
| **CallerRunsPolicy** | 让提交任务的线程自己执行 | ⚡**推荐**，既不丢任务又能降速 |
| **DiscardPolicy** | 默默丢弃，不抛异常 | 无关紧要的任务 |
| **DiscardOldestPolicy** | 丢弃队列中最老的任务 | 只关心最新任务 |

> **为什么不要用 Executors 创建线程池？** `Executors.newFixedThreadPool()` 和 `newSingleThreadExecutor()` 的队列是 ⚡**无界的 LinkedBlockingQueue**，任务堆积会导致 OOM。`newCachedThreadPool()` 的最大线程数是 ⚡**Integer.MAX_VALUE**，可能创建大量线程耗尽资源。

**🎤 面试这样答**：
> "线程池有 7 个核心参数：核心线程数、最大线程数、空闲存活时间、时间单位、工作队列、线程工厂、拒绝策略。生产环境必须用 ThreadPoolExecutor 手动创建，不能用 Executors 的快捷方法，因为 FixedThreadPool 的无界队列会导致 OOM，CachedThreadPool 的无限线程数会耗尽资源。拒绝策略推荐 CallerRunsPolicy，既不丢任务又能起到限流作用。"

---

### 9. 线程池的工作流程？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：先用核心线程，核心满了放队列，队列满了创建临时线程，临时线程也满了就执行拒绝策略。

**通俗理解**：

还是餐厅的例子，来了一个客人（任务）：
1. 正式员工（核心线程）有空的 → 直接接待
2. 正式员工都忙 → 客人去等候区坐着（放入队列）
3. 等候区也坐满了 → 叫临时工来帮忙（创建非核心线程）
4. 临时工也招满了（达到最大线程数）→ 对不起，今天不接客了（拒绝策略）

**回到技术**："正式员工有空直接接待"就是当前线程数小于 corePoolSize 时，直接创建核心线程执行任务。"去等候区坐着"就是任务被放入 workQueue。"叫临时工"就是队列满了才创建非核心线程。注意顺序是核心线程→队列→临时线程，不是核心线程→临时线程→队列。

**原理详解**：

```
提交任务 execute(task)
        │
        ▼
① 当前线程数 < corePoolSize？
        │是                │否
        ▼                  ▼
   创建核心线程执行    ② workQueue 没满？
                        │是          │否
                        ▼            ▼
                    任务入队     ③ 当前线程数 < maximumPoolSize？
                                  │是              │否
                                  ▼                ▼
                             创建临时线程执行    ④ 执行拒绝策略
```

> **关键细节**：注意顺序是"核心线程 → 队列 → 临时线程"，⚡**不是**"核心线程 → 临时线程 → 队列"。也就是说，队列没满之前不会创建临时线程。这是面试常见的坑。

**🎤 面试这样答**：
> "线程池提交任务的流程分四步：第一步，当前线程数小于核心线程数，直接创建核心线程执行；第二步，核心线程满了，任务放入阻塞队列排队；第三步，队列也满了，才创建非核心线程执行；第四步，线程数达到最大值且队列也满了，执行拒绝策略。关键是队列没满之前不会创建临时线程，这个顺序面试经常考。"

---

## 四、并发工具

### 10. ThreadLocal 原理？为什么会内存泄漏？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐⭐

**一句话回答**：ThreadLocal 让每个线程拥有自己独立的变量副本，底层是每个线程维护一个 ThreadLocalMap。内存泄漏是因为 key 是弱引用会被回收，但 value 是强引用不会被回收。

**通俗理解**：

公司每个员工（线程）的工位上有一个抽屉（ThreadLocalMap），抽屉里可以放自己的私人物品（变量副本）。每个人只能打开自己的抽屉，互不干扰。

**回到技术**：每个 Thread 对象内部有一个 `ThreadLocal.ThreadLocalMap` 字段，这个 Map 的 key 是 ThreadLocal 对象本身（弱引用），value 是你存的值（强引用）。

**内存泄漏原理**：

```
Thread 对象
  └── ThreadLocalMap
        └── Entry[]
              ├── Entry: key(弱引用) → ThreadLocal 对象
              │          value(强引用) → 你存的值
              └── ...

当 ThreadLocal 变量被置为 null 后：
  - key 是弱引用 → GC 时被回收，key 变成 null
  - value 是强引用 → 不会被回收 ← ⚡ 泄漏点！
  - 这个 Entry 变成了 key=null 但 value 还在的"脏数据"
```

> **解决办法**：用完 ThreadLocal 后⚡**必须调用 `remove()`**，尤其是在线程池场景下——线程会被复用，如果不 remove，上一个任务存的值会被下一个任务读到。

```java
ThreadLocal<String> userContext = new ThreadLocal<>();
try {
    userContext.set("当前用户信息");
    // 业务逻辑...
} finally {
    userContext.remove(); // 必须清理，防止内存泄漏和数据串线程
}
```

**🎤 面试这样答**：
> "ThreadLocal 底层是每个线程维护一个 ThreadLocalMap，key 是 ThreadLocal 对象的弱引用，value 是强引用。内存泄漏的原因是：当 ThreadLocal 变量被回收后，key 变成 null，但 value 还被 Entry 强引用着无法回收。解决办法是用完后必须调用 remove()，特别是线程池场景下线程会复用，不清理还会导致数据串到其他请求。"

---

### 11. AQS 原理？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐⭐

**一句话回答**：AQS（AbstractQueuedSynchronizer）是 JUC 锁和同步工具的底层框架，核心是一个 state 变量 + 一个 CLH 双向队列。

**通俗理解**：

AQS 就像银行的叫号系统：
- **state 变量** = 柜台状态（0 表示空闲，≥1 表示有人在办业务）
- **CLH 队列** = 等候区的排队队伍，先来先排，叫到号才能去柜台

**回到技术**：state 用 `volatile int` 保证可见性，通过 CAS 修改。获取锁就是把 state 从 0 改成 1，获取失败的线程被包装成 Node 节点加入 CLH 双向队列排队等待。

**原理详解**：

```
AQS 核心结构

        ┌─────────────────────┐
        │   state（volatile）   │
        │   0=无锁  1=已锁     │
        └─────────────────────┘
                  │
        ┌─────────▼──────────┐
        │   CLH 双向等待队列   │
        │                    │
        │  head ←→ Node(T1) ←→ Node(T2) ←→ tail
        │         (等待中)      (等待中)
        └────────────────────┘
```

**谁用了 AQS？** 面试常追问：

| 工具 | 怎么用 state |
|------|-------------|
| **ReentrantLock** | state=0 无锁，state=1 已锁，state>1 重入次数 |
| **CountDownLatch** | state=计数值，每次 countDown() 减 1，到 0 唤醒等待线程 |
| **Semaphore** | state=许可数，acquire() 减 1，release() 加 1 |

**🎤 面试这样答**：
> "AQS 是 JUC 中锁和同步器的基础框架，核心是一个 volatile 的 state 变量和一个 CLH 双向等待队列。获取锁时通过 CAS 修改 state，成功就获取锁，失败就封装成 Node 加入队列排队。ReentrantLock、CountDownLatch、Semaphore 底层都是基于 AQS 实现的，区别在于对 state 的语义不同。"

---

### 12. CountDownLatch、CyclicBarrier、Semaphore 的区别？

> ⭐⭐⭐ 常问 | 难度：⭐⭐⭐

**回答**：三个都是并发控制工具，区别在于"谁等谁"和"能不能复用"。

| 工具 | 通俗理解 | 核心方法 | 可复用 |
|------|---------|---------|--------|
| **CountDownLatch** | 班长等所有人交作业，⚡**全交齐了班长才走** | `countDown()` 交一份，`await()` 等全部交齐 | ❌ 一次性 |
| **CyclicBarrier** | 旅行团等所有人到齐才发车，⚡**到齐后可以再来一轮** | `await()` 每个人到了就等，人齐了一起走 | ✅ 可复用 |
| **Semaphore** | 停车场有 N 个车位，⚡**满了就等，走一辆进一辆** | `acquire()` 占车位，`release()` 让车位 | ✅ 可复用 |

```java
// CountDownLatch：主线程等 3 个子任务全部完成
CountDownLatch latch = new CountDownLatch(3);
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        // 执行任务...
        latch.countDown(); // 完成一个，计数减 1
    }).start();
}
latch.await(); // 主线程阻塞，直到计数为 0

// Semaphore：限流，同时最多 3 个线程执行
Semaphore semaphore = new Semaphore(3);
semaphore.acquire(); // 获取许可（车位）
try {
    // 执行业务...
} finally {
    semaphore.release(); // 释放许可
}
```

---

### 13. 什么是死锁？怎么排查和避免？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐

**一句话回答**：死锁是两个或多个线程互相持有对方需要的锁，导致所有线程都永远等待下去。死锁必须同时满足四个条件：互斥、持有并等待、不可剥夺、循环等待。

**通俗理解**：

两个人面对面过独木桥，你等我让路，我等你让路，谁也不退，两个人就卡死在桥上了。

**回到技术**："两个人"就是两个线程，"独木桥"就是锁资源。线程 A 持有锁 1 等待锁 2，线程 B 持有锁 2 等待锁 1，形成循环等待，就是死锁。

```java
// 死锁示例
Object lockA = new Object();
Object lockB = new Object();

// 线程 1：先拿 A，再拿 B
new Thread(() -> {
    synchronized (lockA) {
        // 拿到了 lockA，等一下再去拿 lockB
        synchronized (lockB) { /* 业务逻辑 */ }
    }
}).start();

// 线程 2：先拿 B，再拿 A → 和线程 1 顺序相反，死锁！
new Thread(() -> {
    synchronized (lockB) {
        synchronized (lockA) { /* 业务逻辑 */ }
    }
}).start();
```

**死锁的四个必要条件**：

| 条件 | 说明 |
|------|------|
| **互斥** | 资源同一时刻只能被一个线程持有 |
| **持有并等待** | 线程持有至少一个资源，同时等待获取其他资源 |
| **不可剥夺** | 线程持有的资源不能被强制夺走 |
| **循环等待** | 线程之间形成环形等待链 |

> ⚡**破坏任意一个条件就能避免死锁**。最常用的方法是破坏"循环等待"——所有线程按相同的顺序获取锁。

**排查死锁**：
- `jstack <pid>`：打印线程堆栈，会直接提示 "Found one Java-level deadlock"
- `jconsole` / `jvisualvm`：图形化工具，线程 Tab 可检测死锁
- `ThreadMXBean`：代码中调用 `findDeadlockedThreads()` 检测

**🎤 面试这样答**：
> "死锁是两个或多个线程互相持有对方需要的锁，导致永远等待。产生死锁需要同时满足四个条件：互斥、持有并等待、不可剥夺、循环等待。避免死锁最常用的方法是破坏循环等待条件，让所有线程按相同顺序获取锁。也可以用 tryLock 设置超时时间来避免。排查死锁可以用 jstack 命令打印线程堆栈，它会直接提示死锁信息。"

---

## 面试高频程度排序（3~5 年）

| 优先级 | 题目 |
|--------|------|
| ⭐⭐⭐⭐⭐ | 线程池的核心参数有哪些？ |
| ⭐⭐⭐⭐⭐ | 线程池的工作流程？ |
| ⭐⭐⭐⭐⭐ | synchronized 的原理和锁升级？ |
| ⭐⭐⭐⭐⭐ | synchronized 和 ReentrantLock 的区别？ |
| ⭐⭐⭐⭐⭐ | volatile 关键字的作用？ |
| ⭐⭐⭐⭐⭐ | JMM 和 happens-before 规则？ |
| ⭐⭐⭐⭐⭐ | CAS 是什么？有什么问题？ |
| ⭐⭐⭐⭐ | 死锁的条件、排查和避免？ |
| ⭐⭐⭐⭐ | ThreadLocal 原理和内存泄漏？ |
| ⭐⭐⭐⭐ | AQS 原理？ |
| ⭐⭐⭐ | 线程的创建方式？ |
| ⭐⭐⭐ | 线程有哪几种状态？ |
| ⭐⭐⭐ | CountDownLatch、CyclicBarrier、Semaphore？ |
