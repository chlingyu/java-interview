# 07-Java集合

---

## 一、List
### 1. ArrayList 和 LinkedList 的区别？什么场景用哪个？

一句话：**ArrayList 底层是数组，查询快、增删慢；LinkedList 底层是双向链表，增删快、查询慢。** 但实际开发中 **99% 的场景用 ArrayList**。

| 对比项 | ArrayList | LinkedList |
|--------|-----------|------------|
| 底层结构 | `Object[]` 数组 | 双向链表（每个节点存 `prev`、`item`、`next` 三个引用） |
| 随机访问 | **O(1)**，直接按下标算偏移量 | **O(n)**，要从头/尾一个一个遍历 |
| 头部插入/删除 | **O(n)**，要把后面所有元素往后搬一位 | **O(1)**，改几个指针就行 |
| 尾部插入 | **均摊 O(1)**（扩容时拷贝数组） | **O(1)** |
| 中间插入/删除 | **O(n)**，搬元素 | **O(n)**（找到位置是 O(n)，改指针是 O(1)） |
| 内存占用 | 紧凑，只存元素值 | 每个节点额外存两个指针引用，**占用约为 ArrayList 的 3~4 倍** |
| 实现的接口 | `List`、`RandomAccess` | `List`、`Deque`（可以当队列/栈用） |

**这题面试经常挖坑：LinkedList 的「增删快」是有条件的！**

只有在**已经定位到节点**的情况下（比如用迭代器遍历到那个位置），LinkedList 的增删才是 O(1)。如果你调用 `list.add(index, element)`，它还是要先 O(n) 遍历到那个位置，整体仍然是 O(n)。

**ArrayList 的扩容机制：**
- 默认初始容量：**10**（首次 `add` 时创建）
- 扩容公式：**新容量 = 旧容量 + (旧容量 >> 1)**，即**扩为原来的 1.5 倍**
- 扩容时用 `Arrays.copyOf()` 把旧数组整体拷贝到新数组，所以频繁扩容有性能损耗
- 优化建议：预估大小时用 `new ArrayList<>(1000)` 指定初始容量

**什么时候用 LinkedList？**

实际开发中**几乎不用 LinkedList**。因为现代 CPU 有缓存行优化（cache line），数组内存连续，**CPU 缓存命中率远高于链表**。LinkedList 的指针跳来跳去，缓存频繁失效，实测性能通常不如 ArrayList。只有在需要频繁在**头部**增删（如实现一个队列）时，LinkedList 才有优势。

**背诵口诀：** 数组连续查询快，链表指针增删快。实际开发用 ArrayList，LinkedList 当队列。

> 面试话术：「ArrayList 底层数组，随机访问 O(1)，增删要移动元素是 O(n)；LinkedList 底层双向链表，增删改指针 O(1) 但要先遍历定位。实际开发几乎都用 ArrayList，因为 CPU 缓存对数组更友好，LinkedList 只在需要频繁头部操作时考虑。」

## 二、Map
### 2. HashMap 的底层实现原理？JDK 1.8 做了哪些优化？

**HashMap 底层是「数组 + 链表 + 红黑树」（JDK 1.8）。** 数组的每个位置叫一个**桶（bucket）**，通过 key 的 hash 值决定放在哪个桶里；同一个桶内的元素形成链表，链表过长时转为红黑树。

**存储流程（`put(key, value)` 过程）：**
1. 计算 key 的 hash 值：`(h = key.hashCode()) ^ (h >>> 16)`（高 16 位异或低 16 位，叫**扰动函数**，让 hash 分布更均匀）
2. 用 `hash & (n - 1)` 算出数组下标（n 是数组长度，等价于 `hash % n`，但位运算更快）
3. 如果该位置为空，直接放入新节点
4. 如果该位置有元素（发生**哈希冲突**）：
   - 遍历链表，用 `equals()` 比较 key，找到相同的 key 就**覆盖**旧 value
   - 没找到就在**链表尾部**追加新节点（JDK 1.7 是头插法，1.8 改为**尾插法**）
5. 如果链表长度 ≥ **8** 且数组长度 ≥ **64**，链表转为**红黑树**（查不到时从 O(n) 变成 O(log n)）
6. 如果红黑树节点 ≤ **6**，退化回链表

**JDK 1.7 vs 1.8 对比：**

