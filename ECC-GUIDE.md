# Everything Claude Code (ECC) 项目导览

> 本文档基于 ECC 官方项目 https://github.com/affaan-m/everything-claude-code 整理

---

## 项目概览

| 属性 | 数据 |
|------|------|
| **Stars** | 180K+ |
| **Forks** | 27K+ |
| **贡献者** | 170+ |
| **主要语言** | JavaScript (56%), Rust (34%), Python (5%) |
| **官网** | https://ecc.tools |
| **来源** | Anthropic Hackathon 获奖项目 |

### 核心定位

**ECC 是一个 AI Agent Harness 性能优化系统**，不只是配置文件，而是一个完整的系统，包含：

- **Skills (技能)** - 主要工作流表面
- **Instincts (本能)** - 持续学习系统
- **Memory (记忆)** - 跨会话持久化
- **Security (安全)** - AgentShield 安全扫描
- **Research-first development** - 研究优先开发

---

## 组件统计

| 组件 | 数量 |
|------|------|
| **Agents** | 58 个专业子代理 |
| **Skills** | 220+ 个工作流技能 |
| **Commands** | 74 个斜杠命令 |
| **Rules** | 34 条规则 |
| **Hook Events** | 8-20+ 种事件类型 |
| **MCP Servers** | 14 个预配置服务器 |

---

## 跨平台支持

| 平台 | 支持状态 |
|------|----------|
| **Claude Code** | ✅ 原生支持 (主要目标) |
| **Cursor IDE** | ✅ 完整支持 |
| **Codex (App + CLI)** | ✅ 一流支持 |
| **OpenCode** | ✅ 完整插件支持 |
| **Gemini CLI** | 🧪 实验性支持 |
| **Antigravity** | ✅ 紧密集成 |

---

## 安装方式

### 推荐：插件安装

```bash
/plugin marketplace add https://github.com/affaan-m/everything-claude-code
/plugin install ecc@ecc
```

### 手动安装

```bash
git clone https://github.com/affaan-m/everything-claude-code.git
./install.sh --profile full
```

---

## ECC 开发生命周期工作流

### 阶段 1：需求理解与探索

**目标：** 理解项目背景、现有代码结构、技术栈

> 📖 **详细指南：** [ECC-GUIDE-requirements-exploration.md](./ECC-GUIDE-requirements-exploration.md) - 需求理解与探索阶段的完整使用手册

| 命令/Skill | 用途 | 使用时机 |
|------------|------|----------|
| `/ecc:plan` | 创建实现计划，分解任务 | 接到新需求时，先规划再动手 |
| `search-first` skill | 研究优先，先搜索再编码 | 任何编码前，先研究现有方案 |
| `documentation-lookup` skill | 查询最新 API 文档 | 需要了解库/框架用法时 |
| `deep-research` skill | 多源研究 + 来源引用 | 复杂需求，需要多方资料综合 |
| `codebase-onboarding` skill | 快速理解项目结构 | 新项目/新团队入职时 |
| `market-research` skill | 市场/竞品研究 | 产品需求分析阶段 |

**典型流程：**
```
/ecc:plan "添加用户认证功能"
→ planner agent 分析需求，生成实现蓝图
→ search-first skill 研究现有认证方案
→ documentation-lookup 查询 OAuth/JWT 最新文档
```

---

### 阶段 2：架构设计

**目标：** 系统设计、技术决策、架构蓝图

> 📖 **详细指南：** [ECC-GUIDE-architecture-design.md](./ECC-GUIDE-architecture-design.md) - 架构设计阶段的完整使用手册

| 命令/Skill | 用途 | 使用时机 |
|------------|------|----------|
| `architect` agent | 系统设计决策 | 复杂架构需要专业设计时 |
| `api-design` skill | REST API 设计规范 | 设计 API 接口时 |
| `backend-patterns` skill | 后端架构模式 | 选择数据库、缓存、消息队列方案 |
| `frontend-patterns` skill | 前端架构模式 | React/Next.js 组件设计 |
| `database-migrations` skill | 数据库迁移策略 | 涉及 schema 变更时 |
| `hexagonal-architecture` skill | 六边形架构 | 需要清晰分层架构时 |

**典型流程：**
```
/ecc:plan "设计微服务架构"
→ 调用 architect agent 进行系统设计
→ api-design skill 定义 API 规范
→ backend-patterns skill 选择技术栈
```

---

### 阶段 3：开发阶段

> 📖 **详细指南：** [ECC-GUIDE-development.md](./ECC-GUIDE-development.md) - 开发阶段的完整使用手册

