# 17 - IO 与网络编程

> 覆盖 IO 核心知识：BIO/NIO/AIO 模型对比、NIO 核心组件、零拷贝、Netty 原理、Reactor 模式等高频面试考点。BIO/NIO/AIO 的区别几乎每场必问。

---

## 一、IO 模型

### 1. BIO、NIO、AIO 的区别？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：BIO 是同步阻塞，一个连接一个线程；NIO 是同步非阻塞，一个线程处理多个连接；AIO 是异步非阻塞，操作系统完成 IO 后通知线程。

**通俗理解**：

把 IO 想象成去餐厅吃饭：

- **BIO（同步阻塞）** = 你点完菜后坐在座位上干等，什么都不做，直到菜上来。一个服务员只能盯一桌客人 → ⚡**一个连接一个线程**
- **NIO（同步非阻塞）** = 你点完菜后可以先玩手机，时不时抬头看一眼菜好了没。一个服务员可以同时照看多桌 → ⚡**一个线程处理多个连接**（多路复用）
- **AIO（异步非阻塞）** = 你点完菜后该干嘛干嘛，菜好了服务员主动端到你面前通知你 → ⚡**操作系统完成后回调通知**

**回到技术**："干等"就是线程阻塞在 IO 操作上，直到数据就绪。"时不时看一眼"就是 NIO 的 Selector 轮询多个 Channel，哪个就绪就处理哪个。"主动通知"就是 AIO 的回调机制，IO 操作由操作系统内核完成后通知应用程序。

| 对比项 | BIO | NIO | AIO |
|--------|-----|-----|-----|
| 模型 | 同步阻塞 | ⚡**同步非阻塞** | 异步非阻塞 |
| 线程模型 | 一连接一线程 | ⚡**一线程多连接**（Selector） | 回调通知 |
| 吞吐量 | 低 | ⚡**高** | 高 |
| 编程复杂度 | 简单 | 复杂 | 复杂 |
| 适用场景 | 连接数少且固定 | ⚡**高并发**（Netty 底层就是 NIO） | Linux 下支持不好，实际用得少 |

> **实际开发中**：BIO 适合连接数少的场景；⚡**NIO 是主流**，Netty、Tomcat（NIO 模式）都基于它；AIO 在 Linux 上支持不完善，实际项目中很少直接用。

**🎤 面试这样答**：
> "BIO 是同步阻塞，一个连接占一个线程，连接多了线程资源耗尽。NIO 是同步非阻塞，核心是 Selector 多路复用，一个线程可以监听多个连接，哪个就绪就处理哪个，适合高并发场景，Netty 底层就是基于 NIO。AIO 是异步非阻塞，IO 操作由内核完成后回调通知，但 Linux 上支持不好，实际用得少。"

---

### 2. NIO 的核心组件？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐

**一句话回答**：NIO 有三大核心组件——Channel（通道）、Buffer（缓冲区）、Selector（选择器）。

**通俗理解**：把 NIO 想象成一个快递分拣中心。Channel 是传送带（数据通道），Buffer 是包裹箱（数据暂存区），Selector 是分拣员（监听多条传送带，哪条有包裹就处理哪条）。

**回到技术**："传送带"就是 Channel，和 BIO 的 Stream 不同，Channel 是双向的，可读可写。"包裹箱"就是 Buffer，NIO 的读写都要经过 Buffer，不能直接操作 Channel。"分拣员"就是 Selector（多路复用器），一个线程通过 Selector 可以同时监听多个 Channel 的事件（连接、读、写），这就是 NIO 能用少量线程处理大量连接的关键。

| 组件 | 作用 | 类比 |
|------|------|------|
| **Channel** | 双向数据通道，可读可写 | 传送带（BIO 的 Stream 是单向的） |
| **Buffer** | 数据缓冲区，读写数据都要经过 Buffer | 包裹箱 |
| **Selector** | ⚡**多路复用器**，一个线程监听多个 Channel | 分拣员 |

