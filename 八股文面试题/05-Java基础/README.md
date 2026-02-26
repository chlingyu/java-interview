# 05 - Java 基础

> 覆盖 Java 语言核心基础：数据类型、字符串、面向对象、关键字、异常处理、反射与泛型等高频面试考点。

---

## 一、数据类型与基础语法

### 1. Java 有哪些基本数据类型？

> ⭐⭐⭐ 常问 | 难度：⭐

**回答**：Java 有 **8 种基本数据类型（primitive types）**，分为 4 类：

| 类型 | 关键字 | 字节数 | 默认值 | 取值范围 |
|------|--------|--------|--------|----------|
| 整型 | `byte` | ⚡**1** | 0 | -128 ~ 127 |
| 整型 | `short` | ⚡**2** | 0 | -32768 ~ 32767 |
| 整型 | `int` | ⚡**4** | 0 | -2^31 ~ 2^31-1 |
| 整型 | `long` | ⚡**8** | 0L | -2^63 ~ 2^63-1 |
| 浮点型 | `float` | ⚡**4** | 0.0f | IEEE 754 单精度 |
| 浮点型 | `double` | ⚡**8** | 0.0d | IEEE 754 双精度 |
| 字符型 | `char` | ⚡**2** | '\u0000' | 0 ~ 65535 |
| 布尔型 | `boolean` | **不确定** | false | true / false |

> 注意：`boolean` 在 JVM 规范中没有明确规定字节数，HotSpot 实现中单独使用时占 ⚡**4 字节**（当作 int 处理），在 boolean 数组中占 ⚡**1 字节**。

---

### 2. 自动装箱与拆箱是什么？有什么陷阱？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐

**一句话回答**：自动装箱是基本类型自动转为包装类型，拆箱是反过来。陷阱在于缓存范围和 NPE。

**通俗理解**：

想象你有一堆散装硬币（基本类型），要寄快递时必须装进盒子里（包装类型）才能发出去。Java 5 之后，编译器帮你自动装盒/拆盒，这就是 **自动装箱（Auto-boxing）** 和 **自动拆箱（Auto-unboxing）**。

但这里有个"坑"——Java 为了省钱，提前做好了一批"常用盒子"放在仓库里复用。如果你的硬币面值在这个范围内，拿到的是同一个盒子；超出范围，就会新做一个盒子。

**原理详解**：

装箱本质调用 `Integer.valueOf()`，拆箱本质调用 `intValue()`。

**Integer 缓存池**：`Integer.valueOf()` 对 ⚡**-128 ~ 127** 范围内的值会返回缓存对象，超出范围则 `new` 一个新对象。

```java
Integer a = 127;  // 自动装箱：Integer.valueOf(127)，命中缓存
Integer b = 127;
System.out.println(a == b);  // true —— 同一个缓存对象

Integer c = 128;  // 自动装箱：Integer.valueOf(128)，超出缓存，new 新对象
Integer d = 128;
System.out.println(c == d);  // false —— 两个不同的对象！

Integer e = null;
int f = e;  // 自动拆箱：e.intValue() → NullPointerException！
```

**各包装类型的缓存范围**：

| 包装类型 | 缓存范围 |
|----------|----------|
| `Byte` | ⚡**-128 ~ 127**（全部） |
| `Short` | ⚡**-128 ~ 127** |
| `Integer` | ⚡**-128 ~ 127**（可通过 JVM 参数调整上限） |
| `Long` | ⚡**-128 ~ 127** |
| `Character` | ⚡**0 ~ 127** |
| `Boolean` | `TRUE` / `FALSE`（两个实例） |
| `Float` / `Double` | **无缓存** |

**🎤 面试这样答**：
> "自动装箱就是编译器自动调用 `valueOf()` 把基本类型转成包装类，拆箱就是调用 `xxxValue()` 转回来。主要陷阱有两个：一是 `==` 比较时，Integer 在 -128 到 127 范围内会复用缓存对象，超出范围就是不同对象，所以包装类比较必须用 `equals()`；二是包装类型可能为 null，拆箱时会抛 NPE。"

---

### 3. == 和 equals() 的区别？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐

**一句话回答**：`==` 比较的是地址（基本类型比值），`equals()` 比较的是内容（需要重写）。

**通俗理解**：

你和朋友各买了一瓶同品牌的可乐。`==` 问的是"你俩手里拿的是不是**同一瓶**可乐？"（比地址），`equals()` 问的是"你俩的可乐**味道一不一样**？"（比内容）。

- 基本类型没有"瓶子"的概念，`==` 直接比味道（值）
- 引用类型有"瓶子"，`==` 比的是瓶子编号（内存地址），`equals()` 比的是里面的内容

**原理详解**：

**① 基本类型**：`==` 直接比较值

```java
int a = 10;
int b = 10;
System.out.println(a == b);  // true —— 值相等
```

**② 引用类型**：`==` 比较的是对象在堆内存中的地址

```java
String s1 = new String("hello");
String s2 = new String("hello");
System.out.println(s1 == s2);      // false —— 两个不同的对象，地址不同
System.out.println(s1.equals(s2)); // true  —— 内容相同
```

**③ Object 的 equals() 默认实现就是 `==`**：

```java
// Object.java 源码
public boolean equals(Object obj) {
    return (this == obj);  // 默认比地址！
}
```

所以自定义类如果不重写 `equals()`，效果和 `==` 一样。String、Integer 等类已经重写了 `equals()` 来比较内容。

**🎤 面试这样答**：
> "对于基本类型，`==` 比较的是值；对于引用类型，`==` 比较的是内存地址，也就是是否是同一个对象。`equals()` 是 Object 类的方法，默认实现也是 `==`，但 String、Integer 等类重写了它来比较内容。所以比较对象内容时应该用 `equals()`，而不是 `==`。"

---

### 4. hashCode() 和 equals() 是什么关系？为什么重写 equals 必须重写 hashCode？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：`equals()` 相等的两个对象，`hashCode()` 必须相等；反过来不一定。不遵守这个约定会导致 HashMap 等集合出 bug。

**通俗理解**：

把 HashMap 想象成一个有很多格子的**快递柜**。你寄快递时，先根据手机尾号（hashCode）决定放哪个格子，取快递时也先找到格子，再核对身份证（equals）确认是不是你的包裹。

如果两个"相同的人"（equals 相等）手机尾号不一样（hashCode 不同），那取快递时就会去错格子，永远找不到自己的包裹——这就是不重写 hashCode 的后果。

**原理详解**：

**Java 规范的三条约定**：
1. `equals()` 返回 `true` → `hashCode()` **必须相等**
2. `equals()` 返回 `false` → `hashCode()` **可以相等**（哈希冲突）
3. `hashCode()` 相等 → `equals()` **不一定** `true`

**反面教材**——只重写 equals 不重写 hashCode：

```java
public class User {
    private String name;

    // 只重写了 equals
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        return Objects.equals(name, ((User) o).name);
    }
    // 没有重写 hashCode！
}

Map<User, String> map = new HashMap<>();
User u1 = new User("张三");
map.put(u1, "开发");

User u2 = new User("张三");
System.out.println(u1.equals(u2));  // true —— 内容相同
System.out.println(map.get(u2));    // null！—— u2 的 hashCode 和 u1 不同，去了不同的桶
```

**正确写法**——equals 和 hashCode 一起重写：

```java
@Override
public int hashCode() {
    return Objects.hash(name);  // 用相同的字段计算 hashCode
}
```

**🎤 面试这样答**：
> "Java 规范要求：equals 相等的两个对象，hashCode 必须相等。因为 HashMap 先用 hashCode 定位桶，再用 equals 比较 key。如果只重写 equals 不重写 hashCode，两个逻辑相等的对象可能落在不同的桶里，导致 put 进去的值 get 不出来。所以重写 equals 时必须同时重写 hashCode，且用相同的字段来计算。"

---

## 二、字符串

### 5. String、StringBuilder、StringBuffer 的区别？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐

**一句话回答**：String 不可变，StringBuilder 可变但线程不安全，StringBuffer 可变且线程安全（方法加了 synchronized）。

**通俗理解**：

