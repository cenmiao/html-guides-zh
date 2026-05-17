# ECC 阶段 9：文档与发布详细指南

> 本文档基于 ECC 官方项目 https://github.com/affaan-m/everything-claude-code 整理

---

## 阶段目标

**核心任务：** 文档更新、发布准备、部署执行

**关键原则：**
- **单一真实来源** — 从代码生成文档，不手动编写生成内容
- **自动化优先** — CI/CD 管道自动化构建、测试、部署
- **可回滚设计** — 每次发布都有快速回滚能力
- **渐进式发布** — 使用蓝绿/金丝雀策略降低风险

---

## 文档更新 Agents

### doc-updater Agent

**用途：** 文档和代码地图专家，主动更新文档

**核心职责：**
1. **代码地图生成** — 从代码库结构创建架构地图
2. **文档更新** — 从代码刷新 README 和指南
3. **AST 分析** — 使用 TypeScript 编译器 API 理解结构
4. **依赖映射** — 跟踪模块间的导入/导出
5. **文档质量** — 确保文档与实际代码一致

**触发时机：**
- 新功能完成后
- API 路由变更后
- 架构变更后
- 依赖添加/删除后

**分析命令：**
```bash
npx tsx scripts/codemaps/generate.ts    # 生成代码地图
npx madge --image graph.svg src/        # 依赖关系图
npx jsdoc2md src/**/*.ts                # 提取 JSDoc
```

---

## 文档更新命令

### /update-docs

**用途：** 从真实来源文件同步文档

**来源与生成目标：**

| 来源 | 生成内容 |
|------|----------|
| `package.json` scripts | 可用命令参考 |
| `.env.example` | 环境变量文档 |
| `openapi.yaml` / 路由文件 | API 端点参考 |
| 源代码导出 | 公共 API 文档 |
| `Dockerfile` / `docker-compose.yml` | 基础设施设置文档 |

**工作流程：**

```
┌─────────────────────────────────────────────────────────────┐
│ Step 1: 识别真实来源                                         │
│    → package.json, .env.example, openapi.yaml, 源代码       │
├─────────────────────────────────────────────────────────────┤
│ Step 2: 生成脚本参考                                         │
│    → 提取所有脚本及描述                                       │
│    → 生成命令参考表                                          │
├─────────────────────────────────────────────────────────────┤
│ Step 3: 生成环境文档                                         │
│    → 提取所有变量及用途                                       │
│    → 分类为必需/可选                                         │
├─────────────────────────────────────────────────────────────┤
│ Step 4: 更新贡献指南                                         │
│    → 开发环境设置                                            │
│    → 可用脚本和用途                                          │
│    → 测试流程                                               │
├─────────────────────────────────────────────────────────────┤
│ Step 5: 更新运维手册                                         │
│    → 部署流程                                               │
│    → 健康检查端点                                           │
│    → 常见问题及修复                                         │
├─────────────────────────────────────────────────────────────┤
│ Step 6: 过时检查                                             │
│    → 查找 90+ 天未修改的文档                                  │
│    → 标记可能过时的文档                                      │
└─────────────────────────────────────────────────────────────┘
```

**输出示例：**
```
Documentation Update
──────────────────────────────
Updated:  docs/CONTRIBUTING.md (scripts table)
Updated:  docs/ENV.md (3 new variables)
Flagged:  docs/DEPLOY.md (142 days stale)
Skipped:  docs/API.md (no changes detected)
──────────────────────────────
```

---

### /update-codemaps

**用途：** 扫描项目结构并生成 Token 优化的架构代码地图

**工作流程：**

```
┌─────────────────────────────────────────────────────────────┐
│ Step 1: 扫描项目结构                                         │
│    → 识别项目类型 (monorepo, single app, library)           │
│    → 查找源目录 (src/, lib/, app/, packages/)               │
│    → 映射入口点 (main.ts, index.ts, app.py, main.go)        │
├─────────────────────────────────────────────────────────────┤
│ Step 2: 生成代码地图                                         │
│    → docs/CODEMAPS/architecture.md (系统概览)               │
│    → docs/CODEMAPS/backend.md (API 路由、服务映射)          │
│    → docs/CODEMAPS/frontend.md (页面树、组件层次)           │
│    → docs/CODEMAPS/data.md (数据库表、关系)                 │
│    → docs/CODEMAPS/dependencies.md (外部依赖)              │
├─────────────────────────────────────────────────────────────┤
│ Step 3: 差异检测                                             │
│    → 计算与旧地图的差异百分比                                 │
│    → 变更 > 30% 时请求用户确认                               │
│    → 变更 <= 30% 时直接更新                                  │
├─────────────────────────────────────────────────────────────┤
│ Step 4: 添加元数据                                           │
│    → 生成时间戳                                              │
│    → 扫描文件数                                              │
│    → Token 估算                                              │
├─────────────────────────────────────────────────────────────┤
│ Step 5: 保存分析报告                                         │
│    → .reports/codemap-diff.txt                              │
│    → 文件增删改记录                                          │
│    → 新依赖检测                                              │
│    → 过时警告                                               │
└─────────────────────────────────────────────────────────────┘
```

