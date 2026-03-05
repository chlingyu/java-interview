# 05-Java基础

---

## 一、基础类型与关键字
### 1. Java 有哪些基本数据类型？各占多少字节？

Java 有 **8 种**基本数据类型，分 4 大类，一句话记住：**1 个布尔、1 个字符、4 个整数、2 个小数**。

| 分类 | 类型 | 字节 | 位数 | 范围 / 说明 |
|------|------|------|------|-------------|
| 布尔 | `boolean` | **不确定**（见下方说明） | — | `true` / `false` |
| 字符 | `char` | 2 | 16 | 一个 Unicode 字符，范围 0~65535 |
| 整数 | `byte` | 1 | 8 | -128 ~ 127 |
| 整数 | `short` | 2 | 16 | -32768 ~ 32767 |
| 整数 | `int` | 4 | 32 | 约 ±21 亿 |
| 整数 | `long` | 8 | 64 | 非常大，赋值时加 `L` 后缀 |
| 小数 | `float` | 4 | 32 | 单精度，赋值时加 `F` 后缀 |
| 小数 | `double` | 8 | 64 | 双精度，小数默认就是 double |

**背诵口诀：** 整数从小到大 1、2、4、8 字节（byte → short → int → long），小数 4、8 字节（float → double），char 占 2 字节。

**boolean 占多少字节？**

这题面试经常挖坑。答案是：**Java 规范里没有明确规定**。

但是在 HotSpot（最常用的 JVM）里：
- 单个 `boolean` 变量 → 底层用 `int` 存，`true` 存为 `1`，`false` 存为 `0`，占 **4 字节**
- `boolean[]` 数组 → 底层用 `byte[]` 存，同样 `1` 表示 `true`、`0` 表示 `false`，每个元素占 **1 字节**

> 面试建议直接说：「规范没有定义 boolean 的大小，HotSpot 中单个 boolean 占 4 字节，boolean 数组中每个元素占 1 字节。」

**基本类型 vs 包装类型：**

每种基本类型都有对应的包装类（`int` → `Integer`，`char` → `Character`，其余首字母大写）。区别：
- 基本类型存在**栈**上，效率高，不能为 `null`
- 包装类型存在**堆**上，是对象，可以为 `null`，能放进集合（如 `List<Integer>`）
- 两者之间通过**自动装箱/拆箱**转换

```java
int a = 10;            // 基本类型
Integer b = a;         // 自动装箱：int → Integer
int c = b;             // 自动拆箱：Integer → int
```

### 2. final、finally、finalize 的区别？

三个长得像但**完全不同**的东西，一句话：**final 修饰不可变，finally 保证必执行，finalize 是垃圾回收钩子（已废弃）**。

| 关键字 | 作用 | 用在哪 |
|--------|------|--------|
| `final` | 修饰**不可变** | 类（不能被继承）、方法（不能被重写）、变量（不能被重新赋值） |
| `finally` | **必定执行**的代码块 | `try-catch-finally` 中，无论是否异常都会执行 |
| `finalize` | 对象被 GC 回收前调用的方法 | `Object` 类的方法，JDK 9 已标记 `@Deprecated`，**不要用** |

**final 修饰变量的细节：**
- 修饰**基本类型**：值不能变
- 修饰**引用类型**：引用（地址）不能变，但对象内部的属性**可以变**

```java
final List<String> list = new ArrayList<>();
list.add("hello");  // ✅ 可以修改内容
list = new ArrayList<>();  // ❌ 编译报错，不能重新赋值
```

**finally 一定会执行吗？**

几乎一定，但有例外：`System.exit()` 终止 JVM、线程被 `kill`、死循环卡在 try 里。面试答「除非 JVM 退出，否则一定执行」就行。

**背诵口诀：** final 不变 finally 必走 finalize 别用。

