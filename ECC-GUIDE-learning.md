# ECC 阶段 10：持续学习详细指南

> 本文档基于 ECC 官方项目 https://github.com/affaan-m/everything-claude-code 整理

---

## 阶段目标

**核心任务：** 从会话中提取模式、进化技能、积累经验

**关键原则：**
- **原子化学习** — 小而精的"本能"(Instincts) 而非大而全的技能
- **置信度加权** — 每个本能都有 0.3-0.9 的置信度分数
- **项目隔离** — 项目特定的模式留在项目内，通用模式共享全局
- **渐进进化** — 本能聚类成技能、命令、代理

---

## 持续学习系统概述

### 什么是持续学习？

ECC 的持续学习系统是一个**自动从开发会话中提取可复用模式**的机制。它通过观察你的工作方式，逐步学习并形成"本能"，最终进化为完整的技能、命令或代理。

### 版本演进

| 版本 | 观察方式 | 分析粒度 | 置信度 | 作用域 |
|------|----------|----------|--------|--------|
| v1 | Stop Hook (会话结束) | 完整技能 | 无 | 全局 |
| v2 | PreToolUse/PostToolUse (100% 可靠) | 原子本能 | 0.3-0.9 | 全局 |
| v2.1 | 同 v2 + 项目检测 | 同 v2 | 同 v2 | **项目隔离 + 全局** |

### 核心架构

```
会话活动 (在 git 仓库中)
      |
      | Hooks 捕获提示 + 工具调用 (100% 可靠)
      | + 检测项目上下文 (git remote / repo path)
      v
+---------------------------------------------+
|  projects/<project-hash>/observations.jsonl  |
|   (提示、工具调用、结果、项目)                 |
+---------------------------------------------+
      |
      | 观察代理读取 (后台, Haiku)
      v
+---------------------------------------------+
|          模式检测                            |
|   * 用户纠正 -> 本能                         |
|   * 错误解决 -> 本能                         |
|   * 重复工作流 -> 本能                       |
|   * 作用域决策: 项目还是全局?                |
+---------------------------------------------+
      |
      | 创建/更新
      v
+---------------------------------------------+
|  projects/<project-hash>/instincts/personal/ |
|   * prefer-functional.yaml (0.7) [项目]     |
|   * use-react-hooks.yaml (0.9) [项目]       |
+---------------------------------------------+
|  instincts/personal/  (全局)                |
|   * always-validate-input.yaml (0.85) [全局]|
|   * grep-before-edit.yaml (0.6) [全局]      |
+---------------------------------------------+
      |
      | /evolve 聚类 + /promote 提升
      v
+---------------------------------------------+
|  projects/<hash>/evolved/ (项目作用域)       |
|  evolved/ (全局)                            |
|   * commands/new-feature.md                 |
|   * skills/testing-workflow.md              |
|   * agents/refactor-specialist.md           |
+---------------------------------------------+
```

---

## 本能 (Instincts) 概念与管理

### 什么是本能？

本能是一个**小型学习行为**，包含：

```yaml
---
id: prefer-functional-style
trigger: "when writing new functions"
confidence: 0.7
domain: "code-style"
source: "session-observation"
scope: project
project_id: "a1b2c3d4e5f6"
project_name: "my-react-app"
---

# Prefer Functional Style

## Action
Use functional patterns over classes when appropriate.

## Evidence
- Observed 5 instances of functional pattern preference
- User corrected class-based approach to functional on 2025-01-15
```

### 本能属性

| 属性 | 说明 |
|------|------|
| **Atomic (原子性)** | 一个触发器，一个动作 |
| **Confidence-weighted (置信度加权)** | 0.3 = 初步，0.9 = 近乎确定 |
| **Domain-tagged (领域标签)** | code-style, testing, git, debugging, workflow 等 |
| **Evidence-backed (证据支持)** | 追踪创建它的观察记录 |
| **Scope-aware (作用域感知)** | `project` (默认) 或 `global` |

### 置信度评分

| 分数 | 含义 | 行为 |
|------|------|------|
| 0.3 | 初步 | 建议但不强制执行 |
| 0.5 | 中等 | 相关时应用 |
| 0.7 | 强 | 自动批准应用 |
| 0.9 | 近乎确定 | 核心行为 |

