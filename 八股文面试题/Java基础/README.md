# Java 基础

---

## 一、语言概述

### 1. JDK、JRE、JVM 分别是什么？有什么关系？

**一句话总结**：JDK 是开发工具包（能编译+运行），JRE 是运行环境（只能运行），JVM 是虚拟机（执行字节码的核心引擎）。三者是**套娃关系：JDK 包含 JRE，JRE 包含 JVM**。

| 名称 | 全称 | 作用 | 包含内容 |
|------|------|------|---------|
| **JDK** | Java Development Kit | 开发 + 编译 + 运行 | JRE + `javac`编译器 + `jar`打包工具 + `javadoc`文档工具 + 调试器 |
| **JRE** | Java Runtime Environment | 运行 Java 程序 | JVM + 核心类库（JDK 8 及之前为 `rt.jar` 等） |
| **JVM** | Java Virtual Machine | 加载并执行字节码 | 类加载器 + 执行引擎 + 内存管理 |

**背诵口诀**：「**JDK 管开发，JRE 管运行，JVM 管执行**」

> **面试话术**：JDK、JRE、JVM 是包含关系。JDK 是最大的，包含 JRE 和开发工具如编译器 javac；JRE 包含 JVM 和运行所需的核心类库；JVM 是最底层的，负责把字节码翻译成机器码执行。如果只是运行 Java 程序装 JRE 就够了，如果要开发则必须装 JDK。

**⚠️ 面试挖坑提醒**：面试官可能追问「JDK 9 之后有什么变化？」—— JDK 9 引入了模块化系统（Jigsaw），JRE 不再作为独立目录分发，而是通过 `jlink` 按需定制运行时。

---

### 2. Java 为什么能跨平台？

**一句话总结**：Java 源码先编译成**字节码**（`.class`），再由各平台的 JVM 翻译成对应的机器码执行，所以**一次编译，到处运行**。

流程：`.java` → `javac`编译 → `.class`字节码 → JVM 解释/JIT编译 → 机器码

> **面试话术**：Java 的跨平台靠的是 JVM。Java 源代码编译后生成的不是机器码而是字节码，字节码由 JVM 来执行。不同操作系统都有对应版本的 JVM，所以同一份字节码可以在不同平台上运行，这就是"Write Once, Run Anywhere"。

**⚠️ 常见追问**：「Java 是编译型还是解释型语言？」—— 答：**都有**。先通过 `javac` 编译成字节码（编译阶段），再由 JVM 解释执行字节码，热点代码还会被 JIT（Just-In-Time，即时编译）编译器编译成本地机器码以提升性能。所以说 Java 是**编译与解释并存**的语言。

---

### 3. Java 有哪些基本数据类型？

**一句话总结**：Java 有 **8 种**基本数据类型，分为 4 类：整数、浮点、字符、布尔。

| 类型 | 关键字 | 字节数 | 取值范围 | 默认值 | 对应包装类 |
|------|--------|--------|---------|--------|-----------|
| 整数 | `byte` | 1 | -128 ~ 127 | 0 | `Byte` |
| 整数 | `short` | 2 | -3万 ~ 3万 | 0 | `Short` |
| 整数 | `int` | 4 | 约 -21亿 ~ 21亿（最常用） | 0 | `Integer` |
| 整数 | `long` | 8 | 特别大（比 int 大得多，一般不用记） | 0L | `Long` |
| 浮点 | `float` | 4 | 小数，精度约 6~7 位有效数字 | 0.0f | `Float` |
| 浮点 | `double` | 8 | 小数，精度约 15~16 位有效数字（最常用） | 0.0d | `Double` |
| 字符 | `char` | 2 | 一个字符（如 'A'、'中'） | 空字符 | `Character` |
| 布尔 | `boolean` | - | true / false | false | `Boolean` |

**背诵口诀**：「**1248-48-2-?**」→ byte/short/int/long 分别占 1/2/4/8 字节，float/double 占 4/8 字节，char 占 2 字节。

