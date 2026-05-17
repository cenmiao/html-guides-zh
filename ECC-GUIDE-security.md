# ECC 阶段 7：安全审计详细指南

> 本文档基于 ECC 官方项目 https://github.com/affaan-m/everything-claude-code 整理

---

## 阶段目标

**核心任务：** 漏洞检测、OWASP Top 10、配置安全、Secrets 管理

**关键原则：**
- **主动防御** — 在代码进入生产前发现漏洞
- **深度防御** — 多层安全措施
- **最小权限** — 只授予必要的权限
- **零信任** — 验证所有输入和输出

---

## 安全审计 Agents

| Agent | 用途 | 适用场景 |
|-------|------|----------|
| `security-reviewer` | 安全漏洞检测与修复 | **任何涉及用户输入/认证/API 的代码** |

---

## 安全审计 Skills

| Skill | 用途 | 适用场景 |
|-------|------|----------|
| `security-review` | 综合安全检查清单 | 认证、用户输入、API 端点 |
| `django-security` | Django 安全最佳实践 | Django 项目 |
| `laravel-security` | Laravel 安全最佳实践 | Laravel 项目 |
| `cloud-infrastructure-security` | 云基础设施安全 | AWS、Vercel、Cloudflare 部署 |

---

## AgentShield 安全扫描

### 安装与使用

```bash
# 快速扫描
npx ecc-agentshield scan

# 指定路径
npx ecc-agentshield scan --path ./my-project

# 指定输出格式
npx ecc-agentshield scan --format json
npx ecc-agentshield scan --format html
npx ecc-agentshield scan --format markdown

# 自动修复
npx ecc-agentshield scan --fix

# 三代理红队审计 (深度分析)
npx ecc-agentshield scan --opus
```

### 扫描范围

AgentShield 检测以下安全问题：

| 类别 | 检测项 |
|------|--------|
| **Secrets 检测** | API Keys、密码、Token、证书 (14+ 种模式) |
| **权限审计** | 过宽权限、危险操作、未授权访问 |
| **Hook 注入** | 恶意 Hook、危险命令、代码注入 |
| **MCP 服务器风险** | Shell 访问、文件系统访问、远程传输、未固定 npx |
| **Agent 配置** | 不安全的提示词、未处理的不可信内容 |
| **OWASP Top 10** | 注入、XSS、CSRF、认证问题等 |

---

## OWASP Top 10 检查清单

### 1. 注入攻击 (Injection)

**检测项：** SQL 注入、命令注入、LDAP 注入

```typescript
// ❌ 危险：字符串拼接 SQL
const query = `SELECT * FROM users WHERE email = '${userEmail}'`

// ✅ 安全：参数化查询
const { data } = await supabase
  .from('users')
  .select('*')
  .eq('email', userEmail)

// 或使用原生参数化
await db.query('SELECT * FROM users WHERE email = $1', [userEmail])
```

**验证步骤：**
- [ ] 所有数据库查询使用参数化
- [ ] 不使用字符串拼接构建 SQL
- [ ] ORM 使用正确
- [ ] 用户输入经过验证

### 2. 失效的身份认证 (Broken Authentication)

**检测项：** 弱密码、会话管理、Token 存储

```typescript
// ❌ 危险：localStorage 存储 Token (易受 XSS)
localStorage.setItem('token', token)

// ✅ 安全：httpOnly Cookie
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

**验证步骤：**
- [ ] Token 存储在 httpOnly Cookie
- [ ] 密码使用 bcrypt/argon2 哈希
- [ ] JWT 验证正确
- [ ] 会话安全配置

### 3. 敏感数据泄露 (Sensitive Data Exposure)

**检测项：** 明文存储、日志泄露、错误信息泄露

```typescript
// ❌ 危险：日志记录敏感信息
console.log('User login:', { email, password })

// ✅ 安全：脱敏日志
console.log('User login:', { email, userId })
```

**验证步骤：**
- [ ] 敏感数据加密存储
- [ ] HTTPS 强制执行
- [ ] 日志不包含敏感信息
- [ ] 错误信息不暴露内部细节

### 4. XML 外部实体 (XXE)

**检测项：** XML 解析器配置、外部实体

```python
# ❌ 危险：允许外部实体
from lxml import etree
parser = etree.XMLParser(resolve_entities=True)