把字符串想象成**写字**：
- **String** = 用钢笔写在石碑上，刻完就不能改了。每次"修改"其实是刻一块新石碑
- **StringBuilder** = 用铅笔写在白板上，随便擦写，但只能一个人用（线程不安全）
- **StringBuffer** = 也是白板，但配了一把锁，同一时间只允许一个人写（线程安全，但慢）

**原理详解**：

| 特性 | String | StringBuilder | StringBuffer |
|------|--------|---------------|--------------|
| 可变性 | ⚡**不可变**（`final char[]`） | 可变 | 可变 |
| 线程安全 | 安全（不可变天然安全） | ⚡**不安全** | ⚡**安全**（synchronized） |
| 性能 | 拼接慢（每次创建新对象） | **最快** | 比 StringBuilder 慢 |
| 使用场景 | 少量字符串操作 | **单线程大量拼接**（首选） | 多线程大量拼接 |

> 注意：JDK 9 之后，String 内部从 `char[]` 改为 `byte[]` + `coder` 标志位（**Compact Strings** 优化），Latin-1 字符只占 ⚡**1 字节**。

**性能对比示例**：

```java
// ❌ 反面教材：循环中用 String 拼接
String s = "";
for (int i = 0; i < 10000; i++) {
    s += i;  // 每次都创建新的 String 对象，产生大量垃圾对象
}

// ✅ 正确做法：用 StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append(i);  // 在同一个对象上操作，不创建新对象
}
String result = sb.toString();
```

**🎤 面试这样答**：
> "String 是不可变的，每次修改都会创建新对象。StringBuilder 和 StringBuffer 都是可变的，区别在于 StringBuffer 的方法加了 synchronized 保证线程安全，但性能比 StringBuilder 差。实际开发中，单线程字符串拼接首选 StringBuilder，多线程场景才用 StringBuffer。另外编译器会把简单的字符串 `+` 优化成 StringBuilder，但循环中的拼接不会优化，需要手动使用 StringBuilder。"

---

### 6. String 为什么设计成不可变的？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐

**一句话回答**：为了安全、性能（字符串常量池复用）和线程安全。

**通俗理解**：

String 就像人民币上的面额——印好了就不能改。为什么？因为如果能改，你把 10 块改成 100 块，所有持有这张钞票引用的人都会受影响，整个经济系统就乱了。

**原理详解**：

**① 实现层面**——三重保护：

```java
// String.java 源码（JDK 8）
public final class String {           // final：不能被继承
    private final char value[];       // final：引用不可变
    // value 数组是 private 的，且没有提供修改方法
}
```

**② 为什么要这样设计？**

| 原因 | 说明 |
|------|------|
| **字符串常量池** | 相同内容的字符串可以共享同一个对象，节省内存。如果可变，一个引用修改了内容，其他引用全受影响 |
| **hashCode 缓存** | String 的 hashCode 计算后会缓存，不可变保证 hashCode 不变，HashMap 的 key 才可靠 |
| **线程安全** | 不可变对象天然线程安全，多线程共享无需同步 |
| **安全性** | 网络连接的 URL、文件路径、数据库连接串都是 String，不可变防止被恶意篡改 |

**🎤 面试这样答**：
> "String 不可变主要有三个原因：一是支持字符串常量池，相同内容的字符串可以复用同一个对象节省内存；二是 hashCode 可以缓存，作为 HashMap 的 key 更高效；三是天然线程安全，多线程环境下无需同步。实现上，String 类是 final 的不能继承，内部的 char 数组也是 private final 的，且不提供修改方法。"

---

### 7. String 的 intern() 方法有什么作用？

> ⭐⭐⭐ 常问 | 难度：⭐⭐⭐

**回答**：`intern()` 会把字符串放入 **字符串常量池（String Pool）**，如果池中已有相同内容的字符串，则直接返回池中的引用。

- JDK 6：常量池在 **永久代（PermGen）**，`intern()` 会把字符串**复制**一份到池中
- JDK 7+：常量池移到了 **堆（Heap）** 中，`intern()` 只在池中记录**堆对象的引用**，不再复制

```java
String s1 = new String("hello");  // 堆中创建对象
String s2 = s1.intern();          // 返回常量池中 "hello" 的引用
String s3 = "hello";              // 字面量，直接指向常量池

System.out.println(s1 == s2);  // false —— s1 是堆对象，s2 是常量池引用
System.out.println(s2 == s3);  // true  —— 都指向常量池中的同一个对象
```

