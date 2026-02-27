# 11 - Spring

> 覆盖 Spring 全家桶核心知识：IOC、AOP、Bean 生命周期、循环依赖、MVC 执行流程、Boot 自动配置原理等高频面试考点。IOC/AOP 和 Bean 生命周期是面试重灾区。

---

## 一、IOC 与 AOP

### 1. 什么是 IOC？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：IOC（Inversion of Control，控制反转）就是把对象的创建和依赖管理交给 Spring 容器，而不是自己 new。DI（依赖注入）是 IOC 的具体实现方式。

**通俗理解**：

以前你想吃饭，得自己买菜、洗菜、炒菜（自己 new 对象、管理依赖）。现在有了 Spring 这个"食堂"，你只需要说"我要一份红烧肉"，食堂就给你端上来，你不用关心肉从哪买的、怎么做的。

- **没有 IOC** = 你在代码里到处 `new UserService(new UserDao(new DataSource(...)))`，自己管理所有依赖
- **有了 IOC** = 你只需要声明"我需要一个 UserService"，Spring 自动帮你创建好并注入所有依赖

**回到技术**："食堂"就是 Spring 的 **IOC 容器（ApplicationContext）**，它负责创建 Bean、管理 Bean 之间的依赖关系。"说我要什么"就是通过 `@Autowired` 或 `@Resource` 注解声明依赖，Spring 自动注入。

```java
// 没有 IOC：自己管理依赖，耦合严重
public class UserController {
    private UserService userService = new UserServiceImpl(new UserDaoImpl());
}

// 有了 IOC：声明依赖，Spring 自动注入
@RestController
public class UserController {
    @Autowired  // Spring 自动注入，你不用关心 UserService 怎么创建的
    private UserService userService;
}
```

**🎤 面试这样答**：
> "IOC 就是控制反转，把对象的创建和依赖管理从代码中转移到 Spring 容器。以前是程序主动 new 对象，现在是容器创建好对象后注入给你，控制权从程序转移到了容器，所以叫控制反转。DI 依赖注入是 IOC 的实现方式，通过构造器注入、setter 注入或字段注入来完成。IOC 的好处是解耦，对象之间不直接依赖具体实现，方便测试和替换。"

---

### 2. 什么是 AOP？实现原理？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：AOP（Aspect-Oriented Programming，面向切面编程）是把日志、事务、权限等横切关注点从业务代码中抽出来，统一处理。底层用动态代理实现。

**通俗理解**：

你开了一家公司，每个部门（业务方法）干活之前都要打卡、干完之后都要写日报。你不可能让每个部门自己写打卡和日报的代码，而是请一个"行政部"统一处理——所有人进出公司时自动打卡、自动生成日报。

**回到技术**："行政部"就是 AOP 的切面（Aspect），"打卡"就是前置通知（Before），"写日报"就是后置通知（After）。AOP 通过动态代理在不修改原始代码的情况下，给方法"织入"额外逻辑。

**AOP 的核心概念**：

| 概念 | 说明 |
|------|------|
| **切面（Aspect）** | 横切关注点的模块化，比如日志切面、事务切面 |
| **切点（Pointcut）** | 定义在哪些方法上生效（用表达式匹配） |
| **通知（Advice）** | 在切点处执行的逻辑（前置、后置、环绕、异常、最终） |
| **织入（Weaving）** | 把切面应用到目标对象的过程 |

**底层实现——两种动态代理**：

| 代理方式 | 条件 | 原理 |
|---------|------|------|
| **JDK 动态代理** | 目标类⚡**实现了接口** | 基于接口生成代理类 |
| **CGLIB 代理** | 目标类⚡**没有实现接口** | 基于继承，生成目标类的子类 |

> SpringBoot 2.x 之后默认使用 ⚡**CGLIB** 代理（`spring.aop.proxy-target-class=true`）。

**🎤 面试这样答**：
> "AOP 是面向切面编程，把日志、事务、权限校验等横切逻辑从业务代码中抽出来，通过切面统一处理。底层通过动态代理实现：如果目标类实现了接口就用 JDK 动态代理，没有接口就用 CGLIB 生成子类代理。SpringBoot 2.x 之后默认用 CGLIB。常见应用场景有 @Transactional 事务管理、@Aspect 自定义日志切面等。"