```java
// NIO 服务端核心代码（简化版）
Selector selector = Selector.open();
ServerSocketChannel serverChannel = ServerSocketChannel.open();
serverChannel.configureBlocking(false); // 非阻塞模式
serverChannel.register(selector, SelectionKey.OP_ACCEPT); // 注册到 Selector

while (true) {
    selector.select(); // 阻塞等待就绪事件
    Set<SelectionKey> keys = selector.selectedKeys();
    for (SelectionKey key : keys) {
        if (key.isAcceptable()) { /* 处理新连接 */ }
        if (key.isReadable()) { /* 处理读事件 */ }
    }
}
```

> **为什么不直接用 NIO 而用 Netty？** 原生 NIO 编程复杂（要处理半包/粘包、空轮询 Bug 等），Netty 封装了 NIO 的底层细节，提供了更简洁的 API 和更完善的功能。

**🎤 面试这样答**：
> "NIO 有三大核心组件：Channel 是双向数据通道，Buffer 是数据缓冲区，Selector 是多路复用器。Selector 是关键，一个线程通过 Selector 可以同时监听多个 Channel 的 IO 事件，实现用少量线程处理大量连接。但原生 NIO 编程复杂，实际开发中一般用 Netty 框架。"

---

## 二、零拷贝

### 3. 什么是零拷贝？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐

**一句话回答**：零拷贝是指数据从磁盘发送到网络时，不需要经过用户态的拷贝，减少 CPU 拷贝次数和上下文切换，提升 IO 性能。Kafka 和 Netty 都用了零拷贝。

**通俗理解**：

传统方式就像你从仓库搬货到卡车：先从仓库搬到办公室（内核→用户态），再从办公室搬到卡车（用户态→内核）。零拷贝就是直接从仓库搬到卡车，不经过办公室。

**回到技术**："仓库"就是磁盘，"办公室"就是用户态内存，"卡车"就是网卡。传统 IO 数据要从内核缓冲区拷贝到用户缓冲区再拷贝回内核，零拷贝通过 `sendfile` 系统调用让数据直接在内核态从磁盘缓冲区传到网卡缓冲区，跳过用户态，减少了拷贝次数和上下文切换。

| 方式 | 拷贝次数 | 上下文切换 |
|------|---------|-----------|
| 传统 IO | ⚡**4 次**拷贝，4 次切换 | 磁盘→内核→用户→内核→网卡 |
| **mmap** | 3 次拷贝，4 次切换 | 内核缓冲区和用户空间共享映射 |
| **sendfile** | ⚡**2 次**拷贝，2 次切换 | 磁盘→内核→网卡（跳过用户态） |

> **Java 中的零拷贝**：`FileChannel.transferTo()` 底层调用 `sendfile()`，Netty 的 `FileRegion` 也是基于此实现。Kafka 消费者读消息时用的就是 `sendfile` 零拷贝。

**🎤 面试这样答**：
> "零拷贝是指数据从磁盘到网络不经过用户态拷贝。传统 IO 需要 4 次拷贝和 4 次上下文切换，sendfile 零拷贝只需要 2 次拷贝和 2 次切换。Java 中通过 FileChannel.transferTo() 实现，Kafka 消费消息和 Netty 文件传输都用了零拷贝来提升性能。"

---

## 三、Netty

### 4. Netty 是什么？为什么用 Netty？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐

**一句话回答**：Netty 是一个基于 NIO 的高性能网络通信框架，封装了 NIO 的复杂细节，提供简洁的 API。Dubbo、RocketMQ、Elasticsearch 底层都用了 Netty。

| 为什么用 Netty | 说明 |
|---------------|------|
| **API 简洁** | 原生 NIO 编程复杂，Netty 大幅简化 |
| **高性能** | 基于 Reactor 模型 + 零拷贝 + 内存池 |
| **解决 NIO Bug** | 原生 NIO 有 epoll 空轮询 Bug，Netty 已修复 |
| **协议支持丰富** | 内置 HTTP、WebSocket、自定义协议的编解码器 |
| **社区活跃** | 大量开源项目使用，久经生产验证 |

---

### 5. 什么是 Reactor 模式？Netty 的线程模型？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐⭐

**一句话回答**：Reactor 模式是一种事件驱动的 IO 处理模式，核心思想是用一个或少量线程监听事件，事件就绪后分发给对应的处理器。Netty 采用的是主从 Reactor 多线程模型。