**经典面试题**：`String s = new String("hello")` 创建了几个对象？

答：最多 ⚡**2 个**。一个是常量池中的 `"hello"`（如果之前不存在），另一个是堆中 `new` 出来的 String 对象。

---

## 三、面向对象

### 8. 接口和抽象类的区别？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐

**一句话回答**：抽象类是"是什么"（is-a），接口是"能做什么"（can-do）。抽象类单继承，接口多实现。

**通俗理解**：

- **抽象类**像一份"岗位说明书"——你是"程序员"这个岗位，必须会写代码（抽象方法），公司还给你配了工位和电脑（成员变量和具体方法）。但你只能属于一个岗位（单继承）
- **接口**像一张"技能证书"——你可以同时持有"驾照""英语六级""PMP"等多张证书（多实现），每张证书只规定你要会什么技能，不管你怎么学的

**原理详解**：

| 对比项 | 抽象类（abstract class） | 接口（interface） |
|--------|--------------------------|-------------------|
| 继承/实现 | 单继承（`extends`） | ⚡**多实现**（`implements`） |
| 构造方法 | **有** | 无 |
| 成员变量 | 可以有普通成员变量 | 只能有 `public static final` 常量 |
| 方法 | 可以有抽象方法和具体方法 | JDK 8 前只能有抽象方法 |
| JDK 8+ | — | 可以有 `default` 方法和 `static` 方法 |
| JDK 9+ | — | 可以有 `private` 方法 |
| 设计理念 | **is-a**（是什么） | **can-do / has-a**（能做什么） |

```java
// 抽象类：定义"是什么"
public abstract class Animal {
    protected String name;          // 可以有成员变量
    public Animal(String name) {    // 可以有构造方法
        this.name = name;
    }
    public abstract void speak();   // 抽象方法
    public void breathe() {         // 具体方法
        System.out.println("呼吸");
    }
}

// 接口：定义"能做什么"
public interface Swimmable {
    void swim();                    // 抽象方法
    default void float_() {         // JDK 8 默认方法
        System.out.println("漂浮");
    }
}

// 单继承 + 多实现
public class Duck extends Animal implements Swimmable, Flyable {
    // ...
}
```

**🎤 面试这样答**：
> "抽象类和接口最核心的区别是：抽象类是 is-a 关系，用于抽取公共代码，只能单继承；接口是 can-do 关系，定义行为规范，可以多实现。抽象类可以有构造方法、成员变量和具体方法；接口在 JDK 8 之前只能有抽象方法和常量，JDK 8 之后可以有 default 和 static 方法。实际开发中，优先用接口定义行为契约，用抽象类复用公共逻辑。"

---

### 9. 重载（Overload）和重写（Override）的区别？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐

**一句话回答**：重载是同一个类中方法名相同、参数不同；重写是子类覆盖父类的同名同参方法。

**通俗理解**：

- **重载**像餐厅的"套餐"——同一家店（同一个类），菜名都叫"套餐"，但 A 套餐是一荤一素，B 套餐是两荤一素（参数列表不同）
- **重写**像"子承父业"——儿子继承了父亲的饭店，菜名和规格都一样，但做法换成了自己的风格（实现不同）

**原理详解**：

| 对比项 | 重载（Overload） | 重写（Override） |
|--------|------------------|------------------|
| 发生位置 | **同一个类**中 | **子类与父类**之间 |
| 方法名 | 相同 | 相同 |
| 参数列表 | ⚡**必须不同**（类型/个数/顺序） | ⚡**必须相同** |
| 返回值 | 无要求 | 相同或是其子类（协变返回） |
| 访问修饰符 | 无要求 | ⚡**不能更严格**（子类 ≥ 父类） |
| 异常 | 无要求 | ⚡**不能抛更大的受检异常** |
| 绑定时机 | **编译期**（静态分派） | **运行期**（动态分派，多态） |