#### 通用开发原则

| 命令/Skill | 用途 | 使用时机 |
|------------|------|----------|
| `tdd-workflow` skill | 测试驱动开发 | **任何编码前，先写测试** |
| `coding-standards` skill | 编码规范 | 编码过程中持续遵循 |
| `strategic-compact` skill | 战略性压缩 | 研究完成后、实现前压缩上下文 |

#### 后端开发

| 命令/Skill | 用途 | 适用场景 |
|------------|------|----------|
| `golang-patterns` skill | Go 语言最佳实践 | Go 项目 |
| `python-patterns` skill | Python 最佳实践 | Python 项目 |
| `django-patterns` / `django-tdd` | Django 开发 | Django 项目 |
| `springboot-patterns` / `springboot-tdd` | Spring Boot 开发 | Java 项目 |
| `fastapi-patterns` skill | FastAPI 开发 | Python API 项目 |
| `postgres-patterns` skill | PostgreSQL 优化 | 数据库优化 |
| `redis-patterns` skill | Redis 缓存模式 | 缓存设计 |

#### 前端开发

| 命令/Skill | 用途 | 适用场景 |
|------------|------|----------|
| `frontend-patterns` skill | React/Next.js 模式 | 前端项目 |
| `frontend-slides` skill | HTML 演示文稿 | 需要展示/文档时 |
| `liquid-glass-design` skill | iOS 26 设计系统 | iOS 开发 |
| `swiftui-patterns` skill | SwiftUI 模式 | Swift 前端 |

**典型 TDD 流程：**
```
tdd-workflow skill
→ 1. 定义接口
→ 2. 写失败测试 (RED)
→ 3. 最小实现 (GREEN)
→ 4. 重构 (IMPROVE)
→ 5. 验证 80%+ 覆盖率
```

---

### 阶段 4：代码审查

**目标：** 质量、安全、可维护性检查

> 📖 **详细指南：** [ECC-GUIDE-code-review.md](./ECC-GUIDE-code-review.md) - 代码审查阶段的完整使用手册

| 命令/Skill | 用途 | 使用时机 |
|------------|------|----------|
| `/code-review` | 代码质量审查 | **每次代码变更后立即审查** |
| `/go-review` | Go 代码专项审查 | Go 项目 |
| `/python-review` | Python 代码审查 | Python 项目 |
| `typescript-reviewer` agent | TypeScript 审查 | TS/JS 项目 |
| `java-reviewer` agent | Java 审查 | Java 项目 |
| `rust-reviewer` agent | Rust 审查 | Rust 项目 |
| `cpp-reviewer` agent | C++ 审查 | C++ 项目 |
| `kotlin-reviewer` agent | Kotlin 审查 | Kotlin/Android/KMP 项目 |
| `django-reviewer` agent | Django 审查 | Django 项目 |
| `database-reviewer` agent | 数据库查询审查 | SQL/数据库变更 |
| `security-reviewer` agent | 安全漏洞分析 | 涉及用户输入/认证的代码 |
| `security-review` skill | 安全检查清单 | 安全审计时 |

**审查流程：**
```
完成代码变更后立即：
/code-review → code-reviewer agent 检查质量/安全/可维护性
→ 发现问题 → 修复 → 再次审查
→ 语言特定审查 (go-reviewer/python-reviewer/typescript-reviewer 等)
→ 安全审查 (security-reviewer)
→ 数据库审查 (database-reviewer)
```

---

### 阶段 5：构建与调试

**目标：** 解决构建错误、编译问题

| 命令/Skill | 用途 | 使用时机 |
|------------|------|----------|
| `/build-fix` | 修复构建错误 | 构建失败时 |
| `/go-build` | Go 构建错误修复 | Go 项目构建失败 |
| `cpp-build-resolver` agent | C++ 构建/CMake 错误 | C++ 项目 |
| `rust-build-resolver` agent | Rust cargo 构建错误 | Rust 项目 |
| `java-build-resolver` agent | Maven/Gradle 错误 | Java 项目 |
| `kotlin-build-resolver` agent | Kotlin/Gradle 错误 | Kotlin 项目 |
| `pytorch-build-resolver` agent | PyTorch/CUDA 错误 | ML 训练崩溃 |

**调试流程：**
```
构建失败 → /build-fix
→ build-error-resolver agent 分析错误
→ 提供修复方案 → 应用修复 → 验证构建成功
```

---

### 阶段 6：测试阶段

