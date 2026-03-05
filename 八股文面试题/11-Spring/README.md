# 11-Spring

---

## 一、IOC 与 Bean 管理
### 1. Spring 的核心特性？IOC 和 AOP 是什么？

**Spring 两大核心：IOC（控制反转）管理对象创建，AOP（面向切面编程）管理横切关注点。**

**IOC（Inversion of Control，控制反转）：**

原来：对象自己 `new` 依赖 → 现在：**Spring 容器**创建对象并注入依赖。「控制」指的是**创建对象的控制权**，从开发者手中**反转**给了 Spring 容器。

```java
// ❌ 传统方式：自己 new
UserService service = new UserService(new UserDaoImpl());

// ✅ IOC 方式：容器注入
@Service
public class UserService {
    @Autowired
    private UserDao userDao; // Spring 自动注入
}
```

**AOP（Aspect Oriented Programming，面向切面编程）：**

把**日志、事务、权限**等与业务无关的通用功能抽取出来，通过**动态代理**织入到业务方法中，避免代码重复。

| AOP 术语 | 含义 |
|---------|------|
| **切面（Aspect）** | 横切关注点的模块化（如日志切面） |
| **通知（Advice）** | 切面的具体行为（前置 Before、后置 After、环绕 Around、返回 AfterReturning、异常 AfterThrowing） |
| **切入点（Pointcut）** | 定义在哪些方法上织入通知 |
| **连接点（JoinPoint）** | 程序执行过程中的某个点（如方法调用） |

**背诵口诀：** IOC 让容器管对象，AOP 让切面管公共逻辑。

> 面试话术：「Spring 核心是 IOC 和 AOP。IOC 把对象的创建和依赖注入交给容器管理，实现解耦。AOP 把日志、事务等横切关注点抽出来，通过动态代理织入业务方法，避免代码侵入。」

### 2. Spring IOC 的实现原理？Bean 的生命周期？

**IOC 容器本质是一个大 Map：key 是 Bean 名称，value 是 Bean 实例。** 底层通过**反射**创建对象、通过**依赖注入**组装对象。

**Bean 的完整生命周期（面试必背）：**

1. **实例化**：通过反射调用构造方法创建对象（`newInstance()`）
2. **属性注入**：通过反射给字段 `set` 值（`@Autowired` 注入依赖）
3. **Aware 接口回调**：如果实现了 `BeanNameAware`、`BeanFactoryAware` 等接口，注入相关资源
4. **BeanPostProcessor 前置处理**：`postProcessBeforeInitialization()`
5. **初始化**：执行 `@PostConstruct` 方法 → `InitializingBean.afterPropertiesSet()` → `init-method`
6. **BeanPostProcessor 后置处理**：`postProcessAfterInitialization()`（AOP 代理在这里生成）
7. **使用**：Bean 就绪，可以被使用
8. **销毁**：容器关闭时执行 `@PreDestroy` → `DisposableBean.destroy()` → `destroy-method`

**背诵口诀：** 实例化 → 注入 → Aware → 前处理 → 初始化 → 后处理 → 使用 → 销毁。记住「实注感前初后用销」。

> 面试话术：「IOC 容器本质是一个 BeanDefinition 注册表加一个 Bean 实例缓存。Bean 生命周期是：反射实例化 → 属性注入 → Aware 回调 → BeanPostProcessor 前置 → 初始化方法 → BeanPostProcessor 后置（这里生成 AOP 代理）→ 使用 → 销毁。」

### 3. Spring 的依赖注入方式有哪些？

**三种注入方式：构造器注入、Setter 注入、字段注入。Spring 官方推荐构造器注入。**

| 方式 | 使用 | 优缺点 |
|------|------|--------|
| **构造器注入**（推荐） | `@Autowired` 在构造方法上 | ✅ 依赖不可变（final）、不会为 null、容易发现循环依赖 |
| **Setter 注入** | `@Autowired` 在 set 方法上 | 可选依赖时适用，但依赖可被修改 |
| **字段注入** | `@Autowired` 直接在字段上 | ⚠️ 最简洁但不推荐：无法 final、难以单元测试 |