```java
// 重载：同一个类，方法名相同，参数不同
public class Calculator {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }  // 参数类型不同
    public int add(int a, int b, int c) { return a + b + c; } // 参数个数不同
}

// 重写：子类覆盖父类方法
public class Animal {
    public void speak() { System.out.println("..."); }
}
public class Dog extends Animal {
    @Override  // 建议加上注解，编译器会帮你检查
    public void speak() { System.out.println("汪汪！"); }
}
```

**🎤 面试这样答**：
> "重载发生在同一个类中，方法名相同但参数列表不同，在编译期就确定调用哪个方法。重写发生在子类和父类之间，方法签名完全相同，在运行期根据对象的实际类型动态分派，这是多态的基础。重写时要注意：访问权限不能更严格，不能抛出更大的受检异常，返回值要相同或是协变类型。"

---

### 10. 深拷贝与浅拷贝的区别？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐

**一句话回答**：浅拷贝只复制对象本身和基本类型字段，引用类型字段仍指向同一个对象；深拷贝会递归复制所有引用对象，完全独立。

**通俗理解**：

你有一份简历，上面贴了一张照片：
- **浅拷贝** = 复印简历，但照片没复印，两份简历上贴的是**同一张照片**。你在照片上画了胡子，两份简历都受影响
- **深拷贝** = 复印简历 + 重新冲洗一张照片。两份简历完全独立，互不影响

**原理详解**：

```java
public class Address implements Cloneable {
    String city;
}

public class Person implements Cloneable {
    String name;       // 基本类型（String 不可变，效果等同基本类型）
    Address address;   // 引用类型

    // 浅拷贝：Object.clone() 默认行为
    @Override
    protected Person clone() throws CloneNotSupportedException {
        return (Person) super.clone();
        // address 字段只复制了引用，没有复制对象本身！
    }

    // 深拷贝：手动递归复制引用对象
    protected Person deepClone() throws CloneNotSupportedException {
        Person copy = (Person) super.clone();
        copy.address = (Address) this.address.clone();  // 引用对象也要 clone
        return copy;
    }
}
```

**实现深拷贝的常见方式**：

| 方式 | 优点 | 缺点 |
|------|------|------|
| 重写 `clone()` 递归复制 | 性能好 | 代码繁琐，嵌套深时容易遗漏 |
| **序列化/反序列化** | 通用，不怕嵌套 | 性能差，类必须实现 Serializable |
| JSON 转换（Jackson/Gson） | 简单方便 | 性能一般，可能丢失类型信息 |

**🎤 面试这样答**：
> "浅拷贝只复制对象本身，引用类型字段还是指向原来的对象，修改会互相影响。深拷贝会递归复制所有引用对象，两个对象完全独立。实现深拷贝常见的方式有三种：重写 clone 方法递归复制、通过序列化反序列化、或者用 JSON 工具转换。实际开发中序列化方式最通用，但性能敏感场景建议手动实现 clone。"

---

## 四、关键字与修饰符

### 11. final 关键字的作用？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐

**一句话回答**：final 修饰类不能被继承，修饰方法不能被重写，修饰变量不能被重新赋值。

**通俗理解**：

`final` 就是"盖棺定论"——一旦定了就不能改：
- **final 类** = 绝育了，不能有后代（不能被继承）
- **final 方法** = 祖传秘方，后代不能改配方（不能被重写）
- **final 变量** = 刻在石头上的名字，写完就不能擦（不能重新赋值）

**原理详解**：

```java
// ① final 修饰类：不能被继承
public final class String { }  // String 就是 final 的
// public class MyString extends String { }  // ❌ 编译报错

// ② final 修饰方法：不能被重写
public class Parent {
    public final void doSomething() { }
}
public class Child extends Parent {
    // public void doSomething() { }  // ❌ 编译报错
}

// ③ final 修饰变量：不能重新赋值
final int x = 10;
// x = 20;  // ❌ 编译报错

// ⚠️ 注意：final 修饰引用类型，引用不可变，但对象内容可以变！
final List<String> list = new ArrayList<>();
list.add("hello");  // ✅ 可以修改对象内容
// list = new ArrayList<>();  // ❌ 不能重新指向新对象
```