**目标：** 单元测试、集成测试、E2E 测试、覆盖率

| 命令/Skill | 用途 | 使用时机 |
|------------|------|----------|
| `/test-coverage` | 测试覆盖率分析 | 验证 80%+ 覆盖率要求 |
| `e2e-testing` skill | Playwright E2E 测试 | 关键用户流程测试 |
| `golang-testing` skill | Go 测试模式 | Go 项目 |
| `python-testing` skill | pytest 测试 | Python 项目 |
| `cpp-testing` skill | GoogleTest/CMake 测试 | C++ 项目 |
| `verification-loop` skill | 持续验证循环 | 持续构建/测试/lint/安全检查 |
| `/quality-gate` | 质量门禁 | 发布前的验证关卡 |

**测试流程：**
```
/test-coverage → 验证覆盖率达标
→ e2e-testing skill → Playwright 测试关键流程
→ /quality-gate → 发布前质量门禁
```

---

### 阶段 7：安全审计

**目标：** 漏洞检测、OWASP Top 10、配置安全

| 命令/Skill | 用途 | 使用时机 |
|------------|------|----------|
| `/security-scan` | AgentShield 安全扫描 | **任何涉及用户输入/认证/API 的代码** |
| `security-review` skill | 安全检查清单 | 安全审计时 |
| `django-security` / `laravel-security` | 框架专项安全 | 框架项目 |
| `agentshield` CLI | 深度安全审计 | CI/CD 集成 |

**安全流程：**
```
/security-scan → AgentShield 扫描
→ 检测：secrets、SSRF、injection、OWASP Top 10
→ npx ecc-agentshield scan --opus → 三代理红队审计
```

---

### 阶段 8：重构与清理

**目标：** 删除死代码、优化结构

| 命令/Skill | 用途 | 使用时机 |
|------------|------|----------|
| `/refactor-clean` | 死代码清理 | 项目维护、代码瘦身 |
| `refactor-cleaner` agent | 死代码清理代理 | 大规模清理 |
| `plankton-code-quality` skill | 编写时质量强制 | 实时质量检查 |

---

### 阶段 9：文档与发布

**目标：** 文档更新、发布准备

| 命令/Skill | 用途 | 使用时机 |
|------------|------|----------|
| `/update-docs` | 更新文档 | 功能完成后 |
| `/update-codemaps` | 更新代码地图 | 架构变更后 |
| `doc-updater` agent | 文档同步代理 | 自动文档维护 |
| `deployment-patterns` skill | CI/CD、Docker、回滚 | 发布部署 |
| `docker-patterns` skill | Docker 模式 | 容器化部署 |

---

### 阶段 10：持续学习

**目标：** 从会话中提取模式、进化技能

| 命令/Skill | 用途 | 使用时机 |
|------------|------|----------|
| `/learn` | 会话中提取模式 | 开发过程中发现新模式 |
| `/learn-eval` | 提取 + 评估 + 保存 | 更严格的模式提取 |
| `/instinct-status` | 查看学习的本能 | 查看积累的经验 |
| `/evolve` | 聚类本能成技能 | 本能积累足够后 |
| `/instinct-export` | 导出本能分享 | 团队知识共享 |

---

## 完整开发周期示例