---

## 二、Bean

### 3. Spring Bean 的生命周期？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐⭐

**一句话回答**：Bean 的生命周期可以概括为四个阶段——实例化、属性注入、初始化、销毁。中间穿插了各种扩展点（BeanPostProcessor 等）。

**通俗理解**：

把 Bean 的一生想象成一个员工的入职流程：
1. **实例化** = HR 发了 offer，人到公司了（创建对象）
2. **属性注入** = 分配工位、电脑、门禁卡（注入依赖）
3. **初始化** = 入职培训、熟悉业务（执行初始化方法）
4. **使用** = 正式上班干活
5. **销毁** = 离职，交还电脑门禁卡（执行销毁方法）

**回到技术**："人到公司"就是通过反射调用构造方法创建 Bean 实例。"分配工位电脑"就是 Spring 把 `@Autowired` 标注的依赖注入进来。"入职培训"就是执行 `@PostConstruct`、`InitializingBean.afterPropertiesSet()`、`init-method` 等初始化回调。

**原理详解**：

```
Bean 生命周期完整流程

① 实例化（Instantiation）
   → 通过反射调用构造方法创建 Bean 对象
② 属性注入（Populate）
   → 注入 @Autowired、@Value 等依赖
③ Aware 回调
   → 如果实现了 BeanNameAware、ApplicationContextAware 等接口，注入对应资源
④ BeanPostProcessor.postProcessBeforeInitialization()
   → 初始化前的扩展点（@PostConstruct 在这里执行）
⑤ 初始化
   → InitializingBean.afterPropertiesSet() → 自定义 init-method
⑥ BeanPostProcessor.postProcessAfterInitialization()
   → 初始化后的扩展点（⚡ AOP 代理在这里生成）
⑦ 使用
⑧ 销毁
   → @PreDestroy → DisposableBean.destroy() → 自定义 destroy-method
```

> **面试重点**：BeanPostProcessor 是 Spring 最重要的扩展点，AOP 的代理对象就是在 `postProcessAfterInitialization` 阶段生成的。

**🎤 面试这样答**：
> "Bean 的生命周期分四个大阶段：实例化、属性注入、初始化、销毁。具体来说，先通过反射创建对象，然后注入依赖，接着执行 Aware 回调，再经过 BeanPostProcessor 的前置处理，执行 @PostConstruct 和 InitializingBean 的初始化方法，最后经过 BeanPostProcessor 的后置处理，AOP 代理就是在这一步生成的。销毁时依次执行 @PreDestroy 和 DisposableBean 的销毁方法。"

---

### 4. Spring Bean 的作用域有哪些？

> ⭐⭐⭐ 常问 | 难度：⭐⭐

**回答**：Spring 有 5 种 Bean 作用域，最常用的是 singleton 和 prototype。简单说，singleton 是"全公司共用一台打印机"，prototype 是"每个人发一台自己的"。

| 作用域 | 说明 |
|--------|------|
| **singleton** | ⚡**默认**，整个容器只有一个实例 |
| **prototype** | 每次获取都创建一个新实例 |
| request | 每个 HTTP 请求一个实例（Web 环境） |
| session | 每个 HTTP Session 一个实例（Web 环境） |
| application | 每个 ServletContext 一个实例（Web 环境） |

> **面试追问**：singleton 的 Bean 是线程安全的吗？⚡**不是**。Spring 不保证线程安全，如果 Bean 中有可变的成员变量，多线程并发访问会有问题。解决办法：尽量设计成无状态 Bean，或者用 ThreadLocal。

---

### 5. Spring 怎么解决循环依赖？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐⭐

**一句话回答**：Spring 通过三级缓存解决 singleton Bean 的循环依赖——在 Bean 还没完全初始化时，先把一个"半成品"暴露出去，让对方先引用着。

**通俗理解**：

A 和 B 互相依赖，就像两个人互相等对方先自我介绍：
- A 说："我要先认识 B 才能完成入职"
- B 说："我要先认识 A 才能完成入职"
- 死锁了！

Spring 的解决办法：A 先递一张名片（半成品引用）给 B，B 拿着名片就算"认识"A 了，B 完成入职后，A 再拿到完整的 B，自己也完成入职。

