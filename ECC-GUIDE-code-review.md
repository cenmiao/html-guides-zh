# ECC 代码审查阶段 - 详细指南

> 本文档详细说明 ECC 在代码审查阶段的使用方法

---

## 核心原则

ECC 在代码审查阶段遵循 **"Review After Every Change"**（每次变更后立即审查）的核心原则。

**阶段目标：**
1. 代码质量 - 可读性、可维护性、一致性
2. 安全漏洞 - OWASP Top 10、注入攻击、认证问题
3. 性能问题 - N+1 查询、内存泄漏、不必要的计算
4. 最佳实践 - 语言特定模式、框架约定

---

## 一、核心审查 Agents

### 1. `code-reviewer` agent - 通用代码审查专家

**用途：** 代码质量和可维护性审查

**触发时机：**
- **每次代码变更后立即审查**
- 写入或修改代码后
- 提交前审查

**审查流程：**

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: 收集上下文                                          │
│     - git diff --staged                                     │
│     - git diff                                              │
│     - git log --oneline -5                                  │
├─────────────────────────────────────────────────────────────┤
│  Step 2: 理解范围                                            │
│     - 识别变更文件                                           │
│     - 理解功能/修复目的                                      │
│     - 追踪依赖关系                                           │
├─────────────────────────────────────────────────────────────┤
│  Step 3: 阅读周边代码                                        │
│     - 读取完整文件                                           │
│     - 理解导入和依赖                                         │
│     - 检查调用点                                             │
├─────────────────────────────────────────────────────────────┤
│  Step 4: 应用审查清单                                        │
│     - CRITICAL → HIGH → MEDIUM → LOW                        │
│     - 只报告 >80% 确信的问题                                 │
├─────────────────────────────────────────────────────────────┤
│  Step 5: 输出发现                                            │
│     - 按严重性组织                                           │
│     - 提供具体修复建议                                        │
└─────────────────────────────────────────────────────────────┘
```

**审查清单：**

#### CRITICAL - 安全

| 问题 | 检测方法 | 修复方案 |
|------|----------|----------|
| 硬编码凭证 | 搜索 API keys, passwords, tokens | 使用 `process.env` |
| SQL 注入 | 字符串拼接查询 | 参数化查询 |
| XSS 漏洞 | 未转义用户输入 | DOMPurify.sanitize() |
| 路径遍历 | 用户控制文件路径 | filepath.Clean + 前缀检查 |
| CSRF 漏洞 | 状态变更无保护 | CSRF token |
| 认证绕过 | 缺少认证检查 | 添加认证中间件 |

```typescript
// 错误: SQL 注入
const query = `SELECT * FROM users WHERE id = ${userId}`;

// 正确: 参数化查询
const query = `SELECT * FROM users WHERE id = $1`;
const result = await db.query(query, [userId]);
```

#### HIGH - 代码质量

| 问题 | 阈值 | 修复方案 |
|------|------|----------|
| 大函数 | >50 行 | 拆分为小函数 |
| 大文件 | >800 行 | 按职责拆分模块 |
| 深嵌套 | >4 层 | 使用 early return |
| 缺少错误处理 | 未处理 Promise | 添加 try/catch |
| 变异模式 | 直接修改对象 | 使用 spread 操作符 |
| console.log | 调试日志 | 移除或使用 logger |
| 缺少测试 | 新代码路径无测试 | 添加测试覆盖 |
| 死代码 | 未使用导入/分支 | 删除 |

```typescript
// 错误: 深嵌套 + 变异
function processUsers(users) {
  if (users) {
    for (const user of users) {
      if (user.active) {
        if (user.email) {
          user.verified = true;  // 变异!
          results.push(user);
        }
      }
    }
  }
  return results;
}

