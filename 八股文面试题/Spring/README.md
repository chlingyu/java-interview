# Spring

---

## 一、IoC 与 AOP

### 1. 什么是 IoC？什么是 DI？

**一句话总结**：**IoC**（Inversion of Control，控制反转）是把对象的创建和管理交给 Spring 容器，不再自己 `new`。**DI**（Dependency Injection，依赖注入）是 IoC 的实现方式——容器在创建对象时自动把它依赖的其他对象"注入"进去。

**没有 IoC vs 有 IoC**：

| 传统方式 | IoC 方式 |
|---------|---------|
| 自己 `new` 对象，自己管理依赖 | 告诉 Spring "我需要什么"，Spring 帮你创建和注入 |
| 类之间**强耦合** | 类之间**松耦合** |
| 改一个依赖要改很多地方 | 改一个依赖只需改配置或注解 |

**三种注入方式**：

| 方式 | 写法 | 推荐？ |
|------|------|--------|
| **构造器注入** | `@Autowired` 在构造方法上 | ✅ **推荐**（Spring 官方推荐，对象不可变） |
| **Setter 注入** | `@Autowired` 在 setter 方法上 | 可选依赖时用 |
| **字段注入** | `@Autowired` 直接加在字段上 | ❌ 不推荐（无法做单元测试） |

> **面试话术**：IoC 就是把对象的创建权从我们程序员手里交给了 Spring 容器。以前需要自己 new 对象、管理依赖，现在只需要声明"我需要什么"，Spring 在运行时自动创建并注入。这样做的好处是降低了类之间的耦合度。Spring 推荐使用构造器注入，因为它能保证依赖不可变，也方便做单元测试。

---

### 2. 什么是 AOP？Spring AOP 怎么实现的？

**一句话总结**：**AOP**（Aspect-Oriented Programming，面向切面编程）是把**与业务无关但到处都在用的功能**（如日志、事务、权限校验）抽出来统一处理，避免在业务代码中重复写。Spring AOP 底层用**动态代理**实现。

| 概念 | 含义 | 举例 |
|------|------|------|
| **切面（Aspect）** | 封装横切关注点的模块 | 日志切面、事务切面 |
| **切点（Pointcut）** | 哪些方法需要增强 | `execution(* com.example.service.*.*(..))` |
| **通知（Advice）** | 什么时候增强 | `@Before`（前置）、`@After`（后置）、`@Around`（环绕） |
| **连接点（JoinPoint）** | 可以被增强的点 | 每个方法的执行 |

**Spring AOP 的两种代理方式**：

| 方式 | 条件 | 原理 |
|------|------|------|
| **JDK 动态代理** | 目标类**实现了接口** | 基于 `java.lang.reflect.Proxy`，生成接口的代理类 |
| **CGLIB 动态代理** | 目标类**没有实现接口** | 生成目标类的**子类**作为代理（不能代理 `final` 类和方法） |

**⚠️ 面试挖坑提醒**：Spring Boot 2.x 默认使用 **CGLIB** 代理，即使目标类实现了接口也用 CGLIB。

---

### 3. Spring Bean 的生命周期是怎样的？

**一句话总结**：Bean 的生命周期就是从**创建到销毁**的过程，核心是 4 个阶段：**实例化 → 属性注入 → 初始化 → 销毁**。

**简化版（面试够用）**：

```
1. 实例化（new 出对象）
   ↓
2. 属性注入（@Autowired 等依赖注入）
   ↓
3. Aware 回调（注入容器资源，如 BeanName、ApplicationContext）
   ↓
4. BeanPostProcessor 前置处理（postProcessBeforeInitialization）
   ↓
5. 初始化（@PostConstruct → InitializingBean.afterPropertiesSet() → init-method）
   ↓
6. BeanPostProcessor 后置处理（postProcessAfterInitialization，AOP 代理在这里生成）
   ↓
7. 使用
   ↓
8. 销毁（@PreDestroy → DisposableBean.destroy() → destroy-method）
```

**背诵口诀**：「**实例化 → 注入 → 初始化 → 使用 → 销毁**」（中间穿插 Aware 回调和 BeanPostProcessor）