# ✅ 安全：禁用外部实体
parser = etree.XMLParser(resolve_entities=False, no_network=True)
```

**验证步骤：**
- [ ] XML 解析器安全配置
- [ ] 禁用外部实体
- [ ] 禁用网络访问

### 5. 失效的访问控制 (Broken Access Control)

**检测项：** 路由授权、CORS 配置、水平越权

```typescript
// ❌ 危险：无授权检查
export async function deleteUser(userId: string) {
  await db.users.delete({ where: { id: userId } })
}

// ✅ 安全：授权检查
export async function deleteUser(userId: string, requesterId: string) {
  const requester = await db.users.findUnique({ where: { id: requesterId } })

  if (requester.role !== 'admin') {
    throw new Error('Unauthorized')
  }

  await db.users.delete({ where: { id: userId } })
}
```

**验证步骤：**
- [ ] 每个路由都有授权检查
- [ ] CORS 正确配置
- [ ] 行级安全 (RLS) 启用

### 6. 安全配置错误 (Security Misconfiguration)

**检测项：** 默认凭据、调试模式、安全头

```python
# Django 生产配置
DEBUG = False
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_HSTS_SECONDS = 31536000
X_FRAME_OPTIONS = 'DENY'
```

**验证步骤：**
- [ ] DEBUG 关闭
- [ ] 默认凭据已更改
- [ ] 安全头已配置
- [ ] 错误处理不暴露堆栈

### 7. 跨站脚本 (XSS)

**检测项：** 输出转义、CSP 配置、用户 HTML

```typescript
import DOMPurify from 'isomorphic-dompurify'

// ❌ 危险：直接渲染用户 HTML
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ 安全：净化用户 HTML
const clean = DOMPurify.sanitize(userInput, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p'],
  ALLOWED_ATTR: []
})
<div dangerouslySetInnerHTML={{ __html: clean }} />
```

**验证步骤：**
- [ ] 用户 HTML 经过净化
- [ ] CSP 头已配置
- [ ] 框架自动转义启用

### 8. 不安全的反序列化 (Insecure Deserialization)

**检测项：** Pickle、YAML、JSON 反序列化

```python
# ❌ 危险：不安全的反序列化
import pickle
data = pickle.loads(user_input)

# ✅ 安全：使用 JSON
import json
data = json.loads(user_input)
```

**验证步骤：**
- [ ] 不使用 pickle 处理用户输入
- [ ] YAML 加载器安全
- [ ] 反序列化有类型验证

### 9. 使用含有已知漏洞的组件 (Vulnerable Components)

**检测项：** 依赖漏洞、过期包

```bash
# npm 审计
npm audit --audit-level=high
npm audit fix

# Python 安全检查
pip install safety
safety check

# Go 漏洞检查
go list -m -json all | nancy sleuth
```

**验证步骤：**
- [ ] 依赖定期更新
- [ ] npm audit 无高危漏洞
- [ ] Dependabot 启用
- [ ] 锁文件已提交

### 10. 日志与监控不足 (Insufficient Logging)

**检测项：** 安全事件日志、告警配置

```python
# Django 安全日志配置
LOGGING = {
    'version': 1,
    'loggers': {
        'django.security': {
            'handlers': ['file'],
            'level': 'WARNING',
        },
    },
}
```

**验证步骤：**
- [ ] 认证失败被记录
- [ ] 管理员操作被审计
- [ ] 异常行为有告警
- [ ] 日志保留期合规

---

## 安全代码模式检查

### 立即标记的模式

| 模式 | 严重性 | 修复方案 |
|------|--------|----------|
| 硬编码密钥 | CRITICAL | 使用 `process.env` |
| Shell 命令 + 用户输入 | CRITICAL | 使用安全 API 或 execFile |
| 字符串拼接 SQL | CRITICAL | 参数化查询 |
| `innerHTML = userInput` | HIGH | 使用 `textContent` 或 DOMPurify |
| `fetch(userProvidedUrl)` | HIGH | 白名单域名 |
| 明文密码比较 | CRITICAL | 使用 `bcrypt.compare()` |
| 路由无认证检查 | CRITICAL | 添加认证中间件 |
| 余额检查无锁 | CRITICAL | 事务中使用 `FOR UPDATE` |
| 无速率限制 | HIGH | 添加 `express-rate-limit` |
| 日志记录密码/密钥 | MEDIUM | 脱敏日志输出 |

---

## Secrets 管理

### 禁止做法

```typescript
// ❌ 永远不要这样做
const apiKey = "sk-proj-xxxxx"  // 硬编码密钥
const dbPassword = "password123" // 源码中的密码
```

### 正确做法

```typescript
// ✅ 使用环境变量
const apiKey = process.env.OPENAI_API_KEY
const dbUrl = process.env.DATABASE_URL