// 正确: Early return + 不可变
function processUsers(users) {
  if (!users) return [];
  return users
    .filter(user => user.active && user.email)
    .map(user => ({ ...user, verified: true }));
}
```

#### HIGH - React/Next.js 模式

| 问题 | 检测 | 修复 |
|------|------|------|
| 缺少依赖数组 | useEffect 无 deps | 添加完整依赖 |
| 渲染中 setState | 导致无限循环 | 移到 effect |
| 列表用索引作 key | key={index} | 使用稳定 ID |
| Prop drilling | 3+ 层传递 | 使用 context |
| 不必要重渲染 | 缺少 memoization | useMemo/useCallback |

```tsx
// 错误: 缺少依赖，闭包陈旧
useEffect(() => {
  fetchData(userId);
}, []); // userId 缺失

// 正确: 完整依赖
useEffect(() => {
  fetchData(userId);
}, [userId]);
```

#### HIGH - Node.js/后端模式

| 问题 | 检测 | 修复 |
|------|------|------|
| 未验证输入 | 请求体无 schema | 使用 zod/joi |
| 缺少限流 | 公开端点无限制 | express-rate-limit |
| 无界查询 | SELECT * 无 LIMIT | 添加分页 |
| N+1 查询 | 循环中查询 | JOIN 或批量 |
| 缺少超时 | 外部 HTTP 调用 | 配置 timeout |
| 错误信息泄露 | 返回内部错误 | 通用错误消息 |

**输出格式：**

```text
[CRITICAL] 硬编码 API key
File: src/api/client.ts:42
Issue: API key "sk-abc..." 暴露在源代码中
Fix: 移动到环境变量并添加到 .gitignore

  const apiKey = "sk-abc123";           // 错误
  const apiKey = process.env.API_KEY;   // 正确

## 审查摘要

| 严重性 | 数量 | 状态 |
|--------|------|------|
| CRITICAL | 0 | pass |
| HIGH | 2 | warn |
| MEDIUM | 3 | info |
| LOW | 1 | note |

结论: WARNING — 2 个 HIGH 问题应在合并前解决。
```

**批准标准：**
- **Approve**: 无 CRITICAL 或 HIGH 问题（包括干净的审查）
- **Warning**: 只有 HIGH 问题（可谨慎合并）
- **Block**: 发现 CRITICAL 问题

---

### 2. `security-reviewer` agent - 安全漏洞专家

**用途：** 识别和修复安全漏洞

**触发时机：**
- 处理用户输入的代码
- 认证/授权相关代码
- API 端点变更
- 敏感数据处理
- 提交前安全检查

**OWASP Top 10 检查：**

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 注入攻击 (Injection)                                     │
│    - SQL 注入: 参数化查询                                    │
│    - 命令注入: 避免用户输入到 shell                          │
│    - LDAP/XPath 注入: 转义输入                               │
├─────────────────────────────────────────────────────────────┤
│ 2. 失效的认证 (Broken Auth)                                  │
│    - 密码哈希: bcrypt/argon2                                │
│    - JWT 验证: 检查过期和签名                                │
│    - 会话安全: httpOnly, Secure, SameSite                    │
├─────────────────────────────────────────────────────────────┤
│ 3. 敏感数据暴露 (Sensitive Data)                             │
│    - HTTPS 强制                                              │
│    - 敏感数据加密                                             │
│    - 日志脱敏                                                │
├─────────────────────────────────────────────────────────────┤
│ 4. XML 外部实体 (XXE)                                        │
│    - 禁用外部实体                                            │
│    - 安全解析器配置                                          │
├─────────────────────────────────────────────────────────────┤
│ 5. 访问控制失效 (Broken Access)                              │
│    - 每个路由检查认证                                        │
│    - CORS 正确配置                                           │
│    - 最小权限原则                                            │
├─────────────────────────────────────────────────────────────┤
│ 6. 安全配置错误 (Misconfiguration)                          │
│    - 默认凭证已更改                                          │
│    - 生产环境 DEBUG 关闭                                     │
│    - 安全头已设置                                            │
├─────────────────────────────────────────────────────────────┤
│ 7. XSS (Cross-Site Scripting)                               │
│    - 输出转义                                                │
│    - CSP 设置                                                │
│    - 框架自动转义                                            │
├─────────────────────────────────────────────────────────────┤
│ 8. 不安全的反序列化                                          │
│    - 安全反序列化用户输入                                    │
│    - 大小/深度限制                                          │
├─────────────────────────────────────────────────────────────┤
│ 9. 已知漏洞组件                                              │
│    - 依赖更新                                                │
│    - npm audit 清洁                                         │
├─────────────────────────────────────────────────────────────┤
│ 10. 日志和监控不足                                           │
│    - 安全事件日志                                            │
│    - 告警配置                                                │
└─────────────────────────────────────────────────────────────┘
```