**置信度增加条件：**
- 模式被反复观察到
- 用户没有纠正建议的行为
- 来自其他来源的类似本能一致

**置信度降低条件：**
- 用户明确纠正行为
- 长时间未观察到模式
- 出现矛盾证据

### 项目作用域 vs 全局作用域

| 模式类型 | 作用域 | 示例 |
|----------|--------|------|
| 语言/框架约定 | **项目** | "使用 React hooks", "遵循 Django REST 模式" |
| 文件结构偏好 | **项目** | "测试放在 `__tests__/`", "组件放在 src/components/" |
| 代码风格 | **项目** | "使用函数式风格", "偏好 dataclasses" |
| 错误处理策略 | **项目** | "使用 Result 类型处理错误" |
| 安全实践 | **全局** | "验证用户输入", "清理 SQL" |
| 通用最佳实践 | **全局** | "测试先行", "始终处理错误" |
| 工具工作流偏好 | **全局** | "编辑前 Grep", "写入前读取" |
| Git 实践 | **全局** | "约定式提交", "小而专注的提交" |

---

## 学习命令速查表

| 命令 | 用途 | 使用时机 |
|------|------|----------|
| `/learn` | 从当前会话提取模式 | 发现可复用模式时 |
| `/learn-eval` | 提取 + 评估 + 保存 | 更严格的模式提取 |
| `/instinct-status` | 查看学习的本能 | 查看积累的经验 |
| `/evolve` | 聚类本能成技能/命令/代理 | 本能积累足够后 |
| `/instinct-export` | 导出本能分享 | 团队知识共享 |
| `/instinct-import` | 导入他人的本能 | 接收团队知识 |
| `/promote` | 提升项目本能到全局 | 跨项目复用时 |
| `/prune` | 清理过期本能 | 定期维护 |

---

## 从会话中提取模式

### /learn 命令

**触发时机：** 在会话中解决了非平凡问题时运行。

**提取内容：**

1. **错误解决模式**
   - 发生了什么错误？
   - 根本原因是什么？
   - 如何修复的？
   - 对类似错误是否可复用？

2. **调试技术**
   - 非显而易见的调试步骤
   - 有效的工具组合
   - 诊断模式

3. **变通方案**
   - 库的怪异行为
   - API 限制
   - 版本特定修复

4. **项目特定模式**
   - 发现的代码库约定
   - 架构决策
   - 集成模式

**输出格式：**

```markdown
# [描述性模式名称]

**提取时间：** [日期]
**上下文：** [适用场景简述]

## 问题
[解决什么问题 - 具体描述]

## 解决方案
[模式/技术/变通方案]

## 示例
[代码示例（如适用）]

## 使用时机
[触发条件 - 什么应该激活此技能]
```

### /learn-eval 命令

**扩展 `/learn` 的功能：** 添加质量门控、保存位置决策和知识放置意识。

**流程：**

1. **审查会话** — 识别可提取的模式
2. **确定保存位置：**
   - **全局** (`~/.claude/skills/learned/`): 跨 2+ 项目可用的通用模式
   - **项目** (`.claude/skills/learned/`): 项目特定知识
   - 不确定时选择全局（全局迁移到项目比反向更容易）

3. **质量门控检查清单：**
   - [ ] Grep `~/.claude/skills/` 和项目 `.claude/skills/` 检查内容重叠
   - [ ] 检查 MEMORY.md（项目和全局）是否有重叠
   - [ ] 考虑追加到现有技能是否足够
   - [ ] 确认这是可复用模式，而非一次性修复

4. **综合判定：**

   | 判定 | 含义 | 下一步 |
   |------|------|--------|
   | **Save** | 唯一、具体、范围明确 | 保存 |
   | **Improve then Save** | 有价值但需改进 | 列出改进 → 修订 → 重新评估 |
   | **Absorb into [X]** | 应追加到现有技能 | 显示目标技能和添加内容 |
   | **Drop** | 琐碎、冗余或太抽象 | 解释原因并停止 |

5. **确认后保存**

---

## 本能聚类成技能的过程

### /evolve 命令

**功能：** 分析本能并将相关本能聚类为更高级的结构。

**进化规则：**

### 本能 -> 命令 (用户调用)

**条件：** 本能描述用户会显式请求的操作

