# ECC 架构设计阶段 - 详细指南

> 本文档详细说明 ECC 在架构设计阶段的使用方法

---

## 核心原则

ECC 在架构设计阶段遵循 **"Design Before You Build"**（先设计再构建）的核心原则。

**阶段目标：**
1. 系统设计决策 - 选择合适的架构模式
2. 技术选型 - 评估和选择技术栈
3. 接口定义 - API 规范、数据模型、契约
4. 架构蓝图 - 可落地的设计方案

---

## 七大核心 Skill/Agent

### 1. `architect` agent - 系统设计专家

**用途：** 复杂系统的架构设计决策

**触发时机：**
- 新系统架构设计
- 大规模重构决策
- 技术选型评估
- 扩展性/性能设计

**工作流程：**

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: 需求理解                                            │
│     - 分析功能需求                                            │
│     - 识别非功能需求（性能、安全、可扩展性）                   │
│     - 理解约束条件                                            │
├─────────────────────────────────────────────────────────────┤
│  Step 2: 架构分析                                            │
│     - 现有系统评估                                            │
│     - 痛点和瓶颈识别                                          │
│     - 技术债务分析                                            │
├─────────────────────────────────────────────────────────────┤
│  Step 3: 方案设计                                            │
│     - 提出多个候选方案                                        │
│     - 权衡分析（Trade-offs）                                 │
│     - 推荐最佳方案                                            │
├─────────────────────────────────────────────────────────────┤
│  Step 4: 详细蓝图                                            │
│     - 组件划分和职责                                          │
│     - 依赖关系和接口                                          │
│     - 数据流和控制流                                          │
├─────────────────────────────────────────────────────────────┤
│  Step 5: 风险评估                                            │
│     - 技术风险                                                │
│     - 实施风险                                                │
│     - 运维风险                                                │
└─────────────────────────────────────────────────────────────┘
```

**使用示例：**

```
用户: 设计一个电商系统的订单服务架构