**通俗理解**：

把 Reactor 想象成一家餐厅的运营模式：

- **单 Reactor 单线程** = 一个人既当前台接待客人，又当服务员点菜上菜，还当厨师炒菜。人少时没问题，人多了忙不过来
- **单 Reactor 多线程** = 一个前台接待客人、点菜，但炒菜交给后厨的多个厨师并行做。前台还是一个人，高峰期接待可能成瓶颈
- **主从 Reactor 多线程** = 一个大堂经理（主 Reactor）专门在门口接待新客人，接待完交给多个服务员（从 Reactor）负责后续点菜上菜，厨房有多个厨师（Worker 线程）并行炒菜 → ⚡**Netty 用的就是这种模型**

**回到技术**："大堂经理"就是 BossGroup，专门处理连接事件（accept）；"服务员"就是 WorkerGroup，负责处理已建立连接的读写事件；"厨师"就是业务线程池，处理耗时的业务逻辑。

**三种 Reactor 模型对比**：

| 模型 | 线程分配 | 优缺点 |
|------|---------|--------|
| 单 Reactor 单线程 | 一个线程干所有事 | 简单，但无法利用多核，一个 Handler 阻塞会卡住所有连接 |
| 单 Reactor 多线程 | 一个线程监听事件，业务交给线程池 | Reactor 线程仍是单点瓶颈 |
| ⚡**主从 Reactor 多线程** | BossGroup 接收连接，WorkerGroup 处理 IO | Netty 默认模型，高并发下性能最优 |

**Netty 的主从 Reactor 模型**：

```
                    ┌─────────────┐
  新连接 ──────────→│  BossGroup   │  主 Reactor：只负责 accept 新连接
                    │ (1~2 个线程) │
                    └──────┬──────┘
                           │ 把新连接注册到 WorkerGroup
                    ┌──────▼──────┐
                    │ WorkerGroup  │  从 Reactor：负责已连接的读写事件
                    │ (多个线程)    │  默认线程数 = ⚡CPU 核心数 × 2
                    └──────┬──────┘
                           │ 读到数据后交给 Pipeline 处理
                    ┌──────▼──────┐
                    │ ChannelPipeline │  Handler 链：编解码、业务逻辑
                    └─────────────┘
```

```java
// Netty 服务端代码（主从 Reactor 模型）
EventLoopGroup bossGroup = new NioEventLoopGroup(1);    // 主 Reactor，1 个线程
EventLoopGroup workerGroup = new NioEventLoopGroup();   // 从 Reactor，默认 CPU×2 个线程

ServerBootstrap bootstrap = new ServerBootstrap();
bootstrap.group(bossGroup, workerGroup)                 // 设置主从线程组
    .channel(NioServerSocketChannel.class)
    .childHandler(new ChannelInitializer<SocketChannel>() {
        @Override
        protected void initChannel(SocketChannel ch) {
            ch.pipeline()
                .addLast(new StringDecoder())            // 解码器
                .addLast(new MyBusinessHandler());       // 业务处理器
        }
    });
bootstrap.bind(8080).sync();
```

**🎤 面试这样答**：
> "Reactor 模式是一种事件驱动的 IO 处理模型，核心是用少量线程监听 IO 事件，事件就绪后分发给对应的 Handler 处理。有三种变体：单 Reactor 单线程、单 Reactor 多线程、主从 Reactor 多线程。Netty 采用的是主从 Reactor 多线程模型——BossGroup 负责接收新连接，WorkerGroup 负责处理已连接的读写事件，默认线程数是 CPU 核心数的 2 倍。每个连接的处理逻辑通过 ChannelPipeline 中的 Handler 链来完成。"

---

## 面试高频程度排序（3~5 年）

| 优先级 | 题目 |
|--------|------|
| ⭐⭐⭐⭐⭐ | BIO、NIO、AIO 的区别？ |
| ⭐⭐⭐⭐ | NIO 的核心组件？ |
| ⭐⭐⭐⭐ | 什么是零拷贝？ |
| ⭐⭐⭐⭐ | Netty 是什么？为什么用 Netty？ |
| ⭐⭐⭐⭐ | Reactor 模式？Netty 线程模型？ |