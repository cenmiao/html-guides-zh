# ECC 阶段 6：测试阶段详细指南

> 本文档基于 ECC 官方项目 https://github.com/affaan-m/everything-claude-code 整理

---

## 阶段目标

**核心任务：** 单元测试、集成测试、E2E 测试、测试覆盖率验证

**关键原则：**
- **测试优先** — TDD 红绿重构循环
- **覆盖率达标** — 80%+ 最低要求
- **自动化验证** — CI/CD 质量门禁

---

## 测试类型层次

```
┌─────────────────────────────────────────────────────────────┐
│                    测试金字塔                                │
├─────────────────────────────────────────────────────────────┤
│                      E2E 测试                                │
│              (Playwright, Cypress, Selenium)                │
│                    10-20%                                   │
├─────────────────────────────────────────────────────────────┤
│                    集成测试                                   │
│           (API 测试, 数据库测试, 服务集成)                    │
│                    20-30%                                   │
├─────────────────────────────────────────────────────────────┤
│                    单元测试                                   │
│           (Jest, pytest, Go testing, JUnit)                 │
│                    50-70%                                   │
└─────────────────────────────────────────────────────────────┘
```

---

## 测试 Agents

| Agent | 用途 | 适用场景 |
|-------|------|----------|
| `tdd-guide` | 测试驱动开发指导 | 任何编码前，先写测试 |
| `e2e-runner` | Playwright E2E 测试 | 关键用户流程测试 |
| `pr-test-analyzer` | PR 测试覆盖分析 | PR 审查时评估测试质量 |

---

## TDD 工作流

### RED-GREEN-REFACTOR 循环

```
┌─────────────────────────────────────────────────────────────┐
│                    TDD 循环                                  │
├─────────────────────────────────────────────────────────────┤
│  1. RED: 写失败测试                                          │
│     → 定义接口/行为                                          │
│     → 写测试期望                                             │
│     → 运行测试 → 失败 ✓                                      │
├─────────────────────────────────────────────────────────────┤
│  2. GREEN: 最小实现                                          │
│     → 写最少的代码让测试通过                                  │
│     → 不追求完美                                             │
│     → 运行测试 → 通过 ✓                                      │
├─────────────────────────────────────────────────────────────┤
│  3. REFACTOR: 重构优化                                       │
│     → 清理代码                                               │
│     → 优化结构                                               │
│     → 运行测试 → 仍然通过 ✓                                  │
├─────────────────────────────────────────────────────────────┤
│  4. 重复                                                     │
│     → 添加下一个测试                                         │
│     → 继续循环                                               │
└─────────────────────────────────────────────────────────────┘
```

### TDD 原则

| 原则 | 说明 |
|------|------|
| **先写测试** | 任何实现代码前必须先有失败测试 |
| **最小实现** | 只写让测试通过的最少代码 |
| **小步快跑** | 每次只测试一个行为 |
| **快速反馈** | 测试必须在秒级完成 |
| **隔离测试** | 测试之间无依赖 |

---

## 测试 Skills

### 通用测试 Skills

| Skill | 用途 | 适用场景 |
|-------|------|----------|
| `tdd-workflow` | TDD 工作流 | **任何编码前** |
| `verification-loop` | 持续验证循环 | CI/CD 质量门禁 |
| `e2e-testing` | Playwright E2E | 关键用户流程 |

### 语言特定测试 Skills

| Skill | 用途 | 适用场景 |
|-------|------|----------|
| `golang-testing` | Go 测试模式 | Go 项目 |
| `python-testing` | pytest 测试 | Python 项目 |
| `cpp-testing` | GoogleTest/CMake | C++ 项目 |
| `rust-testing` | cargo test | Rust 项目 |
| `kotlin-testing` | Kotest/MockK | Kotlin 项目 |

---

## 语言特定测试框架

### TypeScript/JavaScript

```bash
# Jest/Vitest
npm test
npm run test:coverage

# 测试命令
npx jest --coverage --watchAll=false
npx vitest run --coverage

# 覆盖率报告
npx jest --coverage --coverageReporters=text --coverageReporters=lcov
```

**测试结构：**
```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid data', () => {
      // Arrange
      const data = { name: 'test', email: 'test@example.com' };

      // Act
      const result = userService.createUser(data);

      // Assert
      expect(result).toBeDefined();
      expect(result.name).toBe('test');
    });

    it('should throw error for invalid email', () => {
      expect(() => userService.createUser({ email: 'invalid' }))
        .toThrow('Invalid email');
    });
  });
});
```

### Go

```bash
# Go 测试
go test ./...
go test -v ./...
go test -cover ./...
go test -coverprofile=coverage.out ./...

# 覆盖率报告
go tool cover -html=coverage.out -o coverage.html
go tool cover -func=coverage.out

# 基准测试
go test -bench=. -benchmem ./...

# 竞态检测
go test -race ./...
```