architect agent 分析:
┌─────────────────────────────────────────────────────────────┐
│ # 架构设计: 电商订单服务                                      │
│                                                              │
│ ## 需求分析                                                   │
│ - 功能: 订单创建、支付、履约、退款                            │
│ - 非功能: 高可用(99.9%)、低延迟(<100ms)、最终一致性            │
│ - 约束: 现有微服务架构、PostgreSQL、Kafka                      │
│                                                              │
│ ## 候选方案                                                   │
│                                                              │
│ 方案 A: 单体订单服务                                         │
│   优点: 简单、事务简单、开发快                                │
│   缺点: 扩展性差、单点故障                                    │
│   评分: 6/10                                                 │
│                                                              │
│ 方案 B: CQRS + 事件溯源                                      │
│   优点: 高扩展、审计完整、读写分离                            │
│   缺点: 复杂度高、学习曲线陡                                  │
│   评分: 8/10                                                 │
│                                                              │
│ 方案 C: Saga 编排模式                                         │
│   优点: 分布式事务、服务解耦、可扩展                          │
│   缺点: 最终一致性、调试复杂                                  │
│   评分: 9/10 ✓ 推荐                                          │
│                                                              │
│ ## 推荐架构蓝图                                               │
│                                                              │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│   │ 订单服务    │───▶│ Saga 编排器 │───▶│ 支付服务    │     │
│   │ (命令侧)   │    │             │    │             │     │
│   └─────────────┘    └─────────────┘    └─────────────┘     │
│          │                  │                              │
│          ▼                  ▼                              │
│   ┌─────────────┐    ┌─────────────┐                        │
│   │ 订单读模型  │    │ 库存服务    │                        │
│   │ (查询侧)   │    │             │                        │
│   └─────────────┘    └─────────────┘                        │
│                                                              │
│ ## 关键决策                                                   │
│ 1. 使用 Kafka 作为事件总线                                   │
│ 2. PostgreSQL 作为命令侧存储                                 │
│ 3. Redis 作为读模型缓存                                      │
│ 4. Saga 编排器管理分布式事务                                 │
│                                                              │
│ ## 风险评估                                                   │
│ - HIGH: Saga 补偿逻辑复杂度 → 需要完善的测试覆盖              │
│ - MEDIUM: Kafka 消息顺序 → 使用分区键保证                     │
│ - LOW: 读模型同步延迟 → 可接受的最终一致性                    │
└─────────────────────────────────────────────────────────────┘
```

---

### 2. `api-design` skill - REST API 设计规范

**用途：** 设计符合 RESTful 规范的 API 接口

**触发时机：**
- 设计新的 API 接口
- API 版本升级
- API 文档编写
- 前后端契约定义

**设计原则：**

| 原则 | 说明 |
|------|------|
| **资源导向** | URL 表示资源，HTTP 方法表示操作 |
| **统一接口** | 一致的命名、错误格式、认证方式 |
| **无状态** | 每个请求包含所有必要信息 |
| **分层系统** | 客户端无需知道后端架构 |
| **可缓存** | 响应明确标记缓存策略 |

**URL 设计规范：**

```
┌─────────────────────────────────────────────────────────────┐
│ 资源命名规范                                                  │
├─────────────────────────────────────────────────────────────┤
│ ✓ 使用名词复数                                                │
│   /users, /orders, /products                                │
│                                                              │
│ ✓ 使用小写连字符                                              │
│   /order-items, /user-profiles                              │
│                                                              │
│ ✗ 避免动词                                                    │
│   /getUsers (错误) → /users (正确)                           │
│                                                              │
│ ✗ 避免嵌套过深                                                │
│   /users/123/orders/456/items/789 (过深)                     │
│   → /users/123/orders (合理)                                │
│   → /order-items/789 (扁平化)                               │
└─────────────────────────────────────────────────────────────┘
```

**HTTP 方法映射：**

| 方法 | 用途 | 幂等性 | 示例 |
|------|------|--------|------|
| `GET` | 获取资源 | ✓ | `GET /users/123` |
| `POST` | 创建资源 | ✗ | `POST /users` |
| `PUT` | 全量更新 | ✓ | `PUT /users/123` |
| `PATCH` | 部分更新 | ✗ | `PATCH /users/123` |
| `DELETE` | 删除资源 | ✓ | `DELETE /users/123` |

**响应状态码：**

```
┌─────────────────────────────────────────────────────────────┐
│ 成功响应 (2xx)                                                │
│   200 OK          - 成功获取/更新                            │
│   201 Created     - 成功创建                                 │
│   204 No Content  - 成功删除                                 │
│                                                              │
│ 客户端错误 (4xx)                                              │
│   400 Bad Request   - 请求格式错误                           │
│   401 Unauthorized - 未认证                                  │
│   403 Forbidden     - 无权限                                  │
│   404 Not Found     - 资源不存在                             │
│   422 Unprocessable - 业务验证失败                           │
│   429 Too Many Requests - 限流                               │
│                                                              │
│ 服务端错误 (5xx)                                              │
│   500 Internal Server Error - 服务器错误                     │
│   503 Service Unavailable - 服务不可用                       │
└─────────────────────────────────────────────────────────────┘
```

**错误响应格式：**

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "请求参数验证失败",
    "details": [
      {
        "field": "email",
        "message": "邮箱格式不正确"
      }
    ],
    "requestId": "req_abc123",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

**分页设计：**

```
请求: GET /users?page=2&limit=20&sort=-createdAt

