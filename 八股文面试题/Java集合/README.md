# Java 集合

---

## 一、集合框架概述

### 1. Java 集合框架有哪些？List、Set、Map 有什么区别？

**一句话总结**：Java 集合分为两大体系 —— `Collection`（存单个元素）和 `Map`（存键值对）。`Collection` 下又分 `List`（有序可重复）、`Set`（无序不重复）、`Queue`（队列）。

```
集合框架
├── Collection（单个元素）
│   ├── List（有序、可重复）→ ArrayList、LinkedList
│   ├── Set（无序、不重复）→ HashSet、TreeSet
│   └── Queue（队列）→ LinkedList、PriorityQueue
└── Map（键值对，key 不重复）
    ├── HashMap
    ├── LinkedHashMap（保持插入顺序）
    ├── TreeMap（按 key 排序）
    └── ConcurrentHashMap（线程安全）
```

| 对比项 | List | Set | Map |
|--------|------|-----|-----|
| 元素 | 单个元素 | 单个元素 | 键值对（key-value） |
| 有序性 | ✅ 有序（按插入顺序） | ❌ 无序（TreeSet 除外） | ❌ 无序（LinkedHashMap/TreeMap 除外） |
| 重复 | ✅ 允许重复 | ❌ 不允许重复 | key 不重复，value 可重复 |
| 常用实现 | `ArrayList`、`LinkedList` | `HashSet`、`TreeSet` | `HashMap`、`ConcurrentHashMap` |

> **面试话术**：Java 集合框架分两大类：Collection 存单个元素，Map 存键值对。Collection 下面有 List、Set 和 Queue 三个子接口。List 有序可重复，最常用的是 ArrayList；Set 无序不重复，最常用的是 HashSet；Map 的 key 不能重复，最常用的是 HashMap。

---

## 二、List 相关

### 2. ArrayList 和 LinkedList 有什么区别？

**一句话总结**：`ArrayList` 底层是**动态数组**，查询快、增删慢；`LinkedList` 底层是**双向链表**，增删快、查询慢。**实际开发中 99% 的场景用 ArrayList**。

| 对比项 | ArrayList | LinkedList |
|--------|-----------|------------|
| 底层结构 | 动态数组（`Object[]`） | 双向链表 |
| 随机访问（get） | ✅ 快，直接按下标定位 | ❌ 慢，需要从头/尾一个个找 |
| 尾部添加（add） | ✅ 快（不触发扩容时） | ✅ 快 |
| 中间插入/删除 | ❌ 慢（需要移动后面的元素） | ✅ 快（只需改前后节点的指针） |
| 内存占用 | 较少（只存数据） | 较多（每个节点额外存前后两个指针） |
| 线程安全 | ❌ | ❌ |

**⚠️ 面试挖坑提醒**：「LinkedList 插入真的比 ArrayList 快吗？」—— **不一定**！LinkedList 在中间插入虽然改指针是快的，但**先要找到插入位置**，这个查找过程很慢。所以只有在**头部插入/删除**时 LinkedList 才明显快。实际开发中大部分场景 ArrayList 性能更好。

---

### 3. ArrayList 的扩容机制是怎样的？

**一句话总结**：JDK 7 中 ArrayList 初始容量为 **10**；**JDK 8+** 无参构造创建的是**空数组 `{}`**，首次 `add` 时才扩容到 10。之后每次扩容为原来的 **1.5 倍**（`oldCapacity + oldCapacity >> 1`），底层调用 `Arrays.copyOf()` 把旧数组元素复制到新数组。

**扩容过程（4 步）**：

1. 加入新元素时，先检查当前数组是否还有空位
2. 如果满了，计算新容量 = 旧容量 × 1.5（用位运算 `>> 1` 实现除以 2）
3. 创建一个新的更大的数组
4. 把旧数组的元素全部复制到新数组中（`Arrays.copyOf()`）

```java
// 源码简化版（JDK 8）
int oldCapacity = elementData.length;
int newCapacity = oldCapacity + (oldCapacity >> 1);  // 1.5 倍
elementData = Arrays.copyOf(elementData, newCapacity);
```

**⚠️ 面试挖坑提醒**：「频繁扩容有什么问题？怎么优化？」—— 每次扩容都要复制整个数组，**性能开销大**。如果提前知道大致数据量，应该在创建时**指定初始容量**：`new ArrayList<>(1000)`，避免反复扩容。

