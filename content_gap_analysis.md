# 八股文面试题 · 内容缺口分析报告

> 基于通文件审读全部 13 个模块，对照写作规范中的题量规划表、JavaGuide / 面渣逆袭 / 小林 coding 等主流面试资源的高频考点进行交叉比对。
>
> **最后更新**：2026-03-13，P0 缺口全部补齐（Round 1-5），总题数 159 → 166。

---

## 总览

| 技术栈 | 建议题数 | 现有题数 | 差距 | 状态 |
|--------|---------|---------|------|------|
| Java 基础 | 20~25 | **22** | — | ✅ 达标 |
| Java 集合 | 12~15 | **12** | 0~3 | ⚠️ 刚及下限 |
| Java 并发 | 15~20 | **17** | — | ✅ 达标 |
| JVM | 12~15 | **16** | — | ✅ 达标（+2 P0） |
| Spring | 12~15 | **13** | — | ✅ 达标 |
| MySQL | 15~20 | **17** | — | ✅ 达标（+2 P0） |
| Redis | 10~15 | **12** | — | ✅ 达标 |
| 计算机网络 | 8~12 | **11** | — | ✅ 达标（+1 P0） |
| 操作系统 | 6~10 | **9** | — | ✅ 达标（+1 P0） |
| 设计模式 | 6~8 | **7** | — | ✅ 达标 |
| MyBatis | 5~8 | **7** | — | ✅ 达标 |
| 消息队列 | 7~10 | **8** | — | ✅ 达标（+1 P0） |
| 分布式与微服务 | 15~18 | **15** | 0~3 | ⚠️ 刚及下限 |
| **总计** | **143~201** | **166** | — | — |

> [!NOTE]
> P0 缺口已全部补齐（+7 题），原先踩线的 MySQL、消息队列均已超出下限。4 个模块曾踩线现在只剩分布式与微服务仍在下限。下面重点分析的是**考点覆盖缺口**——即使题量达标，某些中频考点也可能被遗漏。

---

## 逐模块缺口分析

### 1. Java 基础（22 题）✅ 题量达标

**已覆盖**：JDK/JRE/JVM、跨平台、基本类型、装箱拆箱、== vs equals、String 不可变、String/Builder/Buffer、重载重写、三大特性、接口 vs 抽象类、final、异常体系、值传递、hashCode/equals、深浅拷贝、static、泛型/类型擦除、反射、Object 方法、访问修饰符、序列化、Lambda

**缺口**：无明显高频缺口。覆盖非常全面。

---

### 2. Java 集合（12 题）⚠️ 踩线

**已覆盖**：框架概述、ArrayList vs LinkedList、ArrayList 扩容、HashMap 底层原理、HashMap put 流程、HashMap 扩容、HashMap vs Hashtable、ConcurrentHashMap、HashSet 底层、LinkedHashMap/TreeMap、Iterator/fail-fast、Comparable vs Comparator

**缺口（建议补 2~3 题）**：

| 缺失考点 | 高频程度 | 说明 |
|---------|---------|------|
| **PriorityQueue 优先队列** | 中频 | 堆的实现原理，面试会结合 TopK 问题来问 |
| **ArrayList vs Vector** | 低频 | 可以合并到已有题中作为追问 |
| **CopyOnWriteArrayList** | 中频 | 线程安全的 List 方案，fail-fast 那题已提及但没展开 |

---

### 3. Java 并发（17 题）✅ 题量达标

**已覆盖**：线程创建方式、线程状态、sleep vs wait、synchronized 原理/锁升级、volatile、JMM、CAS、乐观悲观锁、ReentrantLock vs synchronized、ThreadLocal、线程池好处、线程池 7 大参数、AQS、死锁、CountDownLatch/CyclicBarrier/Semaphore、synchronized vs volatile、CompletableFuture

**缺口**：