**🎤 面试这样答**：
> "final 有三种用法：修饰类表示不能被继承，比如 String；修饰方法表示不能被子类重写；修饰变量表示不能重新赋值。需要注意的是，final 修饰引用类型变量时，只是引用不可变，对象本身的内容还是可以修改的。另外，final 修饰的局部变量如果在编译期就能确定值，会被当作编译期常量内联优化。"

---

### 12. static 关键字有哪些用法？

> ⭐⭐⭐ 常问 | 难度：⭐⭐

**回答**：`static` 表示"属于类，而非属于实例"，有以下用法：

| 用法 | 说明 |
|------|------|
| **静态变量** | 类级别共享，所有实例共用一份，通过 `类名.变量名` 访问 |
| **静态方法** | 不依赖实例，不能访问 `this` 和非静态成员 |
| **静态代码块** | 类加载时执行一次，常用于初始化静态资源 |
| **静态内部类** | 不持有外部类引用，可以独立创建实例 |
| **静态导入** | `import static`，直接使用静态方法/常量而不需要类名前缀 |

```java
public class Counter {
    private static int count = 0;  // 静态变量：所有实例共享

    static {  // 静态代码块：类加载时执行一次
        System.out.println("类被加载了");
    }

    public static int getCount() {  // 静态方法
        // this.xxx  // ❌ 静态方法中不能用 this
        return count;
    }

    static class Inner {  // 静态内部类：不持有外部类引用
        // ...
    }
}
```

**执行顺序**：父类静态代码块 → 子类静态代码块 → 父类构造代码块 → 父类构造方法 → 子类构造代码块 → 子类构造方法。

---

## 五、异常处理

### 13. Java 异常体系是怎样的？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：所有异常的根类是 Throwable，分为 Error（系统级错误）和 Exception（程序可处理的异常），Exception 又分为受检异常和非受检异常（RuntimeException）。

**通俗理解**：

把异常体系想象成"事故分类"：
- **Error** = 天灾（地震、海啸）——程序无能为力，比如内存爆了（OOM）、栈溢出（StackOverflow），不需要也不应该 catch
- **受检异常（Checked Exception）** = 可预见的风险——比如开车上路可能遇到堵车（IOException），法律要求你必须买保险（必须 try-catch 或 throws）
- **非受检异常（RuntimeException）** = 人为失误——比如闯红灯（NullPointerException）、超速（ArrayIndexOutOfBoundsException），属于代码 bug，应该修复而不是 catch

**原理详解**：

```
                    Throwable
                   /         \
               Error        Exception
              /    \        /         \
     OutOfMemory  StackOverflow   RuntimeException    IOException
                                  /        \          /        \
                          NullPointer  ClassCast  FileNotFound  SQL
                          IndexOutOf   IllegalArg
```

| 分类 | 说明 | 是否必须处理 | 常见例子 |
|------|------|-------------|----------|
| **Error** | JVM 系统级错误 | 不需要 | `OutOfMemoryError`、`StackOverflowError` |
| **受检异常** | 编译器强制要求处理 | ⚡**必须** try-catch 或 throws | `IOException`、`SQLException`、`ClassNotFoundException` |
| **非受检异常** | RuntimeException 及其子类 | 不强制 | `NullPointerException`、`IndexOutOfBoundsException`、`ClassCastException` |

**try-catch-finally 执行顺序的坑**：

```java
public static int test() {
    try {
        return 1;
    } finally {
        return 2;  // ⚠️ finally 中的 return 会覆盖 try 中的 return！
    }
}
// 结果返回 2，不是 1！
```

**🎤 面试这样答**：
> "Java 异常体系的根类是 Throwable，分为 Error 和 Exception。Error 是 JVM 层面的严重错误，比如 OOM，程序不应该 catch。Exception 分为受检异常和非受检异常：受检异常编译器强制要求处理，比如 IOException；非受检异常是 RuntimeException 的子类，比如 NPE，通常是代码 bug，应该通过逻辑判断避免而不是 catch。实际开发中推荐用 try-with-resources 处理资源关闭，避免在 finally 中写 return。"

---

### 14. throw 和 throws 的区别？

> ⭐⭐ 了解 | 难度：⭐

**回答**：`throw` 是在方法体内**主动抛出**一个异常对象；`throws` 是在方法签名上**声明**该方法可能抛出的异常类型。

