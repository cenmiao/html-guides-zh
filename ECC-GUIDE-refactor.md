# ECC 阶段 8：重构与清理详细指南

> 本文档基于 ECC 官方项目 https://github.com/affaan-m/everything-claude-code 整理

---

## 阶段目标

**核心任务：** 删除死代码、优化结构、清理依赖

**关键原则：**
- **安全优先** — 每次删除后验证测试
- **最小化变更** — 一次只删除一类代码
- **保守策略** — 不确定时保留代码
- **原子提交** — 每批清理单独提交

---

## 重构与清理 Agents

### 核心清理 Agent

| Agent | 用途 | 适用场景 |
|-------|------|----------|
| `refactor-cleaner` | 死代码检测与清理 | **项目维护、代码瘦身** |
| `code-simplifier` | 代码简化与优化 | 复杂逻辑简化 |
| `performance-optimizer` | 性能优化重构 | 性能瓶颈代码 |

### refactor-cleaner Agent 详解

**核心职责：**
1. **死代码检测** — 查找未使用的代码、导出、依赖
2. **重复消除** — 识别并合并重复代码
3. **依赖清理** — 移除未使用的包和导入
4. **安全重构** — 确保变更不破坏功能

**使用时机：**
- 项目维护阶段
- 功能开发完成后
- 代码审查前
- 发布准备期间

---

## 重构与清理 Skills

### 核心清理 Skills

| Skill | 用途 | 适用场景 |
|-------|------|----------|
| `plankton-code-quality` | 编写时质量强制 | **实时质量检查、自动格式化** |
| `ecc:refactor-clean` | 死代码清理命令 | 大规模清理 |

### plankton-code-quality Skill 详解

**三阶段架构：**

```
阶段 1: 自动格式化 (静默)
├─ 运行格式化工具 (ruff format, biome, shfmt, taplo)
├─ 静默修复 40-50% 的问题
└─ 不输出到主代理

阶段 2: 收集违规 (JSON)
├─ 运行 linter 并收集不可修复的违规
├─ 返回结构化 JSON: {line, column, code, message, linter}
└─ 仍不输出到主代理

阶段 3: 委托 + 验证
├─ 启动 claude -p 子进程处理违规
├─ 根据违规复杂度路由到模型层级:
│   ├─ Haiku: 格式化、导入、样式 (E/W/F codes) — 120s 超时
│   ├─ Sonnet: 复杂度、重构 (C901, PLR codes) — 300s 超时
│   └─ Opus: 类型系统、深度推理 (unresolved-attribute) — 600s 超时
├─ 重新运行阶段 1+2 验证修复
└─ 干净则退出 0，违规仍存在则退出 2
```

**配置保护（防止规则篡改）：**

LLM 可能修改 `.ruff.toml` 或 `biome.json` 来禁用规则而非修复代码。Plankton 通过三层保护：

1. **PreToolUse hook** — `protect_linter_configs.sh` 阻止对 linter 配置的编辑
2. **Stop hook** — `stop_config_guardian.sh` 在会话结束时通过 `git diff` 检测配置变更
3. **保护文件列表** — `.ruff.toml`, `biome.json`, `.shellcheckrc`, `.yamllint`, `.hadolint.yaml` 等

---

## 死代码检测工具

### 语言特定检测工具

| 工具 | 检测内容 | 命令 | 适用语言 |
|------|----------|------|----------|
| **knip** | 未使用的导出、文件、依赖 | `npx knip` | TypeScript/JavaScript |
| **depcheck** | 未使用的 npm 依赖 | `npx depcheck` | TypeScript/JavaScript |
| **ts-prune** | 未使用的 TypeScript 导出 | `npx ts-prune` | TypeScript |
| **vulture** | 未使用的 Python 代码 | `vulture src/` | Python |
| **deadcode** | 未使用的 Go 代码 | `deadcode ./...` | Go |
| **cargo-udeps** | 未使用的 Rust 依赖 | `cargo +nightly udeps` | Rust |
| **knip** | 未使用的导出、依赖 | `npx knip` | TypeScript |

### 检测命令详解

#### TypeScript/JavaScript

```bash
# knip - 综合检测
npx knip                                    # 检测未使用的文件、导出、依赖
npx knip --include=files,exports,deps       # 指定检测类型
npx knip --fix                              # 自动修复（谨慎使用）

# depcheck - 依赖检测
npx depcheck                                # 检测未使用的 npm 依赖
npx depcheck --ignore-bin-package          # 忽略 bin 包
npx depcheck --json                        # JSON 输出

# ts-prune - TypeScript 导出检测
npx ts-prune                                # 检测未使用的 TypeScript 导出
npx ts-prune -p tsconfig.json              # 指定 tsconfig

# ESLint 未使用检测
npx eslint . --rule 'no-unused-vars: error'
npx eslint . --report-unused-disable-directives
```

#### Python