| 缺失考点 | 高频程度 | 说明 |
|---------|---------|------|
| **线程池如何设置核心线程数** | 高频 | IO 密集 vs CPU 密集的公式，面试经常追问 |
| **ReadWriteLock 读写锁** | 中频 | ReentrantReadWriteLock，读写分离场景 |
| **Atomic 原子类体系** | 中频 | AtomicInteger/AtomicReference/LongAdder 等，CAS 题虽提到但没展开 |

> [!TIP]
> 线程池核心线程数设置是**极高频追问点**，建议合并到第 12 题「线程池参数」中作为追问补充。

---

### 4. JVM（14 题）✅ 题量达标

**已覆盖**：内存结构、堆栈区别、对象创建过程、可达性分析、GC 算法、CMS vs G1、类加载过程、双亲委派、引用类型、调优参数、OOM 排查、内存泄漏、对象头、直接内存

**缺口**：

| 缺失考点 | 高频程度 | 说明 |
|---------|---------|------|
| **新生代对象晋升老年代的条件** | 高频 | 年龄阈值 15、大对象直接进老年代、Survivor 空间不足等 |
| **Minor GC / Major GC / Full GC 的区别** | 高频 | GC 触发条件和范围，面试常考 |
| **ZGC / Shenandoah** | 低频 | JDK 11+ 新一代收集器，了解即可 |

> [!IMPORTANT]
> 「对象晋升条件」和「GC 类型区分」是 JVM 模块的高频必考点，当前完全缺失，建议各补 1 题。

---

### 5. Spring（13 题）✅ 题量达标

**已覆盖**：IoC/DI、AOP/动态代理、Bean 生命周期、Bean 作用域、循环依赖/三级缓存、事务传播行为、事务失效、SpringMVC 流程、自动配置原理、@Autowired vs @Resource、四大注解区别、启动流程、Filter vs Interceptor

**缺口**：

| 缺失考点 | 高频程度 | 说明 |
|---------|---------|------|
| **@Configuration 和 @Component 的区别** | 中频 | Full 模式 vs Lite 模式，CGLIB 代理的差异 |
| **Spring Boot Starter 自定义** | 中频 | 自动装配扩展点，3~5 年常问 |

> [!NOTE]
> Spring 模块整体覆盖良好，上述 2 个点属于"有则加分"级别，不补也不影响大局。

---

### 6. MySQL（15 题）⚠️ 踩线

**已覆盖**：InnoDB vs MyISAM、B+ 树、聚簇索引/回表/覆盖索引、索引失效、ACID、隔离级别、MVCC、锁体系、三大日志、Explain、慢 SQL 优化、最左前缀、分库分表、主从复制、Binlog 格式

**缺口（建议补 2~3 题）**：

| 缺失考点 | 高频程度 | 说明 |
|---------|---------|------|
| **Buffer Pool 缓冲池** | 高频 | InnoDB 读写性能的核心，redo log 题提到了但没展开 |
| **两阶段提交（Redo Log + Binlog 一致性）** | 高频 | prepare → commit 两阶段写日志，保证崩溃恢复一致性 |
| **count(*) vs count(1) vs count(字段)** | 中频 | 面试常考小题，可以合并到 SQL 优化中 |
| **联合索引底层结构** | 中频 | 最左前缀题虽有但没解释 B+ 树上联合索引怎么排列 |

> [!IMPORTANT]
> 「Buffer Pool」和「两阶段提交」是 MySQL 进阶高频题，当前完全缺失。

---

### 7. Redis（12 题）✅ 题量达标

**已覆盖**：为什么快、数据类型及场景、RDB vs AOF、过期删除、内存淘汰、穿透/击穿/雪崩、缓存一致性、分布式锁、集群方案、消息队列、大 Key、Redis vs Memcached

**缺口**：

| 缺失考点 | 高频程度 | 说明 |
|---------|---------|------|
| **Redis 的数据结构底层实现** | 中频 | SDS / 跳表 / 压缩列表 / listpack / quicklist 等，第 2 题表格列了底层结构但没展开讲原理 |
| **Redis 哨兵选举原理** | 低频 | Raft 算法，了解即可 |

---

### 8. 计算机网络（10 题）✅ 题量达标