**代码地图格式：**
```markdown
# Backend Architecture

## Routes
POST /api/users -> UserController.create -> UserService.create -> UserRepo.insert
GET  /api/users/:id -> UserController.get -> UserService.findById -> UserRepo.findById

## Key Files
src/services/user.ts (business logic, 120 lines)
src/repos/user.ts (database access, 80 lines)

## Dependencies
- PostgreSQL (primary data store)
- Redis (session cache, rate limiting)
- Stripe (payment processing)
```

---

## 发布部署 Skills

### deployment-patterns Skill

**用途：** 部署工作流、CI/CD 管道模式、健康检查、回滚策略

**触发时机：**
- 设置 CI/CD 管道
- Docker 化应用
- 规划部署策略（蓝绿、金丝雀、滚动）
- 实现健康检查和就绪探针
- 准备生产发布
- 配置环境特定设置

---

## 部署策略

### 滚动部署 (Rolling Deployment)

**默认策略：** 逐步替换实例，新旧版本同时运行

```
Instance 1: v1 -> v2  (先更新)
Instance 2: v1        (仍运行 v1)
Instance 3: v1        (仍运行 v1)

Instance 1: v2
Instance 2: v1 -> v2  (第二个更新)
Instance 3: v1

Instance 1: v2
Instance 2: v2
Instance 3: v1 -> v2  (最后更新)
```

**优点：** 零停机、渐进式发布
**缺点：** 两版本同时运行，需要向后兼容
**适用场景：** 标准部署、向后兼容变更

---

### 蓝绿部署 (Blue-Green Deployment)

**策略：** 运行两个相同环境，原子切换流量

```
Blue  (v1) <- traffic
Green (v2)   idle, 运行新版本

# 验证后：
Blue  (v1)   idle (成为备用)
Green (v2) <- traffic
```

**优点：** 即时回滚（切回蓝色）、干净切换
**缺点：** 部署期间需要 2x 基础设施
**适用场景：** 关键服务、零容忍问题

---

### 金丝雀部署 (Canary Deployment)

**策略：** 先将小比例流量路由到新版本

```
v1: 95% of traffic
v2:  5% of traffic  (canary)

# 指标良好时：
v1: 50% of traffic
v2: 50% of traffic

# 最终：
v2: 100% of traffic
```

**优点：** 用真实流量在全面发布前发现问题
**缺点：** 需要流量分割基础设施、监控
**适用场景：** 高流量服务、风险变更、功能开关

---

## Docker 容器化模式

### 多阶段 Dockerfile (Node.js)

```dockerfile
# Stage 1: 安装依赖
FROM node:22-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --production=false

# Stage 2: 构建
FROM node:22-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build
RUN npm prune --production

# Stage 3: 生产镜像
FROM node:22-alpine AS runner
WORKDIR /app

RUN addgroup -g 1001 -S appgroup && adduser -S appuser -u 1001
USER appuser

COPY --from=builder --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --from=builder --chown=appuser:appgroup /app/package.json ./

ENV NODE_ENV=production
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/server.js"]
```

### 多阶段 Dockerfile (Go)

```dockerfile
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /server ./cmd/server

FROM alpine:3.19 AS runner
RUN apk --no-cache add ca-certificates
RUN adduser -D -u 1001 appuser
USER appuser

COPY --from=builder /server /server

EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://localhost:8080/health || exit 1
CMD ["/server"]
```

### 多阶段 Dockerfile (Python/Django)

```dockerfile
FROM python:3.12-slim AS builder
WORKDIR /app
RUN pip install --no-cache-dir uv
COPY requirements.txt .
RUN uv pip install --system --no-cache -r requirements.txt

FROM python:3.12-slim AS runner
WORKDIR /app

RUN useradd -r -u 1001 appuser
USER appuser

COPY --from=builder /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin
COPY . .

ENV PYTHONUNBUFFERED=1
EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=3s CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health/')" || exit 1
CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000", "--workers", "4"]
```

---

## Docker Compose 配置

### 标准开发栈