响应:
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": true
  },
  "links": {
    "self": "/users?page=2&limit=20",
    "next": "/users?page=3&limit=20",
    "prev": "/users?page=1&limit=20",
    "first": "/users?page=1&limit=20",
    "last": "/users?page=8&limit=20"
  }
}
```

**版本控制策略：**

| 策略 | 示例 | 优点 | 缺点 |
|------|------|------|------|
| URL 路径 | `/v1/users` | 简单直观 | URL 变长 |
| Header | `Accept: application/vnd.api+json;version=1` | URL 干净 | 不够直观 |
| Query | `/users?version=1` | 灵活 | 非标准 |

---

### 3. `backend-patterns` skill - 后端架构模式

**用途：** 选择和实现后端架构模式

**核心模式速查：**

#### 分层架构 (Layered Architecture)

```
┌─────────────────────────────────────────────────────────────┐
│                      Presentation Layer                     │
│                    (Controllers / Routes)                    │
├─────────────────────────────────────────────────────────────┤
│                      Business Logic Layer                    │
│                      (Services / Use Cases)                  │
├─────────────────────────────────────────────────────────────┤
│                      Data Access Layer                       │
│                    (Repositories / DAOs)                     │
├─────────────────────────────────────────────────────────────┤
│                      Data Layer                              │
│                    (Database / External APIs)                │
└─────────────────────────────────────────────────────────────┘
```

**适用场景：** 传统企业应用、CRUD 为主的应用

#### 六边形架构 (Hexagonal Architecture)

```
                    ┌─────────────────────┐
                    │      Domain Core     │
                    │   (Business Logic)   │
                    │                      │
                    │  ┌───────────────┐  │
                    │  │   Entities    │  │
                    │  │   Use Cases   │  │
                    │  │   Ports       │  │
                    │  └───────────────┘  │
                    └─────────────────────┘
                              │
         ┌────────────────────┼────────────────────┐
         │                    │                    │
    ┌────▼────┐          ┌────▼────┐          ┌────▼────┐
    │ Primary │          │ Primary │          │Secondary│
    │ Adapter │          │ Adapter │          │ Adapter │
    │  (REST) │          │ (GraphQL)│         │  (DB)   │
    └─────────┘          └─────────┘          └─────────┘
```

**适用场景：** 需要清晰边界、多入口/出口、测试优先的项目

#### CQRS (Command Query Responsibility Segregation)

```
┌─────────────┐         ┌─────────────┐
│   Command   │         │    Query    │
│   (写操作)  │         │   (读操作)   │
└──────┬──────┘         └──────┬──────┘
       │                       │
       ▼                       ▼
┌─────────────┐         ┌─────────────┐
│ Write Model │         │  Read Model │
│ (优化写入)  │         │ (优化读取)   │
└──────┬──────┘         └──────┬──────┘
       │                       │
       ▼                       ▼
┌─────────────┐         ┌─────────────┐
│   Master    │  同步   │    Slave    │
│  Database   │───────▶│  Database   │
└─────────────┘         └─────────────┘
```

**适用场景：** 读写比例严重失衡、复杂查询需求、需要读写分离优化

#### 事件驱动架构 (Event-Driven Architecture)

```
┌─────────────┐         ┌─────────────┐
│  Producer   │         │  Consumer   │
│  Service A  │         │  Service B  │
└──────┬──────┘         └──────▲──────┘
       │                       │
       │   ┌───────────────────┤
       │   │                   │
       ▼   │                   │
┌─────────────┐         ┌─────────────┐
│   Event     │         │   Event     │
│   Broker    │         │   Store     │
│ (Kafka/SQS) │         │             │
└─────────────┘         └─────────────┘
```

**适用场景：** 微服务通信、异步处理、事件溯源、实时数据处理

#### 微服务架构 (Microservices)

```
┌─────────────────────────────────────────────────────────────┐
│                        API Gateway                           │
└───────────────────────────┬─────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
   ┌────▼────┐         ┌────▼────┐         ┌────▼────┐
   │ User    │         │ Order   │         │Product  │
   │ Service │         │ Service │         │ Service │
   └────┬────┘         └────┬────┘         └────┬────┘
        │                   │                   │
   ┌────▼────┐         ┌────▼────┐         ┌────▼────┐
   │  User   │         │ Order   │         │Product  │
   │   DB    │         │   DB    │         │   DB    │
   └─────────┘         └─────────┘         └─────────┘
```

**适用场景：** 大规模系统、独立部署需求、团队自治、高扩展性需求

**模式选择决策树：**

```
开始
  │
  ├─ 团队规模 < 5 人？
  │     └─ 是 → 单体分层架构
  │
  ├─ 读写比例 > 10:1？
  │     └─ 是 → CQRS
  │
  ├─ 需要异步解耦？
  │     └─ 是 → 事件驱动
  │
  ├─ 需要多入口/出口？
  │     └─ 是 → 六边形架构
  │
  └─ 独立部署/扩展需求？
        └─ 是 → 微服务