**已覆盖**：TCP/IP 模型、TCP vs UDP、三次握手、四次挥手、HTTP vs HTTPS、状态码、GET vs POST、URL 到页面全过程、Cookie vs Session、TCP 可靠传输

**缺口**：

| 缺失考点 | 高频程度 | 说明 |
|---------|---------|------|
| **HTTP/1.0 vs HTTP/1.1 vs HTTP/2 vs HTTP/3** | 高频 | 版本演进对比，持久连接/多路复用/QUIC，面试高频 |
| **HTTPS 的 TLS 握手过程** | 中频 | 对称加密 + 非对称加密 + 证书验证 |
| **WebSocket** | 低频 | 实时通信场景，了解即可 |

> [!IMPORTANT]
> HTTP 版本演进是网络模块的高频题，当前完全缺失，强烈建议补充。

---

### 9. 操作系统（8 题）✅ 题量达标

**已覆盖**：进程 vs 线程、IPC 方式、进程状态、调度算法、死锁、虚拟内存、内存布局、用户态 vs 内核态

**缺口**：

| 缺失考点 | 高频程度 | 说明 |
|---------|---------|------|
| **零拷贝（Zero Copy）** | 中频 | sendfile / mmap，Kafka / Netty / NIO 都用到，可与 JVM 直接内存题关联 |
| **IO 模型（BIO/NIO/AIO）** | 高频 | 可以放在网络或操作系统章节，Java NIO 面试高频 |

> [!IMPORTANT]
> 「IO 模型」是 Java 后端高频考点（BIO/NIO/AIO/多路复用），当前整个仓库都没有覆盖，这是一个**跨模块的全局缺口**。建议放在操作系统或新建「IO 与网络编程」章节。

---

### 10. 设计模式（7 题）✅ 题量达标

**已覆盖**：单例、工厂、代理、策略、观察者、模板方法、常见设计模式总结

**缺口**：

| 缺失考点 | 高频程度 | 说明 |
|---------|---------|------|
| **适配器模式** | 低频 | Spring 中 HandlerAdapter 就是适配器 |
| **装饰器模式** | 低频 | Java IO 流就是装饰器模式 |

> 设计模式模块整体够用，以上 2 个低频了。

---

### 11. MyBatis（7 题）✅ 题量达标

**已覆盖**：#{} vs ${}、一级二级缓存、动态 SQL、Mapper 绑定原理、分页实现、执行流程、延迟加载

**缺口**：

| 缺失考点 | 高频程度 | 说明 |
|---------|---------|------|
| **MyBatis 的插件/拦截器原理** | 中频 | PageHelper 那题提到了拦截器但没展开四大接口（Executor/StatementHandler/ParameterHandler/ResultSetHandler） |
| **MyBatis vs JPA/Hibernate** | 低频 | 了解即可 |

---

### 12. 消息队列（7 题）⚠️ 踩线

**已覆盖**：为什么用 MQ、MQ 选型对比、消息不丢失、幂等性、顺序性、死信队列、消息积压

**缺口（建议补 2~3 题）**：

| 缺失考点 | 高频程度 | 说明 |
|---------|---------|------|
| **RocketMQ 事务消息原理** | 高频 | half 消息 + 回查机制，分布式事务场景常考 |
| **Kafka 的架构和核心概念** | 高频 | Partition / Consumer Group / Offset / ISR，大数据方向必问 |
| **延迟队列的实现方案** | 中频 | RabbitMQ 的 TTL+DLQ、RocketMQ 延迟级别、时间轮 |

> [!WARNING]
> 消息队列模块目前只有通用概念，缺少**具体 MQ 产品的原理题**。选型对比题虽然列了 Kafka/RabbitMQ/RocketMQ，但没有任何一个产品的架构、核心概念深入讲解。建议至少补 Kafka 或 RocketMQ 的架构题 1~2 道。

---

### 13. 分布式与微服务（15 题）⚠️ 踩线