**⚠️ 面试挖坑提醒**：boolean 的大小问题！

- **JVM 规范没有规定 boolean 的确切大小**。在 HotSpot JVM 中，单个 boolean **局部变量**在栈上按 int 槽位存储（true 存为 1，false 存为 0，占 4 字节的栈槽位）；但作为**对象字段**时，HotSpot 通常只分配 **1 字节**
- `boolean` 数组中每个元素用 `byte` 存储，占 **1 字节**
- JVM 规范没有明确规定 boolean 的大小，以上是 HotSpot 的实现

> **面试话术**：Java 有 8 种基本数据类型。整数类型 4 种：byte、short、int、long，分别占 1、2、4、8 字节；浮点类型 2 种：float 和 double，分别占 4 和 8 字节；字符类型 char 占 2 字节，能存一个中文或一个英文字母；布尔类型 boolean，JVM 规范没定死大小；HotSpot 中局部变量按 int 槽位占 4 字节，对象字段通常只占 1 字节。

---

### 4. 自动装箱和拆箱是什么？

**一句话总结**：**装箱**是基本类型自动转为包装类（如 `int` → `Integer`），**拆箱**是包装类自动转为基本类型（如 `Integer` → `int`），底层分别调用 `valueOf()` 和 `xxxValue()`。

```java
// 装箱：编译器自动调用 Integer.valueOf(10)
Integer a = 10;

// 拆箱：编译器自动调用 a.intValue()
int b = a;
```

**⚠️ 面试挖坑提醒**：`Integer` 缓存池问题（这题面试经常挖坑）

```java
Integer a = 127, b = 127;
System.out.println(a == b);  // true（命中缓存）

Integer c = 128, d = 128;
System.out.println(c == d);  // false（超出缓存范围，new 了两个不同对象）
```

- `Integer.valueOf()` 对 **-128 ~ 127** 范围内的值做了缓存（IntegerCache），直接返回缓存对象
- 超出这个范围会 `new Integer()`，所以 `==` 比较的是地址会返回 `false`
- `Byte`、`Short`、`Long` 也有 -128~127 的缓存；`Character` 缓存 0~127；`Float` 和 `Double` **没有缓存**

**常见追问**：「拆箱可能出什么问题？」—— 如果包装类对象为 `null`，拆箱时会抛 `NullPointerException`。

---

## 二、面向对象与核心概念

### 5. `==` 和 `equals()` 有什么区别？

**一句话总结**：`==` 比较的是**值**（基本类型）或**地址**（引用类型），`equals()` 默认也比较地址，但很多类（如 `String`、`Integer`）**重写了它来比较内容**。

| 比较方式 | 基本类型 | 引用类型（对象） |
|---------|---------|---------------|
| `==` | 比较**值** | 比较**内存地址**（是不是同一个对象） |
| `equals()` | 不能用（基本类型不是对象，没有方法） | 默认比较地址，但 `String`/`Integer` 等**重写后比较内容** |

**背诵口诀**：「**== 看地址，equals 看内容（前提是重写了）**」

**⚠️ 面试挖坑提醒**（这题面试经常挖坑）：分三种场景理解 String 的 `==`

```java
// 场景1：字面量 → 指向常量池（JVM 专门存放字符串的一块内存区域）同一个对象
String a = "hello";
String b = "hello";
System.out.println(a == b);  // true

// 场景2：new → 堆上创建新对象
String c = new String("hello");
System.out.println(a == c);  // false

// 场景3：intern() → 强制使用常量池引用
System.out.println(a == c.intern());  // true
```

**常见追问**：「重写 `equals()` 为什么必须同时重写 `hashCode()`？」—— 因为 `HashMap`、`HashSet` 等集合先用 `hashCode()` 定位桶，再用 `equals()` 比较内容。如果两个 `equals()` 相等的对象 `hashCode()` 不同，就会被放进不同的桶里，导致集合行为异常。**规则：equals 相等 → hashCode 必须相等；hashCode 相等 → equals 不一定相等。**