```java
// throws：声明方法可能抛出的异常，调用者必须处理
public void readFile(String path) throws IOException {
    if (path == null) {
        throw new IllegalArgumentException("路径不能为空");  // throw：主动抛出异常
    }
    // ...
}
```

| 对比项 | throw | throws |
|--------|-------|--------|
| 位置 | 方法体内 | 方法签名上 |
| 作用 | 抛出一个具体的异常**对象** | 声明可能抛出的异常**类型** |
| 数量 | 一次只能抛一个 | 可以声明多个（逗号分隔） |

---

## 六、高级特性

### 15. 反射机制是什么？有什么应用场景？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：反射是在运行时动态获取类的信息并操作类的属性和方法的机制，是框架的灵魂。

**通俗理解**：

正常写代码就像"按菜单点菜"——你知道有什么菜（类），直接点就行。**反射（Reflection）** 就像"闯进后厨"——你可以在运行时打开任何一个锅（类），看看里面有什么食材（字段）、什么做法（方法），甚至自己动手炒一盘（调用私有方法）。

Spring 的依赖注入、MyBatis 的 Mapper 代理、Jackson 的 JSON 序列化……这些框架底层全靠反射。

**原理详解**：

**获取 Class 对象的三种方式**：

```java
// 方式一：类名.class（编译期就确定）
Class<?> clazz1 = String.class;

// 方式二：对象.getClass()（运行时获取）
String s = "hello";
Class<?> clazz2 = s.getClass();

// 方式三：Class.forName()（最灵活，常用于框架）
Class<?> clazz3 = Class.forName("java.lang.String");
```

**反射的核心操作**：

```java
Class<?> clazz = Class.forName("com.example.User");

// 创建实例
Object obj = clazz.getDeclaredConstructor().newInstance();

// 获取并调用方法
Method method = clazz.getDeclaredMethod("setName", String.class);
method.invoke(obj, "张三");

// 获取并修改私有字段
Field field = clazz.getDeclaredField("name");
field.setAccessible(true);  // 突破 private 限制
String name = (String) field.get(obj);
```

**反射的代价**：性能比直接调用慢 ⚡**几倍到几十倍**（JIT 优化后差距缩小），且会破坏封装性。

**🎤 面试这样答**：
> "反射是 Java 在运行时动态获取类信息、创建对象、调用方法的能力。获取 Class 对象有三种方式：类名.class、对象.getClass()、Class.forName()。反射是 Spring IoC、AOP、MyBatis 等框架的基础，比如 Spring 通过反射创建 Bean 并注入依赖。缺点是性能比直接调用差，而且会破坏封装性。JDK 9 模块化之后对反射做了更严格的访问控制。"

---

### 16. 泛型是什么？什么是类型擦除？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐

**一句话回答**：泛型是编译期的类型参数化机制，编译后泛型信息会被擦除（类型擦除），运行时不保留泛型类型。

**通俗理解**：

泛型就像快递箱上贴的"标签"——你在箱子上贴了"只装书"（`List<Book>`），快递员（编译器）会帮你检查，不让你往里塞衣服。但箱子到了仓库（JVM 运行时），标签就被撕掉了，仓库只看到一个普通箱子（`List`）。这就是 **类型擦除（Type Erasure）**。

**原理详解**：

**类型擦除规则**：
- `List<String>` → 擦除为 `List`
- `T` → 擦除为 `Object`（无界）
- `T extends Comparable` → 擦除为 `Comparable`（有界）

```java
// 编译前
List<String> list = new ArrayList<>();
list.add("hello");
String s = list.get(0);

// 编译后（类型擦除）
List list = new ArrayList();
list.add("hello");
String s = (String) list.get(0);  // 编译器自动插入强制转换
```

**类型擦除带来的限制**：

```java
// ❌ 不能 new 泛型数组
T[] arr = new T[10];  // 编译报错

// ❌ 不能 instanceof 泛型
if (obj instanceof List<String>) { }  // 编译报错

// ❌ 运行时泛型信息丢失
List<String> a = new ArrayList<>();
List<Integer> b = new ArrayList<>();
System.out.println(a.getClass() == b.getClass());  // true！都是 ArrayList.class
```