**示例：**
```
- new-table-step1: "添加数据库表时，创建迁移"
- new-table-step2: "添加数据库表时，更新 schema"
- new-table-step3: "添加数据库表时，重新生成类型"
```
-> 创建: **new-table** 命令

### 本能 -> 技能 (自动触发)

**条件：** 本能描述应该自动发生的行为

**示例：**
```
- prefer-functional: "编写函数时，偏好函数式风格"
- use-immutable: "修改状态时，使用不可变模式"
- avoid-classes: "设计模块时，避免基于类的设计"
```
-> 创建: `functional-patterns` 技能

### 本能 -> 代理 (需要深度/隔离)

**条件：** 本能描述复杂的多步骤流程，受益于隔离

**示例：**
```
- debug-step1: "调试时，首先检查日志"
- debug-step2: "调试时，隔离失败组件"
- debug-step3: "调试时，创建最小复现"
- debug-step4: "调试时，用测试验证修复"
```
-> 创建: **debugger** 代理

### 输出格式

```
============================================================
  EVOLVE 分析 - 12 个本能
  项目: my-app (a1b2c3d4e5f6)
  项目作用域: 8 | 全局: 4
============================================================

高置信度本能 (>=80%): 5

## 技能候选
1. 聚类: "添加测试"
   本能数: 3
   平均置信度: 82%
   领域: testing
   作用域: project

## 命令候选 (2)
  /adding-tests
    来自: test-first-workflow [project]
    置信度: 84%

## 代理候选 (1)
  adding-tests-agent
    覆盖 3 个本能
    平均置信度: 82%
```

---

## 知识共享与导入导出

### /instinct-export 命令

**用途：**
- 与队友分享
- 迁移到新机器
- 贡献到项目约定

**用法：**

```bash
/instinct-export                           # 导出所有个人本能
/instinct-export --domain testing          # 只导出测试领域本能
/instinct-export --min-confidence 0.7      # 只导出高置信度本能
/instinct-export --output team-instincts.yaml
/instinct-export --scope project --output project-instincts.yaml
```

**输出格式：**

```yaml
# 本能导出
# 生成时间: 2025-01-22
# 来源: personal
# 数量: 12 个本能

---
id: prefer-functional-style
trigger: "when writing new functions"
confidence: 0.8
domain: code-style
source: session-observation
scope: project
project_id: a1b2c3d4e5f6
project_name: my-app
---

# Prefer Functional Style

## Action
Use functional patterns over classes.
```

### /instinct-import 命令

**用法：**

```bash
/instinct-import team-instincts.yaml
/instinct-import https://github.com/org/repo/instincts.yaml
/instinct-import team-instincts.yaml --dry-run
/instinct-import team-instincts.yaml --scope global --force
```

**导入流程：**

```
导入本能来自: team-instincts.yaml
================================================

发现 12 个本能待导入。

分析冲突...

## 新本能 (8)
将被添加:
  ✓ use-zod-validation (置信度: 0.7)
  ✓ prefer-named-exports (置信度: 0.65)
  ✓ test-async-functions (置信度: 0.8)
  ...

## 重复本能 (3)
已有类似本能:
  WARNING: prefer-functional-style
     本地: 0.8 置信度, 12 次观察
     导入: 0.7 置信度
     → 保留本地 (更高置信度)

  WARNING: test-first-workflow
     本地: 0.75 置信度
     导入: 0.9 置信度
     → 更新为导入 (更高置信度)

导入 8 个新本能，更新 1 个？
```

### /promote 命令

**用途：** 将项目本能提升到全局作用域

**自动提升条件：**
- 同一本能 ID 出现在 2+ 项目中
- 平均置信度 >= 0.8

**用法：**

```bash
# 提升特定本能
python3 instinct-cli.py promote prefer-explicit-errors

# 自动提升所有符合条件的本能
python3 instinct-cli.py promote

# 预览不实际更改
python3 instinct-cli.py promote --dry-run
```

### /prune 命令

**用途：** 删除过期的待定本能

**用法：**

```bash
/prune                    # 删除超过 30 天的本能
/prune --max-age 60      # 自定义年龄阈值（天）
/prune --dry-run         # 预览不删除
```

---

## 典型学习流程示例

### 场景 1：自动学习流程