| 对比项 | JDK 1.7 | JDK 1.8 |
|--------|---------|---------|
| 底层结构 | 数组 + 链表 | 数组 + 链表 + **红黑树** |
| 插入方式 | **头插法**（新元素插到链表头部） | **尾插法**（新元素追加到链表尾部） |
| 扰动函数 | 4 次位运算 + 5 次异或 | **1 次位运算 + 1 次异或**（简化了） |
| 扩容时机 | 先判断是否需要扩容，再插入 | 先插入，再判断是否需要扩容 |
| hash 冲突严重时 | 链表 O(n) | 链表长度 ≥ 8 转红黑树 O(log n) |

**为什么 1.8 改为尾插法？** 因为头插法在多线程扩容时会导致**链表成环**，造成 `get()` 死循环。尾插法不会。（后面第 3 题详细说）

**背诵口诀：** 一个数组一堆桶，hash 取模定位桶，冲突了就挂链表，链太长转红黑树。七头八尾是关键。

> 面试话术：「HashMap 底层是数组+链表+红黑树。put 时先算 key 的 hash 值，通过扰动函数让高低位都参与计算，再用 hash & (n-1) 定位桶。冲突时用链表，链表长度超过 8 且数组长度超过 64 时转红黑树。JDK 1.8 的主要优化是引入红黑树、改头插法为尾插法、简化了扰动函数。」

### 3. HashMap 为什么线程不安全？会出现什么问题？

**HashMap 没有加任何同步机制，多线程并发操作时会出现数据丢失、死循环、数据覆盖等问题。**

**具体有三类问题：**

**① JDK 1.7：扩容时链表成环 → `get()` 死循环**

JDK 1.7 扩容时用**头插法**转移链表节点。假设链表是 A → B，两个线程同时扩容：
- 线程 1 暂停在中间状态
- 线程 2 完成扩容，头插法把链表变成 B → A
- 线程 1 恢复后继续用旧的指针操作，结果形成 A → B → A 的环
- 后续 `get()` 遍历到这个桶就会**死循环，CPU 飙到 100%**

JDK 1.8 改为**尾插法**解决了成环问题，但仍然不安全。

**② JDK 1.8：并发 `put` 导致数据覆盖**

两个线程同时 `put`，hash 到同一个桶且桶为空：
1. 线程 1 判断桶为空，准备写入，但还没写就被挂起
2. 线程 2 也判断桶为空，写入成功
3. 线程 1 恢复后直接写入，**覆盖了线程 2 的数据**

**③ `size()` 不准确**

`HashMap` 的 `size` 字段用普通 `int`（不是 `volatile`），并发 `put` 时多个线程同时 `size++`，最终 `size` 可能比实际元素数少。

**怎么解决？**

| 方案 | 说明 |
|------|------|
| `ConcurrentHashMap` | **首选**，分段锁（1.7）/ CAS + synchronized（1.8），性能最好 |
| `Collections.synchronizedMap()` | 把整个 map 用一把大锁包起来，简单粗暴但性能差 |
| `Hashtable` | 所有方法加 `synchronized`，性能最差，**已过时不推荐** |

**背诵口诀：** 七成环八覆盖，线程安全用 Concurrent。

> 面试话术：「HashMap 线程不安全。JDK 1.7 扩容时头插法会导致链表成环，get 死循环；1.8 虽然改成尾插法避免了成环，但并发 put 仍然可能数据覆盖。多线程环境应该用 ConcurrentHashMap。」

### 4. HashMap 的扩容机制？为什么容量是 2 的幂次？

**当元素数量超过 `容量 × 负载因子`（默认 16 × 0.75 = 12）时，HashMap 触发扩容，容量翻倍为原来的 2 倍。**

**扩容流程（JDK 1.8 `resize()` 方法）：**
1. 创建一个**容量为原来 2 倍**的新数组
2. 遍历旧数组的每个桶，把每个节点重新分配到新数组中
3. 重新定位的规则：对于旧数组中下标为 `i` 的桶里的每个节点——
   - 如果 `hash & oldCap == 0`，留在**原位置 i**
   - 如果 `hash & oldCap != 0`，移到**新位置 i + oldCap**

这个巧妙设计意味着扩容时**不需要重新计算 hash**，只需要看一位二进制就能判断新位置，效率很高。