> 面试话术：「final 是修饰符，表示不可变；finally 是异常处理的一部分，保证代码必定执行；finalize 是 GC 回收前的钩子方法，JDK 9 已废弃，不推荐使用。」

## 二、对象与面向对象
### 3. `==` 和 `equals()` 有什么区别？为什么重写 equals 必须重写 hashCode？

**`==` 比的是地址，`equals()` 比的是内容**（前提是类重写了 `equals()`）。

| 比较方式 | 基本类型 | 引用类型 |
|----------|----------|----------|
| `==` | 比**值** | 比**内存地址**（是不是同一个对象） |
| `equals()` | 不适用 | 默认和 `==` 一样比地址；`String`、`Integer` 等重写后比**内容** |

```java
String a = new String("hello");
String b = new String("hello");
System.out.println(a == b);      // false — 两个不同的对象
System.out.println(a.equals(b)); // true  — 内容相同
```

**这题面试经常挖坑：字符串常量池的 `==`**

```java
String x = "hello";           // 字面量，放入常量池
String y = "hello";           // 常量池已有，直接复用同一个对象
System.out.println(x == y);   // true — 同一个对象！

String z = new String("hello"); // new 强制在堆上创建新对象
System.out.println(x == z);   // false — 不同对象
System.out.println(x == z.intern()); // true — intern() 返回常量池中的对象
```

面试常考这三种写法的区别，关键就是：**字面量走常量池（复用），`new` 走堆（新建），`intern()` 把堆上的字符串放回常量池**。

**为什么重写 equals 必须重写 hashCode？**

因为 Java 有一条**约定**（不是语法强制，但必须遵守）：
- **两个对象 `equals()` 为 true → `hashCode()` 必须相同**
- `hashCode()` 相同 → `equals()` 不一定为 true（哈希冲突）

如果只重写 `equals()` 不重写 `hashCode()`，会导致：
1. 两个「内容相同」的对象，`hashCode` 不同
2. 放进 `HashMap` / `HashSet` 时，会被当成**不同的 key**，出现重复数据

```java
// 只重写了 equals，没重写 hashCode
User u1 = new User("张三");
User u2 = new User("张三");
Set<User> set = new HashSet<>();
set.add(u1);
set.add(u2);
System.out.println(set.size()); // 2！预期应该是 1
```

**背诵口诀：** == 看地址，equals 看内容；重写 equals 必重写 hashCode，否则哈希集合会翻车。

> 面试话术：「== 比地址，equals 比内容。重写 equals 必须重写 hashCode，否则在 HashMap、HashSet 等哈希结构中会出现逻辑错误——两个相等的对象会被当成不同的 key。」

### 4. Java 是值传递还是引用传递？

**Java 只有值传递，没有引用传递。** 这是面试高频坑题。

关键在于理解「传的是什么值」：
- **基本类型**：传的是**值的副本**，方法内修改不影响原变量
- **引用类型**：传的是**引用（地址）的副本**，方法内可以通过这个地址修改对象属性，但不能让原引用指向新对象

```java
void change(int[] arr) {
    arr[0] = 99;       // ✅ 能改——通过地址副本修改了堆上的数组
    arr = new int[]{1}; // ❌ 不影响外面——只是让副本指向了新地址
}
```

**怎么理解「值传递」？** 想象你把家门钥匙**复印了一把**给别人：
- 他拿复印钥匙能进你家、搬你家具（修改对象属性）
- 但他把复印钥匙扔了换一把新钥匙，你手里的原钥匙**不受影响**（不能改变原引用指向）

> 面试话术：「Java 只有值传递。基本类型传值的副本，引用类型传引用地址的副本。所以方法内能通过引用修改对象属性，但不能让外部引用指向新对象。」

### 5. 接口和抽象类的区别？什么时候用接口，什么时候用抽象类？

一句话：**接口定义「能做什么」（能力），抽象类定义「是什么」（本质）**。