**立即标记的模式：**

| 模式 | 严重性 | 修复 |
|------|--------|------|
| 硬编码密钥 | CRITICAL | `process.env` |
| 用户输入到 shell 命令 | CRITICAL | 安全 API 或 execFile |
| 字符串拼接 SQL | CRITICAL | 参数化查询 |
| `innerHTML = userInput` | HIGH | textContent 或 DOMPurify |
| `fetch(userProvidedUrl)` | HIGH | 白名单域名 |
| 明文密码比较 | CRITICAL | `bcrypt.compare()` |
| 路由无认证检查 | CRITICAL | 认证中间件 |
| 余额检查无锁 | CRITICAL | `FOR UPDATE` |
| 无限流 | HIGH | express-rate-limit |
| 日志中记录密码/密钥 | MEDIUM | 脱敏日志 |

**常见误报：**
- `.env.example` 中的环境变量（非实际密钥）
- 测试文件中的测试凭证（需明确标记）
- 公开 API 密钥（确实需要公开的）
- 用于校验和的 SHA256/MD5（非密码）

**紧急响应：**
1. 记录详细报告
2. 立即通知项目负责人
3. 提供安全代码示例
4. 验证修复有效
5. 如凭证暴露，立即轮换

---

## 二、语言特定审查 Agents

### 1. `go-reviewer` agent - Go 代码审查

**触发时机：** 所有 Go 代码变更

**审查优先级：**

#### CRITICAL - 安全
- SQL 注入: 字符串拼接查询
- 命令注入: 未验证输入到 `os/exec`
- 路径遍历: 用户控制路径无 `filepath.Clean`
- 竞态条件: 共享状态无同步
- unsafe 包: 无理由使用
- 硬编码密钥: API keys, passwords
- 不安全 TLS: `InsecureSkipVerify: true`

#### CRITICAL - 错误处理
- 忽略错误: 使用 `_` 丢弃错误
- 缺少错误包装: `return err` 无 `fmt.Errorf("context: %w", err)`
- 可恢复错误用 panic: 使用错误返回
- 缺少 errors.Is/As: 使用 `errors.Is(err, target)` 而非 `err == target`

#### HIGH - 并发
- Goroutine 泄漏: 无取消机制（使用 `context.Context`）
- 无缓冲 channel 死锁: 发送无接收
- 缺少 sync.WaitGroup: Goroutines 无协调
- Mutex 误用: 未使用 `defer mu.Unlock()`

#### HIGH - 代码质量
- 大函数: >50 行
- 深嵌套: >4 层
- 非惯用: if/else 而非 early return
- 包级变量: 可变全局状态
- 接口污染: 定义未使用的抽象

#### MEDIUM - 性能
- 循环中字符串拼接: 使用 `strings.Builder`
- 缺少切片预分配: `make([]T, 0, cap)`
- N+1 查询: 循环中数据库查询
- 不必要分配: 热路径中的对象

**诊断命令：**

```bash
go vet ./...
staticcheck ./...
golangci-lint run
go build -race ./...
go test -race ./...
govulncheck ./...
```

---

### 2. `python-reviewer` agent - Python 代码审查

**触发时机：** 所有 Python 代码变更

**审查优先级：**

#### CRITICAL - 安全
- SQL 注入: f-string 查询 → 参数化查询
- 命令注入: 未验证输入到 shell → subprocess 列表参数
- 路径遍历: 用户控制路径 → normpath 验证，拒绝 `..`
- eval/exec 滥用, 不安全反序列化, 硬编码密钥
- 弱加密: 安全用途用 MD5/SHA1, YAML unsafe load

#### CRITICAL - 错误处理
- 裸 except: `except: pass` → 捕获特定异常
- 吞掉异常: 静默失败 → 日志并处理
- 缺少上下文管理器: 手动文件管理 → 使用 `with`

