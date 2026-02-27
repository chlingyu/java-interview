# 12 - MyBatis

> 覆盖 MyBatis 核心知识：#{}与${}区别、执行流程、缓存机制、动态 SQL、插件原理等高频面试考点。#{}与${}的区别几乎每场必问。

---

## 一、基础

### 1. #{} 和 ${} 的区别？

> ⭐⭐⭐⭐⭐ 必考 | 难度：⭐⭐

**一句话回答**：`#{}` 是预编译参数占位符，会用 `?` 替代并通过 PreparedStatement 设值，防止 SQL 注入；`${}` 是字符串拼接，直接把值拼到 SQL 里，有 SQL 注入风险。

**通俗理解**：

- `#{}` = 你去银行填表，表格上有固定的格子让你填名字，银行只把你填的内容当作"数据"处理 → ⚡**安全**
- `${}` = 你直接在一张白纸上写转账指令，如果有人在你的名字后面偷偷加了"并转 100 万给我"，银行也会照做 → ⚡**不安全**

**回到技术**："固定格子"就是 PreparedStatement 的参数占位符 `?`，数据库会把传入的值当作纯数据，不会解析成 SQL 语句。"白纸"就是字符串直接拼接到 SQL 中，恶意输入可以改变 SQL 语义。

| 对比项 | #{} | ${} |
|--------|-----|-----|
| 底层实现 | ⚡**PreparedStatement**，参数用 `?` 占位 | Statement，字符串直接拼接 |
| SQL 注入 | ⚡**防止** | ⚠️ 有风险 |
| 使用场景 | 绝大多数场景 | 动态表名、列名、ORDER BY |

```sql
-- #{} 编译后：SELECT * FROM user WHERE name = ?
SELECT * FROM user WHERE name = #{name}

-- ${} 编译后：SELECT * FROM user WHERE name = '张三'
SELECT * FROM user WHERE name = '${name}'
-- 如果传入 ' OR 1=1 --，就变成了：
-- SELECT * FROM user WHERE name = '' OR 1=1 --'  → 查出所有数据！
```

> **什么时候必须用 ${}？** 表名、列名不能用占位符，只能用 `${}`。比如动态排序 `ORDER BY ${column}`。但要在代码层做白名单校验，防止注入。

**🎤 面试这样答**：
> "#{} 和 ${} 的核心区别是：#{} 底层用 PreparedStatement 的参数占位符，会对传入值做预编译处理，能有效防止 SQL 注入；${} 是直接字符串拼接到 SQL 中，有注入风险。日常开发中绝大多数场景用 #{}，只有动态表名、列名、ORDER BY 这种不能用占位符的场景才用 ${}，而且必须在代码层做白名单校验。"

---

### 2. MyBatis 的执行流程？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐

**一句话回答**：读取配置 → 创建 SqlSessionFactory → 获取 SqlSession → 通过 Mapper 代理执行 SQL → 返回结果。

**通俗理解**：

把 MyBatis 执行 SQL 想象成去餐厅点餐：
1. **读菜单**（加载配置文件和 Mapper XML）
2. **开餐厅**（创建 SqlSessionFactory，全局只有一个）
3. **拿号排队**（获取 SqlSession，一次数据库会话）
4. **点菜**（调用 Mapper 接口方法）
5. **厨房做菜**（Executor 执行 SQL）
6. **上菜**（结果映射，返回 Java 对象）

**回到技术**：MyBatis 的核心组件：

| 组件 | 职责 |
|------|------|
| **SqlSessionFactory** | 全局唯一，负责创建 SqlSession |
| **SqlSession** | 一次数据库会话，线程不安全，用完要关闭 |
| **Executor** | SQL 执行器，有 Simple、Reuse、Batch 三种 |
| **MappedStatement** | 封装了一条 SQL 的所有信息（SQL 语句、参数映射、结果映射） |

```
执行流程：
Mapper 接口方法 → MapperProxy（JDK 动态代理）
  → SqlSession → Executor → StatementHandler
  → ParameterHandler（设置参数）→ 执行 SQL
  → ResultSetHandler（结果映射）→ 返回 Java 对象
```

> **Mapper 接口没有实现类，为什么能调用？** MyBatis 用 ⚡**JDK 动态代理**为 Mapper 接口生成代理对象（MapperProxy），调用接口方法时，代理对象根据方法的全限定名找到对应的 MappedStatement，交给 SqlSession 执行。

**🎤 面试这样答**：
> "MyBatis 的执行流程是：先加载配置文件和 Mapper XML 创建 SqlSessionFactory，然后通过它获取 SqlSession。调用 Mapper 接口方法时，实际上是通过 JDK 动态代理生成的 MapperProxy 对象，它根据方法全限定名找到对应的 MappedStatement，交给 Executor 执行 SQL，最后通过 ResultSetHandler 把结果集映射成 Java 对象返回。"

---

## 二、缓存

### 3. MyBatis 的一级缓存和二级缓存？