| 对比项 | 接口（interface） | 抽象类（abstract class） |
|--------|-------------------|--------------------------|
| 实例化 | 不能 | 不能 |
| 多继承 | 一个类可以实现**多个**接口 | 一个类只能继承**一个**抽象类 |
| 构造方法 | 没有 | 有 |
| 成员变量 | 只能是 `public static final` 常量 | 可以有普通成员变量 |
| 方法实现 | JDK 8 前只能有抽象方法；JDK 8+ 可以有 `default`、`static` 方法 | 可以有抽象方法，也可以有普通方法 |
| 设计理念 | **has-a / can-do**（像一种能力） | **is-a**（像一种分类） |

**JDK 8 的 default 方法具体怎么用？**

```java
public interface Loggable {
    // 抽象方法：实现类必须重写
    void doWork();

    // default 方法：提供默认实现，实现类可以不重写
    default void log(String msg) {
        System.out.println("[LOG] " + msg);
    }
}
// 实现类自动拥有 log() 方法，不需要自己写
```

这让接口也能有「部分实现」，但和抽象类的区别是：接口的 default 方法**不能访问实例状态**（没有成员变量），抽象类可以。

**什么时候用哪个？**
- **接口**：定义一组能力 / 规范，如 `Serializable`（可序列化）、`Comparable`（可比较）
- **抽象类**：多个子类有**公共代码**需要复用，且需要共享**状态**（成员变量），如模板方法模式中的基类

**背诵口诀：** 接口管「能不能」，抽象类管「是不是」。多能力用接口，共状态用抽象类。

> 面试话术：「接口侧重定义能力规范，支持多实现；抽象类侧重代码复用，提供公共实现。JDK 8 之后接口也能有默认方法，但抽象类仍然可以有状态（成员变量）和构造方法，这是接口做不到的。」

### 6. 重载（overload）和重写（override）的区别？

一句话：**重载是同一个类里方法名相同、参数不同；重写是子类覆盖父类的同名同参方法**。

| 对比项 | 重载（Overload） | 重写（Override） |
|--------|------------------|------------------|
| 发生位置 | **同一个类**中 | **子类与父类**之间 |
| 方法名 | 必须相同 | 必须相同 |
| 参数列表 | **必须不同**（类型、个数、顺序） | **必须相同** |
| 返回值 | 无要求 | 必须相同或是父类返回值的**子类型** |
| 访问权限 | 无要求 | 不能比父类**更严格**（如父类 `protected`，子类不能 `private`） |
| 异常 | 无要求 | 不能抛比父类**更宽**的 checked 异常 |
| 绑定时机 | **编译期**（静态分派） | **运行期**（动态分派，多态的基础） |

```java
// 重载：同一个类，参数不同
void print(String s) { }
void print(int n) { }

// 重写：子类覆盖父类方法
class Animal { void speak() { System.out.println("..."); } }
class Dog extends Animal {
    @Override void speak() { System.out.println("汪！"); }
}
```

**背诵口诀：** 重载看参数，重写看父子。重载编译期决定，重写运行期决定。

> 面试话术：「重载是同类中方法名相同参数不同，编译期决定调用哪个；重写是子类覆盖父类方法，运行期通过动态分派实现多态。重写时访问权限不能更严格，异常不能更宽泛。」

## 三、String 与包装类
### 7. String 为什么不可变？有什么好处？

**String 不可变是因为它的底层 `char[]`（JDK 9+ 改为 `byte[]`）被 `final` 修饰且是 `private` 的，并且 String 类本身也是 `final` 的，不能被继承。**

具体来说，在 HotSpot JDK 8 中 `String` 的核心字段：
```java
public final class String {       // final 类，不能被继承
    private final char[] value;   // final 数组引用不可变 + private 外部不可访问
    // 没有任何方法会修改 value 数组的内容
}
```

**不可变有什么好处？**

