# 19 - 设计模式

> 覆盖设计模式核心知识：单例、工厂、代理、策略、观察者等高频模式，结合 Spring 源码中的实际应用。单例模式几乎每场必问。

---

## 一、创建型

### 1. 单例模式怎么实现？有哪些写法？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：单例模式保证一个类只有一个实例。常见写法有饿汉式、懒汉式（双重检查锁）、静态内部类、枚举。面试最爱问双重检查锁为什么要用 volatile。

**通俗理解**：

单例就像一个国家只有一个总统。不管谁去找总统办事，找到的都是同一个人，不会凭空冒出第二个。

**回到技术**："只有一个总统"就是全局只有一个实例对象。"不管谁去找"就是任何线程调用 `getInstance()` 拿到的都是同一个对象。关键问题是怎么在多线程环境下保证只创建一次。

**① 饿汉式**（类加载时就创建，简单安全）：

```java
public class Singleton {
    // 类加载时就创建实例，线程安全
    private static final Singleton INSTANCE = new Singleton();
    private Singleton() {} // 私有构造方法
    public static Singleton getInstance() { return INSTANCE; }
}
```

> 优点：简单，线程安全。缺点：不管用不用都会创建，浪费内存（实际上大部分场景这点内存可以忽略）。

**② 双重检查锁（DCL）**（面试最爱问）：

```java
public class Singleton {
    // ⚡ 必须加 volatile，防止指令重排序
    private static volatile Singleton instance;
    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {              // 第一次检查：避免不必要的加锁
            synchronized (Singleton.class) {
                if (instance == null) {      // 第二次检查：防止重复创建
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

> **为什么要加 volatile？** `new Singleton()` 不是原子操作，JVM 实际执行三步：① 分配内存 ② 初始化对象 ③ 把引用指向内存地址。指令重排序可能导致顺序变成 ①③②，线程 A 执行到 ③ 时，线程 B 第一次检查 `instance != null` 就直接返回了一个⚡**还没初始化完的对象**。volatile 禁止重排序，解决这个问题。

**③ 静态内部类**（推荐写法）：

```java
public class Singleton {
    private Singleton() {}
    // 静态内部类在第一次使用时才加载，天然线程安全
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }
    public static Singleton getInstance() { return Holder.INSTANCE; }
}
```

**④ 枚举**（最简洁，防反射和序列化破坏）：

```java
public enum Singleton {
    INSTANCE; // 天然单例，线程安全，防反射
    public void doSomething() { /* 业务方法 */ }
}
```

> **Spring 中的单例**：Spring Bean 默认就是单例的（`@Scope("singleton")`），但它不是通过上面的写法实现的，而是通过⚡**三级缓存 + ConcurrentHashMap** 来管理单例 Bean。

**🎤 面试这样答**：
> "单例模式常见写法有四种：饿汉式类加载时就创建，简单安全；双重检查锁懒加载，要加 volatile 防止指令重排导致拿到未初始化的对象；静态内部类利用类加载机制实现懒加载，推荐使用；枚举最简洁，还能防反射和序列化破坏。实际开发中 Spring Bean 默认就是单例的。"

---

### 2. 工厂模式？简单工厂、工厂方法、抽象工厂的区别？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐

**一句话回答**：工厂模式把对象的创建逻辑封装起来，调用方不需要知道具体怎么创建。简单工厂用一个工厂类搞定，工厂方法每个产品一个工厂，抽象工厂生产一族产品。

**通俗理解**：

- **简单工厂** = 一家奶茶店，你说要"珍珠奶茶"还是"柠檬茶"，店员根据你说的做对应的饮品。所有饮品都在一家店做 → 一个工厂类，用 if/switch 判断
- **工厂方法** = 奶茶连锁品牌，珍珠奶茶有专门的珍珠奶茶店，柠檬茶有专门的柠檬茶店。新增一种饮品就开一家新店 → 每个产品一个工厂类
- **抽象工厂** = 一家饮品集团，旗下有"夏日系列"和"冬日系列"，每个系列包含奶茶、果汁、咖啡一整套产品 → 一个工厂生产一族相关产品

**回到技术**："店员根据你说的做饮品"就是简单工厂里的 `create(type)` 方法，用 if/switch 判断创建哪个对象。"每种饮品开一家店"就是工厂方法模式，每个具体工厂只负责创建一种产品，新增产品只需新增工厂类，符合开闭原则。

```java
// 简单工厂（最常用，Spring 的 BeanFactory 就是工厂模式的体现）
public class PayFactory {
    public static Pay create(String type) {
        if ("alipay".equals(type)) return new AliPay();
        if ("wechat".equals(type)) return new WechatPay();
        throw new IllegalArgumentException("不支持的支付方式");
    }
}
// 使用：Pay pay = PayFactory.create("alipay");
```

| 模式 | 特点 | 适用场景 |
|------|------|---------|
| **简单工厂** | 一个工厂类，if/switch 判断 | 产品种类少且固定 |
| **工厂方法** | 每个产品一个工厂，符合开闭原则 | 产品种类可能扩展 |
| **抽象工厂** | 一个工厂生产一族产品 | 产品有多个维度的组合 |

> **Spring 中的工厂模式**：`BeanFactory` 是 Spring 的核心工厂接口，`ApplicationContext` 是它的增强版。`FactoryBean` 可以自定义 Bean 的创建逻辑。

**🎤 面试这样答**：
> "工厂模式的核心是把对象创建逻辑封装起来，调用方不直接 new 对象。简单工厂用一个类的静态方法根据参数创建不同对象，简单但不符合开闭原则；工厂方法每个产品对应一个工厂类，新增产品只需新增工厂，符合开闭原则；抽象工厂用于创建一族相关产品。实际开发中简单工厂最常用，Spring 的 BeanFactory 就是工厂模式的典型应用。"

---

## 二、结构型

### 3. 代理模式？静态代理和动态代理的区别？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：代理模式是给目标对象提供一个代理对象，通过代理对象控制对目标对象的访问，可以在不修改目标对象的前提下增强功能。Spring AOP 的底层就是动态代理。

**通俗理解**：

代理就像明星的经纪人。粉丝想找明星拍广告，不能直接找明星本人，要先找经纪人。经纪人可以在接活之前谈价格（前置增强），拍完之后收尾款（后置增强），明星只管拍广告（核心业务）。

**回到技术**："经纪人"就是代理对象，"明星"就是目标对象，"谈价格/收尾款"就是代理在目标方法前后添加的增强逻辑（日志、事务、权限校验等）。

| 对比项 | 静态代理 | JDK 动态代理 | CGLIB 动态代理 |
|--------|---------|-------------|---------------|
| 实现方式 | 手动编写代理类 | ⚡**反射 + Proxy.newProxyInstance** | ⚡**字节码生成子类** |
| 要求 | 代理类和目标类实现同一接口 | 目标类必须实现接口 | 目标类不能是 final |
| 性能 | 编译期确定，快 | 反射调用，稍慢 | 生成子类，首次慢，后续快 |
| 使用场景 | 很少用 | Spring AOP 默认（有接口时） | Spring AOP 默认（无接口时） |

```java
// JDK 动态代理示例
public class LogProxy implements InvocationHandler {
    private Object target; // 目标对象