---

### 6. String 为什么是不可变的？

**一句话总结**：`String` 底层用 `final` 修饰的 `char[]`（JDK 9+ 改为 `byte[]`）存储，且 `String` 类本身也是 `final` 不可继承，**创建后内容不可更改**。

不可变的原因（从设计角度）：

1. **安全性**：`String` 常用于存储敏感信息（URL、密码、文件路径），不可变可以防止值被意外篡改
2. **线程安全**：不可变对象天然线程安全，多线程共享无需加锁
3. **字符串常量池**：相同内容的字符串可以复用同一个对象节省内存，如果可变就无法安全共享
4. **缓存 hashCode**：`String` 常做 `HashMap` 的 key，不可变保证 hashCode 只算一次就能缓存

> **面试话术**：String 不可变有两层保障：底层 char 数组用 private final 修饰，外部无法修改；String 类本身用 final 修饰，不能被继承去破坏不可变性。这样设计主要是为了安全性、线程安全、支持字符串常量池复用和 hashCode 缓存。

**⚠️ 常见追问**：「String 真的完全不可变吗？」—— 通过**反射**可以修改底层 char 数组的内容，但这属于"破坏性"操作，实际开发中绝不应该这样做。

---

### 7. String、StringBuilder、StringBuffer 有什么区别？

**一句话总结**：`String` 不可变，每次修改都会创建新对象；`StringBuilder` 和 `StringBuffer` 都可变，区别是 **`StringBuffer` 线程安全（方法加了 `synchronized`）但慢，`StringBuilder` 线程不安全但快**。

| 特性 | `String` | `StringBuilder` | `StringBuffer` |
|------|----------|-----------------|----------------|
| 可变性 | ❌ 不可变 | ✅ 可变 | ✅ 可变 |
| 线程安全 | ✅（不可变天然安全） | ❌ | ✅（`synchronized`） |
| 性能 | 慢（频繁创建新对象） | **最快** | 中等 |
| 使用场景 | 字符串少量操作 | **单线程**大量拼接 | **多线程**大量拼接 |

**背诵口诀**：「**String 不可变，Builder 快但不安全，Buffer 安全但慢**」

> **面试话术**：三者的核心区别在于可变性和线程安全。String 不可变，每次拼接都会创建新对象，频繁操作性能差。StringBuilder 和 StringBuffer 底层都是可变的 char 数组，不会频繁创建新对象。区别在于 StringBuffer 的方法加了 synchronized 关键字，线程安全但性能稍差；StringBuilder 没加锁，单线程下性能最好。实际开发中大多数场景用 StringBuilder 就够了。

---

### 8. 重载和重写有什么区别？

**一句话总结**：**重载（Overload）** 是同一个类中方法名相同但参数不同；**重写（Override）** 是子类重新实现父类的方法。

| 对比项 | 重载 Overload | 重写 Override |
|--------|-------------|-------------|
| 发生位置 | **同一个类**中 | **父子类**之间 |
| 方法名 | 必须相同 | 必须相同 |
| 参数列表 | **必须不同**（个数/类型/顺序） | **必须相同** |
| 返回类型 | 可以不同 | 必须相同或是子类（如父类返回 `Number`，子类可返回 `Integer`） |
| 访问修饰符 | 无限制 | 子类 ≥ 父类（不能更严格） |
| 异常 | 无限制 | 子类 ≤ 父类（不能抛更多受检异常） |
| 多态类型 | **编译时多态**（编译器根据参数确定调哪个） | **运行时多态**（运行时根据实际对象确定调哪个） |

**背诵口诀**：「**重载看参数，重写看父子**」「**重写两小一大**：子类抛异常更小、返回类型更小（或相同）、访问权限更大（或相同）」