| 好处 | 说明 |
|------|------|
| **字符串常量池** | 因为不可变，多个变量可以安全地指向池中同一个对象，节省内存 |
| **线程安全** | 不可变对象天生线程安全，多线程共享无需加锁 |
| **hashCode 缓存** | `String` 内部有个 `private int hash` 字段，第一次调用 `hashCode()` 计算后缓存起来，后续直接返回，作为 `HashMap` 的 key 时性能更高 |
| **安全性** | 网络连接的 URL、文件路径、数据库连接等用 String 传递，不可变可以防止被恶意篡改 |

**字符串常量池在哪里？（版本差异）**

| JDK 版本 | 常量池位置 | 说明 |
|----------|-----------|------|
| JDK 6 及之前 | **方法区（永久代 PermGen）** | 永久代空间有限，放多了容易 `OutOfMemoryError: PermGen space` |
| JDK 7 | **堆（Heap）** | 移到堆中，可以被 GC 回收，减少 OOM 风险 |
| JDK 8+ | **堆（Heap）** | 永久代被**元空间（Metaspace）**替代，常量池仍在堆中 |

**JDK 9 为什么把 `char[]` 改成 `byte[]`？**

因为大部分字符串只包含 Latin-1 字符（英文、数字），用 `char`（2 字节/字符）浪费空间。JDK 9 引入**紧凑字符串（Compact Strings）**：Latin-1 字符用 `byte[]`（1 字节/字符），非 Latin-1 才用 2 字节编码，**内存占用几乎减半**。

**常见追问：String 真的完全不可变吗？**

通过**反射**可以修改 `value` 数组的内容，但这属于破坏设计契约的行为，正常开发不应该这么做。

**背诵口诀：** String 不可变靠三招：private 藏数据、final 锁引用、类也 final 不让继承。好处四个字：池、线、哈、安（常量池、线程安全、哈希缓存、安全性）。

> 面试话术：「String 不可变是因为底层 char 数组是 private final 的，类也是 final 的。好处有四个：支持字符串常量池节省内存、天生线程安全、hashCode 可以缓存、保证安全性。常量池 JDK 7 之前在永久代，JDK 7+ 移到了堆中。」

### 8. String、StringBuilder、StringBuffer 的区别和使用场景？

一句话：**String 不可变，StringBuilder 可变且快但线程不安全，StringBuffer 可变且线程安全但慢。**

| 对比项 | String | StringBuilder | StringBuffer |
|--------|--------|---------------|--------------|
| 可变性 | **不可变** | 可变 | 可变 |
| 线程安全 | 安全（不可变） | **不安全** | 安全（方法加了 `synchronized`） |
| 性能 | 拼接时最慢（每次创建新对象） | **最快** | 比 StringBuilder 慢（有锁开销） |
| 使用场景 | 字符串不常变化 | **单线程**下大量拼接 | **多线程**下大量拼接 |

**为什么 String 拼接慢？**

每次用 `+` 拼接，实际上是创建新的 String 对象。循环拼接 1 万次就创建 1 万个对象，GC 压力大。而 `StringBuilder` 在同一个 `char[]` 上追加，不创建新对象。

```java
// ❌ 慢：循环中用 String 拼接
String s = "";
for (int i = 0; i < 10000; i++) s += i;

// ✅ 快：用 StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) sb.append(i);
```

**这题面试经常挖坑：编译器不是会自动优化 `+` 吗？**

JDK 5+ 编译器确实会把 `String a = b + c` 优化成 `new StringBuilder(b).append(c).toString()`。但**在循环体内**，每次迭代都会 `new` 一个新的 `StringBuilder`，优化无效。所以循环拼接必须手动用 `StringBuilder`。

**StringBuilder 的底层扩容机制：**
- 初始容量：**16**（默认无参构造）或 `初始字符串长度 + 16`
- 扩容规则：**旧容量 × 2 + 2**，然后用 `Arrays.copyOf()` 把旧数组复制到新数组
- 优化建议：如果预估长度，用 `new StringBuilder(256)` 指定初始容量，避免多次扩容拷贝