**回到技术**："名片"就是 Bean 的早期引用（还没完成属性注入和初始化的对象）。Spring 用三级缓存来存放不同阶段的 Bean：

| 缓存 | 名称 | 存什么 |
|------|------|--------|
| 一级缓存 | `singletonObjects` | ⚡**完整的 Bean**（初始化完毕） |
| 二级缓存 | `earlySingletonObjects` | ⚡**半成品 Bean**（已实例化，未初始化） |
| 三级缓存 | `singletonFactories` | ⚡**Bean 工厂**（ObjectFactory，用于生成早期引用） |

**解决流程（A 和 B 互相依赖）**：

```
① 创建 A：实例化 A → 把 A 的工厂放入三级缓存
② A 注入属性：发现依赖 B → 去创建 B
③ 创建 B：实例化 B → 把 B 的工厂放入三级缓存
④ B 注入属性：发现依赖 A → 从三级缓存拿到 A 的工厂，生成 A 的早期引用，放入二级缓存
⑤ B 初始化完成 → 放入一级缓存
⑥ 回到 A：拿到完整的 B，A 初始化完成 → 放入一级缓存
```

> **为什么需要三级缓存而不是两级？** 因为如果 A 需要被 AOP 代理，三级缓存的工厂可以在需要时生成代理对象，而不是提前生成。这样保证了代理对象的创建时机正确。

> **哪些循环依赖 Spring 解决不了？** ① ⚡**构造器注入**的循环依赖（实例化阶段就需要对方，还没机会放入缓存）；② **prototype** 作用域的循环依赖（不使用缓存）。

**🎤 面试这样答**：
> "Spring 通过三级缓存解决 singleton Bean 的属性注入循环依赖。一级缓存存完整 Bean，二级缓存存半成品 Bean，三级缓存存 Bean 工厂。当 A 依赖 B、B 也依赖 A 时，A 实例化后先把工厂放入三级缓存，然后去创建 B。B 需要 A 时从三级缓存获取 A 的早期引用，B 完成后 A 也能完成。三级缓存的作用是延迟 AOP 代理的创建。但构造器注入和 prototype 的循环依赖无法解决。"

---

## 三、Spring MVC

### 6. Spring MVC 的执行流程？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐

**一句话回答**：请求经过 DispatcherServlet → HandlerMapping → HandlerAdapter → Controller → ViewResolver → 返回响应。

**通俗理解**：

把 Spring MVC 想象成一家餐厅的点餐流程：
1. 客人进门 → **DispatcherServlet**（前台接待，所有请求都先到它这里）
2. 前台查预约表 → **HandlerMapping**（根据 URL 找到对应的 Controller 方法）
3. 前台叫服务员 → **HandlerAdapter**（适配器，负责调用具体的 Controller）
4. 服务员去厨房下单 → **Controller**（执行业务逻辑，返回结果）
5. 厨房做好菜装盘 → **ViewResolver**（视图解析器，渲染页面）
6. 端给客人 → 返回响应

**回到技术**："前台接待"就是 DispatcherServlet，它是 Spring MVC 的核心，所有 HTTP 请求都先经过它。"查预约表"就是 HandlerMapping 根据请求 URL 匹配到对应的 Controller 方法。现在前后端分离项目中，Controller 直接返回 JSON（`@ResponseBody`），不再需要 ViewResolver 渲染页面。

```
请求 → DispatcherServlet
         → HandlerMapping（找到 Controller 方法）
         → HandlerAdapter（调用 Controller）
         → Controller（执行业务，返回数据）
         → HttpMessageConverter（@ResponseBody 时，把对象转成 JSON）
         → 响应
```

**🎤 面试这样答**：
> "Spring MVC 的核心是 DispatcherServlet，所有请求先到它这里。它通过 HandlerMapping 根据 URL 找到对应的 Controller 方法，再通过 HandlerAdapter 调用该方法。Controller 执行完业务逻辑后返回结果，如果是前后端分离项目，通过 HttpMessageConverter 把对象序列化成 JSON 直接返回；如果是传统项目，通过 ViewResolver 解析视图并渲染页面。"

---

## 四、Spring Boot

### 7. SpringBoot 的自动配置原理？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐⭐⭐