> **面试话术**：Bean 的生命周期主要分四步。首先通过反射实例化对象，然后进行属性注入也就是依赖注入，接着执行初始化方法如 PostConstruct 注解标记的方法，最后容器关闭时执行销毁方法。在初始化前后还有 BeanPostProcessor 的前置和后置处理，AOP 的代理对象就是在后置处理阶段生成的。

---

### 4. Spring Bean 的作用域有哪些？

**一句话总结**：最常用的是 **singleton**（单例，默认）和 **prototype**（多例），Web 环境下还有 request、session、application 三种。

| 作用域 | 含义 | 创建时机 |
|--------|------|---------|
| **singleton**（默认） | 整个 Spring 容器中**只有一个实例** | 容器启动时 |
| **prototype** | 每次获取都**创建新实例** | 每次 `getBean()` 时 |
| request | 每个 HTTP 请求一个实例 | Web 环境 |
| session | 每个 HTTP 会话一个实例 | Web 环境 |
| application | 整个 Web 应用一个实例 | Web 环境 |

**⚠️ 面试挖坑提醒**：「singleton 的 Bean 是线程安全的吗？」—— **不一定**！如果 Bean 中有**可变的成员变量**（如 `List`、`Map`），多线程操作会有线程安全问题。解决方法：① 不在 Bean 中定义可变状态；② 用 `ThreadLocal`（详见 Java 并发第 10 题）；③ 将 Bean 改为 `prototype` 作用域。

---

## 二、核心机制

### 5. Spring 怎么解决循环依赖？

**一句话总结**：Spring Framework 底层可以通过**三级缓存**解决 **setter 注入**的单例 Bean 循环依赖；但 **Spring Boot 2.6+/3.x 默认禁止循环依赖**，除非显式打开 `spring.main.allow-circular-references=true`。构造器注入的循环依赖**无法解决**。

**三级缓存**：

| 缓存 | 存了什么 | 作用 |
|------|---------|------|
| **一级缓存** `singletonObjects` | 完全初始化好的 Bean | 正式对外使用 |
| **二级缓存** `earlySingletonObjects` | 已实例化但**未完成初始化**的 Bean | 提前暴露"半成品" |
| **三级缓存** `singletonFactories` | 创建 Bean 的工厂方法 | 可以动态决定返回原始对象还是代理对象 |

**解决过程（以 A 依赖 B，B 依赖 A 为例）**：

```
1. 创建 A：实例化 A → 把 A 的工厂放入三级缓存 → 注入属性，发现需要 B
2. 创建 B：实例化 B → 把 B 的工厂放入三级缓存 → 注入属性，发现需要 A
3. 从三级缓存拿到 A 的工厂 → 获取 A 的早期引用 → 放入二级缓存 → B 完成初始化
4. 回到 A → 注入已完成的 B → A 完成初始化 → 放入一级缓存
```

**⚠️ 面试挖坑提醒**：「为什么需要三级缓存而不是两级？」—— 因为第三级缓存存的是**工厂方法**，可以在获取时**动态决定**返回原始对象还是 AOP 代理对象。如果只有两级缓存，就无法在需要时延迟创建代理。

---

### 6. Spring 事务是怎么实现的？有哪些传播行为？

**一句话总结**：Spring 事务底层基于 **AOP** 实现，通过 `@Transactional` 注解在方法执行前后**自动开启/提交/回滚事务**。传播行为定义了方法调用时事务如何传递。

**7 种事务传播行为**：

| 传播行为 | 含义 | 通俗理解 |
|---------|------|---------|
| **REQUIRED**（默认） | 有事务就加入，没有就新建 | "有车坐车，没车自己开" |
| SUPPORTS | 有事务就加入，没有就不用事务 | "有车坐车，没车走路" |
| MANDATORY | 必须在事务中运行，没有就报错 | "没车不出门" |
| **REQUIRES_NEW** | 无论如何都新建事务，挂起当前事务 | "不管有没有车，我都自己开一辆新的" |
| NOT_SUPPORTED | 不用事务，有的话挂起 | "我不坐车，你们先等着" |
| NEVER | 不能在事务中运行，有就报错 | "有车我就不去了" |
| **NESTED** | 有事务就在嵌套事务中执行 | "在大车里坐一辆小车" |