    public LogProxy(Object target) { this.target = target; }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("方法执行前：记录日志");  // 前置增强
        Object result = method.invoke(target, args); // 调用目标方法
        System.out.println("方法执行后：记录日志");  // 后置增强
        return result;
    }
}

// 创建代理对象
UserService proxy = (UserService) Proxy.newProxyInstance(
    target.getClass().getClassLoader(),
    target.getClass().getInterfaces(),
    new LogProxy(target)
);
```

> **Spring AOP 的选择策略**：目标类实现了接口 → 用 JDK 动态代理；没实现接口 → 用 CGLIB。Spring Boot 2.x 之后默认都用 CGLIB（可通过配置切换）。

**🎤 面试这样答**：
> "代理模式通过代理对象控制对目标对象的访问，可以在不修改目标代码的前提下增强功能。静态代理需要手写代理类，不灵活。动态代理在运行时生成代理类，JDK 动态代理基于反射和接口，要求目标类实现接口；CGLIB 通过生成目标类的子类实现，不需要接口但目标类不能是 final。Spring AOP 底层就是动态代理，有接口用 JDK 代理，没接口用 CGLIB。"

---

## 三、行为型

### 4. 策略模式？怎么消除 if-else？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐

**一句话回答**：策略模式把一组算法封装成独立的策略类，运行时动态选择使用哪个策略，避免大量 if-else 判断。

**通俗理解**：

出门上班可以坐地铁、打车、骑车，每种方式就是一个"策略"。你不需要在脑子里写一堆 if（下雨就打车、不远就骑车……），而是把每种出行方式封装好，根据情况直接切换。

**回到技术**："每种出行方式"就是一个策略实现类，它们都实现同一个策略接口。调用方持有策略接口的引用，运行时注入不同的实现即可切换行为。

```java
// 策略接口
public interface PayStrategy {
    void pay(BigDecimal amount);
}

// 具体策略
@Component("alipay")
public class AliPayStrategy implements PayStrategy {
    public void pay(BigDecimal amount) { /* 支付宝支付 */ }
}