#### HIGH - 类型提示
- 公共函数无类型注解
- 使用 `Any` 当具体类型可行
- 可空参数缺少 `Optional`

#### HIGH - Pythonic 模式
- 使用列表推导而非 C 风格循环
- 使用 `isinstance()` 而非 `type() ==`
- 使用 `Enum` 而非魔法数字
- 使用 `"".join()` 而非循环中字符串拼接
- 可变默认参数: `def f(x=[])` → `def f(x=None)`

#### HIGH - 代码质量
- 函数 >50 行, >5 参数（使用 dataclass）
- 深嵌套 >4 层
- 重复代码模式
- 魔法数字无命名常量

**诊断命令：**

```bash
mypy .                                     # 类型检查
ruff check .                               # 快速 linting
black --check .                            # 格式检查
bandit -r .                                # 安全扫描
pytest --cov=app --cov-report=term-missing # 测试覆盖率
```

---

### 3. `typescript-reviewer` agent - TypeScript/JavaScript 审查

**触发时机：** 所有 TypeScript/JavaScript 代码变更

**审查优先级：**

#### CRITICAL - 安全
- eval/new Function 注入: 用户输入到动态执行
- XSS: 未清理用户输入到 innerHTML, dangerouslySetInnerHTML
- SQL/NoSQL 注入: 字符串拼接查询 → 参数化查询或 ORM
- 路径遍历: 用户输入到 fs.readFile 无 path.resolve + 前缀验证
- 硬编码密钥: API keys, tokens, passwords → 环境变量
- 原型污染: 合并未验证对象无 Object.create(null)
- child_process 用户输入: 验证白名单

#### HIGH - 类型安全
- `any` 无理由: 禁用类型检查 → 使用 `unknown` 并收窄
- 非空断言滥用: `value!` 无前置守卫 → 添加运行时检查
- `as` 转换绕过检查: 转换到不相关类型 → 修复类型
- 放宽编译器设置: tsconfig.json 弱化严格性

#### HIGH - 异步正确性
- 未处理的 Promise 拒绝: async 函数无 await 或 .catch()
- 独立工作顺序 await: 循环中 await → Promise.all
- 浮动 Promise: 事件处理器中无错误处理
- async forEach: `array.forEach(async fn)` 不等待 → for...of 或 Promise.all

#### HIGH - 错误处理
- 吞掉错误: 空 catch 块
- JSON.parse 无 try/catch: 无效输入抛异常
- 抛出非 Error 对象: `throw "message"` → `throw new Error("message")`
- 缺少错误边界: React 树无 ErrorBoundary

**诊断命令：**

```bash
npm run typecheck --if-present
tsc --noEmit -p <relevant-config>
eslint . --ext .ts,.tsx,.js,.jsx
prettier --check .
npm audit
vitest run / jest --ci
```

---

### 4. `java-reviewer` agent - Java/Spring Boot 审查

**触发时机：** 所有 Java 代码变更

**框架检测：**
- 包含 `quarkus` → 应用 QUARKUS 规则
- 包含 `spring-boot` → 应用 SPRING 规则

**审查优先级：**

#### CRITICAL - 安全
- SQL 注入: 字符串拼接 → 绑定参数 (`:param` 或 `?`)
- 命令注入: 用户输入到 ProcessBuilder
- 代码注入: 用户输入到 ScriptEngine.eval()
- 路径遍历: 用户输入到 new File() 无 getCanonicalPath()
- 硬编码密钥: API keys, passwords, tokens
- PII/token 日志: 认证代码附近日志暴露密码
- 缺少输入验证: 请求体无 Bean Validation
- CSRF 无理由禁用

#### CRITICAL - 错误处理
- 吞掉异常: 空 catch 块
- Optional.get() 无 isPresent(): 使用 orElseThrow()
- 缺少集中异常处理: 无 @RestControllerAdvice
- 错误 HTTP 状态: 200 OK 返回 null 而非 404