**背诵口诀：** String 冻住不能动，Builder 快但不安全，Buffer 加锁保安全。循环拼接用 Builder，单句拼接随便写。

> 面试话术：「三者的核心区别是可变性和线程安全。String 不可变，每次拼接都创建新对象；StringBuilder 可变，单线程下用它最快；StringBuffer 方法加了 synchronized，多线程安全但有锁的性能开销。编译器虽然会把 + 优化为 StringBuilder，但循环内每次迭代都会 new 新的，所以循环拼接必须手动用 StringBuilder。」

### 9. Integer 装箱缓存范围是多少？为什么 `Integer a = 128; a == b` 可能是 false？

**Integer 缓存了 -128 ~ 127 的对象。这个范围内 `==` 比较的是同一个缓存对象，所以为 true；超出这个范围则是不同对象，`==` 为 false。**

底层原理：`Integer.valueOf(int)` 方法内部有一个 `IntegerCache`，在 HotSpot JVM 中启动时预先创建了 -128 ~ 127 共 256 个 Integer 对象放在缓存数组里。自动装箱 `Integer a = 127` 实际调用的就是 `Integer.valueOf(127)`。

```java
Integer a = 127, b = 127;
System.out.println(a == b);  // true  — 命中缓存，同一个对象

Integer c = 128, d = 128;
System.out.println(c == d);  // false — 超出缓存，两个不同的 new Integer(128)

// 正确做法：比较值一律用 equals()
System.out.println(c.equals(d)); // true
```

**其他包装类的缓存情况：**

| 包装类 | 缓存范围 |
|--------|----------|
| `Byte` | -128 ~ 127（全部） |
| `Short` | -128 ~ 127 |
| `Integer` | -128 ~ 127（上限可通过 JVM 参数 `-XX:AutoBoxCacheMax` 调） |
| `Long` | -128 ~ 127 |
| `Character` | 0 ~ 127 |
| `Boolean` | `TRUE` / `FALSE` 两个实例 |
| `Float` / `Double` | **不缓存** |

> 面试话术：「Integer 缓存了 -128 到 127 的对象，自动装箱时走 valueOf() 方法，命中缓存就返回同一个对象，所以 == 是 true；超出范围就 new 新对象，== 是 false。比较包装类型的值一定要用 equals()。」

## 四、异常与泛型
### 10. Java 异常体系？checked 和 unchecked 异常的区别？

Java 异常体系的根是 `Throwable`，往下分两条线：**`Error`（JVM 级别，管不了）和 `Exception`（程序级别，要处理）**。

```
Throwable
├── Error（不可恢复，如 OutOfMemoryError、StackOverflowError）
└── Exception
    ├── RuntimeException（unchecked，运行时异常）
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   ├── ClassCastException
    │   └── IllegalArgumentException ...
    └── 其他 Exception（checked，编译时异常）
        ├── IOException
        ├── SQLException
        └── ClassNotFoundException ...
```

| 对比项 | checked 异常 | unchecked 异常 |
|--------|-------------|----------------|
| 继承关系 | `Exception` 的子类（不含 `RuntimeException`） | `RuntimeException` 及其子类 |
| 编译器检查 | **强制处理**（try-catch 或 throws 声明） | 不强制处理 |
| 典型场景 | IO 操作、数据库操作、反射 | 编程错误（空指针、越界、类型转换） |
| 设计意图 | 提醒调用者「这里可能出问题，你必须考虑」 | 属于 bug，应该修代码而不是 catch |

**常见追问：Error 和 Exception 的区别？**
- `Error`：JVM 层面的严重问题，程序**无法恢复**，如内存溢出（`OutOfMemoryError`）、栈溢出（`StackOverflowError`）。不应该 catch。
- `Exception`：程序逻辑层面的问题，**可以恢复**，应该处理。