**一句话回答**：SpringBoot 通过 `@SpringBootApplication` 中的 `@EnableAutoConfiguration` 注解，加载 `META-INF/spring.factories`（或 Spring Boot 3.x 的 `AutoConfiguration.imports`）中定义的自动配置类，根据条件注解（`@ConditionalOnXxx`）决定是否生效。

**通俗理解**：

以前用 Spring 要写一堆 XML 配置，就像自己组装电脑——CPU、内存、硬盘都要自己选、自己装。SpringBoot 的自动配置就像买品牌整机，开箱即用，它帮你把常用的配置都装好了，你只需要改改不满意的地方。

**回到技术**："帮你装好"就是 SpringBoot 启动时自动加载一堆预定义的配置类。"改不满意的地方"就是你可以在 `application.yml` 中覆盖默认配置，或者自己定义同类型的 Bean 来替代自动配置的。

**原理详解**：

**注解链路**：

```
@SpringBootApplication
  ├── @SpringBootConfiguration  → 标记这是一个配置类
  ├── @ComponentScan            → 扫描当前包及子包的组件
  └── @EnableAutoConfiguration  → ⚡ 核心：开启自动配置
        └── @Import(AutoConfigurationImportSelector.class)
              → 加载 META-INF/spring.factories 中的自动配置类
```

**条件注解**——决定配置类是否生效：

| 注解 | 含义 |
|------|------|
| `@ConditionalOnClass` | 类路径下有某个类才生效 |
| `@ConditionalOnMissingBean` | 容器中没有该 Bean 才生效（⚡用户自定义优先） |
| `@ConditionalOnProperty` | 配置文件中有某个属性才生效 |

**🎤 面试这样答**：
> "SpringBoot 自动配置的核心是 @EnableAutoConfiguration 注解，它通过 AutoConfigurationImportSelector 加载 META-INF/spring.factories 文件中定义的所有自动配置类。这些配置类通过 @ConditionalOnClass、@ConditionalOnMissingBean 等条件注解判断是否生效。比如引入了 spring-boot-starter-web 依赖，类路径下有 DispatcherServlet，对应的 WebMvcAutoConfiguration 就会自动生效。如果用户自己定义了同类型的 Bean，自动配置会让步。"

---

## 五、事务

### 8. Spring 事务失效的场景有哪些？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐

**一句话回答**：Spring 事务基于 AOP 代理实现，凡是绕过代理或不满足代理条件的场景，事务都会失效。

**通俗理解**：

`@Transactional` 的事务是通过 AOP 代理实现的，你可以理解为 Spring 在你的方法外面包了一层"事务外套"。如果调用方式绕过了这层外套，事务就不生效了。

**回到技术**："事务外套"就是 Spring AOP 生成的代理对象，它在目标方法前后添加了开启事务和提交/回滚事务的逻辑。"绕过外套"就是调用没有经过代理对象，比如同类中 `this.methodB()` 是直接调用原始对象的方法，不走代理，所以事务注解不生效。

**常见失效场景**：

| 场景 | 原因 |
|------|------|
| **同类中方法内部调用** | `this.methodB()` 是直接调用，⚡**没走代理**，事务不生效 |
| **方法不是 public** | Spring AOP 默认只代理 public 方法 |
| **异常被 catch 吞掉了** | 事务靠异常触发回滚，异常被捕获了就不会回滚 |
| **抛了非 RuntimeException** | 默认只回滚 `RuntimeException` 和 `Error`，受检异常不回滚 |
| **类没被 Spring 管理** | 没加 `@Service` 等注解，不是 Spring Bean，自然没有代理 |
| **propagation 设置错误** | 比如 `NOT_SUPPORTED` 会挂起当前事务 |

```java
// ❌ 事务失效：同类内部调用，没走代理
@Service
public class UserService {
    public void methodA() {
        this.methodB(); // 直接调用，不走代理，methodB 的事务不生效
    }

    @Transactional
    public void methodB() {
        // 业务逻辑...
    }
}
```

> **解决内部调用问题**：① 注入自身 `@Autowired UserService self`，用 `self.methodB()` 调用；② 使用 `AopContext.currentProxy()` 获取代理对象。