```yaml
# docker-compose.yml
services:
  app:
    build:
      context: .
      target: dev                     # 使用开发阶段
    ports:
      - "3000:3000"
    volumes:
      - .:/app                        # 绑定挂载用于热重载
      - /app/node_modules             # 匿名卷保护容器依赖
    environment:
      - DATABASE_URL=postgres://postgres:postgres@db:5432/app_dev
      - REDIS_URL=redis://redis:6379/0
      - NODE_ENV=development
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    command: npm run dev

  db:
    image: postgres:16-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app_dev
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redisdata:/data

  mailpit:                            # 本地邮件测试
    image: axllent/mailpit
    ports:
      - "8025:8025"                   # Web UI
      - "1025:1025"                   # SMTP

volumes:
  pgdata:
  redisdata:
```

### 开发/生产覆盖文件

```yaml
# docker-compose.override.yml (自动加载，仅开发设置)
services:
  app:
    environment:
      - DEBUG=app:*
      - LOG_LEVEL=debug
    ports:
      - "9229:9229"                   # Node.js 调试器

# docker-compose.prod.yml (显式用于生产)
services:
  app:
    build:
      target: production
    restart: always
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
```

**使用命令：**
```bash
# 开发 (自动加载 override)
docker compose up

# 生产
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

## CI/CD 管道配置

### GitHub Actions 标准管道

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run typecheck
      - run: npm test -- --coverage
      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: coverage
          path: coverage/

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - name: Deploy to production
        run: |
          # 平台特定部署命令
          # Railway: railway up
          # Vercel: vercel --prod
          # K8s: kubectl set image deployment/app app=ghcr.io/${{ github.repository }}:${{ github.sha }}
          echo "Deploying ${{ github.sha }}"
```

### 管道阶段流程

```
PR 打开:
  lint -> typecheck -> unit tests -> integration tests -> preview deploy

合并到 main:
  lint -> typecheck -> unit tests -> integration tests -> build image -> deploy staging -> smoke tests -> deploy production
```

---

## 健康检查配置

### 健康检查端点

```typescript
// 简单健康检查
app.get("/health", (req, res) => {
  res.status(200).json({ status: "ok" });
});

// 详细健康检查 (用于内部监控)
app.get("/health/detailed", async (req, res) => {
  const checks = {
    database: await checkDatabase(),
    redis: await checkRedis(),
    externalApi: await checkExternalApi(),
  };

  const allHealthy = Object.values(checks).every(c => c.status === "ok");

  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? "ok" : "degraded",
    timestamp: new Date().toISOString(),
    version: process.env.APP_VERSION || "unknown",
    uptime: process.uptime(),
    checks,
  });
});

async function checkDatabase(): Promise<HealthCheck> {
  try {
    await db.query("SELECT 1");
    return { status: "ok", latency_ms: 2 };
  } catch (err) {
    return { status: "error", message: "Database unreachable" };
  }
}
```

### Kubernetes 探针配置

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 30
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 10
  failureThreshold: 2

startupProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 0
  periodSeconds: 5
  failureThreshold: 30    # 30 * 5s = 150s 最大启动时间
```

---

## 环境配置

### 十二要素应用模式

```bash
# 所有配置通过环境变量 — 不在代码中
DATABASE_URL=postgres://user:pass@host:5432/db
REDIS_URL=redis://host:6379/0
API_KEY=${API_KEY}           # 由 secrets manager 注入
LOG_LEVEL=info
PORT=3000

# 环境特定行为
NODE_ENV=production          # 或 staging, development
APP_ENV=production           # 显式应用环境
```

### 配置验证

```typescript
import { z } from "zod";

const envSchema = z.object({
  NODE_ENV: z.enum(["development", "staging", "production"]),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  LOG_LEVEL: z.enum(["debug", "info", "warn", "error"]).default("info"),
});

// 启动时验证 — 配置错误时快速失败
export const env = envSchema.parse(process.env);
```

---

## 回滚策略

### 即时回滚命令

```bash
# Docker/Kubernetes: 指向上一镜像
kubectl rollout undo deployment/app

# Vercel: 提升上一部署
vercel rollback

# Railway: 重新部署上一提交
railway up --commit <previous-sha>