> **面试话术**：ArrayList 在 JDK 8+ 中无参构造先创建空数组，首次 add 时扩到 10，之后空间不足每次扩容为原来的 1.5 倍。扩容底层通过 Arrays.copyOf 创建新数组并复制旧数据，所以如果数据量能预估，最好创建时指定容量，避免频繁扩容带来的性能损耗。

---

## 三、HashMap 相关（重点）

### 4. HashMap 的底层原理是什么？

**一句话总结**：JDK 8 中 HashMap 底层是**数组 + 链表 + 红黑树**。用数组存储数据，哈希冲突时用链表串起来，链表太长（≥8）就转成红黑树提升查找速度。

**数据结构图示**：

```
数组（Node[]，也叫哈希桶）
 ┌────┬────┬────┬────┬────┬────┐
 │ 0  │ 1  │ 2  │ 3  │ 4  │... │
 └────┴──┬─┴────┴──┬─┴────┴────┘
         │         │
       [A→B→C]   [D→E]     ← 链表（冲突 < 8 个）
         ↓
      [红黑树]              ← 链表长度 ≥ 8 且数组长度 ≥ 64 时转换
```

**核心参数**：

| 参数 | 默认值 | 含义 |
|------|--------|------|
| 初始容量 | 16 | 数组的初始大小 |
| 负载因子 | 0.75 | 元素数量达到 `容量 × 0.75` 时触发扩容 |
| 树化阈值 | 8 | 链表长度 ≥ 8 时转红黑树 |
| 反树化阈值 | 6 | 红黑树节点 ≤ 6 时退化回链表 |
| 最小树化容量 | 64 | 数组长度 < 64 时，即使链表长度 ≥ 8 也不会树化，而是先扩容 |

> **面试话术**：HashMap 在 JDK 8 中底层是数组加链表加红黑树的结构。每个元素通过 key 的哈希值确定存在数组的哪个位置，如果多个 key 落到同一个位置就形成链表。当链表长度达到 8 且数组长度达到 64 时，链表会转换为红黑树，查找效率从逐个遍历变成了类似二分查找。负载因子默认 0.75，也就是当元素数量达到容量的 75% 时会触发扩容。

**⚠️ 面试挖坑提醒**：「为什么树化阈值是 8 而不是其他数字？」—— 根据**泊松分布**（一种统计学概率分布），在负载因子 0.75 的条件下，单个桶中元素个数达到 8 的概率极低（约千万分之六），所以选 8 是在时间和空间之间的平衡。

---

### 5. HashMap 的 put 流程是怎样的？

**一句话总结**：计算 key 的哈希值 → 定位到数组下标 → 判断该位置有没有元素 → 没有就直接放，有就处理冲突（链表或红黑树）→ 最后检查是否需要扩容。

**完整流程（6 步）**：

1. **计算哈希值**：对 key 调用 `hashCode()`，再做一次扰动处理（把哈希值的高位和低位混合，让分布更均匀，减少冲突）
2. **定位下标**：用 `(n - 1) & hash` 计算出数组下标（n 是数组长度，必须是 2 的幂，这样取模运算可以用位运算代替，更快）
3. **该位置为空**：直接放入新节点
4. **该位置有元素且 key 相同**：覆盖旧值（用 `equals()` 判断）
5. **该位置有元素且 key 不同（哈希冲突）**：
   - 如果是链表：遍历链表，相同 key 则覆盖，否则追加到尾部。如果链表长度 ≥ 8 且数组长度 ≥ 64，转为红黑树
   - 如果已经是红黑树：按红黑树规则插入
6. **插入后检查**：如果元素总数 > 容量 × 负载因子，触发扩容

**背诵口诀**：「**算哈希、找位置、空就放、同就换、冲突挂链/挂树、最后查扩容**」

---

### 6. HashMap 的扩容机制是怎样的？

**一句话总结**：当元素数量超过 `容量 × 负载因子`（默认 16 × 0.75 = 12）时触发扩容，新数组大小是原来的 **2 倍**，所有元素重新计算位置。

**JDK 7 vs JDK 8 扩容对比**：

| 对比项 | JDK 7 | JDK 8 |
|--------|-------|-------|
| 元素迁移 | 重新计算每个元素的哈希值和下标 | 通过 `hash & oldCap` 判断元素在原位置还是 `原位置 + 旧容量` |
| 链表顺序 | **头插法**（会导致链表反转） | **尾插法**（保持原有顺序） |
| 并发问题 | 头插法在多线程下可能形成**死循环链表** | 尾插法不会出现死循环，但**仍然不是线程安全的** |