#### HIGH - 架构
- 依赖注入风格: @Autowired 字段是代码异味 → 构造器注入
- 业务逻辑在控制器: 必须立即委托给服务层
- @Transactional 在错误层: 必须在服务层
- 实体暴露在响应: 直接返回 JPA 实体 → 使用 DTO

#### HIGH - JPA/关系数据库
- N+1 查询: FetchType.EAGER → JOIN FETCH 或 @EntityGraph
- 无界列表端点: 返回 List<T> 无分页
- 缺少 @Modifying: 变更数据 @Query 需要 @Modifying + @Transactional

---

### 5. `rust-reviewer` agent - Rust 代码审查

**触发时机：** 所有 Rust 代码变更

**审查优先级：**

#### CRITICAL - 安全
- 未检查 unwrap()/expect(): 生产代码路径 → 使用 `?` 或显式处理
- unsafe 无理由: 缺少 `// SAFETY:` 注释
- SQL 注入: 字符串插值查询 → 参数化查询
- 命令注入: 未验证输入到 std::process::Command
- 路径遍历: 用户控制路径无规范化
- 硬编码密钥: API keys, passwords, tokens
- 不安全反序列化: 未验证数据无大小/深度限制
- 原始指针释放后使用: 无生命周期保证

#### CRITICAL - 错误处理
- 静默错误: `let _ = result;` 在 #[must_use] 类型
- 缺少错误上下文: `return Err(e)` 无 .context() 或 .map_err()
- 可恢复错误用 panic: panic!(), todo!(), unreachable!()
- 库中 Box<dyn Error>: 使用 thiserror 类型化错误

#### HIGH - 所有权和生命周期
- 不必要克隆: .clone() 满足借用检查器而不理解根因
- String 而非 &str: 当 &str 或 impl AsRef<str> 足够时
- Vec 而非 slice: 当 &[T] 足够时
- 缺少 Cow: 当 Cow<'_, str> 可避免分配时

#### HIGH - 并发
- async 中阻塞: std::thread::sleep, std::fs → 使用 tokio 等效
- 无界 channel: 需要理由 → 优先有界 channel
- Mutex poisoning 忽略: 未处理 .lock() PoisonError
- 缺少 Send/Sync 边界: 跨线程共享类型无适当边界

**诊断命令：**

```bash
cargo clippy -- -D warnings
cargo fmt --check
cargo test
cargo audit
cargo deny check
cargo build --release
```

---

### 6. `cpp-reviewer` agent - C++ 代码审查

**触发时机：** 所有 C++ 代码变更

**审查优先级：**

#### CRITICAL - 内存安全
- 原始 new/delete: 使用 std::unique_ptr 或 std::shared_ptr
- 缓冲区溢出: C 风格数组, strcpy, sprintf 无边界
- 释放后使用: 悬空指针, 失效迭代器
- 未初始化变量: 赋值前读取
- 内存泄漏: 缺少 RAII, 资源未绑定对象生命周期
- 空指针解引用: 指针访问无空检查

#### CRITICAL - 安全
- 命令注入: 未验证输入到 system() 或 popen()
- 格式字符串攻击: 用户输入到 printf 格式字符串
- 整数溢出: 未验证算术运算
- 硬编码密钥: API keys, passwords
- 不安全转换: reinterpret_cast 无理由

#### HIGH - 并发
- 数据竞争: 共享可变状态无同步
- 死锁: 多个 mutex 以不一致顺序锁定
- 缺少锁守卫: 手动 lock()/unlock() → std::lock_guard
- 分离线程: std::thread 无 join() 或 detach()

**诊断命令：**

```bash
clang-tidy --checks='*,-llvmlibc-*' src/*.cpp -- -std=c++17
cppcheck --enable=all --suppress=missingIncludeSystem src/
cmake --build build
```

---

### 7. `database-reviewer` agent - 数据库审查

**触发时机：**
- 编写 SQL
- 创建迁移
- 设计 Schema
- 排查数据库性能