**背诵口诀**：最常考的就三个——「**REQUIRED 默认加入、REQUIRES_NEW 独立新建、NESTED 嵌套回滚**」

---

### 7. Spring 事务在哪些情况下会失效？

**一句话总结**：事务失效的场景都是因为**AOP 代理没有生效**或者**异常没有被正确捕获**。

**常见失效场景**：

| # | 失效场景 | 原因 |
|---|---------|------|
| 1 | **接口代理下方法不是 public** | JDK 接口代理仍要求 `public`；Spring 6 的类代理对 `protected`/包可见方法已放宽，但面试里最稳妥仍写 `public` |
| 2 | **同一个类中方法自调用** | 内部调用走的是 `this.method()`，不经过代理对象 |
| 3 | **异常被 catch 吞掉了** | Spring 事务默认只在**抛出异常**时回滚，catch 了就不会回滚 |
| 4 | **抛的是 checked 异常** | 默认只回滚 `RuntimeException` 和 `Error`，需要 `@Transactional(rollbackFor = Exception.class)` |
| 5 | **Bean 没被 Spring 管理** | 类上没有 `@Service`/`@Component` 等注解 |
| 6 | **数据库引擎不支持事务** | MySQL 的 MyISAM 引擎不支持事务，要用 InnoDB |

**背诵口诀**：「**非 public、自调用、异常被吞、checked 异常、没被管理、引擎不支持**」

---

### 8. SpringMVC 的执行流程是怎样的？

**一句话总结**：请求进来后，经过 **DispatcherServlet**（前端控制器）统一调度，找到对应的 Controller 处理，最后返回视图或 JSON。

**执行流程（7 步）**：

```
1. 用户发送请求 → DispatcherServlet（前端控制器，统一入口）
2. DispatcherServlet → HandlerMapping（处理器映射器）：找到处理请求的 Controller 方法
3. DispatcherServlet → HandlerAdapter（处理器适配器）：调用 Controller 方法
4. Controller 处理业务逻辑，返回 ModelAndView 或 @ResponseBody 数据
5. 如果返回视图 → ViewResolver（视图解析器）解析视图名
6. 渲染视图，填充数据
7. 返回响应给用户
```

> **面试话术**：请求首先到达 DispatcherServlet，它是 SpringMVC 的核心，负责统一调度。DispatcherServlet 通过 HandlerMapping 找到对应的 Controller 方法，再通过 HandlerAdapter 去调用。Controller 处理完业务后返回结果。如果是返回页面就走 ViewResolver 解析视图，如果是 REST 接口标了 @ResponseBody 就直接通过 HttpMessageConverter 转换为 JSON 返回。

---

## 三、Spring Boot

### 9. Spring Boot 的自动配置原理是什么？

**一句话总结**：`@SpringBootApplication` 注解中的 `@EnableAutoConfiguration` 会加载自动配置候选类，再通过**条件注解**（`@ConditionalOnClass` 等）判断是否生效。**Spring Boot 2.x** 常从 `META-INF/spring.factories` 读取候选类，**Spring Boot 3.x** 主流入口改为 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`。

**核心流程**：

```
@SpringBootApplication
  └→ @EnableAutoConfiguration
       └→ Boot 2.x 读取 `spring.factories`；Boot 3.x 主流读取 `AutoConfiguration.imports`
            └→ 每个配置类通过条件注解判断是否生效
                 ├── @ConditionalOnClass：classpath 中有某个类才生效
                 ├── @ConditionalOnMissingBean：容器中没有某个 Bean 才生效
                 └── @ConditionalOnProperty：配置文件中有某个属性才生效