**⚠️ 面试挖坑提醒**（这题面试经常挖坑）：
- 「JDK 7 的 HashMap 在多线程下会出什么问题？」—— **死循环**！因为扩容时头插法会导致链表反转，多线程同时扩容可能形成环形链表，导致 CPU 100%
- 「JDK 8 解决了这个问题吗？」—— 解决了死循环（改用尾插法），但 **HashMap 本身仍然不是线程安全的**，多线程应该用 `ConcurrentHashMap`

---

### 7. HashMap 和 Hashtable 有什么区别？

**一句话总结**：`HashMap` 线程不安全但快，`Hashtable` 线程安全但慢（整个方法加 `synchronized`）。**Hashtable 已过时**，线程安全场景应该用 `ConcurrentHashMap`。

| 对比项 | HashMap | Hashtable |
|--------|---------|-----------|
| 线程安全 | ❌ | ✅（方法加了 `synchronized`） |
| 性能 | **快** | 慢（锁整个表） |
| null 键/值 | ✅ 允许一个 null key，多个 null value | ❌ key 和 value 都不能为 null |
| 初始容量 | 16 | 11 |
| 扩容方式 | 2 倍 | 2 倍 + 1 |
| 继承关系 | 继承 `AbstractMap` | 继承 `Dictionary`（已过时） |
| 推荐使用 | ✅ 单线程场景 | ❌ **已淘汰** |

> **面试话术**：HashMap 和 Hashtable 最大的区别是线程安全性。Hashtable 是线程安全的，因为所有方法都加了 synchronized，但这种粗粒度的锁导致性能很差。HashMap 线程不安全但性能好。如果需要线程安全的 Map，应该用 ConcurrentHashMap 而不是 Hashtable，因为 ConcurrentHashMap 用了更细粒度的锁，并发性能好得多。

---

### 8. ConcurrentHashMap 怎么保证线程安全？JDK 7 和 JDK 8 有什么区别？

**一句话总结**：JDK 7 用**分段锁**（把数组分成多个段，每段一把锁），JDK 8 改为 **CAS + synchronized 锁单个桶的头节点**，锁粒度更细，并发度更高。

| 对比项 | JDK 7 | JDK 8 |
|--------|-------|-------|
| 数据结构 | Segment 数组 + 链表 | **数组 + 链表 + 红黑树** |
| 锁机制 | 分段锁（Segment 继承 ReentrantLock，一种可重入锁） | **CAS + synchronized（锁单个桶的头节点）** |
| 锁粒度 | Segment 级别（默认 16 个段） | **Node 级别**（更细） |
| 并发度 | 最多 16 个线程同时写 | **理论上等于数组长度** |
| null 键/值 | ❌ 不允许 | ❌ 不允许 |

**JDK 8 的 ConcurrentHashMap 工作方式**：
- **读操作**：不加锁，用 `volatile`（一种轻量级同步机制，详见 Java 并发第 7 题）保证其他线程能立即看到最新值
- **写操作**：用 **CAS**（比较并交换，详见 Java 并发第 5 题）尝试写入空桶；如果桶已有数据，用 `synchronized` 锁住该桶的头节点，只锁一个桶不影响其他桶的并发访问

**⚠️ 面试挖坑提醒**：「ConcurrentHashMap 的 key 和 value 能不能为 null？」—— **都不能**！因为在并发场景下，如果 `get(key)` 返回 null，无法区分是"key 不存在"还是"value 就是 null"，会造成歧义。

---

## 四、其他集合

### 9. HashSet 的底层原理是什么？

**一句话总结**：HashSet 底层就是一个 **HashMap**，元素作为 HashMap 的 **key** 存储，value 是一个固定的占位对象（`PRESENT`）。所以 HashSet 不允许重复，本质上是因为 HashMap 的 key 不允许重复。

```java
// HashSet 的 add 方法（源码简化）
private static final Object PRESENT = new Object();

public boolean add(E e) {
    return map.put(e, PRESENT) == null;  // 本质上就是往 HashMap 里放
}
```

**因此**：
- HashSet 的**去重**依赖元素的 `hashCode()` 和 `equals()` 方法（详见 Java 基础第 14 题）
- 自定义对象放入 HashSet 时，**必须重写这两个方法**，否则无法正确去重