**核心职责：**

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 查询性能                                                  │
│    - WHERE/JOIN 列有索引？                                   │
│    - EXPLAIN ANALYZE 检查 Seq Scan                          │
│    - 避免 N+1 模式                                           │
│    - 复合索引列顺序正确？                                     │
├─────────────────────────────────────────────────────────────┤
│ 2. Schema 设计                                               │
│    - 正确类型: bigint IDs, text strings                     │
│    - timestamptz 而非 timestamp                             │
│    - numeric 而非 float 用于货币                            │
│    - 约束: PK, FK, NOT NULL, CHECK                          │
├─────────────────────────────────────────────────────────────┤
│ 3. 安全                                                      │
│    - 多租户表启用 RLS                                        │
│    - RLS 策略列有索引                                        │
│    - 最小权限访问                                            │
│    - 公共 schema 权限撤销                                    │
├─────────────────────────────────────────────────────────────┤
│ 4. 连接管理                                                  │
│    - 连接池配置                                              │
│    - 超时设置                                                │
│    - 连接限制                                                │
├─────────────────────────────────────────────────────────────┤
│ 5. 并发                                                      │
│    - 防止死锁                                                │
│    - 优化锁定策略                                            │
│    - SKIP LOCKED 用于队列                                    │
└─────────────────────────────────────────────────────────────┘
```

**关键原则：**
- **外键必须有索引** — 无例外
- **使用部分索引** — `WHERE deleted_at IS NULL`
- **覆盖索引** — `INCLUDE (col)` 避免表查找
- **队列用 SKIP LOCKED** — 10x 吞吐量
- **游标分页** — `WHERE id > $last` 而非 OFFSET
- **批量插入** — 多行 INSERT 或 COPY
- **短事务** — 外部 API 调用期间不持有锁

**反模式标记：**
- 生产代码中 `SELECT *`
- ID 用 int（用 bigint）
- 无理由 varchar(255)（用 text）
- timestamp 无时区（用 timestamptz）
- 随机 UUID 作主键（用 UUIDv7 或 IDENTITY）
- 大表 OFFSET 分页
- 未参数化查询（SQL 注入风险）
- 应用用户 GRANT ALL
- RLS 策略每行调用函数（未包装在 SELECT 中）

---

### 8. `django-reviewer` agent - Django 代码审查

**触发时机：** 所有 Django 代码变更

**审查优先级：**

#### CRITICAL - 安全
- SQL 注入: f-string 或 % 格式原始 SQL → %s 参数或 ORM
- 用户输入 mark_safe: 无显式 escape()
- 无理由 CSRF 豁免: @csrf_exempt 非 webhook 视图
- 生产 DEBUG = True: 泄露完整堆栈跟踪
- 硬编码 SECRET_KEY: 必须来自环境变量
- DRF 视图缺少 permission_classes: 默认全局
- 用户输入 eval()/exec(): 立即阻止
- 文件上传无验证: 路径遍历风险

#### CRITICAL - ORM 正确性
- N+1 查询: 循环中访问关联对象无 select_related/prefetch_related
- 多步写入缺少 atomic(): 使用 transaction.atomic()
- bulk_create 无 update_conflicts: 重复键静默数据丢失
- get() 无 DoesNotExist 处理: 未处理异常风险
- delete() 后使用 queryset: 陈旧引用

#### CRITICAL - 迁移安全
- 模型变更无迁移: python manage.py makemigrations --check
- 向后不兼容列删除: 必须两阶段部署
- RunPython 无 reverse_code: 迁移无法回滚
- atomic = False 无理由: 失败时 DB 处于部分状态

#### HIGH - DRF 模式
- Serializer 无显式 fields: fields = '__all__' 暴露敏感列
- 列表端点无分页: 无界查询可返回百万行
- 缺少 read_only_fields: 自动生成字段可被 API 编辑
- 未使用 perform_create: 用户上下文应注入在 perform_create

---

### 9. `kotlin-reviewer` agent - Kotlin/Android/KMP 审查

**触发时机：** 所有 Kotlin 代码变更

**审查优先级：**

#### CRITICAL - 安全
- SQL 注入: 字符串拼接 → 参数化查询
- 硬编码密钥: API keys, passwords
- 不安全 Intent: 未验证组件
- WebView 配置: JavaScriptInterface 风险
- 加密误用: ECB 模式, 硬编码 IV

#### HIGH - Kotlin 惯用模式
- 使用 `val` 而非 `var` 不可变数据
- 数据类用于数据持有
- when 表达式而非 if-else 链
- 扩展函数提高可读性
- 作用域函数正确使用 (let, apply, run, also, takeIf)

#### HIGH - 协程
- 阻塞协程: runBlocking 在生产代码
- 结构化并发: 正确作用域
- 取消处理: 检查 isActive
- 异常处理: CoroutineExceptionHandler

---

## 三、审查 Skills

### 1. `security-review` skill - 安全审查清单

**激活时机：**
- 添加认证
- 处理用户输入
- 创建 API 端点
- 处理密钥或凭证
- 实现支付功能
- 存储或传输敏感数据

**安全检查清单：**

```
┌─────────────────────────────────────────────────────────────┐
│ 部署前安全检查                                                │
├─────────────────────────────────────────────────────────────┤
│ [ ] 密钥: 无硬编码密钥，全部在环境变量                         │
│ [ ] 输入验证: 所有用户输入已验证                              │
│ [ ] SQL 注入: 所有查询参数化                                  │
│ [ ] XSS: 用户内容已清理                                      │
│ [ ] CSRF: 保护已启用                                          │
│ [ ] 认证: 正确 token 处理                                    │
│ [ ] 授权: 角色检查到位                                        │
│ [ ] 限流: 所有端点已启用                                      │
│ [ ] HTTPS: 生产环境强制                                       │
│ [ ] 安全头: CSP, X-Frame-Options 已配置                      │
│ [ ] 错误处理: 错误中无敏感数据                                │
│ [ ] 日志: 无敏感数据日志                                      │
│ [ ] 依赖: 最新，无漏洞                                        │
│ [ ] RLS: Supabase 已启用                                     │
│ [ ] CORS: 正确配置                                            │
│ [ ] 文件上传: 已验证（大小、类型）                             │
│ [ ] 钱包签名: 已验证（如区块链）                               │
└─────────────────────────────────────────────────────────────┘
```

---

### 2. `coding-standards` skill - 编码规范基线

**激活时机：**
- 开始新项目或模块
- 审查代码质量和可维护性
- 重构现有代码遵循约定
- 执行命名、格式或结构一致性

**代码质量原则：**

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 可读性优先                                                │
│    - 代码被阅读次数远多于被编写次数                           │
│    - 清晰的变量和函数名                                       │
│    - 自文档代码优于注释                                       │
│    - 一致的格式                                              │
├─────────────────────────────────────────────────────────────┤
│ 2. KISS (保持简单)                                           │
│    - 最简单的可行方案                                        │
│    - 避免过度工程                                            │
│    - 无过早优化                                              │
│    - 易理解 > 巧妙代码                                       │
├─────────────────────────────────────────────────────────────┤
│ 3. DRY (不要重复自己)                                        │
│    - 提取公共逻辑到函数                                       │
│    - 创建可复用组件                                          │
│    - 跨模块共享工具                                          │
│    - 避免复制粘贴编程                                         │
├─────────────────────────────────────────────────────────────┤
│ 4. YAGNI (你不会需要它)                                      │
│    - 不要提前构建不需要的功能                                 │
│    - 避免推测性泛化                                          │
│    - 仅在需要时添加复杂性                                     │
│    - 从简单开始，需要时重构                                   │
└─────────────────────────────────────────────────────────────┘
```