**为什么容量必须是 2 的幂次？**

核心原因是：**只有容量为 2 的幂时，`hash & (n - 1)` 才等价于 `hash % n`。**

- 2 的幂减 1 后二进制全是 1（如 16 - 1 = 15 = `1111`）
- `hash & 1111` 就是取 hash 的低 4 位，结果均匀分布在 0~15
- 如果容量不是 2 的幂，比如 15 - 1 = 14 = `1110`，最低位永远是 0，所有奇数下标都不会被用到，**冲突率翻倍**

**默认参数：**

| 参数 | 值 | 说明 |
|------|-----|------|
| 默认初始容量 | **16** | `1 << 4` |
| 默认负载因子 | **0.75** | 空间与时间的折中：太小浪费空间，太大冲突多、链表长 |
| 扩容阈值 | 容量 × 负载因子 | 默认 16 × 0.75 = 12，超过 12 个元素就扩容 |
| 链表转红黑树 | 链表长度 ≥ **8** 且数组长度 ≥ **64** | 否则优先扩容而不是树化 |
| 红黑树退化链表 | 节点 ≤ **6** | 小于等于 6 转回链表 |

**这题面试经常挖坑：为什么负载因子是 0.75？**

0.75 是数学上在**时间和空间之间的最佳折中**。负载因子越大，空间利用率越高但冲突越多（查询变慢）；越小，冲突少但浪费空间。0.75 在泊松分布下使得**桶中出现 8 个及以上节点的概率不到千万分之一**，所以 8 作为树化阈值也是配套设计的。

**背诵口诀：** 超过阈值就翻倍，与运算定下标，2 的幂减一全是 1，分布均匀少冲突。

> 面试话术：「HashMap 容量超过阈值（容量 × 0.75）时扩容为 2 倍。容量必须是 2 的幂次，这样 hash & (n-1) 等价于取模但更快，且分布均匀。JDK 1.8 扩容时只需看 hash 的多出来的一位就能判断新位置，不需要重新计算 hash。」

### 5. ConcurrentHashMap 的实现原理？JDK 1.7 和 1.8 有什么区别？

**ConcurrentHashMap 是线程安全的 HashMap，核心思想是「减小锁粒度」。** JDK 1.7 用分段锁，1.8 用 CAS + synchronized 锁单个桶。

**JDK 1.7 实现：Segment 分段锁**

```
ConcurrentHashMap
├── Segment[0] → HashEntry[] → 链表
├── Segment[1] → HashEntry[] → 链表
├── ...
└── Segment[15] → HashEntry[] → 链表
```

- 数据分成 **16 个 Segment**（默认），每个 Segment 继承 `ReentrantLock`，**相当于 16 把独立的锁**
- 不同 Segment 的操作可以**并行**，最大并发度 = Segment 个数 = **16**
- 同一个 Segment 内的操作需要**竞争同一把锁**
- 每个 Segment 内部是一个小型 HashMap（数组 + 链表）

**JDK 1.8 实现：CAS + synchronized 锁桶头节点**

- 取消了 Segment，回归到和 HashMap 一样的**数组 + 链表 + 红黑树**结构
- 锁的粒度更细：**只锁数组中的某一个桶（头节点）**，不同桶的操作完全并行
- 空桶插入用 **CAS**（无锁操作），有冲突时才用 **synchronized** 锁住头节点
- `size()` 用 `baseCount + CounterCell[]` 分散累加，类似 `LongAdder` 的思想，避免所有线程竞争同一个计数器

**JDK 1.7 vs 1.8 对比：**

| 对比项 | JDK 1.7 | JDK 1.8 |
|--------|---------|---------|
| 数据结构 | Segment[] + HashEntry[] + 链表 | Node[] + 链表 + 红黑树 |
| 锁机制 | Segment 分段锁（`ReentrantLock`） | CAS + `synchronized`（锁桶头节点） |
| 锁粒度 | 锁一个 Segment（含多个桶） | **锁一个桶**（更细） |
| 最大并发度 | Segment 个数（默认 16） | **数组长度**（可以很大） |
| 查询复杂度 | O(n)（链表） | O(log n)（红黑树） |

**常见追问：ConcurrentHashMap 的 key 和 value 能不能为 null？**