**try-with-resources（JDK 7+）：**

实现了 `AutoCloseable` 接口的资源（如流、连接），可以在 `try()` 中声明，**代码块结束后自动关闭**，不需要手动写 `finally { xxx.close() }`。

```java
// ❌ JDK 7 之前：手动关闭，代码冗长
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("file.txt"));
    String line = br.readLine();
} finally {
    if (br != null) br.close(); // 还可能抛异常
}

// ✅ JDK 7+：try-with-resources，自动关闭
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line = br.readLine();
} // 出了 try 块自动调用 br.close()
```

底层原理：编译器会自动在 `finally` 中插入 `close()` 调用，如果 `close()` 本身也抛异常，会作为 **suppressed exception** 附加到主异常上（通过 `Throwable.getSuppressed()` 获取）。

**背诵口诀：** Error 管不了，Exception 要处理；checked 编译器逼你 catch，unchecked 是你代码的 bug。

> 面试话术：「Throwable 分 Error 和 Exception。Error 是 JVM 级错误不可恢复；Exception 分 checked 和 unchecked，checked 异常编译器强制处理，unchecked 是 RuntimeException 及其子类，通常代表编程 bug。JDK 7 引入了 try-with-resources，实现 AutoCloseable 的资源会自动关闭。」

### 11. 泛型是什么？为什么会有类型擦除？

**泛型是「参数化类型」，让类、接口、方法在定义时不指定具体类型，使用时再确定。** 好处是**编译期类型检查 + 消除强制转换**。

```java
// 没有泛型：需要强转，运行时可能 ClassCastException
List list = new ArrayList();
list.add("hello");
String s = (String) list.get(0);  // 手动强转

// 有泛型：编译期检查，不需要强转
List<String> list = new ArrayList<>();
list.add("hello");
String s = list.get(0);  // 自动推断类型
list.add(123);  // ❌ 编译报错，类型不匹配
```

**什么是类型擦除？**

Java 的泛型是**编译期特性**，编译后字节码中不保留泛型信息，全部替换为原始类型（通常是 `Object`），这就是**类型擦除**。

比如 `List<String>` 和 `List<Integer>` 编译后都变成 `List`，字节码里全是 `Object`，取出来时编译器**自动插入强转指令**。

**为什么要类型擦除？**

历史兼容。泛型是 JDK 5 才引入的，为了让**新代码和旧代码（无泛型的 List）能在同一个 JVM 上运行**，Java 选择了擦除方案，保证了字节码层面的向后兼容。

**类型擦除带来的限制：**
- 不能 `new T()`、`new T[]`（运行时不知道 T 是什么）
- 不能用基本类型做泛型参数（`List<int>` ❌，只能 `List<Integer>`）
- 不能 `instanceof List<String>`（运行时只有 `List`）

**通配符 `? extends T` 和 `? super T`（高频追问）：**

| 通配符 | 含义 | 能读 | 能写 | 典型场景 |
|--------|------|------|------|----------|
| `? extends T` | **上界**：T 或 T 的子类 | ✅ 取出来当 T 用 | ❌ 不能写入（不知道具体是哪个子类） | 只读场景，如遍历集合 |
| `? super T` | **下界**：T 或 T 的父类 | ❌ 只能取出为 Object | ✅ 可以写入 T 及其子类 | 只写场景，如往集合里添加元素 |

**PECS 原则（Producer Extends, Consumer Super）：**
- 如果集合是**生产者**（你从中取数据）→ 用 `extends`
- 如果集合是**消费者**（你往里放数据）→ 用 `super`

```java
// extends：只读，从集合中取 Number
void sum(List<? extends Number> list) {
    for (Number n : list) { /* 读取 OK */ }
    // list.add(1);  // ❌ 编译报错，不能写入
}

// super：只写，往集合中放 Integer
void fill(List<? super Integer> list) {
    list.add(1);     // ✅ 写入 OK
    // Integer n = list.get(0);  // ❌ 只能取出为 Object
}
```