```bash
# vulture - 死代码检测
vulture src/                                # 检测未使用的 Python 代码
vulture src/ --min-confidence 80           # 设置最小置信度
vulture src/ --exclude tests/              # 排除测试目录

# pip-autoremove - 依赖清理
pip-autoremove                              # 移除未使用的包

# pyflakes - 快速检测
pyflakes src/                               # 检测未使用的导入和变量
```

#### Go

```bash
# deadcode - 死代码检测
deadcode ./...                              # 检测未使用的 Go 代码

# go vet - 静态分析
go vet ./...                                # 检测可疑代码结构

# depcheck - 依赖检测
go mod tidy -v                              # 清理未使用的依赖
```

#### Rust

```bash
# cargo-udeps - 未使用依赖检测
cargo +nightly udeps                        # 检测未使用的 Rust 依赖
cargo +nightly udeps --all-targets          # 包含测试和示例

# cargo-machete - 快速依赖检测
cargo machete                               # 快速检测未使用依赖
```

---

## 重构流程

### 标准重构流程

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 分析阶段                                                   │
│    → 并行运行检测工具                                         │
│    → 按风险分类: SAFE / CAREFUL / RISKY                      │
├─────────────────────────────────────────────────────────────┤
│ 2. 验证阶段                                                   │
│    → Grep 检查所有引用（包括动态导入）                         │
│    → 检查是否属于公共 API                                     │
│    → 查看 git 历史了解上下文                                   │
├─────────────────────────────────────────────────────────────┤
│ 3. 安全删除                                                   │
│    → 只从 SAFE 项开始                                         │
│    → 一次删除一类: deps -> exports -> files -> duplicates    │
│    → 每批后运行测试                                           │
│    → 每批后提交                                               │
├─────────────────────────────────────────────────────────────┤
│ 4. 合并重复                                                   │
│    → 查找重复组件/工具函数                                     │
│    → 选择最佳实现（最完整、测试最好）                          │
│    → 更新所有导入，删除重复                                    │
│    → 验证测试通过                                             │
├─────────────────────────────────────────────────────────────┤
│ 5. 总结报告                                                   │
│    → 统计删除的代码行数                                        │
│    → 列出删除的文件和依赖                                      │
│    → 确认所有测试通过                                         │
└─────────────────────────────────────────────────────────────┘
```

### 风险分类

| 层级 | 示例 | 操作 |
|------|------|------|
| **SAFE** | 未使用的工具函数、测试辅助、内部函数 | 放心删除 |
| **CAREFUL** | 组件、API 路由、中间件 | 验证无动态导入或外部消费者 |
| **RISKY** | 配置文件、入口点、类型定义 | 调查后再处理 |

---

## 安全重构原则

### 删除前检查清单

- [ ] 检测工具确认未使用
- [ ] Grep 确认无引用（包括动态导入）
- [ ] 不属于公共 API
- [ ] 测试在删除后通过

### 每批删除后检查

- [ ] 构建成功
- [ ] 测试通过
- [ ] 使用描述性提交信息提交

### 核心原则

1. **从小开始** — 一次只处理一类
2. **频繁测试** — 每批后测试
3. **保守策略** — 不确定时保留
4. **文档记录** — 每批使用描述性提交信息
5. **时机选择** — 不在活跃功能开发或部署前删除

### 禁止行为

| 行为 | 原因 |
|------|------|
| 在活跃功能开发期间删除 | 可能删除正在使用的代码 |
| 在生产部署前删除 | 风险窗口太短 |
| 没有测试覆盖时删除 | 无法验证功能完整性 |
| 删除不理解的代码 | 可能是隐式依赖 |

---

## 命令速查表

### 重构清理命令

| 命令 | 用途 | 适用场景 |
|------|------|----------|
| `/refactor-clean` | 死代码清理 | 项目维护、代码瘦身 |
| `ecc:refactor-clean` | 死代码清理命令 | 大规模清理 |

### 检测命令

| 命令 | 用途 |
|------|------|
| `npx knip` | TypeScript 未使用代码检测 |
| `npx depcheck` | npm 未使用依赖检测 |
| `npx ts-prune` | TypeScript 未使用导出检测 |
| `vulture src/` | Python 死代码检测 |
| `deadcode ./...` | Go 死代码检测 |
| `cargo +nightly udeps` | Rust 未使用依赖检测 |

---

## 典型重构场景示例

### 场景 1：TypeScript 项目清理

```
┌─────────────────────────────────────────────────────────────┐
│ npm run build 成功，准备清理                                 │
├─────────────────────────────────────────────────────────────┤
│ 1. 运行检测工具                                              │
│    npx knip                                                  │
│    npx depcheck                                              │
│    npx ts-prune                                              │
│    → 发现: 5 个未使用导出，3 个未使用依赖                     │
├─────────────────────────────────────────────────────────────┤
│ 2. 分类                                                      │
│    SAFE: utils/deprecated.ts, old-helper.ts                  │
│    CAREFUL: components/LegacyButton.tsx                      │
│    RISKY: types/legacy.d.ts                                  │
├─────────────────────────────────────────────────────────────┤
│ 3. 删除 SAFE 项                                              │
│    rm utils/deprecated.ts                                    │
│    npm test → PASS                                           │
│    git commit -m "remove: deprecated utils"                  │
├─────────────────────────────────────────────────────────────┤
│ 4. 清理依赖                                                  │
│    npm uninstall unused-dep-1 unused-dep-2 unused-dep-3     │
│    npm test → PASS                                           │
│    git commit -m "chore: remove unused dependencies"         │
├─────────────────────────────────────────────────────────────┤
│ 5. 总结                                                      │
│    删除: 5 个未使用函数，3 个未使用依赖                       │
│    节省: ~200 行代码                                          │
│    状态: 所有测试通过 PASS                                    │
└─────────────────────────────────────────────────────────────┘
```

### 场景 2：Python 项目清理

```
┌─────────────────────────────────────────────────────────────┐
│ Python 项目维护清理                                          │
├─────────────────────────────────────────────────────────────┤
│ 1. 运行检测工具                                              │
│    vulture src/ --min-confidence 80                          │
│    pyflakes src/                                             │
│    → 发现: 10 个未使用函数，5 个未使用导入                    │
├─────────────────────────────────────────────────────────────┤
│ 2. 验证                                                      │
│    grep -r "unused_function" src/                            │
│    grep -r "import unused_module" src/                       │
│    → 确认无动态引用                                          │
├─────────────────────────────────────────────────────────────┤
│ 3. 删除                                                      │
│    # 删除未使用函数                                          │
│    # 移除未使用导入                                          │
│    pytest → PASS                                             │
│    git commit -m "refactor: remove dead code"                │
├─────────────────────────────────────────────────────────────┤
│ 4. 清理依赖                                                  │
│    pip-autoremove                                            │
│    pip freeze > requirements.txt                             │
│    pytest → PASS                                             │
└─────────────────────────────────────────────────────────────┘
```

### 场景 3：合并重复代码

```
┌─────────────────────────────────────────────────────────────┐
│ 发现重复组件                                                  │
│    components/Button.tsx                                     │
│    components/ButtonV2.tsx                                   │
│    shared/Button.tsx                                          │
├─────────────────────────────────────────────────────────────┤
│ 1. 分析                                                      │
│    → 比较三个组件的功能                                       │
│    → ButtonV2 最完整，测试最好                                │
│    → 其他两个是旧版本                                         │
├─────────────────────────────────────────────────────────────┤
│ 2. 选择最佳实现                                               │
│    → 保留 ButtonV2，重命名为 Button                           │
│    → 更新所有导入                                             │
├─────────────────────────────────────────────────────────────┤
│ 3. 迁移                                                      │
│    grep -r "from.*Button" src/                               │
│    → 更新所有导入路径                                         │
│    → 删除旧文件                                               │
├─────────────────────────────────────────────────────────────┤
│ 4. 验证                                                      │
│    npm test → PASS                                           │
│    npm run build → SUCCESS                                    │
│    git commit -m "refactor: consolidate Button components"   │
└─────────────────────────────────────────────────────────────┘
```

---

## Plankton 与 ECC 配合

### 互补关系

| 关注点 | ECC | Plankton |
|--------|-----|----------|
| 代码质量强制 | PostToolUse hooks (Prettier, tsc) | PostToolUse hooks (20+ linters + 子进程修复) |
| 安全扫描 | AgentShield, security-reviewer agent | Bandit (Python), Semgrep (TypeScript) |
| 配置保护 | — | PreToolUse 阻止 + Stop hook 检测 |
| 包管理器 | 检测 + 设置 | 强制（阻止旧版 PM） |
| CI 集成 | — | Git pre-commit hooks |
| 模型路由 | 手动 (`/model opus`) | 自动（违规复杂度 -> 层级） |

### 推荐组合

1. 安装 ECC 作为插件（agents, skills, commands, rules）
2. 添加 Plankton hooks 用于编写时质量强制
3. 使用 AgentShield 进行安全审计
4. 使用 ECC 的 verification-loop 作为 PR 前的最终门控

---

## 输出格式

### 标准输出格式

```text
Dead Code Cleanup
──────────────────────────────
Deleted:   12 unused functions
           3 unused files
           5 unused dependencies
Skipped:   2 items (tests failed)
Saved:     ~450 lines removed
──────────────────────────────
All tests passing PASS:
```

### 最终状态报告

```text
Cleanup Status: SUCCESS/FAILED | Items Removed: N | Files Modified: list
```

---

## 相关 Skills

| Skill | 用途 |
|-------|------|
| `plankton-code-quality` | 编写时质量强制 |
| `golang-patterns` | Go 语言最佳实践 |
| `rust-patterns` | Rust 模式和错误处理 |
| `python-patterns` | Python 最佳实践 |
| `springboot-patterns` | Spring Boot 模式 |

---

## 相关链接

- **GitHub 仓库：** https://github.com/affaan-m/everything-claude-code
- **官网：** https://ecc.tools
- **作者：** [@affaanmustafa](https://x.com/affaanmustafa)
- **Plankton 原作者：** @alxfazio

---

**Star 这个项目如果有帮助。构建伟大的东西。**