```java
// 重载：同一类，方法名相同，参数不同
public int add(int a, int b) { return a + b; }
public double add(double a, double b) { return a + b; }

// 重写：子类重新实现父类方法
class Animal { void speak() { System.out.println("..."); } }
class Dog extends Animal {
    @Override void speak() { System.out.println("汪汪！"); }
}
```

**⚠️ 面试挖坑提醒**：「能不能只改返回类型算重载？」—— **不能**！仅返回类型不同不构成重载，编译会报错。重载必须参数列表不同。

---

### 9. 面向对象的三大特性是什么？

**一句话总结**：**封装**把数据藏起来只留接口、**继承**让子类复用父类代码、**多态**让同一个方法在不同对象上表现不同。

1. **封装（Encapsulation）**：把数据（属性）和操作数据的方法包在一起，对外只暴露必要的接口，用 `private`/`protected`/`public` 控制访问权限
2. **继承（Inheritance）**：子类可以复用父类的属性和方法，用 `extends` 实现，Java **只支持单继承**（一个类只能有一个父类），但可以多层继承
3. **多态（Polymorphism）**：同一个方法调用，根据实际对象类型执行不同的行为。实现条件：**继承 + 重写 + 父类引用指向子类对象**

```java
// 多态示例
Animal animal = new Dog();  // 父类引用指向子类对象
animal.speak();  // 运行时调用 Dog 的 speak()，而不是 Animal 的
```

**背诵口诀**：「**封装藏细节，继承省代码，多态活接口**」

> **面试话术**：面向对象三大特性是封装、继承和多态。封装是把属性和方法包在类里面，通过访问修饰符控制外部访问权限；继承让子类可以复用父类的代码，Java 只支持单继承；多态是指父类引用可以指向子类对象，运行时根据实际类型调用对应的方法，实现多态需要继承、方法重写和向上转型三个条件。

---

### 10. 接口和抽象类有什么区别？

**一句话总结**：抽象类是对**事物**的抽象（is-a），接口是对**行为**的抽象（can-do）。一个类只能继承一个抽象类，但可以实现多个接口。

| 对比项 | 抽象类 `abstract class` | 接口 `interface` |
|--------|----------------------|-----------------|
| 关键字 | `extends` | `implements` |
| 多继承 | ❌ 单继承 | ✅ 可以实现多个 |
| 构造器 | ✅ 有 | ❌ 没有 |
| 成员变量 | 任意类型 | 只能是 `public static final` 常量 |
| 方法 | 可以有普通方法和抽象方法 | JDK 8 前只能有抽象方法；JDK 8+ 可以有 `default`/`static` 方法 |
| 设计目的 | 代码复用，模板模式 | 定义行为规范，解耦 |
| 关系 | is-a（"猫**是**动物"） | can-do（"猫**能**爬树"） |

**背诵口诀**：「**抽象类是模板，接口是规范；抽象单继承，接口多实现**」

> **面试话术**：抽象类和接口最核心的区别有两个：第一，一个类只能继承一个抽象类，但可以实现多个接口；第二，抽象类是对事物本质的抽象，比如"猫是动物"，而接口是对行为的抽象，比如"猫能爬树"。JDK 8 之后接口也可以有 default 方法了，两者的界限变得模糊了一些，但设计意图还是不同的。

**⚠️ 面试挖坑提醒**：「JDK 8 之后接口能有方法实现了，那接口和抽象类还有啥区别？」—— 即使接口有了 default 方法，抽象类仍然可以有**构造器**、**成员变量**（非 static final）、**非 public 方法**，这些接口都做不到。

---

### 11. final 关键字有什么作用？

**一句话总结**：`final` 修饰**类**不可继承，修饰**方法**不可重写，修饰**变量**不可重新赋值。