```
1. 开发者在 React 项目中工作
   ↓
2. Hooks 自动捕获所有工具调用
   ↓
3. 观察代理（后台 Haiku）分析模式
   ↓
4. 检测到用户偏好函数式组件
   ↓
5. 创建本能: prefer-functional-components (置信度 0.5)
   ↓
6. 后续会话中模式重复出现
   ↓
7. 置信度提升到 0.7
   ↓
8. 开发者运行 /evolve
   ↓
9. 与其他 React 相关本能聚类
   ↓
10. 生成 react-patterns 技能
```

### 场景 2：手动提取模式

```
1. 开发者解决了一个复杂的构建错误
   ↓
2. 运行 /learn-eval
   ↓
3. 系统分析会话，识别模式
   ↓
4. 质量门控检查
   ↓
5. 判定: Save
   ↓
6. 确认保存位置（项目 vs 全局）
   ↓
7. 保存到 ~/.claude/skills/learned/resolve-circular-deps.md
```

### 场景 3：团队知识共享

```
1. 团队负责人整理项目约定
   ↓
2. 运行 /instinct-export --scope project --output team-conventions.yaml
   ↓
3. 提交到项目仓库
   ↓
4. 新成员克隆项目
   ↓
5. 运行 /instinct-import team-conventions.yaml
   ↓
6. 新成员立即获得项目约定本能
```

---

## 配置与文件结构

### 启用观察 Hooks

**插件安装（推荐）：**

Claude Code v2.1+ 自动加载插件 `hooks/hooks.json`，无需额外配置。

**手动安装：**

添加到 `~/.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh"
      }]
    }],
    "PostToolUse": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "~/.claude/skills/continuous-learning-v2/hooks/observe.sh"
      }]
    }]
  }
}
```

### 配置文件

编辑 `config.json` 控制后台观察器：

```json
{
  "version": "2.1",
  "observer": {
    "enabled": false,
    "run_interval_minutes": 5,
    "min_observations_to_analyze": 20
  }
}
```

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `observer.enabled` | `false` | 启用后台观察代理 |
| `observer.run_interval_minutes` | `5` | 分析观察的频率 |
| `observer.min_observations_to_analyze` | `20` | 分析前的最小观察数 |

### 文件结构

```
${XDG_DATA_HOME:-~/.local/share}/ecc-homunculus/
+-- identity.json           # 你的配置，技术等级
+-- projects.json           # 注册表: 项目 hash -> 名称/路径/remote
+-- observations.jsonl      # 全局观察 (回退)
+-- instincts/
|   +-- personal/           # 全局自动学习本能
|   +-- inherited/          # 全局导入本能
+-- evolved/
|   +-- agents/             # 全局生成的代理
|   +-- skills/             # 全局生成的技能
|   +-- commands/           # 全局生成的命令
+-- projects/
    +-- a1b2c3d4e5f6/       # 项目 hash (来自 git remote URL)
    |   +-- project.json    # 项目元数据
    |   +-- observations.jsonl
    |   +-- instincts/
    |   |   +-- personal/   # 项目特定自动学习
    |   |   +-- inherited/  # 项目特定导入
    |   +-- evolved/
    |       +-- skills/
    |       +-- commands/
    |       +-- agents/
    +-- f6e5d4c3b2a1/       # 另一个项目
```

---

## 项目检测机制

系统自动检测当前项目：

1. **`CLAUDE_PROJECT_DIR` 环境变量** (最高优先级)
2. **`git remote get-url origin`** — 哈希生成可移植的项目 ID
3. **`git rev-parse --show-toplevel`** — 使用仓库路径（机器特定）
4. **全局回退** — 无项目检测时，本能进入全局作用域

每个项目获得一个 12 字符的 hash ID（如 `a1b2c3d4e5f6`）。

---

## 隐私说明

- 观察数据**保留在本地**机器上
- 项目作用域本能按项目隔离
- 只有**本能**（模式）可以导出 — 不是原始观察
- 不共享实际代码或对话内容
- 你控制导出和提升的内容

---

## 相关资源

- [ECC-Tools GitHub App](https://github.com/apps/ecc-tools) - 从仓库历史生成本能
- [The Longform Guide](https://x.com/affaanmustafa/status/2014040193557471352) - 持续学习章节
- Homunculus - 启发 v2 本能架构的社区项目

---

*基于本能的学习：一个项目一个项目地教会 Claude 你的模式。*