**🎤 面试这样答**：
> "Spring 事务基于 AOP 代理实现，常见失效场景有：同类内部调用没走代理、方法不是 public、异常被 catch 吞掉、抛的是受检异常默认不回滚、类没被 Spring 管理。最常踩的坑是同类内部调用，因为 this 调用不经过代理对象，解决办法是注入自身或用 AopContext 获取代理。"

---

### 9. @Autowired 和 @Resource 的区别？

> ⭐⭐⭐ 常问 | 难度：⭐⭐

**回答**：简单说，`@Autowired` 是 Spring 的注解，默认按类型注入；`@Resource` 是 JDK 的注解，默认按名称注入。

| 对比项 | @Autowired | @Resource |
|--------|-----------|-----------|
| 来源 | ⚡**Spring** 提供 | ⚡**JDK** 提供（`javax.annotation`） |
| 注入方式 | 默认按⚡**类型**（byType） | 默认按⚡**名称**（byName） |
| 找不到时 | 可配合 `@Qualifier` 指定名称 | 按名称找不到再按类型找 |
| 支持位置 | 字段、构造器、setter | 字段、setter（不支持构造器） |

> **怎么选？** 如果项目只用 Spring，两个都行。如果考虑框架无关性（比如未来可能换框架），用 `@Resource`。Spring 官方推荐⚡**构造器注入**，而不是字段注入。

---

### 10. Spring 事务的传播行为有哪些？

> ⭐⭐⭐ 常问 | 难度：⭐⭐⭐

**回答**：事务传播行为定义的是"一个事务方法调用另一个事务方法时，事务怎么传递"。简单说就是：方法 A 有事务，方法 A 调了方法 B，B 是加入 A 的事务，还是自己新开一个？

Spring 定义了 ⚡**7 种**传播行为，但常用的就前三种：

| 传播行为 | 含义 | 通俗理解 |
|---------|------|---------|
| ⚡**REQUIRED**（默认） | 有事务就加入，没有就新建 | "有车搭车，没车自己开" |
| ⚡**REQUIRES_NEW** | 不管有没有，都新建事务 | "不管你开不开车，我自己开" |
| **NESTED** | 有事务就在里面嵌套一个子事务（savepoint） | "搭你的车，但我坐后排，出事我先下车不影响你" |
| SUPPORTS | 有事务就加入，没有就不用事务 | "有车搭车，没车走路" |
| NOT_SUPPORTED | 不管有没有，都不用事务 | "我就走路，不坐车" |
| MANDATORY | 必须在事务中调用，否则抛异常 | "必须有车，没车不出门" |
| NEVER | 必须不在事务中调用，否则抛异常 | "必须走路，有车也不坐" |

```java
@Service
public class OrderService {
    @Autowired
    private LogService logService;

    @Transactional // 默认 REQUIRED
    public void createOrder() {
        // 创建订单...
        logService.saveLog(); // 调用另一个事务方法
    }
}

@Service
public class LogService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void saveLog() {
        // 记录日志：新开事务，即使 createOrder 回滚，日志也不会丢
    }
}
```

> **面试重点**：REQUIRED 和 REQUIRES_NEW 的区别。REQUIRED 是加入调用方的事务，一荣俱荣一损俱损；REQUIRES_NEW 是挂起调用方事务，自己新开一个，互不影响。典型场景：操作日志记录用 REQUIRES_NEW，即使业务回滚，日志也要保留。

---

## 面试高频程度排序（3~5 年）

| 优先级 | 题目 |
|--------|------|
| ⭐⭐⭐⭐⭐ | IOC 是什么？ |
| ⭐⭐⭐⭐⭐ | AOP 原理？ |
| ⭐⭐⭐⭐⭐ | Bean 的生命周期？ |
| ⭐⭐⭐⭐⭐ | Spring 怎么解决循环依赖？ |
| ⭐⭐⭐⭐⭐ | Spring MVC 执行流程？ |
| ⭐⭐⭐⭐⭐ | SpringBoot 自动配置原理？ |
| ⭐⭐⭐⭐ | Spring 事务失效场景？ |
| ⭐⭐⭐ | 事务的传播行为？ |
| ⭐⭐⭐ | Bean 的作用域？ |
| ⭐⭐⭐ | @Autowired 和 @Resource 的区别？ |