**已覆盖**：CAP、BASE、分布式事务、分布式 ID、微服务核心组件、熔断降级、Nacos vs ZK、RPC vs HTTP、Gateway 原理、OpenFeign 原理、Nacos 配置中心、Sentinel 限流熔断、调用链路、负载均衡、Spring Cloud vs Dubbo

**缺口（建议补 2~3 题）**：

| 缺失考点 | 高频程度 | 说明 |
|---------|---------|------|
| **分布式锁（完整版）** | 高频 | Redis 分布式锁在 Redis 章已有，但 ZooKeeper 分布式锁、Redisson 的 WatchDog 机制没有在分布式章节完整串联 |
| **接口幂等性设计** | 高频 | 消息队列章讲了 MQ 幂等，但通用的接口幂等（Token 机制、唯一请求 ID）没有 |
| **限流算法** | 中频 | 令牌桶、漏桶、滑动窗口，Sentinel 题只讲了规则没讲算法原理 |
| **服务链路追踪（Skywalking / Zipkin）** | 低频 | 了解即可 |

---

## 全局缺口汇总（优先级排序）

### 🔴 P0 — 高频必补（✅ 已全部补齐，Round 1-5）

| # | 缺失考点 | 归属章节 | 状态 | 补充轮次 |
|---|---------|--------|------|----------|
| 1 | **IO 模型（BIO/NIO/AIO/多路复用）** | 操作系统 Q9 | ✅ 已补 | Round 4 |
| 2 | **HTTP 版本演进（1.0/1.1/2/3）** | 计算机网络 Q6 | ✅ 已补 | Round 3 |
| 3 | **Minor/Major/Full GC 区别与触发条件** | JVM Q7 | ✅ 已补 | Round 1 |
| 4 | **对象晋升老年代的条件** | JVM Q8 | ✅ 已补 | Round 1 |
| 5 | **Buffer Pool 缓冲池** | MySQL Q10 | ✅ 已补 | Round 2 |
| 6 | **两阶段提交（Redo Log + Binlog 一致性）** | MySQL Q11 | ✅ 已补 | Round 2 |
| 7 | **Kafka 核心架构** | 消息队列 Q2 | ✅ 已补 | Round 5 |

### 🟡 P1 — 中频建议补

| # | 缺失考点 | 建议归属章节 |
|---|---------|------------|
| 8 | 线程池核心线程数怎么设置（IO密集 vs CPU密集） | Java 并发 |
| 9 | CopyOnWriteArrayList | Java 集合 |
| 10 | HTTPS TLS 握手过程 | 计算机网络 |
| 11 | 零拷贝 | 操作系统 |
| 12 | RocketMQ 事务消息原理 | 消息队列 |
| 13 | 限流算法（令牌桶/漏桶/滑动窗口） | 分布式与微服务 |
| 14 | 接口幂等性设计（通用版） | 分布式与微服务 |
| 15 | 联合索引底层 B+ 树结构 | MySQL |
| 16 | Redis 底层数据结构详解 | Redis |

### 🟢 P2 — 低频可选

| # | 缺失考点 | 建议归属章节 |
|---|---------|------------|
| 17 | PriorityQueue 堆实现 | Java 集合 |
| 18 | ReadWriteLock 读写锁 | Java 并发 |
| 19 | Atomic 原子类体系 | Java 并发 |
| 20 | ZGC / Shenandoah | JVM |
| 21 | @Configuration vs @Component | Spring |
| 22 | 适配器/装饰器模式 | 设计模式 |
| 23 | WebSocket | 计算机网络 |
| 24 | Skywalking 链路追踪 | 分布式与微服务 |

---

## 补题进度

✅ **P0（7 题）已全部补齐**，总题数 159 → **166 题**，主流高频考点已覆盖。

下一步补 **P1（9 题）**，补完后总题数 → **175 题**，基本无死角。

P1 建议顺序：① HTTPS TLS 握手（计算机网络）② 线程池核心线程数（Java 并发）③ 零拷贝（操作系统）④ 联合索引底层 B+ 树（MySQL）⑤ RocketMQ 事务消息（消息队列）