| 修饰对象 | 效果 | 典型例子 |
|---------|------|---------|
| **类** | 不能被继承 | `String`、`Integer`、`Math` |
| **方法** | 不能被子类重写 | `Object.getClass()` |
| **基本类型变量** | 值不可变 | `final int MAX = 100;` |
| **引用类型变量** | **引用地址**不可变，但对象**内容**可变 | `final List<String> list = new ArrayList<>(); list.add("ok");` ✅ |

**⚠️ 面试挖坑提醒**（这题面试经常挖坑）：`final` 修饰引用类型时，**不可变的是引用（地址），不是对象本身**。

```java
final int[] arr = {1, 2, 3};
arr[0] = 99;      // ✅ 可以！修改的是数组内容
arr = new int[5];  // ❌ 编译报错！不能改变引用指向
```

**关联知识点**：`final` + `static` 一起用就是**常量**，命名惯例全大写：`public static final double PI = 3.14159;`

---

### 12. Java 的异常体系是怎样的？

**一句话总结**：所有异常都继承自 `Throwable`，分为 `Error`（系统级错误，不处理）和 `Exception`（程序可处理），`Exception` 又分为**受检异常**（编译器强制处理）和**非受检异常**（`RuntimeException`，编译器不管）。

```
Throwable
├── Error（不处理）
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── ...
└── Exception（要处理）
    ├── 受检异常 Checked（编译器强制 try-catch 或 throws）
    │   ├── IOException
    │   ├── SQLException
    │   └── ...
    └── RuntimeException 非受检异常 Unchecked（编译器不管）
        ├── NullPointerException
        ├── ArrayIndexOutOfBoundsException
        ├── ClassCastException
        └── ...
```

| 类型 | 特点 | 处理方式 | 举例 |
|------|------|---------|------|
| **Error** | JVM 层面的严重错误，程序无法恢复 | 不处理 | `OutOfMemoryError`、`StackOverflowError` |
| **受检异常** | 编译器强制要求处理 | `try-catch` 或 `throws` | `IOException`、`SQLException` |
| **非受检异常** | 编译器不强制处理，通常是代码逻辑错误 | 可选处理 | `NullPointerException`、`IllegalArgumentException` |

**背诵口诀**：「**Error 不管，Checked 必管，Runtime 选管**」

> **面试话术**：Java 异常体系的根是 Throwable，下面分 Error 和 Exception。Error 是系统级错误，比如内存溢出，程序无法恢复，不需要我们处理。Exception 分两种：受检异常（Checked Exception）编译器会强制要求用 try-catch 或 throws 处理，比如 IOException；非受检异常（RuntimeException）编译器不会强制处理，通常是程序逻辑错误导致的，比如空指针异常。

**⚠️ 常见追问**：「`try-catch-finally` 中 `finally` 一定会执行吗？」—— **几乎一定**，除了两种极端情况：① `System.exit()` 直接终止 JVM；② JVM 崩溃或线程被强制 kill。如果 `try` 和 `finally` 都有 `return`，以 `finally` 中的 `return` 为准（但不推荐在 finally 中写 return）。

---

## 三、常见机制与关键字

### 13. Java 是值传递还是引用传递？

**一句话总结**：Java **只有值传递**，没有引用传递。传对象时传的是**引用的副本**（地址的拷贝），不是对象本身。

两种场景对比：

```java
// 场景1：基本类型 → 传的是值的拷贝，修改不影响原变量
void change(int x) { x = 100; }
int a = 1;
change(a);  // a 还是 1

// 场景2：引用类型 → 传的是引用（地址）的拷贝
void change(int[] arr) { arr[0] = 100; }
int[] a = {1, 2};
change(a);  // a[0] 变成了 100（通过拷贝的地址修改了同一个对象）

// 场景3：引用类型（重新赋值）
void change(String s) { s = "world"; }
String a = "hello";
change(a);  // a 还是 "hello"（s 指向了新对象，不影响 a）
```

**⚠️ 面试挖坑提醒**（这题面试经常挖坑）：很多人以为传对象就是"引用传递"，其实不是。**关键区别**：引用传递能让调用者的引用变量指向新对象，Java 做不到。Java 传的是引用的副本，在方法内部让副本指向新对象不会影响外面的原引用。