**不可变模式 (CRITICAL)：**

```typescript
// 正确: 总是使用 spread 操作符
const updatedUser = {
  ...user,
  name: 'New Name'
}

const updatedArray = [...items, newItem]

// 错误: 永远不要直接变异
user.name = 'New Name'  // 错误
items.push(newItem)     // 错误
```

---

### 3. `flutter-dart-code-review` skill - Flutter/Dart 审查

**激活时机：** 所有 Flutter/Dart 代码变更

**审查清单：**

#### Dart 语言陷阱
- 隐式 dynamic: 缺少类型注解 → 启用 strict 设置
- 空安全误用: 过多 `!` 而非正确空检查
- 类型提升失败: 使用 this.field 而非局部变量提升
- 捕获过宽: catch (e) 无 on 子句
- 捕获 Error: Error 子类表示 bug，不应捕获
- 未使用 async: 标记 async 但从未 await
- late 过度使用: 可空或构造函数初始化更安全
- 循环中字符串拼接: 使用 StringBuffer
- 忽略 Future 返回值: 使用 await 或 unawaited()

#### Widget 最佳实践
- 无单个 widget build() 超过 80-100 行
- Widget 按 封装 AND 变化方式 拆分
- 私有 _build*() 方法提取为单独 widget 类
- 无状态 widget 优于有状态
- const 构造函数尽可能使用