**测试结构：**
```go
func TestUserService_Create(t *testing.T) {
    tests := []struct {
        name    string
        input   CreateUserInput
        want    *User
        wantErr bool
    }{
        {
            name:  "valid user",
            input: CreateUserInput{Name: "test", Email: "test@example.com"},
            want:  &User{Name: "test", Email: "test@example.com"},
        },
        {
            name:    "invalid email",
            input:   CreateUserInput{Email: "invalid"},
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := userService.Create(tt.input)
            if tt.wantErr {
                assert.Error(t, err)
                return
            }
            assert.NoError(t, err)
            assert.Equal(t, tt.want, got)
        })
    }
}
```

### Python

```bash
# pytest
pytest
pytest -v
pytest --cov=src --cov-report=term-missing
pytest --cov=src --cov-report=html

# 特定测试
pytest tests/unit/test_user.py -v
pytest tests/ -k "user" -v

# 并行测试
pytest -n auto
```

**测试结构：**
```python
import pytest
from unittest.mock import Mock, patch

class TestUserService:
    def test_create_user_success(self):
        # Arrange
        service = UserService()
        data = {"name": "test", "email": "test@example.com"}

        # Act
        result = service.create_user(data)

        # Assert
        assert result.name == "test"
        assert result.email == "test@example.com"

    def test_create_user_invalid_email(self):
        service = UserService()

        with pytest.raises(ValueError, match="Invalid email"):
            service.create_user({"email": "invalid"})

    @patch("services.user.send_email")
    def test_create_user_sends_welcome_email(self, mock_send):
        service = UserService()
        service.create_user({"name": "test", "email": "test@example.com"})

        mock_send.assert_called_once()
```

### C++

```bash
# GoogleTest
cmake --build build
cd build && ctest --output-on-failure

# 运行特定测试
./build/tests/user_service_test

# 详细输出
./build/tests/user_service_test --gtest_filter=*CreateUser*

# XML 报告
./build/tests/user_service_test --gtest_output=xml:test_results.xml
```

**测试结构：**
```cpp
#include <gtest/gtest.h>
#include "user_service.h"

class UserServiceTest : public ::testing::Test {
protected:
    UserService service;

    void SetUp() override {
        // Setup code
    }

    void TearDown() override {
        // Cleanup code
    }
};

TEST_F(UserServiceTest, CreateUser_ValidData_ReturnsUser) {
    // Arrange
    auto data = CreateUserInput{"test", "test@example.com"};

    // Act
    auto result = service.CreateUser(data);

    // Assert
    EXPECT_TRUE(result.has_value());
    EXPECT_EQ(result->name, "test");
    EXPECT_EQ(result->email, "test@example.com");
}

TEST_F(UserServiceTest, CreateUser_InvalidEmail_Throws) {
    EXPECT_THROW(
        service.CreateUser({"test", "invalid"}),
        std::invalid_argument
    );
}
```

### Rust

```bash
# cargo test
cargo test
cargo test -- --nocapture  # 显示输出
cargo test -- --show-output

# 特定测试
cargo test test_create_user
cargo test --test integration_tests

# 文档测试
cargo test --doc

# 覆盖率 (需要 cargo-tarpaulin)
cargo tarpaulin --out Html --output-dir coverage
```

**测试结构：**
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_create_user_success() {
        // Arrange
        let service = UserService::new();
        let input = CreateUserInput {
            name: "test".to_string(),
            email: "test@example.com".to_string(),
        };

        // Act
        let result = service.create_user(input);

        // Assert
        assert!(result.is_ok());
        let user = result.unwrap();
        assert_eq!(user.name, "test");
    }

    #[test]
    fn test_create_user_invalid_email() {
        let service = UserService::new();
        let input = CreateUserInput {
            email: "invalid".to_string(),
            ..Default::default()
        };

        assert!(matches!(
            service.create_user(input),
            Err(UserError::InvalidEmail)
        ));
    }
}
```

### Kotlin

```bash
# Gradle 测试
./gradlew test
./gradlew test --info

# Kotest
./gradlew test

# 覆盖率 (Kover)
./gradlew koverHtmlReport
./gradlew koverXmlReport

# 特定测试
./gradlew test --tests "UserServiceTest"
```

**测试结构：**
```kotlin
import io.kotest.core.spec.style.DescribeSpec
import io.kotest.matchers.shouldBe
import io.kotest.matchers.should
import io.kotest.matchers.throwable.shouldHaveMessage
import io.mockk.*