```

---

### 4. `frontend-patterns` skill - 前端架构模式

**用途：** React/Next.js 组件设计和状态管理

**组件设计原则：**

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
│  Higher-Order Components (HOC)                               │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ - 复用组件逻辑                                        │   │
│  │ - 接收组件返回增强组件                                 │   │
│  │ - 例: withAuth, withLoading, withErrorBoundary       │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**目录结构最佳实践：**

```
src/
├── app/                    # Next.js App Router
│   ├── (auth)/            # 路由组
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   └── api/                # API Routes
│
├── components/
│   ├── ui/                 # 基础 UI 组件
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   └── index.ts
│   ├── features/           # 功能组件
│   │   ├── UserList/
│   │   └── OrderForm/
│   └── layouts/            # 布局组件
│       ├── Header.tsx
│       └── Sidebar.tsx
│
├── hooks/                  # 自定义 Hooks
│   ├── useAuth.ts
│   └── useFetch.ts
│
├── lib/                    # 工具库
│   ├── api.ts
│   └── utils.ts
│
├── types/                  # TypeScript 类型
│   └── index.ts
│
└── styles/                 # 样式
    └── globals.css
```

**状态管理选择：**

| 场景 | 推荐方案 | 理由 |
|------|----------|------|
| 简单组件状态 | `useState` | 原生、简单 |
| 复杂状态逻辑 | `useReducer` | 可预测、可测试 |
| 跨组件共享 | Context + `useContext` | 无额外依赖 |
| 全局状态 | Zustand / Jotai | 轻量、TS 友好 |
| 服务端状态 | React Query / SWR | 缓存、同步、重试 |
| 表单状态 | React Hook Form | 性能、验证 |

**React Query 数据获取模式：**

```typescript
// 查询
const { data, isLoading, error } = useQuery({
  queryKey: ['users', userId],
  queryFn: () => fetchUser(userId),
  staleTime: 5 * 60 * 1000,  // 5分钟内不重新获取
  gcTime: 30 * 60 * 1000,    // 30分钟后垃圾回收
});

// 变更
const mutation = useMutation({
  mutationFn: updateUser,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['users'] });
  },
});

// 乐观更新
const { mutate } = useMutation({
  mutationFn: updateTodo,
  onMutate: async (newTodo) => {
    await queryClient.cancelQueries({ queryKey: ['todos'] });
    const previousTodos = queryClient.getQueryData(['todos']);
    queryClient.setQueryData(['todos'], (old) => [...old, newTodo]);
    return { previousTodos };
  },
  onError: (err, newTodo, context) => {
    queryClient.setQueryData(['todos'], context.previousTodos);
  },
});
```

---

### 5. `database-migrations` skill - 数据库迁移策略

**用途：** 安全的数据库 Schema 变更管理

**迁移原则：**

```
┌─────────────────────────────────────────────────────────────┐
│ 黄金法则                                                      │
├─────────────────────────────────────────────────────────────┤
│ 1. 向后兼容优先 - 新代码必须能读旧 Schema                      │
│ 2. 小步快跑 - 每次迁移只做一件事                               │
│ 3. 可回滚 - 每个迁移必须有对应的 down 迁移                     │
│ 4. 测试先行 - 在生产数据副本上测试                             │
│ 5. 零停机 - 大表变更需要特殊策略                               │
└─────────────────────────────────────────────────────────────┘
```

**迁移工作流：**

```
Phase 1: 准备阶段
┌─────────────────────────────────────────────────────────────┐
│ 1. 分析当前 Schema                                           │
│ 2. 识别依赖关系                                              │
│ 3. 评估数据量                                                │
│ 4. 制定迁移计划                                              │
└─────────────────────────────────────────────────────────────┘

Phase 2: 开发阶段
┌─────────────────────────────────────────────────────────────┐
│ 1. 创建迁移文件                                              │
│ 2. 编写 up 和 down SQL                                       │
│ 3. 在开发环境执行                                             │
│ 4. 验证结果                                                   │
└─────────────────────────────────────────────────────────────┘

