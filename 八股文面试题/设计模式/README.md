# 设计模式

---

## 一、创建型模式

### 1. 单例模式有几种写法？

**一句话总结**：单例模式保证一个类**只有一个实例**。面试常考 **5 种写法**，推荐**静态内部类**或**枚举**。

| 写法 | 线程安全 | 懒加载 | 推荐？ |
|------|---------|--------|--------|
| 饿汉式 | ✅ | ❌ 类加载就创建 | 简单但浪费资源 |
| 懒汉式（synchronized） | ✅ | ✅ | ❌ 性能差，每次都加锁 |
| **双重检查锁（DCL）** | ✅ | ✅ | ✅ 常考（详见 Java 并发第 5 题 volatile） |
| **静态内部类** | ✅ | ✅ | ✅ **推荐** |
| **枚举** | ✅ | ❌ | ✅ **最推荐**，防反射和序列化破坏 |

```java
// 推荐写法一：静态内部类
public class Singleton {
    private Singleton() {}
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getInstance() {
        return Holder.INSTANCE;  // 调用时才加载内部类
    }
}

// 推荐写法二：枚举（最安全）
public enum Singleton {
    INSTANCE;
}
```

**⚠️ 面试挖坑提醒**：「怎么破坏单例？」—— ① **反射**可以调用私有构造方法（枚举可防）；② **反序列化**会创建新对象（加 `readResolve()` 方法可防）。

---

### 2. 工厂模式有几种？有什么区别？

**一句话总结**：工厂模式把**对象的创建**从业务代码中抽出来，交给工厂类处理。有 **3 种**：简单工厂、工厂方法、抽象工厂。

| 类型 | 原理 | 适用场景 |
|------|------|---------|
| **简单工厂** | 一个工厂类，用 if/switch 创建不同对象 | 产品种类少且固定 |
| **工厂方法** | 每种产品对应一个工厂类（符合开闭原则） | 产品种类可能扩展 |
| **抽象工厂** | 一个工厂生产**一族**相关的产品 | 产品有多个系列（如 Windows 风格组件和 Mac 风格组件） |

```java
// 简单工厂示例
public class AnimalFactory {
    public static Animal create(String type) {
        if ("dog".equals(type)) return new Dog();
        if ("cat".equals(type)) return new Cat();
        throw new IllegalArgumentException("未知类型");
    }
}
```

**背诵口诀**：「**简单工厂一个类搞定，工厂方法一产品一工厂，抽象工厂一族产品一工厂**」

---

## 二、结构型模式

### 3. 代理模式是什么？和 Spring AOP 有什么关系？

**一句话总结**：**代理模式**通过一个代理对象来控制对目标对象的访问，可以在**不修改目标代码**的前提下增强功能。Spring AOP 就是基于**动态代理**实现的（详见 Spring 第 2 题）。

| 类型 | 原理 | 使用场景 |
|------|------|---------|
| **静态代理** | 手动编写代理类，编译期确定 | 代理类少的时候 |
| **JDK 动态代理** | 基于接口，运行时通过 `Proxy.newProxyInstance()` 生成 | 目标类实现了接口 |
| **CGLIB 动态代理** | 基于继承，运行时生成目标类的子类 | 目标类没有接口时使用。**注意：Spring Boot 2.x 起默认 AOP 配置下，常见场景会优先使用 CGLIB 代理**，即使目标类有接口（详见 Spring 第 2 题） |

---

## 三、行为型模式

### 4. 策略模式是什么？

**一句话总结**：把一组算法封装成独立的策略类，客户端可以**根据需要动态切换**算法，避免大量 if-else。

```java
// 定义策略接口
public interface PayStrategy {
    void pay(int amount);
}
// 微信支付策略
public class WechatPay implements PayStrategy {
    public void pay(int amount) { /* 微信支付逻辑 */ }
}
// 支付宝支付策略
public class AliPay implements PayStrategy {
    public void pay(int amount) { /* 支付宝支付逻辑 */ }
}
```

**使用场景**：替代大量 if-else 或 switch-case（如支付方式选择、折扣计算、文件解析策略等）

---

### 5. 观察者模式是什么？

**一句话总结**：**一对多**的依赖关系，当一个对象（被观察者/发布者）状态变化时，所有依赖它的对象（观察者/订阅者）都会**自动收到通知**。

**使用场景**：
- Spring 的**事件机制**（`ApplicationEvent` + `@EventListener`）
- 消息队列的**发布/订阅**模式
- GUI 编程中的**按钮点击事件**

---

### 6. 模板方法模式是什么？

**一句话总结**：在**父类中定义算法的骨架**（执行步骤的顺序），把某些步骤的具体实现**延迟到子类**。

```java
public abstract class AbstractTemplate {
    // 模板方法：定义了算法骨架，final 不可被重写
    public final void execute() {
        step1();
        step2();  // 由子类实现
        step3();
    }
    void step1() { /* 通用实现 */ }
    abstract void step2();  // 子类必须实现
    void step3() { /* 通用实现 */ }
}
```

**使用场景**：Spring 中的 `JdbcTemplate`、`RestTemplate`、`AbstractApplicationContext.refresh()` 等都用了模板方法模式。

---

### 7. 面试中最常问的设计模式有哪些？

**一句话总结**：面试最常问 **6 个**——单例、工厂、代理、策略、观察者、模板方法。知道原理、使用场景和在 Spring 中的应用即可。

| 模式 | 在 Spring/JDK 中的应用 |
|------|----------------------|
| **单例** | Spring Bean 默认是单例 |
| **工厂** | Spring 的 BeanFactory |
| **代理** | Spring AOP（JDK/CGLIB 动态代理） |
| **策略** | Spring 的 Resource 接口（不同资源加载策略） |
| **观察者** | Spring Event（事件发布/监听机制） |
| **模板方法** | JdbcTemplate、RestTemplate |

---

## 复习优先级（3~5 年）

| 优先级 | 题目 |
|--------|------|
| **P0 高频必背** | 1. 单例模式（5 种写法）、2. 工厂模式、3. 代理模式 |
| **P1 中频建议掌握** | 4. 策略模式、5. 观察者模式、6. 模板方法 |
| **P2 低频了解即可** | 7. 设计模式总结 |