// 验证密钥存在
if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured')
}
```

### 验证步骤

- [ ] 无硬编码 API Keys、Tokens、密码
- [ ] 所有密钥在环境变量中
- [ ] `.env.local` 在 `.gitignore` 中
- [ ] Git 历史中无密钥
- [ ] 生产密钥在托管平台 (Vercel、Railway)

---

## 输入验证

### Schema 验证

```typescript
import { z } from 'zod'

// 定义验证 Schema
const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150)
})

// 验证后再处理
export async function createUser(input: unknown) {
  try {
    const validated = CreateUserSchema.parse(input)
    return await db.users.create(validated)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, errors: error.errors }
    }
    throw error
  }
}
```

### 文件上传验证

```typescript
function validateFileUpload(file: File) {
  // 大小检查 (5MB 最大)
  const maxSize = 5 * 1024 * 1024
  if (file.size > maxSize) {
    throw new Error('File too large (max 5MB)')
  }

  // 类型检查
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif']
  if (!allowedTypes.includes(file.type)) {
    throw new Error('Invalid file type')
  }

  // 扩展名检查
  const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif']
  const extension = file.name.toLowerCase().match(/\.[^.]+$/)?.[0]
  if (!extension || !allowedExtensions.includes(extension)) {
    throw new Error('Invalid file extension')
  }

  return true
}
```

---

## 认证与授权

### JWT Token 处理

```typescript
// ❌ 错误：localStorage (易受 XSS)
localStorage.setItem('token', token)

// ✅ 正确：httpOnly Cookie
res.setHeader('Set-Cookie',
  `token=${token}; HttpOnly; Secure; SameSite=Strict; Max-Age=3600`)
```

### Supabase 行级安全 (RLS)

```sql
-- 启用 RLS
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- 用户只能查看自己的数据
CREATE POLICY "Users view own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

-- 用户只能更新自己的数据
CREATE POLICY "Users update own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);
```

---

## 速率限制

### API 速率限制

```typescript
import rateLimit from 'express-rate-limit'

// 通用限制
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 100, // 每窗口 100 请求
  message: 'Too many requests'
})

app.use('/api/', limiter)

// 敏感操作更严格限制
const authLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 分钟
  max: 5, // 每分钟 5 次
  message: 'Too many attempts'
})

