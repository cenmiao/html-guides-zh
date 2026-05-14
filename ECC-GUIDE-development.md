# ECC 开发阶段 - 详细指南

> 本文档详细说明 ECC 在开发阶段的使用方法

---

## 核心原则

ECC 在开发阶段遵循 **"Tests Before Code"**（测试先行）的核心原则。

**阶段目标：**
1. 测试驱动开发 - 先写测试，再写代码
2. 编码规范 - 遵循语言特定的最佳实践
3. 代码质量 - 保持简洁、可读、可维护
4. 覆盖率保证 - 最低 80% 测试覆盖率

---

## 一、通用开发原则

### 1. 核心原则

| 原则 | 说明 |
|------|------|
| **Readability First** | 代码被阅读的次数远多于被编写的次数 |
| **KISS** | 保持简单，避免过度工程化 |
| **DRY** | 不要重复自己，提取公共逻辑 |
| **YAGNI** | 不要提前实现不需要的功能 |

### 2. 代码质量标准

```
┌─────────────────────────────────────────────────────────────┐
│ 代码质量检查清单                                              │
├─────────────────────────────────────────────────────────────┤
│ ✓ 函数长度 < 50 行                                           │
│ ✓ 嵌套深度 < 5 层（使用 early return）                        │
│ ✓ 无魔法数字（使用命名常量）                                   │
│ ✓ 有意义的命名（描述性变量名）                                 │
│ ✓ 单一职责（每个函数只做一件事）                               │
│ ✓ 无重复代码                                                  │
│ ✓ 适当的注释（解释 WHY，不是 WHAT）                           │
└─────────────────────────────────────────────────────────────┘
```

---

## 二、TDD 工作流 (tdd-workflow skill)

### 核心原则

**"Tests BEFORE Code - ALWAYS write tests first"**

**覆盖率要求：** 最低 80%（单元测试 + 集成测试 + E2E 测试）

### TDD 循环

```
┌─────────────────────────────────────────────────────────────┐
│                    TDD Red-Green-Refactor 循环               │
│                                                              │
│     ┌─────────┐                                             │
│     │  RED    │ ── 写一个失败的测试                          │
│     │  🔴     │    (测试必须能编译和执行)                     │
│     └────┬────┘                                             │
│          │                                                   │
│          ▼                                                   │
│     ┌─────────┐                                             │
│     │  GREEN  │ ── 写最少的代码让测试通过                     │
│     │  🟢     │    (不要过度实现)                             │
│     └────┬────┘                                             │
│          │                                                   │
│          ▼                                                   │
│     ┌─────────┐                                             │
│     │REFACTOR │ ── 重构代码，保持测试通过                     │
│     │  🔵     │    (消除重复，改善设计)                       │
│     └────┬────┘                                             │
│          │                                                   │
│          └──────────────────▶ 重复                           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 工作流程步骤

```
Step 1: 编写用户旅程
  → 描述功能从用户角度的行为

Step 2: 生成测试用例
  → 基于用户旅程生成测试场景

Step 3: 运行测试 (RED)
  → 确认测试失败
  → ⚠️ 测试必须能编译和执行才算有效的 RED

Step 4: 实现最小代码
  → 只写让测试通过的最少代码

Step 5: 运行测试 (GREEN)
  → 确认测试通过

Step 6: 重构
  → 改善代码结构，保持测试通过

Step 7: 验证覆盖率
  → 确保达到 80% 覆盖率