class UserServiceTest : DescribeSpec({
    describe("createUser") {
        val service = UserService()

        context("with valid data") {
            it("should create user successfully") {
                val input = CreateUserInput(
                    name = "test",
                    email = "test@example.com"
                )

                val result = service.createUser(input)

                result.name shouldBe "test"
                result.email shouldBe "test@example.com"
            }
        }

        context("with invalid email") {
            it("should throw InvalidEmailException") {
                val input = CreateUserInput(email = "invalid")

                shouldThrow<InvalidEmailException> {
                    service.createUser(input)
                } shouldHaveMessage "Invalid email format"
            }
        }
    }
})
```

---

## E2E 测试

### Playwright 测试流程

```
┌─────────────────────────────────────────────────────────────┐
│                    E2E 测试流程                              │
├─────────────────────────────────────────────────────────────┤
│  1. 测试计划                                                 │
│     → 识别关键用户流程                                       │
│     → 定义测试场景                                           │
│     → 创建 Page Object Model                                │
├─────────────────────────────────────────────────────────────┤
│  2. 测试实现                                                 │
│     → 使用 Playwright 编写测试                              │
│     → 实现页面对象                                           │
│     → 添加测试数据管理                                       │
├─────────────────────────────────────────────────────────────┤
│  3. 测试执行                                                 │
│     → 本地运行验证                                           │
│     → CI/CD 集成                                            │
│     → 生成测试报告                                           │
├─────────────────────────────────────────────────────────────┤
│  4. 测试维护                                                 │
│     → 定期更新测试数据                                       │
│     → 修复失败的测试                                         │
│     → 隔离不稳定测试                                         │
└─────────────────────────────────────────────────────────────┘
```

### Playwright 测试结构

```typescript
// tests/auth/login.spec.ts
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/login.page';

test.describe('Login Flow', () => {
  let loginPage: LoginPage;

  test.beforeEach(async ({ page }) => {
    loginPage = new LoginPage(page);
    await loginPage.goto();
  });

  test('should login successfully with valid credentials', async ({ page }) => {
    await loginPage.login('user@example.com', 'password123');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="user-menu"]')).toBeVisible();
  });

  test('should show error for invalid credentials', async ({ page }) => {
    await loginPage.login('user@example.com', 'wrong-password');

    await expect(page.locator('[data-testid="error-message"]'))
      .toContainText('Invalid credentials');
  });
});
```

### Page Object Model

```typescript
// pages/login.page.ts
import { Page, Locator } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.locator('[data-testid="email-input"]');
    this.passwordInput = page.locator('[data-testid="password-input"]');
    this.submitButton = page.locator('[data-testid="login-button"]');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }
}
```

---

## 测试覆盖率

### 覆盖率要求

| 指标 | 最低要求 | 推荐值 |
|------|----------|--------|
| **行覆盖率** | 80% | 90%+ |
| **分支覆盖率** | 70% | 85%+ |
| **函数覆盖率** | 80% | 90%+ |

### 覆盖率命令

| 语言 | 命令 |
|------|------|
| **TypeScript/Jest** | `npx jest --coverage` |
| **TypeScript/Vitest** | `npx vitest run --coverage` |
| **Go** | `go test -coverprofile=coverage.out ./...` |
| **Python/pytest** | `pytest --cov=src --cov-report=term-missing` |
| **Rust** | `cargo tarpaulin --out Html` |
| **Kotlin** | `./gradlew koverHtmlReport` |

### 覆盖率报告解读

```
----------|---------|----------|---------|---------|-------------------
File      | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
----------|---------|----------|---------|---------|-------------------
All files |   85.2  |   78.4   |   90.1  |   85.5  |
 user.ts  |   92.3  |   88.9   |   95.0  |   92.5  | 45-48
 auth.ts  |   78.1  |   67.9   |   85.2  |   78.5  | 23,45-50,67