**都不能。** `HashMap` 允许一个 null key 和多个 null value，但 `ConcurrentHashMap` 的 key 和 value 都**不允许 null**，否则抛 `NullPointerException`。原因是多线程下 `get(key)` 返回 null 时，你分不清到底是「key 不存在」还是「value 本身就是 null」，这会导致二义性。

**背诵口诀：** 七段八桶，七分段锁八锁头，CAS 加 synchronized，不允许 null。

> 面试话术：「ConcurrentHashMap 在 JDK 1.7 用 Segment 分段锁，默认 16 段，并发度有限；1.8 改用 CAS + synchronized 锁单个桶头节点，并发度等于数组长度，大幅提升。同时 1.8 引入了红黑树，查询从 O(n) 优化到 O(log n)。注意 key 和 value 都不能为 null。」

### 6. LinkedHashMap 和 TreeMap 的区别和使用场景？

一句话：**LinkedHashMap 按插入或访问顺序排列，TreeMap 按 key 的大小排序。**

| 对比项 | LinkedHashMap | TreeMap |
|--------|--------------|---------|
| 底层结构 | HashMap + **双向链表** | **红黑树** |
| 排序方式 | **插入顺序**（默认）或**访问顺序** | 按 key 的**自然顺序**或自定义 `Comparator` |
| 查询性能 | **O(1)**（继承 HashMap） | **O(log n)**（红黑树） |
| null key | 允许一个 | **不允许**（需要比较大小，null 无法比较） |
| 线程安全 | 不安全 | 不安全 |

**LinkedHashMap 的「访问顺序」模式：**

构造时传 `accessOrder = true`，每次 `get()` 或 `put()` 访问一个节点，会把它**移到链表尾部**。这个特性天然适合实现 **LRU 缓存**（最近最少使用）：

```java
// 简易 LRU 缓存：容量超过 100 时淘汰最早访问的
LinkedHashMap<String, Object> lru = new LinkedHashMap<>(16, 0.75f, true) {
    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > 100; // 超过 100 就删最老的
    }
};
```

**TreeMap 的使用场景：**
- 需要按 key 排序输出（如排行榜、按时间排序的事件）
- 需要范围查询：`subMap(fromKey, toKey)`、`headMap(toKey)`、`tailMap(fromKey)`
- 需要取最大/最小 key：`firstKey()`、`lastKey()`

**背诵口诀：** Linked 记顺序能做 LRU，Tree 排大小能做范围查。

> 面试话术：「LinkedHashMap 在 HashMap 基础上加了双向链表维护顺序，支持插入顺序和访问顺序，访问顺序模式可以用来实现 LRU 缓存。TreeMap 底层是红黑树，按 key 排序，适合需要范围查询的场景。」

## 三、Set
### 7. HashSet 的实现原理？如何保证元素不重复？

**HashSet 底层就是一个 HashMap，元素作为 key 存，value 统一用一个固定的 `PRESENT` 对象占位。** 所以 HashSet 的去重逻辑完全复用了 HashMap 的 key 去重机制。

```java
// HashSet 源码核心（HotSpot JDK 8）
public class HashSet<E> {
    private transient HashMap<E, Object> map;
    private static final Object PRESENT = new Object(); // 所有 value 都是这个

    public boolean add(E e) {
        return map.put(e, PRESENT) == null; // 如果 key 已存在，put 返回旧 value，不为 null → 返回 false
    }
}
```

**如何保证不重复？** 和 HashMap 判断 key 是否相同的逻辑一样：

1. 先比 **`hashCode()`**：hash 值不同 → 一定不同，放进去
2. hash 值相同 → 再比 **`equals()`**：equals 为 true → 重复，不放；equals 为 false → 不重复，挂链表

所以自定义对象放 HashSet 时，**必须同时重写 `hashCode()` 和 `equals()`**，否则「内容相同」的两个对象会被当成不同元素。

**这题面试经常挖坑：只重写 equals 不重写 hashCode 会怎样？**

两个内容相同的对象可能 hashCode 不同，落到**不同的桶**，HashMap 连 equals 都不会调，直接当成不同 key 放进去了。结果 HashSet 里出现两个「相同」的元素。

**常见的 Set 实现对比：**