```

### Git 提交检查点

| 阶段 | 提交消息格式 |
|------|-------------|
| RED 验证后 | `test: add reproducer for <feature>` |
| GREEN 验证后 | `fix: <feature>` |
| 重构后（可选） | `refactor: clean up after <feature>` |

### 测试最佳实践

```
┌─────────────────────────────────────────────────────────────┐
│ 测试原则                                                      │
├─────────────────────────────────────────────────────────────┤
│ ✓ 一个测试一个断言                                            │
│ ✓ Arrange-Act-Assert 结构                                    │
│ ✓ Mock 外部依赖                                              │
│ ✓ 测试边界情况和错误路径                                       │
│ ✓ 单元测试保持 < 50ms                                        │
│ ✓ 测试名称描述预期行为                                         │
└─────────────────────────────────────────────────────────────┘
```

**AAA 结构示例：**

```typescript
test('should calculate total with tax', () => {
  // Arrange
  const items = [{ price: 100 }, { price: 200 }];
  const taxRate = 0.1;

  // Act
  const total = calculateTotal(items, taxRate);

  // Assert
  expect(total).toBe(330);
});
```

---

## 三、后端开发 Skills

### 1. Go 语言模式 (golang-patterns skill)

**触发时机：** 编写/审查 Go 代码

**核心原则：**

| 原则 | 说明 |
|------|------|
| Simplicity | 简单的解决方案 |
| Useful Zero Values | 类型有意义的零值 |
| Accept Interfaces, Return Structs | 接受接口，返回结构体 |

**错误处理：**

```go
// ✓ 正确：包装错误上下文
if err != nil {
    return fmt.Errorf("failed to process order %d: %w", orderID, err)
}

// ✗ 错误：忽略错误
result, _ := doSomething()

// ✓ 正确：使用 errors.Is/As 检查
if errors.Is(err, ErrNotFound) {
    // 处理未找到
}
```

**并发模式：**

```go
// Worker Pool 模式
func processJobs(jobs <-chan Job, results chan<- Result, workers int) {
    var wg sync.WaitGroup
    for i := 0; i < workers; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for job := range jobs {
                results <- process(job)
            }
        }()
    }
    wg.Wait()
}

// Context 超时控制
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
defer cancel()
```

**接口设计：**

```go
// ✓ 正确：小接口
type Reader interface {
    Read(p []byte) (n int, err error)
}

// ✓ 正确：在使用的包定义接口，而非实现的包
```

**性能优化：**

| 场景 | 建议 |
|------|------|
| 频繁分配 | 使用 `sync.Pool` |
| 字符串拼接 | 使用 `strings.Builder` |
| 切片预分配 | `make([]int, 0, expectedCap)` |

**反模式警告：**

| 反模式 | 问题 |
|--------|------|
| 裸返回 | 长函数中降低可读性 |
| panic 控制流 | panic 仅用于不可恢复错误 |
| context 存储在 struct | 应作为参数传递 |
| 混合接收者类型 | 值/指针接收者保持一致 |

---

### 2. Python 模式 (python-patterns skill)

**触发时机：** 编写/审查 Python 代码

**核心原则：**

| 原则 | 说明 |
|------|------|
| Readability Counts | 代码应该显而易见 |
| Explicit > Implicit | 明确优于隐式 |
| EAFP | 宁愿请求原谅而非许可（异常处理优先） |

**类型提示：**

```python
# Python 3.9+
def process_items(items: list[str]) -> dict[str, int]:
    return {item: len(item) for item in items}

# 使用 TypeVar 和 Protocol
T = TypeVar('T')

def first(items: Sequence[T]) -> T | None:
    return items[0] if items else None
```

**错误处理：**

```python
# ✓ 正确：捕获特定异常
try:
    result = process(data)
except ValueError as e:
    logger.error(f"Invalid data: {e}")
    raise ProcessingError(f"Failed to process: {e}") from e

# ✗ 错误：裸 except
try:
    result = process(data)
except:  # 不要这样做
    pass
```

**数据类：**

```python
from dataclasses import dataclass
from typing import NamedTuple

# @dataclass 用于可变数据
@dataclass
class User:
    name: str
    email: str
    active: bool = True

# NamedTuple 用于不可变数据
class Point(NamedTuple):
    x: float
    y: float
```

**反模式警告：**

| 反模式 | 问题 |
|--------|------|
| 可变默认参数 | `def f(x=[])` 每次调用共享同一列表 |
| `type()` 检查 | 应使用 `isinstance()` |
| `== None` 比较 | 应使用 `is None` |
| `from module import *` | 污染命名空间 |

---

### 3. Django 模式 (django-patterns skill)

**触发时机：** 构建 Django 应用

**项目结构：**

```
project/
├── config/
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── production.py
│   │   └── test.py
│   ├── urls.py
│   └── wsgi.py
├── apps/
│   ├── users/
│   ├── orders/
│   └── products/
└── manage.py
```

**Model 设计：**

```python
# 自定义用户模型
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
    email = models.EmailField(unique=True)
    phone = models.CharField(max_length=20, blank=True)

    class Meta:
        db_table = 'users'