----------|---------|----------|---------|---------|-------------------
```

---

## 验证循环

### 持续验证流程

```
┌─────────────────────────────────────────────────────────────┐
│                    验证循环                                  │
├─────────────────────────────────────────────────────────────┤
│  1. 构建 (Build)                                             │
│     → npm run build / cargo build / go build                │
│     → 检查编译错误                                           │
├─────────────────────────────────────────────────────────────┤
│  2. 测试 (Test)                                              │
│     → npm test / cargo test / pytest                        │
│     → 验证测试通过                                           │
├─────────────────────────────────────────────────────────────┤
│  3. Lint (代码检查)                                          │
│     → npm run lint / cargo clippy / golangci-lint           │
│     → 检查代码风格                                           │
├─────────────────────────────────────────────────────────────┤
│  4. 安全扫描 (Security)                                       │
│     → npm audit / cargo audit / safety check                │
│     → 检查安全漏洞                                           │
├─────────────────────────────────────────────────────────────┤
│  5. 覆盖率 (Coverage)                                        │
│     → 验证 80%+ 覆盖率                                       │
│     → 生成覆盖率报告                                         │
├─────────────────────────────────────────────────────────────┤
│  6. 通过 → 继续                                              │
│     失败 → 修复 → 重新验证                                   │
└─────────────────────────────────────────────────────────────┘
```

### 质量门禁

| 检查项 | 通过条件 |
|--------|----------|
| **构建** | 无编译错误 |
| **测试** | 所有测试通过 |
| **Lint** | 无错误，警告 < 10 |
| **安全** | 无高危/中危漏洞 |
| **覆盖率** | ≥ 80% |

---

## 命令速查表

### 测试命令

| 命令 | 用途 | 适用场景 |
|------|------|----------|
| `/test-coverage` | 测试覆盖率分析 | 验证 80%+ 覆盖率 |
| `/quality-gate` | 质量门禁 | 发布前验证 |

### 测试运行命令

| 语言 | 运行测试 | 覆盖率 |
|------|----------|--------|
| **TypeScript** | `npm test` | `npm run test:coverage` |
| **Go** | `go test ./...` | `go test -cover ./...` |
| **Python** | `pytest` | `pytest --cov` |
| **Rust** | `cargo test` | `cargo tarpaulin` |
| **Kotlin** | `./gradlew test` | `./gradlew koverHtmlReport` |
| **C++** | `ctest` | `gcov` + `lcov` |

---

## 测试最佳实践

### 单元测试原则

| 原则 | 说明 |
|------|------|
| **FIRST** | Fast, Independent, Repeatable, Self-validating, Timely |
| **AAA** | Arrange, Act, Assert |
| **单一职责** | 每个测试只验证一个行为 |
| **命名清晰** | `test_shouldThrowError_whenEmailInvalid` |
| **隔离依赖** | 使用 Mock/Stub |

### 集成测试原则

| 原则 | 说明 |
|------|------|
| **真实环境** | 使用真实数据库/服务 |
| **测试数据** | 固定的测试数据集 |
| **清理状态** | 每次测试后清理 |
| **并行安全** | 避免测试间冲突 |

### E2E 测试原则

| 原则 | 说明 |
|------|------|
| **关键路径** | 只测试关键用户流程 |
| **稳定优先** | 避免不稳定测试 |
| **等待策略** | 使用显式等待 |
| **数据隔离** | 独立测试数据 |

---

## 典型测试流程示例

### 场景：添加用户服务功能

```
┌─────────────────────────────────────────────────────────────┐
│ 1. TDD 循环                                                  │
│    → 写失败测试: test_create_user_success                    │
│    → 最小实现让测试通过                                       │
│    → 重构优化                                                │
├─────────────────────────────────────────────────────────────┤
│ 2. 添加更多测试                                              │
│    → test_create_user_invalid_email                          │
│    → test_create_user_duplicate_email                        │
│    → test_create_user_sends_welcome_email                     │
├─────────────────────────────────────────────────────────────┤
│ 3. 运行测试套件                                              │
│    npm test → 所有测试通过                                    │
├─────────────────────────────────────────────────────────────┤
│ 4. 检查覆盖率                                                │
│    npm run test:coverage                                     │
│    → 验证 80%+ 覆盖率                                        │
├─────────────────────────────────────────────────────────────┤
│ 5. E2E 测试                                                  │
│    → 添加 Playwright 测试: login.spec.ts                     │
│    → 验证完整用户流程                                         │
├─────────────────────────────────────────────────────────────┤
│ 6. 质量门禁                                                  │
│    /quality-gate                                             │
│    → build ✓ + test ✓ + lint ✓ + security ✓ + coverage ✓    │
└─────────────────────────────────────────────────────────────┘
```

---

## 测试反模式

### 应避免的做法

| 反模式 | 问题 | 正确做法 |
|--------|------|----------|
| **测试私有方法** | 测试实现细节 | 测试公开行为 |
| **过度 Mock** | 测试变得脆弱 | 只 Mock 外部依赖 |
| **测试依赖顺序** | 测试不可靠 | 每个测试独立 |
| **断言不足** | 测试无意义 | 明确的断言 |
| **测试过大** | 难以定位问题 | 小而专注的测试 |
| **忽略不稳定测试** | 掩盖问题 | 修复或隔离 |

---

## 相关 Skills

| Skill | 用途 |
|-------|------|
| `tdd-workflow` | TDD 工作流 |
| `verification-loop` | 持续验证循环 |
| `e2e-testing` | Playwright E2E |
| `golang-testing` | Go 测试模式 |
| `python-testing` | pytest 测试 |
| `cpp-testing` | GoogleTest |
| `rust-testing` | cargo test |
| `kotlin-testing` | Kotest/MockK |

---

## 相关链接

- **GitHub 仓库：** https://github.com/affaan-m/everything-claude-code
- **官网：** https://ecc.tools
- **作者：** [@affaanmustafa](https://x.com/affaanmustafa)

---

**Star 这个项目如果有帮助。构建伟大的东西。**