> ⭐⭐⭐⭐ 高频 | 难度：⭐⭐⭐

**一句话回答**：一级缓存是 SqlSession 级别，默认开启；二级缓存是 Mapper（namespace）级别，需要手动开启。

**通俗理解**：

- **一级缓存** = 你自己的记事本，只有你能看，换个人（换个 SqlSession）就看不到了
- **二级缓存** = 部门的共享文档，同一个部门（同一个 Mapper）的人都能看到

| 对比项 | 一级缓存 | 二级缓存 |
|--------|---------|---------|
| 作用域 | ⚡**SqlSession** 级别 | ⚡**Mapper（namespace）** 级别 |
| 默认状态 | 默认开启 | 需要手动开启 |
| 失效条件 | SqlSession 关闭、执行增删改、手动清除 | 执行增删改会清空该 namespace 的缓存 |

> **生产环境建议**：一级缓存在 Spring 整合 MyBatis 时基本⚡**无效**（每次请求都是新的 SqlSession）。二级缓存在分布式环境下容易出现脏数据，一般⚡**不推荐使用**，缓存交给 Redis 等中间件处理。

**🎤 面试这样答**：
> "MyBatis 有两级缓存。一级缓存是 SqlSession 级别的，默认开启，同一个 SqlSession 内相同的查询会直接返回缓存结果。但在 Spring 中每次请求都是新的 SqlSession，所以一级缓存基本无效。二级缓存是 Mapper 级别的，需要手动开启，同一个 namespace 下的查询可以共享缓存。但二级缓存在分布式环境下容易出现脏数据，生产中一般不用，缓存交给 Redis 处理。"

---

## 三、动态 SQL

### 4. MyBatis 的动态 SQL 有哪些标签？

> ⭐⭐⭐ 常问 | 难度：⭐⭐

**回答**：MyBatis 提供了一组 XML 标签来实现动态 SQL，根据条件拼接不同的 SQL 片段。你可以理解为 SQL 版的 if-else。

| 标签 | 作用 | 示例场景 |
|------|------|---------|
| `<if>` | 条件判断 | 某个字段不为空才加到 WHERE 中 |
| `<where>` | 自动处理 WHERE 和多余的 AND/OR | 多条件查询 |
| `<choose>/<when>/<otherwise>` | 类似 switch-case | 多选一条件 |
| `<foreach>` | 遍历集合 | `IN (1, 2, 3)` 批量查询 |
| `<set>` | 自动处理 SET 和多余的逗号 | 动态更新字段 |

```xml
<!-- 常见用法：动态条件查询 -->
<select id="findUser" resultType="User">
    SELECT * FROM user
    <where>
        <if test="name != null">AND name = #{name}</if>
        <if test="age != null">AND age = #{age}</if>
    </where>
</select>
```

---

## 四、插件

### 5. MyBatis 插件（拦截器）的原理？

> ⭐⭐⭐ 常问 | 难度：⭐⭐⭐

**回答**：MyBatis 插件本质是**拦截器（Interceptor）**，基于 JDK 动态代理实现。你可以理解为在 SQL 执行的关键节点上"埋钩子"，在 SQL 真正执行前后插入自定义逻辑。

MyBatis 允许拦截以下 ⚡**4 个核心对象**的方法：

| 可拦截对象 | 职责 | 常见用途 |
|-----------|------|---------|
| **Executor** | SQL 执行器 | 分页、缓存处理 |
| **StatementHandler** | SQL 语法构建 | SQL 改写、慢 SQL 监控 |
| **ParameterHandler** | 参数处理 | 参数加密 |
| **ResultSetHandler** | 结果集处理 | 结果解密、字段脱敏 |

**实现原理**：MyBatis 在创建这四个对象时，会调用 `interceptorChain.pluginAll(target)`，用 JDK 动态代理层层包装，形成一条责任链。调用方法时，先经过拦截器的 `intercept()` 方法，再决定是否放行执行原方法。

```java
// 自定义插件示例：SQL 执行耗时监控
@Intercepts({
    @Signature(type = StatementHandler.class, method = "query",
               args = {Statement.class, ResultHandler.class})
})
public class SlowSqlPlugin implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = invocation.proceed(); // 执行原方法
        long cost = System.currentTimeMillis() - start;
        if (cost > 1000) {
            // 记录慢 SQL 日志
        }
        return result;
    }
}
```

> **典型应用**：PageHelper 分页插件就是通过拦截 Executor 的 query 方法，在 SQL 执行前自动拼接 `LIMIT` 语句实现分页的。

---

## 面试高频程度排序（3~5 年）

| 优先级 | 题目 |
|--------|------|
| ⭐⭐⭐⭐⭐ | #{} 和 ${} 的区别？ |
| ⭐⭐⭐⭐ | MyBatis 的执行流程？ |
| ⭐⭐⭐⭐ | 一级缓存和二级缓存？ |
| ⭐⭐⭐ | 动态 SQL 有哪些标签？ |
| ⭐⭐⭐ | 插件（拦截器）的原理？ |