Phase 3: 测试阶段
┌─────────────────────────────────────────────────────────────┐
│ 1. 在 staging 环境测试                                       │
│ 2. 使用生产数据副本                                           │
│ 3. 测量执行时间                                               │
│ 4. 测试回滚                                                   │
└─────────────────────────────────────────────────────────────┘

Phase 4: 生产阶段
┌─────────────────────────────────────────────────────────────┐
│ 1. 备份数据库                                                 │
│ 2. 低峰期执行                                                 │
│ 3. 监控执行过程                                               │
│ 4. 验证数据完整性                                             │
└─────────────────────────────────────────────────────────────┘
```

**大表迁移策略：**

| 场景 | 策略 | 示例 |
|------|------|------|
| 添加可空列 | 直接 ALTER | `ALTER TABLE users ADD COLUMN bio TEXT;` |
| 添加非空列 | 分步迁移 | 1. 添加可空列 → 2. 回填数据 → 3. 设非空 |
| 重命名列 | 双写迁移 | 1. 添加新列 → 2. 双写 → 3. 迁移数据 → 4. 删除旧列 |
| 大表加索引 | CONCURRENTLY | `CREATE INDEX CONCURRENTLY idx_users_email ON users(email);` |
| 分区表 | 渐进迁移 | 1. 创建分区表 → 2. 双写 → 3. 迁移数据 → 4. 切换 |

**添加非空列的完整流程：**

```sql
-- Step 1: 添加可空列
ALTER TABLE users ADD COLUMN status VARCHAR(20);

-- Step 2: 回填默认值 (分批)
UPDATE users SET status = 'active' WHERE id BETWEEN 1 AND 10000;
UPDATE users SET status = 'active' WHERE id BETWEEN 10001 AND 20000;
-- ... 分批执行

-- Step 3: 设置非空约束
ALTER TABLE users ALTER COLUMN status SET NOT NULL;

-- Step 4: 添加索引 (可选)
CREATE INDEX CONCURRENTLY idx_users_status ON users(status);
```

**Prisma 迁移示例：**

```prisma
// schema.prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  status    String   @default("active")
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

```bash
# 创建迁移
npx prisma migrate dev --name add_user_status

# 生产环境部署
npx prisma migrate deploy

# 查看状态
npx prisma migrate status
```

---

### 6. `hexagonal-architecture` skill - 六边形架构

**用途：** 实现清晰的架构边界和依赖反转

**核心概念：**

```
                    ┌─────────────────────────────────────┐
                    │           Domain Layer               │
                    │  ┌─────────────────────────────┐    │
                    │  │        Entities             │    │
                    │  │  - 业务对象                  │    │
                    │  │  - 无外部依赖               │    │
                    │  └─────────────────────────────┘    │
                    │  ┌─────────────────────────────┐    │
                    │  │        Use Cases            │    │
                    │  │  - 业务逻辑                  │    │
                    │  │  - 定义 Ports               │    │
                    │  └─────────────────────────────┘    │
                    │  ┌─────────────────────────────┐    │
                    │  │        Ports                │    │
                    │  │  - Primary (入站接口)        │    │
                    │  │  - Secondary (出站接口)      │    │
                    │  └─────────────────────────────┘    │
                    └─────────────────────────────────────┘
                                      │
        ┌─────────────────────────────┼─────────────────────────────┐
        │                             │                             │
   ┌────▼────┐                  ┌────▼────┐                  ┌────▼────┐
   │ Primary │                  │ Primary │                  │Secondary│
   │ Adapter │                  │ Adapter │                  │ Adapter │
   │         │                  │         │                  │         │
   │ REST API│                  │ GraphQL │                  │Database │
   │Controller│                  │Resolver │                  │Repository│
   └─────────┘                  └─────────┘                  └─────────┘
```

**目录结构：**