```

> **面试话术**：Spring Boot 自动配置的核心是 `@EnableAutoConfiguration`。它会先加载自动配置候选类，再通过条件注解筛选真正生效的配置。需要注意版本差异：Boot 2.x 常见入口是 `spring.factories`，Boot 3.x 主流入口改成了 `AutoConfiguration.imports`。条件满足才生效，这就是"约定大于配置"。

---

### 10. @Autowired 和 @Resource 有什么区别？

**一句话总结**：`@Autowired` 是 Spring 提供的，**按类型**注入；`@Resource` 是 JDK 提供的（JSR-250 标准），**按名称**注入。

| 对比项 | @Autowired | @Resource |
|--------|-----------|-----------|
| 来源 | Spring 框架 | JDK 标准（`javax.annotation`） |
| 默认注入方式 | **按类型**（byType） | **按名称**（byName） |
| 找不到时 | 报错（可加 `@Qualifier` 指定名称） | 退而求其次按类型找 |
| 是否可以用在构造器上 | ✅ | ❌ |

**⚠️ 面试挖坑提醒**：「一个接口有多个实现类时怎么注入？」—— ① `@Autowired` + `@Qualifier("beanName")` 指定名称；② `@Resource(name = "beanName")` 直接按名称注入；③ `@Primary` 标注优先注入的实现类。

---

### 11. @Component、@Service、@Controller、@Repository 有什么区别？

**一句话总结**：功能上**没有区别**，都是把类注册为 Spring Bean。区别是**语义不同**，方便分层管理。

| 注解 | 语义 | 用在哪一层 |
|------|------|----------|
| `@Component` | 通用组件 | 任何层，不确定时用这个 |
| `@Controller` | 控制器 | **表现层**（处理请求） |
| `@Service` | 服务 | **业务层**（业务逻辑） |
| `@Repository` | 数据访问 | **持久层**（数据库操作） |

**额外功能**：`@Repository` 会自动把数据库异常转换为 Spring 的 `DataAccessException`，其他三个没有额外功能。

---

### 12. Spring Boot 的启动流程是怎样的？

**一句话总结**：从 `main()` 方法调用 `SpringApplication.run()` 开始，经过**创建 SpringApplication 对象 → 准备环境 → 创建并刷新容器 → 自动配置 → 启动完成**。

**简化版启动流程**：

```
1. main() → SpringApplication.run()
2. 创建 SpringApplication 对象（推断应用类型、设置初始化器、设置监听器）
3. 准备环境（加载 application.yml/properties 配置）
4. 创建 ApplicationContext（Spring 容器）
5. 刷新容器（扫描注解、注册 Bean、执行自动配置、依赖注入）
6. 执行 CommandLineRunner / ApplicationRunner（启动后回调）
7. 启动完成，等待请求
```

---

### 13. Filter 和 Interceptor 有什么区别？

**一句话总结**：**Filter**（过滤器）属于 Servlet 规范，在 DispatcherServlet **之前**执行；**Interceptor**（拦截器）属于 Spring MVC，在 DispatcherServlet **之后**、Controller **之前**执行。

| 对比项 | Filter | Interceptor |
|--------|--------|-------------|
| 所属 | **Servlet** 规范 | **Spring MVC** 框架 |
| 执行时机 | DispatcherServlet **之前** | DispatcherServlet **之后**，Controller 之前 |
| 能否获取 Spring Bean | 视注册方式而定：原生 Servlet Filter 不依赖 Spring MVC；在 Spring Boot 中常可作为 Bean 注册 | ✅ 可以 |
| 拦截范围 | **所有请求**（包括静态资源） | 只拦截**Controller 请求** |
| 典型用途 | 字符编码、GZIP 压缩、安全防火墙 | 登录校验、权限检查、日志记录 |
| 核心方法 | `doFilter()` | `preHandle()`、`postHandle()`、`afterCompletion()` |

```
请求 → Filter → DispatcherServlet → Interceptor.preHandle → Controller
                                    Interceptor.postHandle ← Controller
                                    Interceptor.afterCompletion
      ← Filter ← 响应
```

---

## 复习优先级（3~5 年）

| 优先级 | 题目 |
|--------|------|
| **P0 高频必背** | 1. IoC/DI、2. AOP、3. Bean 生命周期、5. 循环依赖、6. 事务传播行为、7. 事务失效、9. 自动配置原理 |
| **P1 中频建议掌握** | 4. Bean 作用域、8. SpringMVC 流程、10. @Autowired vs @Resource、12. 启动流程、13. Filter vs Interceptor |
| **P2 低频了解即可** | 11. @Component 等注解区别 |