#### 状态管理（库无关）
- 业务逻辑在 widget 层外
- 状态管理器通过注入接收依赖
- 服务层抽象数据源
- 状态管理器单一职责
- 不可变状态: copyWith() 创建新实例
- 状态类正确实现 == 和 hashCode

---

## 四、审查工作流程

### 典型审查流程

```
┌─────────────────────────────────────────────────────────────┐
│ 代码变更完成                                                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 1: 通用代码审查                                        │
│  → /code-review → code-reviewer agent                       │
│  → 检查质量、安全、可维护性                                   │
│  → 发现问题 → 修复 → 再次审查                                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 2: 语言特定审查                                        │
│  → 根据技术栈选择:                                            │
│    - Go → go-reviewer                                        │
│    - Python → python-reviewer                               │
│    - TypeScript → typescript-reviewer                        │
│    - Java → java-reviewer                                   │
│    - Rust → rust-reviewer                                   │
│    - C++ → cpp-reviewer                                     │
│    - Kotlin → kotlin-reviewer                               │
│    - Django → django-reviewer                               │
│    - Flutter → flutter-reviewer                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 3: 安全审查 (如需要)                                   │
│  → /security-scan → security-reviewer agent                 │
│  → 涉及用户输入/认证/API 的代码                               │
│  → 检测 OWASP Top 10                                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 4: 数据库审查 (如需要)                                 │
│  → database-reviewer agent                                  │
│  → SQL 变更、迁移、Schema 设计                               │
│  → 查询优化、索引检查                                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 5: 审查通过                                            │
│  → 无 CRITICAL 或 HIGH 问题                                  │
│  → 可以继续下一阶段                                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 五、Agent/Skill 协同关系

| Agent/Skill | 与其他的协同 |
|-------------|--------------|
| `code-reviewer` | 所有审查的入口，发现安全问题转交 security-reviewer |
| `security-reviewer` | code-reviewer 发现安全问题后调用 |
| `go-reviewer` | Go 项目的语言特定审查 |
| `python-reviewer` | Python 项目的语言特定审查 |
| `typescript-reviewer` | TypeScript/JavaScript 项目审查 |
| `java-reviewer` | Java/Spring Boot 项目审查 |
| `rust-reviewer` | Rust 项目审查 |
| `cpp-reviewer` | C++ 项目审查 |
| `kotlin-reviewer` | Kotlin/Android/KMP 项目审查 |
| `django-reviewer` | Django 项目审查，依赖 python-reviewer |
| `database-reviewer` | 数据库层审查，配合后端审查 |
| `security-review` skill | 安全检查清单和模式 |
| `coding-standards` skill | 编码规范基线 |

---

## 六、关键建议总结

| 原则 | 流程 |
|------|------|
| 每次变更后立即审查 | 写完代码 → /code-review → 修复 → 再审查 |
| 安全优先 | 涉及用户输入/认证 → /security-scan |
| 语言特定模式 | 根据技术栈选择对应的 reviewer agent |
| 数据库审查 | SQL 变更 → database-reviewer → 索引/性能 |
| 无问题即通过 | 干净审查是有效结果，不要制造问题 |
| 置信度过滤 | 只报告 >80% 确信的问题 |

---

## 相关链接

- [ECC-GUIDE.md](./ECC-GUIDE.md) - ECC 项目总览
- [ECC-GUIDE-development.md](./ECC-GUIDE-development.md) - 开发阶段指南
- [GitHub 仓库](https://github.com/affaan-m/everything-claude-code)
- [官网](https://ecc.tools)
