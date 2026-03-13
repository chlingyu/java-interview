# MyBatis

---

### 1. #{} 和 ${} 有什么区别？

**一句话总结**：`#{}` 是**预编译参数**（安全，防 SQL 注入），`${}` 是**字符串拼接**（不安全，有 SQL 注入风险）。

| 对比项 | #{} | ${} |
|--------|-----|-----|
| 处理方式 | **预编译**（PreparedStatement 的 ?） | **字符串拼接**（直接替换） |
| SQL 注入 | ✅ 安全 | ❌ 有风险 |
| 使用场景 | **绝大多数场景** | 动态表名、列名、ORDER BY |

```sql
-- #{} → 预编译：SELECT * FROM user WHERE id = ?
SELECT * FROM user WHERE id = #{id}

-- ${} → 字符串拼接：SELECT * FROM user ORDER BY name
SELECT * FROM user ORDER BY ${column}
```

**背诵口诀**：「**#预编译防注入，$拼接不安全只用于表名列名**」

---

### 2. MyBatis 的一级缓存和二级缓存是什么？

**一句话总结**：**一级缓存**是 SqlSession 级别（默认开启），同一个 SqlSession 中相同查询直接返回缓存。**二级缓存**是 Mapper 级别（需手动开启），多个 SqlSession 可以共享。

| 缓存级别 | 作用范围 | 默认状态 | 失效条件 |
|---------|---------|---------|---------|
| **一级缓存** | **SqlSession** 级别 | ✅ 默认开启 | SqlSession 关闭、执行增删改、手动清除 |
| **二级缓存** | **Mapper（namespace）** 级别 | ❌ 需手动开启 | 执行增删改后自动清除 |

**⚠️ 面试挖坑提醒**：「分布式环境下能用 MyBatis 二级缓存吗？」—— **不推荐**！二级缓存存在本地内存中，多台服务器之间缓存不一致。分布式环境用 **Redis** 做缓存更合适。

---

### 3. MyBatis 的动态 SQL 有哪些标签？

**一句话总结**：MyBatis 通过 XML 标签实现动态 SQL，最常用的有 `<if>`、`<where>`、`<foreach>`、`<choose>`。

| 标签 | 作用 | 示例场景 |
|------|------|---------|
| `<if>` | 条件判断 | 如果 name 不为空就加 WHERE name = ? |
| `<where>` | 自动处理 WHERE 和多余的 AND/OR | 包裹多个 `<if>` 条件 |
| `<foreach>` | 遍历集合 | `WHERE id IN (1, 2, 3)` |
| `<choose>/<when>/<otherwise>` | 类似 switch-case | 多条件择一 |
| `<set>` | 自动处理 UPDATE 时多余的逗号 | 动态更新字段 |
| `<trim>` | 自定义前后缀和去除多余字符 | 灵活拼接 SQL |

---

### 4. MyBatis 的 Mapper 接口是怎么和 XML 绑定的？

**一句话总结**：MyBatis 通过 **JDK 动态代理**（详见设计模式第 3 题）自动为 Mapper 接口生成实现类。接口的**全限定名**对应 XML 的 `namespace`，方法名对应 XML 中的 `id`。

```
接口：com.example.mapper.UserMapper.findById(int id)
  ↓ 对应
XML：<mapper namespace="com.example.mapper.UserMapper">
       <select id="findById">SELECT * FROM user WHERE id = #{id}</select>
     </mapper>
```

---

### 5. MyBatis 的分页是怎么实现的？

**一句话总结**：MyBatis 本身不支持物理分页，可以通过 **PageHelper 插件**（拦截器原理）或**手写 LIMIT** 实现。

| 方式 | 原理 | 推荐？ |
|------|------|--------|
| 手写 LIMIT | 在 SQL 中直接写 `LIMIT #{offset}, #{size}` | 简单场景可以 |
| **PageHelper 插件** | 通过 MyBatis **拦截器（Interceptor）**在 SQL 执行前自动加 LIMIT | ✅ **推荐** |
| RowBounds | MyBatis 内置逻辑分页（内存分页，查出全部再截取） | ❌ 大数据量会 OOM |

---

### 6. MyBatis 的执行流程是怎样的？

**一句话总结**：读取配置 → 创建 SqlSessionFactory → 获取 SqlSession → 通过 Mapper 接口执行 SQL → 返回结果。

```
1. 加载 mybatis-config.xml 和 Mapper XML 配置
2. 创建 SqlSessionFactory（整个应用只有一个）
3. 通过 SqlSessionFactory.openSession() 获取 SqlSession
4. SqlSession.getMapper(UserMapper.class) 获取 Mapper 代理对象
5. 调用 Mapper 方法 → 执行 SQL → 返回结果
6. 提交事务 / 关闭 SqlSession
```

---

### 7. MyBatis 的延迟加载是什么？怎么实现的？

**一句话总结**：延迟加载（懒加载）就是**用到关联数据时才去查**，不用就不查。MyBatis 只支持 `association`（一对一）和 `collection`（一对多）的延迟加载，底层通过 **CGLIB 代理**实现。

| 对比项 | 立即加载 | 延迟加载 |
|--------|---------|---------|
| 查询时机 | 查主表时**同时查关联表** | 访问关联属性时**才去查** |
| SQL 执行 | 一次查完（JOIN 或多条 SQL） | 分多次查（按需触发） |
| 适用场景 | 关联数据一定会用到 | 关联数据不一定用到 |

**实现原理**：
1. MyBatis 用 **CGLIB** 为查询结果对象创建**代理对象**
2. 访问关联属性时（如 `user.getOrders()`），代理对象拦截方法调用
3. 发现关联数据还没加载（为 null），就**触发之前保存好的 SQL** 去查关联表
4. 查回来后设置到对象属性上，返回结果

**开启方式**：在配置中设置 `lazyLoadingEnabled=true`

---

## 复习优先级（3~5 年）

| 优先级 | 题目 |
|--------|------|
| **P0 高频必背** | 1. #{} vs ${}、2. 一级二级缓存、3. 动态 SQL |
| **P1 中频建议掌握** | 4. Mapper 绑定原理、5. 分页实现、7. 延迟加载 |
| **P2 低频了解即可** | 6. 执行流程 |