```java
// ✅ 推荐：构造器注入
@Service
public class UserService {
    private final UserDao userDao;

    @Autowired // Spring 4.3+ 单构造方法可省略
    public UserService(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

**`@Autowired` vs `@Resource`：**

| 对比项 | `@Autowired` | `@Resource` |
|--------|-------------|------------|
| 来源 | **Spring** 注解 | **JDK** 注解（JSR-250） |
| 匹配方式 | 先按**类型**，再按名称 | 先按**名称**，再按类型 |

**背诵口诀：** 构造器注入最推荐：不可变、不为空、好测试。Autowired 按类型，Resource 按名称。

> 面试话术：「Spring 有三种注入方式：构造器、Setter 和字段注入。官方推荐构造器注入，因为依赖可以声明为 final 不可变，且能在启动时就发现循环依赖。@Autowired 先按类型匹配，@Resource 先按名称匹配。」

### 4. Spring Bean 的作用域有哪些？

**Spring 有 6 种 Bean 作用域，常用的是 `singleton` 和 `prototype`。**

| 作用域 | 含义 | 使用场景 |
|--------|------|---------|
| `singleton`（**默认**） | 整个 IOC 容器中只有**一个**实例 | 无状态的 Service、DAO |
| `prototype` | 每次 `getBean()` 都创建**新实例** | 有状态的 Bean |
| `request` | 每个 **HTTP 请求**一个实例 | Web 应用 |
| `session` | 每个 **HTTP Session** 一个实例 | 用户会话数据 |
| `application` | 每个 **ServletContext** 一个实例 | Web 应用全局 |
| `websocket` | 每个 **WebSocket 会话**一个实例 | WebSocket |

**这题面试经常挖坑：singleton 的 Bean 是线程安全的吗？**

**不是。** Spring 不保证 singleton Bean 的线程安全。如果 Bean 中有**可变的成员变量**，多线程同时访问就可能出问题。解决方法：
- 尽量让 Bean **无状态**（不保存可变实例变量）
- 用 `ThreadLocal` 存线程私有数据
- 改为 `prototype` 作用域

**背诵口诀：** singleton 默认一个实例，prototype 每次新建。singleton 不等于线程安全。

> 面试话术：「Spring 默认 Bean 是 singleton 作用域，整个容器只有一个实例。prototype 每次获取都创建新实例。singleton 的 Bean 不是线程安全的，因为多线程共享同一个实例，如果有可变状态就会有并发问题。」
### 5. Spring 的循环依赖如何解决？

**循环依赖是 A 依赖 B，B 又依赖 A。Spring 通过「三级缓存」解决 singleton Bean 的循环依赖。**

**三级缓存：**

| 缓存 | 变量名 | 存什么 |
|------|--------|--------|
| 一级缓存 | `singletonObjects` | **完整的 Bean**（已初始化完毕） |
| 二级缓存 | `earlySingletonObjects` | **早期暴露的 Bean**（已实例化但未完成属性注入） |
| 三级缓存 | `singletonFactories` | **Bean 工厂**（`ObjectFactory`，用于生成早期引用，可能是代理对象） |

**解决流程（A ↔ B 互相依赖）：**
1. 创建 A：实例化 A → 把 A 的工厂放入**三级缓存**
2. 注入 A 的依赖：发现需要 B → 去创建 B
3. 创建 B：实例化 B → 把 B 的工厂放入三级缓存
4. 注入 B 的依赖：发现需要 A → 从三级缓存拿到 A 的工厂 → 获取 A 的早期引用 → 放入**二级缓存**
5. B 完成初始化 → 放入**一级缓存**
6. 回到 A → 注入 B → A 完成初始化 → 放入一级缓存

**三级缓存存工厂而不是直接存对象的原因：** 如果 A 需要被 AOP 代理，三级缓存的工厂可以在需要时生成**代理对象**而不是原对象，保证注入的是代理后的 Bean。

**哪些循环依赖解决不了？**
- **构造器注入**的循环依赖：实例化阶段就需要依赖，还没来得及放入缓存 → 报错
- **prototype 作用域**：Spring 不缓存 prototype Bean → 报错

**背诵口诀：** 三级缓存：一完整二半成品三工厂。构造器注入和 prototype 的循环依赖解决不了。

> 面试话术：「Spring 通过三级缓存解决 singleton Bean 的字段/Setter 注入循环依赖。一级缓存存完整 Bean，二级存早期引用，三级存 Bean 工厂用于在需要时生成代理对象。构造器注入和 prototype 的循环依赖无法解决。」

## 二、AOP 与事务
### 6. Spring AOP 的实现原理？JDK 动态代理和 CGLIB 的区别？

**Spring AOP 底层通过动态代理实现：有接口用 JDK 动态代理，无接口用 CGLIB。**

| 对比项 | JDK 动态代理 | CGLIB |
|--------|-------------|-------|
| 原理 | 基于**接口**，生成代理类实现同一个接口 | 基于**继承**，生成目标类的子类 |
| 要求 | 目标类**必须实现接口** | 目标类**不能是 final** |
| 性能 | JDK 8+ 性能提升很大，差距不明显 | 生成字节码较慢，但调用速度快 |
| Spring 默认 | Spring Framework 默认有接口用 JDK | **Spring Boot 2.0+ 默认 CGLIB** |

**Spring Boot 2.0+ 为什么默认改用 CGLIB？** 因为 JDK 动态代理需要接口，实际开发中很多类没有接口，统一用 CGLIB 更省心。

**背诵口诀：** 有接口 JDK 代理，无接口 CGLIB 继承。Spring Boot 默认 CGLIB。

> 面试话术：「Spring AOP 通过动态代理实现。JDK 动态代理基于接口，CGLIB 基于继承生成子类。Spring Boot 2.0+ 默认使用 CGLIB。JDK 代理要求目标类实现接口，CGLIB 要求目标类不是 final 的。」

### 7. Spring 的事务传播机制？

**事务传播机制定义了当一个事务方法调用另一个事务方法时，事务如何传播。共 7 种（`@Transactional(propagation = ...)` ）。**

| 传播行为 | 含义 | 一句话 |
|---------|------|--------|
| **`REQUIRED`**（**默认**） | 有事务就加入，没有就新建 | 最常用 |
| `REQUIRES_NEW` | **总是新建**事务，挂起当前事务 | 独立事务，外层回滚不影响内层 |
| `NESTED` | 有事务就新建**嵌套事务**（保存点） | 内层回滚到保存点，外层回滚则全部回滚 |
| `SUPPORTS` | 有事务就加入，没有就以非事务运行 | 可有可无 |
| `NOT_SUPPORTED` | 以非事务运行，有事务就**挂起** | 强制不用事务 |
| `MANDATORY` | 必须在事务中，没事务就**抛异常** | 强制要求事务 |
| `NEVER` | 不能在事务中，有事务就**抛异常** | 强制不能有事务 |

**面试重点记住三个：** REQUIRED、REQUIRES_NEW、NESTED。

**REQUIRES_NEW vs NESTED：**
- **REQUIRES_NEW**：完全独立的新事务，外层事务回滚**不影响**内层
- **NESTED**：嵌套在外层事务中，外层回滚则**内层也回滚**，内层回滚只回到保存点不影响外层

**背诵口诀：** REQUIRED 有就加没就建，REQUIRES_NEW 总新建，NESTED 嵌套回保存点。

> 面试话术：「Spring 有 7 种事务传播机制。REQUIRED 是默认的，有事务就加入否则新建。REQUIRES_NEW 总是创建独立新事务，外层回滚不影响内层。NESTED 是嵌套事务，内层回滚到保存点不影响外层，但外层回滚会连带内层。」

### 8. @Transactional 的实现原理？什么情况下会失效？

**`@Transactional` 底层通过 AOP 动态代理实现：Spring 为标注了该注解的方法生成代理对象，在方法前开启事务，成功则提交，异常则回滚。**

**@Transactional 失效的 8 种场景（必背）：**

| 场景 | 原因 |
|------|------|
| **方法不是 public** | Spring AOP 代理只拦截 public 方法 |
| **自调用**（同类内方法调用） | `this.method()` 走的是原始对象，不经过代理 → 注解无效 |
| **异常被 catch 吞掉** | 事务管理器检测不到异常，以为成功了不会回滚 |
| **抛出的是 checked 异常** | 默认只回滚 `RuntimeException` 和 `Error`，checked 异常不回滚 |
| **数据库引擎不支持事务** | 如 MyISAM 不支持事务 |
| **Bean 没被 Spring 管理** | 没有 `@Service`/`@Component` 等注解，不是 Spring Bean |
| **propagation 设置不当** | 如 `NOT_SUPPORTED` 不使用事务 |
| **多线程** | 事务绑定在 ThreadLocal 上，新线程没有事务上下文 |

**自调用问题如何解决？**
- 注入自身：`@Autowired private UserService self;` 然后 `self.method()`
- 使用 `AopContext.currentProxy()`
- 把被调用方法抽到另一个类中

**rollbackFor 配置：** 建议显式指定 `@Transactional(rollbackFor = Exception.class)`，让 checked 异常也能回滚。

**背诵口诀：** 失效八场景：非 public、自调用、吞异常、checked 异常、不支持事务引擎、不是 Bean、传播设错、多线程。

> 面试话术：「@Transactional 通过 AOP 代理实现。常见失效场景有：非 public 方法、同类内自调用（不走代理）、异常被 catch 吞掉、checked 异常默认不回滚。建议加 rollbackFor = Exception.class，自调用问题可以通过注入自身或抽取到另一个类来解决。」

## 三、MVC 与 Boot
### 9. SpringMVC 的工作流程？

**SpringMVC 处理一个 HTTP 请求的完整流程（面试高频）：**

1. 客户端发送请求到 **`DispatcherServlet`**（前端控制器，所有请求的入口）
2. DispatcherServlet 调用 **`HandlerMapping`**（处理器映射器）根据 URL 找到对应的 Handler（Controller 方法）
3. HandlerMapping 返回 **`HandlerExecutionChain`**（包含 Handler 和拦截器链）
4. DispatcherServlet 调用 **`HandlerAdapter`**（处理器适配器）执行 Handler
5. Handler（Controller）执行业务逻辑，返回 **`ModelAndView`**
6. DispatcherServlet 调用 **`ViewResolver`**（视图解析器）解析视图
7. ViewResolver 返回 **`View`** 对象
8. DispatcherServlet 渲染视图，返回响应给客户端

**前后端分离场景下（@RestController）：** 不走 ViewResolver，直接通过 `HttpMessageConverter` 把返回对象序列化成 JSON 返回。

**核心组件：**

| 组件 | 作用 |
|------|------|
| `DispatcherServlet` | 前端控制器，所有请求入口 |
| `HandlerMapping` | 根据 URL 找到对应的 Controller 方法 |
| `HandlerAdapter` | 适配并执行 Controller 方法 |
| `ViewResolver` | 解析视图名称为 View 对象 |

**背诵口诀：** 请求进 Dispatcher → Mapping 找 Handler → Adapter 执行 → ViewResolver 渲染。

> 面试话术：「SpringMVC 的核心是 DispatcherServlet，它接收所有请求，通过 HandlerMapping 找到对应的 Controller 方法，用 HandlerAdapter 执行，返回 ModelAndView 后通过 ViewResolver 渲染视图。前后端分离场景下用 @RestController，直接返回 JSON 不走视图解析。」

### 10. Spring Boot 的自动配置原理？

**Spring Boot 自动配置核心是 `@EnableAutoConfiguration` 注解 + `spring.factories` 文件（Spring Boot 2.x）/ `AutoConfiguration.imports` 文件（Spring Boot 3.x）。**

**自动配置流程：**
1. `@SpringBootApplication` 包含 `@EnableAutoConfiguration`
2. `@EnableAutoConfiguration` 通过 `@Import(AutoConfigurationImportSelector.class)` 导入自动配置类
3. `AutoConfigurationImportSelector` 从 `META-INF/spring.factories`（2.x）或 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`（3.x）文件中读取所有自动配置类
4. 通过 **`@Conditional` 系列注解**筛选：满足条件的配置类才生效