# 数据库: 回滚迁移 (如果可逆)
npx prisma migrate resolve --rolled-back <migration-name>
```

### 回滚检查清单

- [ ] 上一镜像/构件可用且已标记
- [ ] 数据库迁移向后兼容（无破坏性变更）
- [ ] 功能开关可在不部署情况下禁用新功能
- [ ] 监控警报配置了错误率峰值
- [ ] 回滚在生产发布前在 staging 测试过

---

## 生产就绪检查清单

### 应用层面

- [ ] 所有测试通过（单元、集成、E2E）
- [ ] 代码或配置文件中无硬编码密钥
- [ ] 错误处理覆盖所有边界情况
- [ ] 日志结构化（JSON）且不含 PII
- [ ] 健康检查端点返回有意义的状态

### 基础设施层面

- [ ] Docker 镜像可重复构建（固定版本）
- [ ] 环境变量已文档化并在启动时验证
- [ ] 资源限制已设置（CPU、内存）
- [ ] 水平扩展已配置（最小/最大实例）
- [ ] 所有端点启用 SSL/TLS

### 监控层面

- [ ] 应用指标已导出（请求率、延迟、错误）
- [ ] 错误率超过阈值时配置警报
- [ ] 日志聚合已设置（结构化日志、可搜索）
- [ ] 健康端点有正常运行监控

### 安全层面

- [ ] 依赖项已扫描 CVE
- [ ] CORS 仅配置允许的来源
- [ ] 公共端点启用速率限制
- [ ] 认证和授权已验证
- [ ] 安全头已设置（CSP, HSTS, X-Frame-Options）

### 运维层面

- [ ] 回滚计划已文档化并测试
- [ ] 数据库迁移已针对生产规模数据测试
- [ ] 常见故障场景有运维手册
- [ ] 值班轮换和升级路径已定义

---

## Docker 最佳实践

### 良好实践

```
- 使用特定版本标签 (node:22-alpine, 不是 node:latest)
- 多阶段构建最小化镜像大小
- 以非 root 用户运行
- 先复制依赖文件（层缓存）
- 使用 .dockerignore 排除 node_modules, .git, tests
- 添加 HEALTHCHECK 指令
- 在 docker-compose 或 k8s 中设置资源限制
```

### 不良实践

```
- 以 root 运行
- 使用 :latest 标签
- 在一个 COPY 层复制整个仓库
- 在生产镜像安装开发依赖
- 在镜像中存储密钥（使用环境变量或 secrets manager）
- 一个巨大容器运行所有服务
- 在 docker-compose.yml 中放置密钥
```

---

## 命令速查表

### 文档更新命令

| 命令 | 用途 | 使用时机 |
|------|------|----------|
| `/update-docs` | 从代码同步文档 | 功能完成后 |
| `/update-codemaps` | 生成架构代码地图 | 架构变更后 |
| `doc-updater` agent | 文档同步代理 | 自动文档维护 |

### Docker 常用命令

```bash
# 查看日志
docker compose logs -f app           # 跟踪应用日志
docker compose logs --tail=50 db     # 数据库最后 50 行

# 在运行容器中执行命令
docker compose exec app sh           # 进入应用 shell
docker compose exec db psql -U postgres  # 连接 postgres

# 检查
docker compose ps                     # 运行中的服务
docker compose top                    # 每个容器中的进程
docker stats                          # 资源使用

# 重建
docker compose up --build             # 重建镜像
docker compose build --no-cache app   # 强制完全重建

# 清理
docker compose down                   # 停止并删除容器
docker compose down -v                # 同时删除卷（破坏性）
docker system prune                   # 删除未使用的镜像/容器
```

### 网络调试命令

```bash
# 检查容器内 DNS 解析
docker compose exec app nslookup db

# 检查连接性
docker compose exec app wget -qO- http://api:3000/health

# 检查网络
docker network ls
docker network inspect <project>_default
```

---

## 典型发布流程示例

### 场景：发布新版本 v2.0.0

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 发布前准备                                                │
│    /update-docs                                              │
│    /update-codemaps                                          │
│    → 确保文档与代码同步                                       │
├─────────────────────────────────────────────────────────────┤
│ 2. CI/CD 自动执行                                            │
│    lint -> typecheck -> test -> build                        │
│    → 所有检查通过后构建 Docker 镜像                           │
├─────────────────────────────────────────────────────────────┤
│ 3. 部署到 Staging                                            │
│    docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
│    → 运行冒烟测试                                            │
│    → 验证健康检查端点                                        │
├─────────────────────────────────────────────────────────────┤
│ 4. 生产部署 (金丝雀)                                         │
│    → 5% 流量到新版本                                         │
│    → 监控错误率和延迟                                        │
│    → 逐步增加流量: 5% -> 25% -> 50% -> 100%                  │
├─────────────────────────────────────────────────────────────┤
│ 5. 发布后监控                                                │
│    → 检查日志聚合                                            │
│    → 验证警报阈值                                            │
│    → 准备回滚计划                                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 相关资源

- [ECC-GUIDE.md](./ECC-GUIDE.md) - ECC 主指南
- [ECC-GUIDE-build-debug.md](./ECC-GUIDE-build-debug.md) - 构建与调试指南
- [ECC-GUIDE-security.md](./ECC-GUIDE-security.md) - 安全审查指南
- [deployment-patterns skill](https://github.com/affaan-m/everything-claude-code/tree/main/skills/deployment-patterns)
- [docker-patterns skill](https://github.com/affaan-m/everything-claude-code/tree/main/skills/docker-patterns)

---

> 文档生成日期: 2026-05-17 | 基于 ECC 官方项目整理