**背诵口诀：** 读用 extends，写用 super，记住 PECS 四个字母就行。

> 面试话术：「泛型是参数化类型，编译期做类型检查。Java 用类型擦除实现，编译后擦除为 Object，为了兼容旧代码。通配符方面，extends 用于只读（上界），super 用于只写（下界），遵循 PECS 原则。」

### 12. 反射是什么？有什么应用场景？有什么性能问题？

**反射是指在运行时动态获取类的信息（属性、方法、构造器）并调用它们的能力。** 正常写代码是编译期就确定调用什么方法，反射是运行时才决定。

**怎么用？** 核心入口是 `Class` 对象，获取方式有三种：
```java
Class<?> clazz = Class.forName("com.example.User"); // 1. 全类名
Class<?> clazz = User.class;                         // 2. 类字面量
Class<?> clazz = user.getClass();                     // 3. 对象实例
```

拿到 `Class` 对象后可以：
- `clazz.getDeclaredFields()` — 获取所有字段
- `clazz.getDeclaredMethods()` — 获取所有方法
- `clazz.getDeclaredConstructor().newInstance()` — 创建实例
- `method.invoke(obj, args)` — 调用方法

**应用场景：**
- **Spring IOC**：根据配置文件 / 注解中的类名，用反射创建 Bean 实例并注入依赖
- **MyBatis**：把数据库查询结果通过反射映射到 Java 对象的属性上
- **JDK 动态代理**：`Proxy.newProxyInstance()` 底层通过反射调用目标方法
- **Jackson / Gson**：JSON 序列化 / 反序列化时通过反射读写对象字段

**性能问题：**

| 问题 | 原因 |
|------|------|
| 比直接调用**慢几倍到几十倍** | 反射调用需要进行方法查找、安全检查、参数装箱等额外操作 |
| 无法被 JIT 内联优化 | JIT 编译器难以对反射调用做内联、逃逸分析等优化 |
| 破坏封装 | `setAccessible(true)` 可以绕过 `private` 访问控制 |

**优化手段：**
- 缓存 `Method`、`Field` 对象，避免重复查找
- `setAccessible(true)` 跳过安全检查可以提速
- 高频场景可以用 `MethodHandle`（JDK 7+）替代，性能更接近直接调用

**背诵口诀：** 反射三步走：拿 Class → 拿 Method/Field → invoke/set。慢在查找和检查，缓存加 setAccessible 提速。

> 面试话术：「反射是运行时动态获取类信息并调用的能力，Spring、MyBatis、动态代理底层都依赖反射。性能问题主要是方法查找和安全检查的开销，比直接调用慢几倍到几十倍，可以通过缓存 Method 对象和 setAccessible(true) 优化。」

## 复习优先级（3~5 年）
| 优先级 | 题目 |
|--------|------|
| P0 | 3. `==` 和 `equals()` 有什么区别？为什么重写 equals 必须重写 hashCode？ |
| P0 | 4. Java 是值传递还是引用传递？ |
| P0 | 7. String 为什么不可变？有什么好处？ |
| P0 | 9. Integer 装箱缓存范围是多少？为什么 `Integer a = 128; a == b` 可能是 false？ |
| P0 | 10. Java 异常体系？checked 和 unchecked 异常的区别？ |
| P1 | 5. 接口和抽象类的区别？什么时候用接口，什么时候用抽象类？ |
| P1 | 6. 重载（overload）和重写（override）的区别？ |
| P1 | 8. String、StringBuilder、StringBuffer 的区别和使用场景？ |
| P1 | 11. 泛型是什么？为什么会有类型擦除？ |
| P1 | 12. 反射是什么？有什么应用场景？有什么性能问题？ |
| P2 | 1. Java 有哪些基本数据类型？各占多少字节？ |
| P2 | 2. final、finally、finalize 的区别？ |