**核心条件注解：**

| 注解 | 条件 |
|------|------|
| `@ConditionalOnClass` | classpath 下**存在**指定的类才生效 |
| `@ConditionalOnMissingBean` | 容器中**没有**指定 Bean 才生效（用户自定义优先） |
| `@ConditionalOnProperty` | 配置文件中**存在**指定属性才生效 |

```java
// 示例：DataSourceAutoConfiguration
@Configuration
@ConditionalOnClass(DataSource.class)  // classpath 有 DataSource 类才配置
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration { ... }
```

**背诵口诀：** @SpringBootApplication → @EnableAutoConfiguration → 读 spring.factories → @Conditional 筛选。

> 面试话术：「Spring Boot 自动配置通过 @EnableAutoConfiguration 触发，从 spring.factories 文件加载所有自动配置类，再通过 @ConditionalOnClass、@ConditionalOnMissingBean 等条件注解判断是否生效。用户自定义的 Bean 优先于自动配置，因为有 @ConditionalOnMissingBean 兜底。」

### 11. Spring Boot 和 Spring 的区别？

一句话：**Spring Boot 是 Spring 的脚手架，简化配置、开箱即用。**

| 对比项 | Spring | Spring Boot |
|--------|--------|-------------|
| 配置方式 | 大量 XML / Java Config | **约定大于配置**，自动配置 |
| 依赖管理 | 手动管理版本 | **starter** 一键引入，版本自动管理 |
| 内嵌服务器 | 需要外部 Tomcat | **内嵌 Tomcat/Jetty**，打 jar 包直接运行 |
| 启动方式 | 部署到外部容器 | `java -jar` 直接启动 |
| 监控 | 需要自己搭 | **Actuator** 内置监控端点 |