```
src/
├── domain/                    # 领域层 (核心)
│   ├── entities/
│   │   ├── User.ts
│   │   └── Order.ts
│   ├── ports/
│   │   ├── primary/          # 入站端口 (被外部调用)
│   │   │   ├── UserServicePort.ts
│   │   │   └── OrderServicePort.ts
│   │   └── secondary/        # 出站端口 (调用外部)
│   │       ├── UserRepositoryPort.ts
│   │       └── EmailServicePort.ts
│   └── usecases/
│       ├── CreateUserUseCase.ts
│       └── PlaceOrderUseCase.ts
│
├── adapters/                  # 适配器层
│   ├── primary/              # 入站适配器
│   │   ├── rest/
│   │   │   └── UserController.ts
│   │   └── graphql/
│   │       └── UserResolver.ts
│   └── secondary/            # 出站适配器
│       ├── persistence/
│       │   └── PostgresUserRepository.ts
│       └── notification/
│           └── SMTPEmailService.ts
│
└── infrastructure/            # 基础设施
    ├── database/
    └── config/
```

**代码示例：**

```typescript
// domain/entities/User.ts
export class User {
  constructor(
    public readonly id: string,
    public email: string,
    public name: string,
  ) {}

  updateName(newName: string): void {
    if (newName.length < 2) {
      throw new Error('Name must be at least 2 characters');
    }
    this.name = newName;
  }
}

// domain/ports/secondary/UserRepositoryPort.ts
export interface UserRepositoryPort {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<void>;
  delete(id: string): Promise<void>;
}

// domain/ports/primary/UserServicePort.ts
export interface UserServicePort {
  createUser(email: string, name: string): Promise<User>;
  getUser(id: string): Promise<User | null>;
  updateUser(id: string, name: string): Promise<User>;
}

// domain/usecases/CreateUserUseCase.ts
export class CreateUserUseCase implements UserServicePort {
  constructor(private readonly userRepo: UserRepositoryPort) {}

  async createUser(email: string, name: string): Promise<User> {
    const user = new User(crypto.randomUUID(), email, name);
    await this.userRepo.save(user);
    return user;
  }
}

// adapters/secondary/persistence/PostgresUserRepository.ts
export class PostgresUserRepository implements UserRepositoryPort {
  constructor(private readonly db: Database) {}

  async findById(id: string): Promise<User | null> {
    const row = await this.db.query('SELECT * FROM users WHERE id = $1', [id]);
    if (!row) return null;
    return new User(row.id, row.email, row.name);
  }

  async save(user: User): Promise<void> {
    await this.db.query(
      'INSERT INTO users (id, email, name) VALUES ($1, $2, $3)',
      [user.id, user.email, user.name]
    );
  }
}

// adapters/primary/rest/UserController.ts
export class UserController {
  constructor(private readonly userService: UserServicePort) {}

  async create(req: Request, res: Response) {
    const { email, name } = req.body;
    const user = await this.userService.createUser(email, name);
    res.status(201).json(user);
  }
}
```

**依赖注入配置：**

```typescript
// infrastructure/container.ts
import { Container } from 'inversify';
import { TYPES } from './types';

const container = new Container();

// 绑定 Ports 到 Adapters
container.bind<UserRepositoryPort>(TYPES.UserRepository)
  .to(PostgresUserRepository);

container.bind<UserServicePort>(TYPES.UserService)
  .to(CreateUserUseCase);

export { container };
```

---

### 7. `code-architect` agent - 功能架构设计

**用途：** 分析现有代码库并设计具体功能的实现架构

**触发时机：**
- 实现新功能前
- 需要了解现有代码模式
- 设计文件和接口结构

**工作流程：**

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: 代码库分析                                          │
│     - 扫描现有文件结构                                        │
│     - 识别命名约定                                            │
│     - 发现架构模式                                            │
├─────────────────────────────────────────────────────────────┤
│  Step 2: 依赖映射                                            │
│     - 追踪导入关系                                            │
│     - 识别模块边界                                            │
│     - 发现共享工具                                            │
├─────────────────────────────────────────────────────────────┤
│  Step 3: 设计蓝图                                            │
│     - 需要创建的文件                                          │
│     - 接口定义                                                │
│     - 数据流设计                                              │
├─────────────────────────────────────────────────────────────┤
│  Step 4: 构建顺序                                            │
│     - 依赖排序                                                │
│     - 实现步骤                                                │
│     - 测试策略                                                │
└─────────────────────────────────────────────────────────────┘
```

**输出示例：**

```
用户: 设计用户认证功能的架构