### 场景：添加用户认证功能

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 需求理解                                                  │
│    /ecc:plan "添加 OAuth 用户认证"                           │
│    → planner agent 生成实现蓝图                              │
│    → search-first skill 研究现有方案                         │
│    → documentation-lookup 查询 OAuth 文档                    │
├─────────────────────────────────────────────────────────────┤
│ 2. 架构设计                                                  │
│    → architect agent 设计认证架构                            │
│    → api-design skill 定义 API 规范                          │
│    → backend-patterns skill 选择 JWT/Session 方案            │
├─────────────────────────────────────────────────────────────┤
│ 3. 开发 (TDD)                                                │
│    → tdd-workflow skill                                      │
│    → 1. 定义接口                                             │
│    → 2. 写失败测试                                           │
│    → 3. 最小实现                                             │
│    → 4. 重构                                                 │
│    → 5. 验证覆盖率                                           │
├─────────────────────────────────────────────────────────────┤
│ 4. 代码审查                                                  │
│    /code-review → code-reviewer agent                        │
│    → 检查质量、安全、可维护性                                 │
├─────────────────────────────────────────────────────────────┤
│ 5. 构建                                                      │
│    → 构建失败？/build-fix                                    │
│    → build-error-resolver agent                              │
├─────────────────────────────────────────────────────────────┤
│ 6. 测试                                                      │
│    /test-coverage → 验证 80%+                                │
│    → e2e-testing skill → Playwright 登录流程测试              │
│    /quality-gate → 发布前门禁                                │
├─────────────────────────────────────────────────────────────┤
│ 7. 安全审计                                                  │
│    /security-scan → AgentShield                              │
│    → 检测 secrets、injection、OWASP                          │
│    → npx ecc-agentshield scan --opus                         │
├─────────────────────────────────────────────────────────────┤
│ 8. 文档与发布                                                │
│    /update-docs → 更新文档                                   │
│    → deployment-patterns skill → CI/CD                       │
├─────────────────────────────────────────────────────────────┤
│ 9. 持续学习                                                  │
│    /learn → 提取认证实现模式                                  │
│    /instinct-status → 查看积累                               │
│    /evolve → 聚类成可复用技能                                 │
└─────────────────────────────────────────────────────────────┘
```

---

## 核心建议

### 1. 永远先规划再编码

```
/ecc:plan → planner → blueprint → 再动手
```

### 2. 永远先研究再编码

```
search-first → documentation-lookup → 再编码
```

### 3. 永远先写测试再实现

```
tdd-workflow → RED → GREEN → IMPROVE
```

### 4. 每次代码变更后立即审查

```
写完代码 → /code-review → 修复 → 再审查
```

### 5. 任何涉及安全的代码必须扫描

```
认证/API/用户输入 → /security-scan → AgentShield
```

### 6. 发布前必须通过质量门禁

```
/quality-gate → build + test + lint + security
```

### 7. 从每个会话中学习

```
/learn → /instinct-status → /evolve
```

---

## Token 优化建议

```json
{
  "model": "sonnet",
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50"
  }
}
```

| 设置 | 默认值 | 推荐值 | 影响 |
|------|--------|--------|------|
| `model` | opus | **sonnet** | ~60% 成本降低，处理 80%+ 编码任务 |
| `MAX_THINKING_TOKENS` | 31,999 | **10,000** | ~70% 隐藏思考成本降低 |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` | 95 | **50** | 更早压缩，长会话质量更好 |

---

## 持续学习系统 v2

```bash
/instinct-status        # 查看学习的本能及置信度
/instinct-import <file> # 导入他人的本能
/instinct-export        # 导出你的本能用于分享
/evolve                 # 将相关本能聚类成技能
/prune                  # 删除过期的待定本能 (30天 TTL)
```

---

## AgentShield 安全审计

```bash
npx ecc-agentshield scan          # 快速扫描
npx ecc-agentshield scan --fix    # 自动修复
npx ecc-agentshield scan --opus   # 三代理红队审计
```

**扫描范围：**
- Secrets 检测 (14 种模式)
- 权限审计
- Hook 注入分析
- MCP 服务器风险分析
- Agent 配置审查
- OWASP Top 10

---

## 主要 Agents 速查表

| Agent | 用途 |
|-------|------|
| `planner` | 功能实现规划 |
| `architect` | 系统设计决策 |
| `tdd-guide` | 测试驱动开发 |
| `code-reviewer` | 代码质量审查 |
| `security-reviewer` | 安全漏洞分析 |
| `build-error-resolver` | 构建错误解决 |
| `e2e-runner` | Playwright E2E 测试 |
| `refactor-cleaner` | 死代码清理 |
| `doc-updater` | 文档同步 |
| `docs-lookup` | 文档/API 查询 |
| `go-reviewer` | Go 代码审查 |
| `python-reviewer` | Python 代码审查 |
| `typescript-reviewer` | TypeScript 审查 |
| `java-reviewer` | Java 审查 |
| `rust-reviewer` | Rust 审查 |
| `cpp-reviewer` | C++ 审查 |
| `database-reviewer` | 数据库审查 |

---

## 相关链接

- **GitHub 仓库：** https://github.com/affaan-m/everything-claude-code
- **官网：** https://ecc.tools
- **Shorthand Guide：** [The Shorthand Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2012378465664745795)
- **Longform Guide：** [The Longform Guide to Everything Claude Code](https://x.com/affaanmustafa/status/2014040193557471352)
- **Security Guide：** [Security Guide](https://x.com/affaanmustafa/status/2033263813387223421)
- **作者：** [@affaanmustafa](https://x.com/affaanmustafa)

---

## 许可证

MIT License - 自由使用、修改、贡献

---

**Star 这个项目如果有帮助。阅读两份指南。构建伟大的东西。**