> **面试话术**：Java 只有值传递。传基本类型时传的是值的拷贝；传引用类型时传的是引用地址的拷贝。通过拷贝的地址可以修改对象的内容，但不能让外面的引用指向另一个对象。

---

### 14. hashCode() 和 equals() 是什么关系？

**一句话总结**：`equals()` 用于判断两个对象**内容**是否相等，`hashCode()` 返回哈希值用于**快速定位**。两者必须遵守约定：**equals 相等 → hashCode 必须相等**。

**为什么必须一起重写？**

`HashMap`/`HashSet` 的查找过程：
1. 先算 `hashCode()` → 快速定位到哪个桶（像查字典先查首字母）
2. 再用 `equals()` → 在桶内逐个比较找到目标

如果只重写 `equals()` 不重写 `hashCode()`：两个内容相同的对象可能算出不同的 hashCode，被分到不同的桶里，`HashMap` 就找不到了。

| 规则 | 说明 |
|------|------|
| equals 相等 → hashCode **必须**相等 | 否则 HashMap 会出 bug |
| hashCode 相等 → equals **不一定**相等 | 哈希冲突是正常的 |
| equals 不等 → hashCode 尽量不等 | 减少冲突，提升性能 |

**背诵口诀**：「**equals 相等 hashCode 必等，hashCode 相等 equals 未必**」

---

### 15. 深拷贝和浅拷贝有什么区别？

**一句话总结**：**浅拷贝**只复制对象本身和基本类型字段，引用类型字段**共享**同一个对象；**深拷贝**递归复制所有引用对象，两份完全**独立**。

| 对比项 | 浅拷贝 | 深拷贝 |
|--------|--------|--------|
| 基本类型字段 | ✅ 复制值 | ✅ 复制值 |
| 引用类型字段 | ❌ 只复制引用地址（共享） | ✅ 递归创建新对象（独立） |
| 修改影响 | 改一个影响另一个 | 互不影响 |
| 实现方式 | `Object.clone()`（实现 `Cloneable`） | 递归 clone / 序列化反序列化 |

```java
// 浅拷贝：arr 指向同一个数组对象
Person p1 = new Person("Tom", new int[]{1,2});
Person p2 = p1.clone();  // 浅拷贝
p2.scores[0] = 99;  // p1.scores[0] 也变成 99！

// 深拷贝：完全独立
Person p3 = deepCopy(p1);
p3.scores[0] = 99;  // p1 不受影响
```

> **面试话术**：浅拷贝只复制对象的第一层，引用类型字段还是指向同一个对象，所以修改会互相影响。深拷贝会递归复制所有引用字段创建全新的对象，两份完全独立。实际开发中常用序列化反序列化的方式实现深拷贝。

---

### 16. static 关键字有什么作用？

**一句话总结**：`static` 表示"属于类，不属于实例"。可以修饰**变量**（类变量）、**方法**（类方法）、**代码块**（类加载时执行）、**内部类**（不依赖外部类实例）。

| 修饰对象 | 效果 | 注意事项 |
|---------|------|---------|
| **变量** | 所有实例共享一份，类加载时初始化 | 通过 `类名.变量名` 访问 |
| **方法** | 不需要创建对象就能调用 | **不能访问**非 static 成员（因为没有 this） |
| **代码块** | 类加载时执行且只执行一次 | 用于初始化静态变量 |
| **内部类** | 不持有外部类的引用 | 只能访问外部类的 static 成员 |

**⚠️ 面试挖坑提醒**：「static 方法能不能调用非 static 方法？」—— **不能直接调用**。因为 static 方法属于类，执行时没有 `this` 引用，而非 static 方法需要通过对象实例调用。但可以在 static 方法内**创建对象**后通过对象调用。