# 自定义 QuerySet
class OrderQuerySet(models.QuerySet):
    def pending(self):
        return self.filter(status='pending')

    def for_user(self, user):
        return self.filter(user=user)

class Order(models.Model):
    objects = OrderQuerySet.as_manager()
```

**DRF ViewSet 模式：**

```python
class OrderViewSet(viewsets.ModelViewSet):
    queryset = Order.objects.select_related('user').prefetch_related('items')
    serializer_class = OrderSerializer
    permission_classes = [IsAuthenticated]
    filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
    filterset_fields = ['status']
    search_fields = ['order_number']
    ordering_fields = ['created_at', 'total']

    @action(detail=True, methods=['post'])
    def cancel(self, request, pk=None):
        order = self.get_object()
        order.cancel()
        return Response({'status': 'cancelled'})
```

**性能优化：**

```python
# 避免 N+1 查询
orders = Order.objects.select_related('user').prefetch_related('items')

# 批量操作
Order.objects.bulk_create([Order(...) for _ in range(1000)])

# 索引优化
class Order(models.Model):
    order_number = models.CharField(max_length=50, db_index=True)
    class Meta:
        indexes = [
            models.Index(fields=['user', 'status']),
        ]
```

---

### 4. FastAPI 模式 (fastapi-patterns skill)

**触发时机：** 构建 FastAPI 应用

**项目结构：**

```
app/
├── main.py              # 应用工厂
├── config.py            # 配置
├── dependencies.py      # 依赖注入
├── exceptions.py        # 自定义异常
├── api/
│   └── routes/          # 路由模块
├── models/              # ORM 模型
├── schemas/             # Pydantic 模型
├── services/            # 业务逻辑
└── tests/               # 测试
```

**应用工厂：**

```python
from contextlib import asynccontextmanager
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 启动时初始化
    await database.connect()
    yield
    # 关闭时清理
    await database.disconnect()

app = FastAPI(lifespan=lifespan)
```

**Schema 分离：**

```python
# schemas/user.py
from pydantic import BaseModel, EmailStr

class UserCreate(BaseModel):
    email: EmailStr
    password: str
    name: str

class UserUpdate(BaseModel):
    name: str | None = None

class UserResponse(BaseModel):
    id: str
    email: str
    name: str
    # ⚠️ 响应模型绝不能包含密码、token 等敏感信息
```

**依赖注入：**

```python
from fastapi import Depends

async def get_db():
    async with database.session() as session:
        yield session

async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: AsyncSession = Depends(get_db)
) -> User:
    user = await verify_token(token, db)
    if not user:
        raise HTTPException(status_code=401, detail="Invalid token")
    return user

@router.post("/orders")
async def create_order(
    data: OrderCreate,
    user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    return await OrderService(db).create(user.id, data)
```

**安全检查清单：**

```
┌─────────────────────────────────────────────────────────────┐
│ FastAPI 安全检查                                              │
├─────────────────────────────────────────────────────────────┤
│ ✓ 使用 argon2-cffi/bcrypt 哈希密码                           │
│ ✓ 验证 JWT 属性（过期、签名）                                  │
│ ✓ 环境特定的 CORS 配置                                        │
│ ✓ 所有请求体使用 Pydantic 模型                                │
│ ✓ ORM 参数绑定（禁止 f-string SQL）                           │
│ ✓ 日志中脱敏敏感数据                                          │
│ ✓ CI 中运行依赖审计                                           │
└─────────────────────────────────────────────────────────────┘
```

---

### 5. Spring Boot 模式 (springboot-patterns skill)

**触发时机：** 构建 Spring Boot 应用

**REST API 结构：**

```java
@RestController
@RequestMapping("/api/v1/orders")
@RequiredArgsConstructor
public class OrderController {
    private final OrderService orderService;

    @PostMapping
    public ResponseEntity<OrderResponse> create(
        @Valid @RequestBody OrderCreateRequest request
    ) {
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(orderService.create(request));
    }
}
```

**Service 层：**

```java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository orderRepository;

    @Transactional
    public OrderResponse create(OrderCreateRequest request) {
        // 业务逻辑
    }

    @Transactional(readOnly = true)
    public OrderResponse getById(Long id) {
        return orderRepository.findById(id)
            .map(this::toResponse)
            .orElseThrow(() -> new NotFoundException("Order not found"));
    }
}
```

**异常处理：**

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(NotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("NOT_FOUND", e.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException e) {
        List<String> errors = e.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(f -> f.getField() + ": " + f.getDefaultMessage())
            .toList();
        return ResponseEntity.badRequest()
            .body(new ErrorResponse("VALIDATION_ERROR", errors));
    }
}
```