app.use('/api/auth', authLimiter)
```

---

## 安全头配置

### Content Security Policy (CSP)

```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'Content-Security-Policy',
    value: `
      default-src 'self';
      base-uri 'self';
      object-src 'none';
      frame-ancestors 'none';
      script-src 'self';
      style-src 'self';
      img-src 'self' data: https:;
      font-src 'self';
      connect-src 'self' https://api.example.com;
    `.replace(/\s{2,}/g, ' ').trim()
  },
  {
    key: 'X-Frame-Options',
    value: 'DENY'
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  },
  {
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin'
  }
]
```

---

## 云基础设施安全

### IAM 最小权限

```yaml
# ✅ 正确：最小权限
iam_role:
  permissions:
    - s3:GetObject
    - s3:ListBucket
  resources:
    - arn:aws:s3:::my-bucket/*

# ❌ 错误：过宽权限
iam_role:
  permissions:
    - s3:*
  resources:
    - "*"
```

### 密钥轮换

```bash
# 设置数据库凭据自动轮换
aws secretsmanager rotate-secret \
  --secret-id prod/db-password \
  --rotation-lambda-arn arn:aws:lambda:region:account:function:rotate \
  --rotation-rules AutomaticallyAfterDays=30
```

### 安全组配置

```terraform
# ✅ 正确：限制访问
resource "aws_security_group" "app" {
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]  # 仅内部 VPC
  }
}

# ❌ 错误：开放所有端口
resource "aws_security_group" "bad" {
  ingress {
    from_port   = 0
    to_port     = 65535
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # 所有 IP！
  }
}
```

---

## 安全审计流程

### 标准审计流程

```
┌─────────────────────────────────────────────────────────────┐
│                    安全审计流程                              │
├─────────────────────────────────────────────────────────────┤
│  1. 自动扫描                                                 │
│     npx ecc-agentshield scan                                │
│     → 检测 Secrets、权限、Hook、MCP、配置                    │
├─────────────────────────────────────────────────────────────┤
│  2. OWASP Top 10 检查                                       │
│     → 逐项验证每个漏洞类别                                   │
│     → 记录发现的问题                                         │
├─────────────────────────────────────────────────────────────┤
│  3. 代码模式审查                                             │
│     → 检查危险代码模式                                       │
│     → 验证输入验证、认证、授权                               │
├─────────────────────────────────────────────────────────────┤
│  4. 依赖审计                                                 │
│     npm audit / pip check / cargo audit                     │
│     → 检查已知漏洞                                           │
├─────────────────────────────────────────────────────────────┤
│  5. 配置审计                                                 │
│     → 安全头、CORS、密钥管理                                 │
│     → 云基础设施配置                                         │
├─────────────────────────────────────────────────────────────┤
│  6. 生成报告                                                 │
│     → 按严重性排序                                           │
│     → 提供修复建议                                           │
│     → 验证修复效果                                           │
└─────────────────────────────────────────────────────────────┘
```

---

## 命令速查表

### 安全扫描命令

| 命令 | 用途 | 适用场景 |
|------|------|----------|
| `/security-scan` | AgentShield 安全扫描 | **任何涉及用户输入/认证/API 的代码** |

### AgentShield CLI

```bash
# 基本扫描
npx ecc-agentshield scan

# 指定格式
npx ecc-agentshield scan --format json
npx ecc-agentshield scan --format html

# 自动修复
npx ecc-agentshield scan --fix

# 深度分析
npx ecc-agentshield scan --opus
```

### 依赖审计命令

| 语言 | 命令 |
|------|------|
| **npm** | `npm audit --audit-level=high` |
| **Python** | `pip install safety && safety check` |
| **Go** | `go list -m -json all \| nancy sleuth` |
| **Rust** | `cargo audit` |

---

## 发布前安全检查清单

在任何生产部署前：

- [ ] **Secrets**: 无硬编码密钥，全部在环境变量
- [ ] **输入验证**: 所有用户输入已验证
- [ ] **SQL 注入**: 所有查询参数化
- [ ] **XSS**: 用户内容已净化
- [ ] **CSRF**: 保护已启用
- [ ] **认证**: Token 处理正确
- [ ] **授权**: 角色检查到位
- [ ] **速率限制**: 所有端点已启用
- [ ] **HTTPS**: 生产环境强制执行
- [ ] **安全头**: CSP、X-Frame-Options 已配置
- [ ] **错误处理**: 错误不暴露敏感数据
- [ ] **日志**: 无敏感数据记录
- [ ] **依赖**: 已更新，无漏洞
- [ ] **RLS**: Supabase 行级安全已启用
- [ ] **CORS**: 正确配置
- [ ] **文件上传**: 已验证 (大小、类型)
- [ ] **AgentShield**: 扫描通过，无 CRITICAL 问题

---

## 紧急响应

如果发现 CRITICAL 漏洞：

1. **记录详细报告**
2. **立即通知项目负责人**
3. **提供安全代码示例**
4. **验证修复有效**
5. **如果凭据泄露，轮换密钥**

---

## 相关 Skills

| Skill | 用途 |
|-------|------|
| `security-review` | 综合安全检查清单 |
| `django-security` | Django 安全最佳实践 |
| `laravel-security` | Laravel 安全最佳实践 |
| `cloud-infrastructure-security` | 云基础设施安全 |

---

## 相关链接

- **GitHub 仓库：** https://github.com/affaan-m/everything-claude-code
- **AgentShield：** https://github.com/affaan-m/agentshield
- **官网：** https://ecc.tools
- **OWASP Top 10：** https://owasp.org/www-project-top-ten/
- **作者：** [@affaanmustafa](https://x.com/affaanmustafa)

---

**记住：安全不是可选的。一个漏洞可能导致用户真正的财务损失。要彻底、要偏执、要主动。**