| Set 实现 | 底层 | 是否有序 | 线程安全 |
|----------|------|----------|----------|
| `HashSet` | HashMap | **无序** | 不安全 |
| `LinkedHashSet` | LinkedHashMap | **插入顺序** | 不安全 |
| `TreeSet` | TreeMap（红黑树） | **按元素大小排序** | 不安全 |

**背诵口诀：** HashSet 就是 HashMap 的 key，去重靠 hashCode + equals 两兄弟。

> 面试话术：「HashSet 底层是 HashMap，元素作为 key，value 用一个固定对象占位。去重靠 hashCode 定位桶 + equals 判重。自定义对象必须同时重写 hashCode 和 equals，否则去重失效。」

## 四、并发与安全
### 8. fail-fast 和 fail-safe 是什么？

**fail-fast 是一发现并发修改就立即抛异常；fail-safe 是在副本上遍历，不会抛异常。**

| 对比项 | fail-fast | fail-safe |
|--------|-----------|-----------|
| 含义 | 遍历时如果集合被修改，**立即抛 `ConcurrentModificationException`** | 遍历时在**副本**上操作，不受原集合修改影响 |
| 原理 | 通过 `modCount` 计数器检测：迭代器创建时记录 modCount，每次 `next()` 检查是否变了 | 创建迭代器时拷贝底层数组，遍历的是快照 |
| 典型集合 | `ArrayList`、`HashMap`、`HashSet` 等 `java.util` 包下的集合 | `CopyOnWriteArrayList`、`ConcurrentHashMap` 等 `java.util.concurrent` 包下的集合 |
| 优点 | 快速暴露 bug，防止不可预期的行为 | 遍历安全，不会抛异常 |
| 缺点 | 遍历时不能修改（除了 `iterator.remove()`） | 内存开销大（拷贝），遍历的不是最新数据 |

**实际场景：**

```java
// ❌ fail-fast：边遍历边删除会抛异常
List<String> list = new ArrayList<>(Arrays.asList("a", "b", "c"));
for (String s : list) {
    if ("b".equals(s)) list.remove(s); // 抛 ConcurrentModificationException
}

// ✅ 正确做法：用迭代器的 remove()
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if ("b".equals(it.next())) it.remove(); // 安全
}
```

**ConcurrentHashMap 的迭代器是弱一致性的：** 它既不拷贝也不抛异常，而是**直接遍历底层数组**，能看到遍历开始后的部分修改，但不保证看到所有修改。严格来说不算 fail-safe，而是**弱一致（weakly consistent）**。

**背诵口诀：** fast 抛异常保安全，safe 拷副本不报错。foreach 里别 remove，要删用迭代器。

> 面试话术：「fail-fast 通过 modCount 检测并发修改，一旦发现就抛 ConcurrentModificationException，ArrayList、HashMap 都是这种机制。fail-safe 在副本上遍历，CopyOnWriteArrayList 是典型代表。ConcurrentHashMap 的迭代器是弱一致性的，不抛异常但也不保证看到所有最新修改。」

### 9. 如何实现一个线程安全的 List？

**三种方式：`Vector`（不推荐）、`Collections.synchronizedList()`（简单场景）、`CopyOnWriteArrayList`（读多写少场景首选）。**

| 方案 | 锁机制 | 性能 | 适用场景 |
|------|--------|------|----------|
| `Vector` | 所有方法加 `synchronized` | **最差**，每个操作都加锁 | **已过时，不推荐** |
| `Collections.synchronizedList()` | 用一把 `mutex` 锁包装所有方法 | 一般，全局一把锁 | 临时需要线程安全的简单场景 |
| `CopyOnWriteArrayList` | **写时复制**（COW）：写的时候复制整个数组，读不加锁 | **读极快（无锁），写慢（拷贝数组）** | **读多写少**，如监听器列表、配置缓存 |

**CopyOnWriteArrayList 的底层原理：**

```java
// HotSpot JDK 8 源码核心
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock(); // 写操作加锁
    try {
        Object[] elements = getArray();
        Object[] newElements = Arrays.copyOf(elements, len + 1); // 复制一份新数组
        newElements[len] = e; // 在新数组上修改
        setArray(newElements); // 用新数组替换旧数组（volatile 写）
        return true;
    } finally {
        lock.unlock();
    }
}
// 读操作——完全无锁
public E get(int index) {
    return (E) getArray()[index]; // 直接读 volatile 数组引用
}
```