**🎤 面试这样答**：
> "泛型是 Java 5 引入的编译期类型检查机制，可以在编译时发现类型错误，避免运行时的 ClassCastException。但 Java 的泛型采用类型擦除实现，编译后泛型信息会被擦除，运行时 `List<String>` 和 `List<Integer>` 是同一个类。这导致不能 new 泛型数组、不能 instanceof 泛型类型等限制。不过通过反射可以获取字段、方法参数上声明的泛型信息，因为这些信息保存在 Class 文件的 Signature 属性中。"

---

### 17. 序列化与反序列化是什么？

> ⭐⭐⭐ 常问 | 难度：⭐⭐

**回答**：**序列化（Serialization）** 是把对象转换为字节流以便存储或传输，**反序列化（Deserialization）** 是把字节流还原为对象。

实现方式：类实现 `Serializable` 接口（标记接口，无需实现任何方法）。

```java
public class User implements Serializable {
    private static final long serialVersionUID = 1L;  // 版本号，反序列化时校验兼容性
    private String name;
    private transient String password;  // transient 修饰的字段不参与序列化
}
```

**关键点**：

| 要点 | 说明 |
|------|------|
| `serialVersionUID` | 版本号，不一致会抛 `InvalidClassException`。⚡**强烈建议显式声明** |
| `transient` | 修饰的字段不会被序列化（如密码、临时缓存） |
| `static` 字段 | 不参与序列化（属于类，不属于对象） |
| 常见替代方案 | JSON（Jackson/Gson）、Protobuf、Hessian，性能和跨语言能力更好 |

---

### 18. Object 类有哪些常用方法？

> ⭐⭐⭐ 常问 | 难度：⭐⭐

**回答**：Object 是所有类的根类，常用方法如下：

| 方法 | 作用 |
|------|------|
| `equals(Object obj)` | 判断两个对象是否相等，默认比较地址，通常需要重写 |
| `hashCode()` | 返回对象的哈希码，重写 equals 必须同时重写 |
| `toString()` | 返回对象的字符串表示，默认是 `类名@哈希值`，建议重写 |
| `getClass()` | 返回对象的运行时 Class 对象，是反射的入口 |
| `clone()` | 创建并返回对象的浅拷贝，需实现 `Cloneable` 接口 |
| `finalize()` | GC 回收前调用，⚡**JDK 9 已标记 @Deprecated**，不建议使用 |
| `wait()` / `notify()` / `notifyAll()` | 线程间通信，必须在 `synchronized` 块中调用 |

```java
public class User {
    private String name;
    private int age;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return age == user.age && Objects.equals(name, user.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }

    @Override
    public String toString() {
        return "User{name='" + name + "', age=" + age + "}";
    }
}

---

## 面试高频程度排序（3~5 年）

| 优先级 | 题目 |
|--------|------|
| ⭐⭐⭐⭐⭐ | == 和 equals() 的区别 |
| ⭐⭐⭐⭐⭐ | hashCode() 和 equals() 的关系 |
| ⭐⭐⭐⭐⭐ | String、StringBuilder、StringBuffer 的区别 |
| ⭐⭐⭐⭐⭐ | 接口和抽象类的区别 |
| ⭐⭐⭐⭐⭐ | Java 异常体系 |
| ⭐⭐⭐⭐⭐ | 反射机制及应用场景 |
| ⭐⭐⭐⭐ | 自动装箱与拆箱的陷阱 |
| ⭐⭐⭐⭐ | String 为什么是不可变的 |
| ⭐⭐⭐⭐ | 重载和重写的区别 |
| ⭐⭐⭐⭐ | 深拷贝与浅拷贝 |
| ⭐⭐⭐⭐ | final 关键字的作用 |
| ⭐⭐⭐⭐ | 泛型与类型擦除 |
| ⭐⭐⭐ | Java 基本数据类型 |
| ⭐⭐⭐ | String 的 intern() 方法 |
| ⭐⭐⭐ | static 关键字的用法 |
| ⭐⭐⭐ | 序列化与反序列化 |
| ⭐⭐⭐ | Object 类的常用方法 |
| ⭐⭐ | throw 和 throws 的区别 |