code-architect agent 分析:
┌─────────────────────────────────────────────────────────────┐
│ # 功能架构: 用户认证                                          │
│                                                              │
│ ## 现有代码模式分析                                           │
│ - 目录结构: src/features/[feature]/                         │
│ - 命名约定: PascalCase 组件, camelCase 工具                  │
│ - API 模式: RESTful, /api/v1/ 前缀                          │
│ - 数据库: Prisma ORM, PostgreSQL                            │
│                                                              │
│ ## 需要创建的文件                                             │
│                                                              │
│ 1. src/features/auth/                                        │
│    ├── components/                                           │
│    │   ├── LoginForm.tsx         # 登录表单组件              │
│    │   ├── RegisterForm.tsx       # 注册表单组件              │
│    │   └── AuthProvider.tsx      # 认证上下文                │
│    ├── api/                                                  │
│    │   ├── login/route.ts         # POST /api/v1/auth/login  │
│    │   ├── register/route.ts      # POST /api/v1/auth/register│
│    │   └── logout/route.ts        # POST /api/v1/auth/logout │
│    ├── hooks/                                                │
│    │   └── useAuth.ts             # 认证 Hook                │
│    ├── services/                                             │
│    │   └── authService.ts        # 业务逻辑                  │
│    ├── types/                                                │
│    │   └── auth.types.ts          # 类型定义                  │
│    └── tests/                                                │
│        └── auth.test.ts           # 单元测试                  │
│                                                              │
│ 2. prisma/                                                   │
│    └── migrations/                                           │
│        └── add_user_table/                                  │
│            └── migration.sql     # 用户表迁移                │
│                                                              │
│ ## 接口定义                                                   │
│                                                              │
│ // POST /api/v1/auth/login                                   │
│ Request:  { email: string, password: string }                │
│ Response: { user: User, token: string }                     │
│                                                              │
│ // POST /api/v1/auth/register                                │
│ Request:  { email: string, password: string, name: string }  │
│ Response: { user: User, token: string }                     │
│                                                              │
│ ## 数据流                                                     │
│                                                              │
│ [LoginForm] ──POST──▶ [API Route] ──▶ [AuthService]         │
│                              │               │               │
│                              │               ▼               │
│                              │         [Prisma]              │
│                              │               │               │
│                              ▼               ▼               │
│                         [JWT Token] ◀─── [Database]          │
│                              │                               │
│                              ▼                               │
│                    [Cookie: httpOnly]                        │
│                                                              │
│ ## 构建顺序                                                   │
│                                                              │
│ 1. prisma/schema.prisma - 添加 User model                   │
│ 2. prisma migrate dev - 创建迁移                            │
│ 3. src/features/auth/types/auth.types.ts                    │
│ 4. src/features/auth/services/authService.ts                │
│ 5. src/features/auth/api/login/route.ts                     │
│ 6. src/features/auth/api/register/route.ts                  │
│ 7. src/features/auth/hooks/useAuth.ts                       │
│ 8. src/features/auth/components/AuthProvider.tsx            │
│ 9. src/features/auth/components/LoginForm.tsx               │
│ 10. src/features/auth/tests/auth.test.ts                    │
│                                                              │
│ ## 测试策略                                                   │
│ - 单元测试: authService 业务逻辑                             │
│ - 集成测试: API routes                                       │
│ - E2E 测试: 登录/注册流程                                    │
└─────────────────────────────────────────────────────────────┘
```

---

## 综合使用流程

### 典型架构设计流程

```
┌─────────────────────────────────────────────────────────────┐
│ 用户需求: "设计电商订单系统架构"                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 1: 系统架构设计                                        │
│  → architect agent                                          │
│  → 输出: 架构模式选择、组件划分、技术决策                      │
│  → 推荐: Saga 编排模式 + 事件驱动                            │
│                                                              │
│  Step 2: API 接口设计                                        │
│  → api-design skill                                         │
│  → 输出: RESTful API 规范、端点定义、错误格式                 │
│                                                              │
│  Step 3: 后端架构模式                                        │
│  → backend-patterns skill                                   │
│  → 输出: 分层架构、数据访问层、服务层设计                     │
│                                                              │
│  Step 4: 数据库设计                                          │
│  → database-migrations skill                                │
│  → 输出: Schema 设计、迁移策略、索引优化                     │
│                                                              │
│  Step 5: 前端架构 (如需要)                                    │
│  → frontend-patterns skill                                  │
│  → 输出: 组件结构、状态管理、数据获取                        │
│                                                              │
│  Step 6: 具体功能架构                                        │
│  → code-architect agent                                     │
│  → 输出: 文件结构、接口定义、构建顺序                        │
│                                                              │
│  最终输出: 完整的架构蓝图                                     │
│  - 架构图                                                    │
│  - API 规范                                                  │
│  - 数据模型                                                  │
│  - 文件结构                                                  │
│  - 实现顺序                                                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Skill/Agent 协同关系