**关联知识点**：`static` + `final` = 常量；`static` 代码块的执行顺序：父类 static 块 → 子类 static 块 → 父类构造器 → 子类构造器。

---

### 17. 什么是泛型？什么是类型擦除？

**一句话总结**：**泛型**就是“类型占位符”，写代码时先用 `<T>` 占位，用的时候再指定具体类型（如 `List<String>`）。**类型擦除**是指编译后泛型信息会被擦除，运行时 JVM 不知道泛型是什么类型。

```java
// 不用泛型：需要强转，不安全
List list = new ArrayList();
list.add("hello");
String s = (String) list.get(0);  // 手动转型

// 用泛型：编译期检查类型，安全
List<String> list = new ArrayList<>();
list.add("hello");
String s = list.get(0);  // 不需要转型
```

**类型擦除**：编译后 `List<String>` 和 `List<Integer>` 都变成了 `List`，泛型参数信息被擦除。

**⚠️ 面试挖坑提醒**：「通配符 `?`、`extends`、`super` 的区别」

| 写法 | 含义 | 场景 |
|------|------|------|
| `<?>` | 无界通配符，可接受任何类型 | 只读不写 |
| `<? extends T>` | 上界通配符，只能是 T 或 T 的子类 | **取数据**（生产者） |
| `<? super T>` | 下界通配符，只能是 T 或 T 的父类 | **存数据**（消费者） |

**背诵口诀**：「**PECS 原则：生产者用 Extends（取数据），消费者用 Super（存数据）**」

---

### 18. 什么是反射？有什么用？

**一句话总结**：反射是在**运行时**动态获取类的信息（字段、方法、构造器）并调用的机制，核心类是 `java.lang.reflect` 包下的 `Class`、`Method`、`Field`、`Constructor`。

**获取 Class 对象的 3 种方式**：

```java
Class<?> c1 = String.class;              // 方式1：类名.class
Class<?> c2 = "hello".getClass();        // 方式2：对象.getClass()
Class<?> c3 = Class.forName("java.lang.String");  // 方式3：全限定类名
```

**反射的应用场景**：
- **框架底层**：Spring 的 IoC（控制反转，通过反射创建对象）、MyBatis 的动态代理
- **注解处理**：运行时读取注解信息并执行对应逻辑
- **动态代理**：JDK 动态代理基于反射实现

**⚠️ 面试挖坑提醒**：「反射的缺点？」—— ① 性能开销大（比直接调用慢几倍）；② 破坏封装性（可以访问 private 成员）；③ 编译期无法检查类型安全。

---

### 19. Object 类有哪些常用方法？

**一句话总结**：`Object` 是所有类的根父类，常用方法有 `equals()`、`hashCode()`、`toString()`、`clone()`、`getClass()`、`finalize()` 以及线程相关的 `wait()`/`notify()`/`notifyAll()`。

| 方法 | 作用 | 注意事项 |
|------|------|---------|
| `equals(Object)` | 比较内容是否相等 | 默认比较地址，通常需要重写 |
| `hashCode()` | 返回哈希值 | 重写 equals 必须同时重写 |
| `toString()` | 返回字符串表示 | 默认返回 `类名@哈希值`，建议重写 |
| `clone()` | 创建并返回对象的拷贝 | 需实现 `Cloneable`，默认浅拷贝 |
| `getClass()` | 返回运行时类的 Class 对象 | `final` 方法不可重写 |
| `finalize()` | 对象被回收前**可能**被调用一次，但不保证及时执行 | **已废弃**（JDK 9+），不应依赖 |
| `wait()` / `notify()` | 线程间通信 | 必须在 `synchronized` 块中调用 |

> **面试话术**：Object 是所有类的根类，最常考的方法是 equals 和 hashCode，需要一起重写。toString 建议重写方便调试。wait/notify 用于线程间协作，必须在同步块中使用。finalize 在 JDK 9 之后已被标记为废弃，不应使用。

---

### 20. Java 中有哪些访问修饰符？