---

### 6. PostgreSQL 模式 (postgres-patterns skill)

**触发时机：** 编写 SQL/迁移、设计 Schema、排查慢查询

**索引速查表：**

| 查询模式 | 索引类型 |
|----------|----------|
| 等值/范围查询 | B-tree（默认） |
| 多条件查询 | 复合索引 |
| JSONB 包含查询 | GIN 索引 |
| 全文搜索 | GIN 索引 |
| 时间序列范围 | BRIN 索引 |

**复合索引规则：**

```
"Equality columns first, then range columns"

-- 查询: WHERE user_id = ? AND status = ? AND created_at > ?
-- 索引: CREATE INDEX idx_orders_user_status_date ON orders(user_id, status, created_at);
```

**关键模式：**

```sql
-- 覆盖索引（避免回表）
CREATE INDEX idx_users_email_include ON users(email) INCLUDE (name, created_at);

-- 部分索引（更小的索引）
CREATE INDEX idx_orders_pending ON orders(created_at) WHERE status = 'pending';

-- UPSERT（插入或更新）
INSERT INTO users (id, email, name)
VALUES ($1, $2, $3)
ON CONFLICT (id) DO UPDATE SET email = EXCLUDED.email, name = EXCLUDED.name;

-- 游标分页（避免 OFFSET）
SELECT * FROM orders
WHERE id > $last_id
ORDER BY id
LIMIT 20;

-- 队列处理（跳过锁定）
SELECT * FROM jobs
WHERE status = 'pending'
FOR UPDATE SKIP LOCKED
LIMIT 1;
```

---

### 7. Redis 模式 (redis-patterns skill)

**触发时机：** 缓存设计、会话存储、分布式锁

**数据结构选择：**

| 用例 | 数据结构 |
|------|----------|
| 简单缓存 | String |
| 用户会话 | Hash |
| 排行榜 | Sorted Set |
| 唯一访客 | Set |
| 活动流 | List |
| 事件流 | Stream |
| 计数器/限流 | String (INCR) |

**缓存模式：**

```
┌─────────────────────────────────────────────────────────────┐
│ Cache-Aside 模式                                              │
├─────────────────────────────────────────────────────────────┤
│ 1. 检查缓存                                                   │
│    GET user:123                                              │
│                                                              │
│ 2. 缓存命中 → 返回数据                                        │
│                                                              │
│ 3. 缓存未命中 → 查询数据库                                    │
│    SELECT * FROM users WHERE id = 123                       │
│                                                              │
│ 4. 写入缓存（设置 TTL）                                       │
│    SETEX user:123 3600 '{json_data}'                        │
└─────────────────────────────────────────────────────────────┘
```

**分布式锁：**

```python
import redis

r = redis.Redis()

def acquire_lock(resource: str, ttl: int = 10) -> str | None:
    token = str(uuid.uuid4())
    # SET NX PX 原子操作
    if r.set(f"lock:{resource}", token, nx=True, px=ttl * 1000):
        return token
    return None

def release_lock(resource: str, token: str) -> bool:
    # Lua 脚本保证原子性
    script = """
    if redis.call("get", KEYS[1]) == ARGV[1] then
        return redis.call("del", KEYS[1])
    else
        return 0
    end
    """
    return r.eval(script, 1, f"lock:{resource}", token)
```

**反模式警告：**