**写时复制的特点：**
- **读不加锁**：读的是当前数组引用（`volatile`），性能极高
- **写要加锁**：先复制整个数组，修改后原子替换引用
- **缺点**：每次写都要拷贝整个数组，**写密集场景不适合**
- **数据弱一致**：写操作完成前，其他线程读到的仍是旧数据

**这题面试经常挖坑：`Collections.synchronizedList()` 的迭代需要手动加锁！**

```java
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
// ❌ 遍历时不安全，可能抛 ConcurrentModificationException
for (String s : syncList) { ... }

// ✅ 必须手动加锁
synchronized (syncList) {
    for (String s : syncList) { ... }
}
```

**背诵口诀：** Vector 太老别用了，synchronized 包一层简单粗暴，COW 读快写慢读多写少用它。

> 面试话术：「线程安全的 List 有三种：Vector 已过时，synchronizedList 用一把锁包装但迭代时需要手动同步，CopyOnWriteArrayList 写时复制读不加锁，适合读多写少的场景，比如事件监听器列表。」

### 10. Collections.synchronizedMap 和 ConcurrentHashMap 的区别？

一句话：**synchronizedMap 是一把大锁锁整个 map，ConcurrentHashMap 是细粒度锁，性能差距巨大。**

| 对比项 | `Collections.synchronizedMap()` | `ConcurrentHashMap` |
|--------|--------------------------------|---------------------|
| 锁机制 | 一把 `mutex` 锁，所有操作互斥 | JDK 1.8：CAS + synchronized 锁**单个桶** |
| 并发度 | **1**（同一时刻只有一个线程能操作） | **数组长度**（不同桶可以并行） |
| 迭代安全 | 需要**手动 synchronized** | 弱一致性迭代器，不抛异常 |
| null key/value | 取决于被包装的 map（如 HashMap 允许 null） | **都不允许 null** |
| 适用场景 | 临时需要线程安全，调用少 | **高并发场景首选** |

**synchronizedMap 的原理非常简单：**

```java
// JDK 源码简化版
public V get(Object key) {
    synchronized (mutex) { return m.get(key); } // 每个方法都加同一把锁
}
public V put(K key, V value) {
    synchronized (mutex) { return m.put(key, value); }
}
```

就是把原来的 map 的每个方法都用 `synchronized(mutex)` 包一层，粗暴但有效。问题是所有操作串行化，**高并发下性能极差**。

**ConcurrentHashMap 的优势：**
- 读操作**完全不加锁**（`Node` 的 `val` 和 `next` 都是 `volatile` 的）
- 写操作只锁**一个桶的头节点**
- `size()` 用 `baseCount + CounterCell[]` 分散计数，避免竞争
- 提供原子操作：`putIfAbsent()`、`computeIfAbsent()`、`merge()` 等

**背诵口诀：** synchronized 一把大锁全串行，Concurrent 细锁高并发。90% 的场景用 ConcurrentHashMap。

> 面试话术：「synchronizedMap 用一把互斥锁包装整个 map，所有操作串行化，并发性能差。ConcurrentHashMap 用 CAS + synchronized 只锁单个桶，不同桶可以并行操作，并发度远高于 synchronizedMap。高并发场景应该用 ConcurrentHashMap。」

## 复习优先级（3~5 年）
| 优先级 | 题目 |
|--------|------|
| P0 | 2. HashMap 的底层实现原理？JDK 1.8 做了哪些优化？ |
| P0 | 3. HashMap 为什么线程不安全？会出现什么问题？ |
| P0 | 4. HashMap 的扩容机制？为什么容量是 2 的幂次？ |
| P0 | 5. ConcurrentHashMap 的实现原理？JDK 1.7 和 1.8 有什么区别？ |
| P1 | 1. ArrayList 和 LinkedList 的区别？什么场景用哪个？ |
| P1 | 6. LinkedHashMap 和 TreeMap 的区别和使用场景？ |
| P1 | 7. HashSet 的实现原理？如何保证元素不重复？ |
| P1 | 10. Collections.synchronizedMap 和 ConcurrentHashMap 的区别？ |
| P2 | 8. fail-fast 和 fail-safe 是什么？ |
| P2 | 9. 如何实现一个线程安全的 List？ |