**Spring Boot Starter 是什么？** Starter 是一组依赖的集合，引入一个 starter 就自动引入所有相关依赖。如 `spring-boot-starter-web` 包含了 Spring MVC、Tomcat、Jackson 等。

**背诵口诀：** Spring Boot = Spring + 自动配置 + 内嵌服务器 + Starter 依赖。

> 面试话术：「Spring Boot 是基于 Spring 的快速开发框架，核心优势是自动配置、内嵌服务器和 Starter 依赖管理。开发者不需要写大量 XML 配置，引入 starter 即可使用。Spring Boot 并没有引入新技术，只是简化了 Spring 的使用方式。」

## 四、注解
### 12. Spring 的常用注解？

**按功能分类：**

| 分类 | 注解 | 作用 |
|------|------|------|
| **Bean 定义** | `@Component` | 通用组件 |
| | `@Service` | 业务层 |
| | `@Repository` | 数据访问层 |
| | `@Controller` / `@RestController` | 控制层（@RestController = @Controller + @ResponseBody） |
| **依赖注入** | `@Autowired` | 按类型注入 |
| | `@Qualifier` | 配合 @Autowired，按名称指定 Bean |
| | `@Resource` | 按名称注入（JDK 注解） |
| | `@Value` | 注入配置值 |
| **配置** | `@Configuration` | 标记配置类 |
| | `@Bean` | 方法返回值注册为 Bean |
| | `@ComponentScan` | 指定包扫描路径 |
| **Web** | `@RequestMapping` / `@GetMapping` / `@PostMapping` | URL 映射 |
| | `@RequestBody` | 接收 JSON 请求体 |
| | `@PathVariable` | 获取 URL 路径变量 |
| | `@RequestParam` | 获取查询参数 |
| **AOP/事务** | `@Aspect` | 标记切面类 |
| | `@Transactional` | 声明式事务 |
| **条件** | `@Conditional` | 条件装配 |
| | `@Profile` | 环境（dev/test/prod）条件 |