@Component("wechat")
public class WechatPayStrategy implements PayStrategy {
    public void pay(BigDecimal amount) { /* 微信支付 */ }
}
```

```java
// Spring 中用 Map 自动注入所有策略，彻底消除 if-else
@Service
public class PayService {
    // Spring 会自动把所有 PayStrategy 实现注入到 Map 中，key 是 Bean 名称
    @Autowired
    private Map<String, PayStrategy> strategyMap;

    public void pay(String type, BigDecimal amount) {
        PayStrategy strategy = strategyMap.get(type); // 根据类型获取策略
        if (strategy == null) throw new IllegalArgumentException("不支持的支付方式");
        strategy.pay(amount);
    }
}
```

> **策略模式在实际项目中非常常用**：支付方式选择、优惠券计算、文件导出格式（Excel/CSV/PDF）等场景都适合用策略模式消除 if-else。

**🎤 面试这样答**：
> "策略模式把一组算法封装成独立的策略类，它们实现同一个接口，调用方持有接口引用，运行时动态切换实现。最典型的应用是消除 if-else，比如支付方式选择，每种支付方式是一个策略实现类。在 Spring 中可以用 Map 自动注入所有策略实现，根据类型 key 直接获取对应策略，完全不需要 if-else 判断。"

---

### 5. 观察者模式？Spring 中的事件机制？

> ⭐⭐⭐ 常问 | 难度：⭐⭐

**回答**：观察者模式定义了一对多的依赖关系，当一个对象状态变化时，所有依赖它的对象都会收到通知并自动更新。简单说就是"发布-订阅"，你关注了一个公众号，公众号发文章你就能收到推送。

Spring 的事件机制就是观察者模式的实现：

| 角色 | Spring 对应 |
|------|-----------|
| 事件（消息） | `ApplicationEvent` |
| 发布者 | `ApplicationEventPublisher` |
| 监听者（观察者） | `@EventListener` 注解的方法 |

```java
// 自定义事件
public class OrderCreatedEvent extends ApplicationEvent {
    private Long orderId;
    public OrderCreatedEvent(Object source, Long orderId) {
        super(source);
        this.orderId = orderId;
    }
}

// 发布事件
@Service
public class OrderService {
    @Autowired
    private ApplicationEventPublisher publisher;

    public void createOrder(Order order) {
        // ... 创建订单逻辑
        publisher.publishEvent(new OrderCreatedEvent(this, order.getId()));
    }
}

// 监听事件（可以有多个监听者，互不影响）
@Component
public class SmsListener {
    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        // 发送短信通知
    }
}
```

> **适用场景**：下单后发短信、发邮件、扣库存等操作，用事件机制解耦，主流程不需要关心有多少个后续操作。

---

### 6. 模板方法模式？

> ⭐⭐⭐ 常问 | 难度：⭐⭐

**回答**：模板方法模式在父类中定义算法的骨架（流程步骤），把某些步骤延迟到子类实现。简单说就是"流程固定，细节可变"——父类定好 1234 步，子类只需要重写其中某几步的具体实现。

```java
// 模板方法：父类定义流程骨架
public abstract class AbstractExporter {
    // 模板方法，定义导出流程（final 防止子类修改流程）
    public final void export(List<Data> dataList) {
        List<Data> data = queryData();   // 第一步：查数据（通用）
        byte[] bytes = doExport(data);   // 第二步：导出（子类实现）
        upload(bytes);                   // 第三步：上传（通用）
    }

    protected abstract byte[] doExport(List<Data> data); // 子类实现具体导出逻辑
}

// 子类只需实现差异化的步骤
public class ExcelExporter extends AbstractExporter {
    protected byte[] doExport(List<Data> data) { /* 导出为 Excel */ }
}

public class CsvExporter extends AbstractExporter {
    protected byte[] doExport(List<Data> data) { /* 导出为 CSV */ }
}
```

> **Spring 中的模板方法**：`JdbcTemplate`、`RedisTemplate`、`RestTemplate` 都是模板方法模式的应用，封装了通用流程（获取连接、执行、释放资源），用户只需关注核心逻辑。

---

## 面试高频程度排序（3~5 年）

| 优先级 | 题目 |
|--------|------|
| ⭐⭐⭐⭐⭐ | 单例模式怎么实现？DCL 为什么要 volatile？ |
| ⭐⭐⭐⭐⭐ | 代理模式？JDK 动态代理和 CGLIB 的区别？ |
| ⭐⭐⭐⭐ | 工厂模式？简单工厂、工厂方法、抽象工厂？ |
| ⭐⭐⭐⭐ | 策略模式？怎么消除 if-else？ |
| ⭐⭐⭐ | 观察者模式？Spring 事件机制？ |
| ⭐⭐⭐ | 模板方法模式？ |