> **面试话术**：HashSet 底层就是 HashMap，添加元素时把元素作为 HashMap 的 key，value 是一个固定的常量对象。因为 HashMap 的 key 不能重复，所以 HashSet 也就保证了元素不重复。去重的过程依赖 hashCode 和 equals 方法。

---

### 10. LinkedHashMap 和 TreeMap 有什么特点？

**一句话总结**：`LinkedHashMap` 在 HashMap 基础上用**双向链表**维护插入顺序或访问顺序；`TreeMap` 底层用**红黑树**实现，按 key 自然排序或自定义排序。

| 对比项 | HashMap | LinkedHashMap | TreeMap |
|--------|---------|---------------|---------|
| 底层结构 | 数组+链表+红黑树 | HashMap + **双向链表** | **红黑树** |
| 有序性 | ❌ 无序 | ✅ **按插入顺序**（默认） | ✅ **按 key 排序** |
| 性能 | 最快 | 略慢于 HashMap | 较慢（需要维护排序） |
| null key | ✅ | ✅ | ❌（要比较大小，null 没法比） |
| 适用场景 | 一般用途 | 需要保持顺序，比如实现 **LRU 缓存** | 需要按 key 排序 |

**关联知识点**：LinkedHashMap 可以通过设置 `accessOrder = true` 实现**按访问顺序排列**（最近访问的放最后），这是实现 **LRU 缓存**（最近最少使用淘汰策略）的经典做法。

---

### 11. 什么是 Iterator？什么是 fail-fast 机制？

**一句话总结**：`Iterator`（迭代器）是遍历集合的统一方式。**fail-fast（快速失败）** 是指在遍历集合时，如果集合被其他方式修改了，就立刻抛出 `ConcurrentModificationException` 异常，防止出错。

**fail-fast 原理**：

1. 集合内部维护一个 `modCount`（修改计数器），每次 add/remove 操作都会 +1
2. 创建迭代器时，记录当前的 `expectedModCount = modCount`
3. 每次调用 `next()` 时，检查 `modCount` 是否等于 `expectedModCount`
4. 如果不相等（说明集合在迭代期间被修改了），立刻抛异常

```java
// ❌ 错误写法：遍历时用集合的 remove 会触发 fail-fast
for (String s : list) {
    if (s.equals("hello")) list.remove(s);  // 抛 ConcurrentModificationException！
}

// ✅ 正确写法：用迭代器的 remove
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("hello")) it.remove();  // 安全
}
```

**⚠️ 面试挖坑提醒**：「怎么避免 fail-fast？」—— 三种方式：① 用迭代器自带的 `remove()` 方法；② 用 `CopyOnWriteArrayList`（写时复制，详见 Java 并发）；③ 用 JDK 8 的 `removeIf()` 方法。

---

### 12. Comparable 和 Comparator 有什么区别？

**一句话总结**：`Comparable` 是"我跟别人比"（类自己实现），`Comparator` 是"让别人帮我比"（外部定义比较规则）。

| 对比项 | Comparable | Comparator |
|--------|-----------|------------|
| 所在包 | `java.lang` | `java.util` |
| 核心方法 | `compareTo(T o)` | `compare(T o1, T o2)` |
| 实现方式 | 类**自身**实现接口 | **外部**单独写一个比较器 |
| 排序规则 | 只能有**一种**默认排序 | 可以定义**多种**排序规则 |
| 典型使用 | `Collections.sort(list)` | `Collections.sort(list, comparator)` |

```java
// Comparable：类自己定义排序规则
class Student implements Comparable<Student> {
    public int compareTo(Student o) {
        return this.age - o.age;  // 按年龄升序
    }
}

// Comparator：外部定义排序规则，更灵活
list.sort((s1, s2) -> s1.getName().compareTo(s2.getName()));  // 按名字排序
```

**背诵口诀**：「**Comparable 自比较、Comparator 外比较；一个 compareTo、一个 compare**」

---

## 复习优先级（3~5 年）

| 优先级 | 题目 |
|--------|------|
| **P0 高频必背** | 1. 集合框架概述、2. ArrayList vs LinkedList、4. HashMap 底层原理、5. HashMap put 流程、6. HashMap 扩容、8. ConcurrentHashMap |
| **P1 中频建议掌握** | 3. ArrayList 扩容、7. HashMap vs Hashtable、9. HashSet 底层、11. Iterator 和 fail-fast |
| **P2 低频了解即可** | 10. LinkedHashMap/TreeMap、12. Comparable vs Comparator |