| 反模式 | 问题 |
|--------|------|
| 无 TTL | 内存泄漏 |
| 生产环境 `KEYS *` | 阻塞操作 |
| 大对象 | 序列化开销 |
| 忽略连接限制 | 连接池耗尽 |

---

## 四、前端开发 Skills

### 1. 前端模式 (frontend-patterns skill)

**触发时机：** 构建 React 组件、状态管理、性能优化

**组件模式：**

```
┌─────────────────────────────────────────────────────────────┐
│ 组件分类                                                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Presentational Components (展示组件)                        │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ - 只关注 UI 展示                                      │   │
│  │ - 无业务逻辑                                          │   │
│  │ - 通过 props 接收数据                                 │   │
│  │ - 例: Button, Card, Modal, Table                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
│  Container Components (容器组件)                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ - 关注业务逻辑                                        │   │
│  │ - 管理状态                                            │   │
│  │ - 连接数据源                                          │   │
│  │ - 例: UserListContainer, AuthProvider                │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
│  Compound Components (复合组件)                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ - 通过 Context 共享状态                               │   │
│  │ - 子组件协同工作                                      │   │
│  │ - 例: Tabs, Accordion, Dialog                        │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**自定义 Hooks：**

```tsx
// useToggle - 简单布尔状态
function useToggle(initial: boolean = false) {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue(v => !v), []);
  return [value, toggle] as const;
}

// useDebounce - 防抖值
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}
```

**性能优化：**

```tsx
// useMemo - 缓存计算结果
const sortedItems = useMemo(
  () => items.sort((a, b) => a.name.localeCompare(b.name)),
  [items]
);

// useCallback - 缓存回调函数
const handleSubmit = useCallback(
  (data: FormData) => submitForm(data),
  []
);

// React.memo - 避免不必要渲染
const ExpensiveComponent = React.memo(function ExpensiveComponent({ data }) {
  return <div>{/* 复杂渲染 */}</div>;
});

// 代码分割
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}
```

---

### 2. E2E 测试 (e2e-testing skill)

**触发时机：** 测试关键用户流程

**项目结构：**

```
tests/e2e/
├── auth/
│   ├── login.spec.ts
│   └── register.spec.ts
├── features/
│   ├── checkout.spec.ts
│   └── search.spec.ts
├── fixtures/
│   └── test-data.ts
└── playwright.config.ts
```

**Page Object Model：**

```typescript
// pages/SearchPage.ts
class SearchPage {
  readonly page: Page;
  readonly searchInput: Locator;
  readonly searchButton: Locator;
  readonly results: Locator;

  constructor(page: Page) {
    this.page = page;
    this.searchInput = page.getByTestId('search-input');
    this.searchButton = page.getByTestId('search-button');
    this.results = page.getByTestId('search-results');
  }

  async goto() {
    await this.page.goto('/search');
  }

  async search(query: string) {
    await this.searchInput.fill(query);
    await this.searchButton.click();
    await this.results.waitFor({ state: 'visible' });
  }
}
```

**Flaky 测试处理：**

```
┌─────────────────────────────────────────────────────────────┐
│ Flaky 测试策略                                                │
├─────────────────────────────────────────────────────────────┤
│ 识别：                                                        │
│   npx playwright test --repeat-each=10                       │
│   npx playwright test --retries=3                            │
│                                                              │
│ 隔离：                                                        │
│   test.fixme('flaky test description', async () => {        │
│     // TODO: fix issue #123                                  │
│   });                                                        │
│                                                              │
│ 修复：                                                        │
│   ✓ 使用自动等待的 locator                                    │
│   ✓ 等待特定网络响应                                          │
│   ✗ 避免任意 timeout                                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 五、Rust 开发模式 (rust-patterns skill)

**触发时机：** 编写/审查 Rust 代码

**所有权和借用：**

```rust
// ✓ 正确：不需要所有权时传递引用
fn process(data: &str) -> String {
    data.to_uppercase()
}

// ✓ 正确：需要存储或消费时获取所有权
fn store(value: String) -> StoredValue {
    StoredValue { inner: value }
}
```

**错误处理：**