**一句话总结**：Java 有 4 种访问级别，权限从大到小：`public` > `protected` > `默认（包级私有）` > `private`。

| 修饰符 | 同一个类 | 同一个包 | 不同包的子类 | 不同包的非子类 |
|--------|---------|---------|------------|-------------|
| `public` | ✅ | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| 默认（不写） | ✅ | ✅ | ❌ | ❌ |
| `private` | ✅ | ❌ | ❌ | ❌ |

**背诵口诀**：「**public 全可见，protected 跨包靠继承，默认同包可见，private 仅自己**」

**⚠️ 面试挖坑提醒**：`protected` 和默认修饰符容易混淆。**核心区别**：`protected` 允许不同包的**子类**访问，默认不允许。

---

## 四、进阶特性

### 21. 什么是序列化和反序列化？

**一句话总结**：**序列化**是把 Java 对象转换成**字节流**以便存储或传输，**反序列化**是把字节流恢复成 Java 对象。实现 `Serializable` 接口即可。

| 概念 | 方向 | 用途 |
|------|------|------|
| 序列化 | 对象 → 字节流 | 网络传输、保存到文件、远程方法调用（RPC） |
| 反序列化 | 字节流 → 对象 | 从文件/网络恢复对象 |

**关键知识点**：

- **`serialVersionUID`**：序列化版本号。如果不显式声明，JVM 会自动生成。反序列化时如果版本号不一致会抛 `InvalidClassException`。**建议手动声明**
- **`transient`**：被此关键字修饰的字段**不会被序列化**（如密码等敏感信息）
- 常见序列化方式：JDK 原生（`ObjectOutputStream`）、JSON（Jackson/Gson）、Protobuf、Hessian

**⚠️ 面试挖坑提醒**：「为什么不推荐用 JDK 原生序列化？」—— ① 性能差（速度慢、体积大）；② 存在安全漏洞（反序列化可被利用执行恶意代码）；③ 不能跨语言。实际开发中更推荐用 JSON 或 Protobuf。

---

### 22. Lambda 表达式和函数式接口是什么？

**一句话总结**：**Lambda 表达式**是 JDK 8 引入的匿名函数写法，让代码更简洁。**函数式接口**是只有一个抽象方法的接口（如 `Runnable`、`Comparator`），用 `@FunctionalInterface` 标注。

```java
// 传统写法
new Thread(new Runnable() {
    @Override
    public void run() { System.out.println("hello"); }
}).start();

// Lambda 写法
new Thread(() -> System.out.println("hello")).start();
```

**常用内置函数式接口**（`java.util.function` 包）：

| 接口 | 方法 | 用途 |
|------|------|------|
| `Predicate<T>` | `test(T)` → boolean | 判断条件 |
| `Function<T,R>` | `apply(T)` → R | 转换数据 |
| `Consumer<T>` | `accept(T)` → void | 消费数据 |
| `Supplier<T>` | `get()` → T | 生产数据 |

**关联知识点**：Lambda 常与 **Stream API** 搭配使用，如 `list.stream().filter(x -> x > 5).collect(Collectors.toList())`。方法引用（`System.out::println`）是 Lambda 的简写形式。

---

## 复习优先级（3~5 年）

| 优先级 | 题目 |
|--------|------|
| **P0 高频必背** | 3. 基本数据类型、4. 自动装箱拆箱、5. == 和 equals、6. String 不可变、7. String/Builder/Buffer、8. 重载和重写、9. 三大特性、12. 异常体系、13. 值传递、14. hashCode 和 equals、17. 泛型与类型擦除、18. 反射 |
| **P1 中频建议掌握** | 1. JDK/JRE/JVM、10. 接口和抽象类、11. final、15. 深拷贝浅拷贝、16. static、19. Object 常用方法、21. 序列化、22. Lambda |
| **P2 低频了解即可** | 2. 跨平台原理、20. 访问修饰符 |