| Skill/Agent | 与其他的协同 |
|-------------|--------------|
| `architect` | 为所有其他 Skill 提供顶层架构决策 |
| `api-design` | 依赖 architect 的组件划分，为 backend-patterns 提供接口规范 |
| `backend-patterns` | 实现 architect 的架构决策，遵循 api-design 的接口规范 |
| `frontend-patterns` | 对接 api-design 的接口，实现 architect 的前端架构 |
| `database-migrations` | 实现 backend-patterns 的数据层，遵循 architect 的数据架构 |
| `hexagonal-architecture` | 实现 architect 的分层决策，指导 backend-patterns |
| `code-architect` | 整合所有设计，输出具体实现蓝图 |

---

## 架构决策记录 (ADR)

**用途：** 记录重要的架构决策及其背景

**ADR 模板：**

```markdown
# ADR-001: 使用 Saga 模式处理分布式事务

## 状态
已接受

## 背景
订单系统需要协调多个服务（订单、支付、库存）完成事务。
传统分布式事务（XA）在微服务架构中性能差、可用性低。

## 决策
采用 Saga 编排模式处理分布式事务。

## 考虑的方案

### 方案 A: 两阶段提交 (2PC)
- 优点: 强一致性
- 缺点: 性能差、单点故障、锁资源

### 方案 B: Saga 编排模式
- 优点: 高可用、无锁、可扩展
- 缺点: 最终一致性、补偿逻辑复杂

### 方案 C: Saga 编舞模式
- 优点: 去中心化
- 缺点: 调试困难、循环依赖风险

## 结果
选择方案 B (Saga 编编排模式)。

## 后果
- 正向: 高可用、可扩展、无单点故障
- 负向: 需要实现补偿逻辑、调试复杂度增加
- 风险: 补偿失败需要人工干预

## 参考
- https://microservices.io/patterns/data/saga.html
```

---

## 关键建议总结

| 原则 | 流程 |
|------|------|
| 先设计再构建 | `architect` → 架构蓝图 → 再实现 |
| API 优先设计 | `api-design` → 接口定义 → 前后端并行 |
| 选择合适模式 | `backend-patterns` → 模式决策树 → 应用 |
| 数据库安全变更 | `database-migrations` → 小步迁移 → 零停机 |
| 清晰架构边界 | `hexagonal-architecture` → 依赖反转 → 可测试 |
| 具体实现蓝图 | `code-architect` → 文件结构 → 构建顺序 |
| 记录架构决策 | ADR → 背景/决策/后果 → 可追溯 |

---

## 相关链接

- [ECC-GUIDE.md](./ECC-GUIDE.md) - ECC 项目总览
- [ECC-GUIDE-requirements-exploration.md](./ECC-GUIDE-requirements-exploration.md) - 需求理解阶段指南
- [GitHub 仓库](https://github.com/affaan-m/everything-claude-code)
- [官网](https://ecc.tools)