```rust
// ✓ 正确：使用 Result 和 ? 操作符
fn read_config(path: &str) -> Result<Config, Error> {
    let content = fs::read_to_string(path)?;
    let config: Config = toml::from_str(&content)?;
    Ok(config)
}

// 库：使用 thiserror
#[derive(thiserror::Error, Debug)]
pub enum Error {
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),
}

// 应用：使用 anyhow
use anyhow::{Context, Result};

fn main() -> Result<()> {
    let config = read_config("config.toml")
        .context("Failed to read configuration")?;
    Ok(())
}
```

**Unsafe 代码规则：**

| 可接受 | 不可接受 |
|--------|----------|
| FFI 边界 | 绕过借用检查器 |
| 性能关键路径（已证明正确） | 方便 |
| 有 Safety 注释 | 无 Safety 注释 |

---

## 六、综合使用流程

### 典型开发流程

```
┌─────────────────────────────────────────────────────────────┐
│ 用户需求: "实现用户登录功能"                                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 1: TDD 工作流                                          │
│  → tdd-workflow skill                                       │
│  → 1. 编写登录测试用例                                        │
│  → 2. 运行测试 (RED)                                         │
│  → 3. 实现最小代码                                            │
│  → 4. 运行测试 (GREEN)                                       │
│  → 5. 重构                                                   │
│  → 6. 验证覆盖率 ≥ 80%                                       │
│                                                              │
│  Step 2: 语言特定模式                                        │
│  → 根据技术栈选择:                                            │
│    - Go → golang-patterns                                   │
│    - Python → python-patterns                               │
│    - Django → django-patterns                               │
│    - FastAPI → fastapi-patterns                             │
│    - Spring Boot → springboot-patterns                      │
│    - Rust → rust-patterns                                   │
│                                                              │
│  Step 3: 数据库模式 (如需要)                                  │
│  → postgres-patterns / redis-patterns                       │
│                                                              │
│  Step 4: 前端开发 (如需要)                                    │
│  → frontend-patterns skill                                  │
│                                                              │
│  Step 5: E2E 测试                                            │
│  → e2e-testing skill                                        │
│  → Playwright 测试登录流程                                   │
│                                                              │
│  最终输出: 完整的登录功能                                      │
│  - 单元测试覆盖                                               │
│  - 集成测试覆盖                                               │
│  - E2E 测试覆盖                                               │
│  - 遵循语言最佳实践                                           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 七、Skill 协同关系

| Skill | 与其他的协同 |
|-------|--------------|
| `tdd-workflow` | 所有开发的基础，必须首先遵循 |
| `coding-standards` | 贯穿所有开发过程 |
| `golang-patterns` | Go 项目的开发规范 |
| `python-patterns` | Python 项目的开发规范 |
| `django-patterns` | Django 项目，依赖 python-patterns |
| `fastapi-patterns` | FastAPI 项目，依赖 python-patterns |
| `springboot-patterns` | Spring Boot 项目的开发规范 |
| `rust-patterns` | Rust 项目的开发规范 |
| `postgres-patterns` | 数据库层开发 |
| `redis-patterns` | 缓存层开发 |
| `frontend-patterns` | 前端开发 |
| `e2e-testing` | 功能完成后的端到端测试 |

---

## 八、关键建议总结

| 原则 | 流程 |
|------|------|
| 测试先行 | `tdd-workflow` → RED → GREEN → REFACTOR |
| 80% 覆盖率 | 单元测试 + 集成测试 + E2E 测试 |
| 语言最佳实践 | 根据技术栈选择对应的 patterns skill |
| 代码质量 | 函数 < 50 行，嵌套 < 5 层，无魔法数字 |
| 性能优化 | 避免 N+1 查询，使用缓存，预分配 |
| 安全意识 | 输入验证，参数绑定，敏感数据脱敏 |

---

## 相关链接

- [ECC-GUIDE.md](./ECC-GUIDE.md) - ECC 项目总览
- [ECC-GUIDE-requirements-exploration.md](./ECC-GUIDE-requirements-exploration.md) - 需求理解阶段指南
- [ECC-GUIDE-architecture-design.md](./ECC-GUIDE-architecture-design.md) - 架构设计阶段指南
- [GitHub 仓库](https://github.com/affaan-m/everything-claude-code)
- [官网](https://ecc.tools)