**背诵口诀：** Bean 四注解（Component/Service/Repository/Controller），注入三注解（Autowired/Resource/Value），配置三注解（Configuration/Bean/ComponentScan）。

> 面试话术：「Spring 常用注解按功能分：Bean 定义用 @Component/@Service/@Repository/@Controller，依赖注入用 @Autowired/@Resource，配置用 @Configuration/@Bean，Web 用 @RestController/@RequestMapping/@RequestBody，事务用 @Transactional。」

## 复习优先级（3~5 年）
| 优先级 | 题目 |
|--------|------|
| P0 | 1. Spring 的核心特性？IOC 和 AOP 是什么？ |
| P0 | 2. Spring IOC 的实现原理？Bean 的生命周期？ |
| P0 | 5. Spring 的循环依赖如何解决？ |
| P0 | 6. Spring AOP 的实现原理？JDK 动态代理和 CGLIB 的区别？ |
| P0 | 7. Spring 的事务传播机制？ |
| P0 | 8. @Transactional 的实现原理？什么情况下会失效？ |
| P0 | 10. Spring Boot 的自动配置原理？ |
| P1 | 3. Spring 的依赖注入方式有哪些？ |
| P1 | 4. Spring Bean 的作用域有哪些？ |
| P1 | 9. SpringMVC 的工作流程？ |
| P1 | 11. Spring Boot 和 Spring 的区别？ |
| P2 | 12. Spring 的常用注解？ |