# ECC 阶段 5：构建与调试详细指南

> 本文档基于 ECC 官方项目 https://github.com/affaan-m/everything-claude-code 整理

---

## 阶段目标

**核心任务：** 解决构建错误、编译问题、依赖冲突

**关键原则：**
- **最小化修改** — 只修复错误，不重构架构
- **手术式修复** — 精准定位问题，最小改动
- **快速恢复构建** — 优先让构建通过

---

## 构建错误解决 Agents

### 通用构建错误解决

| Agent | 用途 | 适用场景 |
|-------|------|----------|
| `build-error-resolver` | TypeScript/通用构建错误 | TypeScript 项目、前端构建失败 |
| `cpp-build-resolver` | C++/CMake 构建错误 | C++ 项目、链接器错误 |
| `rust-build-resolver` | Rust/cargo 构建错误 | Rust 项目、借用检查器错误 |
| `go-build-resolver` | Go 构建错误 | Go 项目、go vet 问题 |
| `java-build-resolver` | Java/Maven/Gradle 错误 | Java/Spring Boot/Quarkus 项目 |
| `kotlin-build-resolver` | Kotlin/Gradle 错误 | Kotlin/Android/KMP 项目 |
| `swift-build-resolver` | Swift/Xcode 构建错误 | Swift/iOS/macOS 项目 |
| `dart-build-resolver` | Dart/Flutter 构建错误 | Flutter/Dart 项目 |
| `pytorch-build-resolver` | PyTorch/CUDA 错误 | ML 训练崩溃、张量形状错误 |
| `django-build-resolver` | Django/Python 构建错误 | Django 项目、迁移冲突 |

---

## 构建调试流程

### 标准调试流程

```
┌─────────────────────────────────────────────────────────────┐
│ 构建失败检测                                                  │
│    npm run build / cargo build / ./mvnw compile 失败         │
├─────────────────────────────────────────────────────────────┤
│ 1. 错误诊断                                                   │
│    → 运行诊断命令收集错误信息                                  │
│    → 解析错误消息，识别错误类型                                │
├─────────────────────────────────────────────────────────────┤
│ 2. 定位问题                                                   │
│    → Read affected file 理解上下文                            │
│    → 分析错误根本原因                                         │
├─────────────────────────────────────────────────────────────┤
│ 3. 应用最小修复                                               │
│    → 只修改必要的部分                                         │
│    → 不重构、不改变架构                                       │
├─────────────────────────────────────────────────────────────┤
│ 4. 验证修复                                                   │
│    → 重新运行构建命令                                         │
│    → 确保没有引入新错误                                       │
├─────────────────────────────────────────────────────────────┤
│ 5. 测试确认                                                   │
│    → 运行测试套件                                             │
│    → 确保修复没有破坏现有功能                                  │
└─────────────────────────────────────────────────────────────┘
```

---

## 语言特定诊断命令

### TypeScript/JavaScript

```bash
# TypeScript 类型检查
npx tsc --noEmit --pretty
npx tsc --noEmit --pretty --incremental false   # 显示所有错误

# 构建命令
npm run build
yarn build
pnpm build

# ESLint 检查
npx eslint . --ext .ts,.tsx,.js,.jsx

# 清除缓存（终极方案）
rm -rf .next node_modules/.cache && npm run build
rm -rf node_modules package-lock.json && npm install
```

### Go

```bash
# Go 构建检查
go build ./...
go vet ./...

# Linter 检查
staticcheck ./... 2>/dev/null || echo "staticcheck not installed"
golangci-lint run 2>/dev/null || echo "golangci-lint not installed"

# 模块依赖
go mod verify
go mod tidy -v
go mod why -m package              # 为什么选择某个版本
```

### Rust

```bash
# Rust 编译检查
cargo check 2>&1
cargo clippy -- -D warnings 2>&1
cargo fmt --check 2>&1

# 依赖检查
cargo tree --duplicates 2>&1
cargo tree -i some_crate           # 反向依赖

# 安全审计
cargo audit 2>/dev/null || echo "cargo-audit not installed"
```

### C++

```bash
# CMake 构建
cmake --build build 2>&1 | head -100
cmake -B build -S . 2>&1 | tail -30

# 静态分析
clang-tidy src/*.cpp -- -std=c++17 2>/dev/null || echo "clang-tidy not available"
cppcheck --enable=all src/ 2>/dev/null || echo "cppcheck not available"

# 详细构建
cmake -B build -S . -DCMAKE_VERBOSE_MAKEFILE=ON
cmake --build build --verbose
```

### Java/Maven/Gradle

```bash
# Maven 构建
./mvnw compile -q 2>&1 || mvn compile -q 2>&1
./mvnw test -q 2>&1 || mvn test -q 2>&1

# Gradle 构建
./gradlew build 2>&1

# 依赖树
./mvnw dependency:tree 2>&1 | head -100
./gradlew dependencies --configuration runtimeClasspath 2>&1 | head -100

# 检查工具
./mvnw checkstyle:check 2>&1 || echo "checkstyle not configured"
./mvnw spotbugs:check 2>&1 || echo "spotbugs not configured"
```

### Kotlin

```bash
# Gradle 构建
./gradlew build 2>&1

# Kotlin 特定检查
./gradlew detekt 2>&1 || echo "detekt not configured"
./gradlew ktlintCheck 2>&1 || echo "ktlint not configured"

# 依赖分析
./gradlew dependencies --configuration runtimeClasspath 2>&1 | head -100
./gradlew dependencyInsight --dependency <name> --configuration runtimeClasspath
```

### Swift/Xcode

```bash
# Swift 构建
swift build 2>&1

# SwiftLint（如果安装）
swiftlint lint --quiet 2>&1 || echo "swiftlint not installed"

# SPM 依赖
swift package resolve 2>&1
swift package show-dependencies 2>&1

# Xcode 构建
xcodebuild -scheme <Scheme> -destination 'generic/platform=iOS Simulator' build 2>&1 | tail -50
xcodebuild -showBuildSettings 2>&1 | grep -E 'SWIFT_VERSION|CODE_SIGN'
```

### Dart/Flutter

```bash
# Flutter 分析
flutter analyze 2>&1
dart analyze 2>&1                    # 纯 Dart 项目

# Pub 依赖
flutter pub get 2>&1
flutter pub deps                     # 依赖树

# 代码生成
dart run build_runner build --delete-conflicting-outputs 2>&1

# 平台构建
flutter build apk 2>&1               # Android
flutter build ipa --no-codesign 2>&1 # iOS
flutter build web 2>&1               # Web
```

### Python/Django

```bash
# Python 版本检查
python --version
python -m django --version

# Django 配置验证
python manage.py check --deploy 2>&1 || python manage.py check 2>&1

# 迁移检查
python manage.py showmigrations 2>&1
python manage.py migrate --check 2>&1

# 静态文件
python manage.py collectstatic --dry-run --noinput 2>&1

# 依赖检查
pip check
pip list | grep -E "Django|djangorestframework|celery|psycopg"
```

### PyTorch

```bash
# PyTorch 环境检查
python -c "import torch; print(f'PyTorch: {torch.__version__}, CUDA: {torch.cuda.is_available()}, Device: {torch.cuda.get_device_name(0) if torch.cuda.is_available() else \"CPU\"}')"

# cuDNN 检查
python -c "import torch; print(f'cuDNN: {torch.backends.cudnn.version()}')" 2>/dev/null || echo "cuDNN not available"

# CUDA 测试
python -c "import torch; x = torch.randn(2,3).cuda(); print('CUDA tensor test: OK')" 2>&1

# GPU 内存
nvidia-smi 2>/dev/null || echo "nvidia-smi not available"
```

---

## 常见错误修复模式

### TypeScript 错误

| 错误 | 原因 | 修复 |
|------|------|------|
| `implicitly has 'any' type` | 缺少类型注解 | 添加类型注解 |
| `Object is possibly 'undefined'` | 可能为空 | 使用可选链 `?.` 或空值检查 |
| `Property does not exist` | 属性不存在 | 添加到接口或使用可选 `?` |
| `Cannot find module` | 模块未找到 | 检查 tsconfig paths、安装包、修复导入路径 |
| `Type 'X' not assignable to 'Y'` | 类型不匹配 | 解析/转换类型或修复类型定义 |
| `Hook called conditionally` | Hook 条件调用 | 将 hooks 移到顶层 |

### Go 错误

| 错误 | 原因 | 修复 |
|------|------|------|
| `undefined: X` | 缺少导入、拼写错误、未导出 | 添加导入或修复大小写 |
| `cannot use X as type Y` | 类型不匹配、指针/值问题 | 类型转换或解引用 |
| `X does not implement Y` | 缺少方法 | 用正确的接收器实现方法 |
| `import cycle not allowed` | 循环依赖 | 提取共享类型到新包 |
| `cannot find package` | 缺少依赖 | `go get pkg@version` 或 `go mod tidy` |
| `declared but not used` | 未使用的变量/导入 | 移除或使用空白标识符 |

### Rust 错误

| 错误 | 原因 | 修复 |
|------|------|------|
| `cannot borrow as mutable` | 不可变借用活跃 | 重构借用顺序或使用 `Cell`/`RefCell` |
| `does not live long enough` | 值在借用期间被丢弃 | 延长生命周期或使用拥有类型 |
| `cannot move out of` | 从引用中移动 | 使用 `.clone()`、`.to_owned()` |
| `mismatched types` | 类型不匹配 | 添加 `.into()`、`as` 或显式转换 |
| `trait X is not implemented for Y` | 缺少实现或 derive | 添加 `#[derive(Trait)]` 或手动实现 |
| `lifetime may not live long enough` | 生命周期约束太短 | 添加生命周期约束或使用 `'static` |

### C++ 错误

| 错误 | 原因 | 修复 |
|------|------|------|
| `undefined reference to X` | 缺少实现或库 | 添加源文件或链接库 |
| `no matching function for call` | 参数类型错误 | 修复类型或添加重载 |
| `use of undeclared identifier` | 缺少 include 或拼写错误 | 添加 `#include` 或修复名称 |
| `multiple definition of` | 重复符号 | 使用 `inline`、移到 .cpp 或添加 include guard |
| `cannot convert X to Y` | 类型不匹配 | 添加 cast 或修复类型 |
| `incomplete type` | 前向声明需要完整类型 | 添加 `#include` |

### Java 错误

| 错误 | 原因 | 修复 |
|------|------|------|
| `cannot find symbol` | 缺少导入、拼写错误、缺少依赖 | 添加导入或依赖 |
| `incompatible types: X cannot be converted to Y` | 类型不匹配 | 添加显式 cast 或修复类型 |
| `variable X might not have been initialized` | 未初始化的局部变量 | 在使用前初始化变量 |
| `package X does not exist` | 缺少依赖或导入错误 | 在 `pom.xml`/`build.gradle` 中添加依赖 |
| `Circular dependency involving X` | 构造器注入循环 | 重构打破循环或在一侧使用 `@Lazy` |
| `No qualifying bean of type X` | 缺少 `@Component`/`@Service` | 添加注解或修复扫描基础包 |

### Kotlin 错误

| 错误 | 原因 | 修复 |
|------|------|------|
| `Unresolved reference: X` | 缺少导入、拼写错误、缺少依赖 | 添加导入或依赖 |
| `Type mismatch: Required X, Found Y` | 类型不匹配 | 添加转换或修复类型 |
| `Smart cast impossible` | 可变属性或并发访问 | 使用局部 `val` 复制或 `let` |
| `'when' expression must be exhaustive` | sealed class `when` 缺少分支 | 添加缺失分支或 `else` |
| `Suspend function can only be called from coroutine` | 缺少 `suspend` 或协程作用域 | 添加 `suspend` 修饰符或启动协程 |

### Swift 错误

| 错误 | 原因 | 修复 |
|------|------|------|
| `cannot find type 'X' in scope` | 缺少导入或拼写错误 | 添加 `import Module` 或修复名称 |
| `value of type 'X' has no member 'Y'` | 类型错误或缺少扩展 | 修复类型或添加缺失方法 |
| `type 'X' does not conform to protocol 'Y'` | 缺少必需成员 | 实现缺失的协议要求 |
| `expression is 'async' but is not marked with 'await'` | 缺少 `await` | 添加 `await` 关键字 |
| `non-sendable type 'X' passed in implicitly asynchronous call` | Sendable 违规 | 添加 `Sendable` 一致性或重构 |
| `actor-isolated property cannot be referenced from non-isolated context` | Actor 隔离不匹配 | 添加 `await`、标记调用者为 `async` |

### Dart/Flutter 错误

| 错误 | 原因 | 修复 |
|------|------|------|
| `The name 'X' isn't defined` | 缺少导入或拼写错误 | 添加正确的 `import` 或修复名称 |
| `A value of type 'X?' can't be assigned to type 'X'` | 空安全 — 未处理可空 | 添加 `!`、`?? default` 或空值检查 |
| `The argument type 'X' can't be assigned to 'Y'` | 类型不匹配 | 修复类型、添加显式 cast |
| `Non-nullable instance field 'x' must be initialized` | 缺少初始化器 | 添加初始化器、标记 `late` 或使其可空 |
| `Because X depends on Y >=A and Z depends on Y <B, version solving failed` | Pub 版本冲突 | 调整版本约束或添加 `dependency_overrides` |

### PyTorch 错误

| 错误 | 原因 | 修复 |
|------|------|------|
| `RuntimeError: mat1 and mat2 shapes cannot be multiplied` | Linear 层输入大小不匹配 | 修复 `in_features` 以匹配前一层输出 |
| `RuntimeError: Expected all tensors to be on the same device` | CPU/GPU 张量混合 | 添加 `.to(device)` 到所有张量和模型 |
| `CUDA out of memory` | Batch 太大或内存泄漏 | 减少 batch size、添加 `torch.cuda.empty_cache()` |
| `RuntimeError: element 0 of tensors does not require grad` | 损失计算中的分离张量 | 在梯度计算前移除 `.detach()` |
| `RuntimeError: one of the variables needed for gradient computation has been modified by an inplace operation` | 原位操作破坏 autograd | 替换 `x += 1` 为 `x = x + 1` |

### Django 错误

| 错误 | 原因 | 修复 |
|------|------|------|
| `ModuleNotFoundError: No module named 'X'` | 缺少包 | `pip install X` 或添加到 `requirements.txt` |
| `InconsistentMigrationHistory` | 迁移顺序错误 | Squash 或 fake 迁移 |
| `Multiple leaf nodes in the migration graph` | 迁移分支冲突 | 合并：`python manage.py makemigrations --merge` |
| `django.core.exceptions.ImproperlyConfigured` | 缺少设置或值错误 | 检查 `settings.py` |
| `Invalid HTTP_HOST header` | `ALLOWED_HOSTS` 配置错误 | 添加主机名到 `ALLOWED_HOSTS` |

---

## 快速恢复命令

### TypeScript/JavaScript

```bash
# 清除所有缓存
rm -rf .next node_modules/.cache && npm run build

# 重新安装依赖
rm -rf node_modules package-lock.json && npm install

# ESLint 自动修复
npx eslint . --fix
```

### Go

```bash
# 模块清理
go clean -modcache && go mod download

# 依赖整理
go mod tidy
```

### Rust

```bash
# 清理构建
cargo clean

# 更新特定依赖
cargo update -p specific_crate        # 推荐
cargo update                          # 完全刷新（最后手段）
```

### Java/Maven

```bash
# 强制更新快照
./mvnw clean install -U

# 分析依赖冲突
./mvnw dependency:analyze

# 跳过测试隔离编译错误
./mvnw compile -DskipTests
```

### Kotlin/Gradle

```bash
# 强制刷新依赖
./gradlew build --refresh-dependencies

# 清理构建缓存
./gradlew clean && rm -rf .gradle/build-cache/
```

### Swift/SPM

```bash
# 清除包缓存
swift package reset
swift package resolve
```

### Flutter

```bash
# 清理 Flutter 缓存
flutter clean

# Pub 缓存修复
flutter pub cache repair

# Android 清理
cd android && ./gradlew clean && cd ..

# iOS 清理
cd ios && pod deintegrate && pod install && cd ..
```

### Django

```bash
# 迁移合并
python manage.py makemigrations --merge --no-input

# Fake 迁移（数据库已存在）
python manage.py migrate --fake <app> <migration_number>

# 强制重新安装依赖
pip install --force-reinstall -r requirements.txt
```

---

## 命令速查表

### 通用构建修复命令

| 命令 | 用途 | 适用场景 |
|------|------|----------|
| `/build-fix` | 修复构建错误 | 构建失败时 |
| `/go-build` | Go 构建错误修复 | Go 项目构建失败 |
| `/cpp-build` | C++ 构建/CMake 错误 | C++ 项目 |
| `/rust-build` | Rust cargo 构建错误 | Rust 项目 |
| `/kotlin-build` | Kotlin/Gradle 错误 | Kotlin 项目 |
| `/flutter-build` | Flutter/Dart 构建错误 | Flutter 项目 |

---

## 调试原则

### 核心原则

1. **最小化修改** — 只修复错误，不重构
2. **手术式修复** — 精准定位，最小改动
3. **验证优先** — 每次修复后立即验证
4. **根本原因** — 修复根本原因而非抑制症状
5. **不抑制警告** — 不使用 `// ignore`、`@SuppressWarnings` 等（除非明确批准）

### 禁止行为

| 行为 | 原因 |
|------|------|
| 重构无关代码 | 只修复错误，不改进 |
| 改变架构 | 构建修复不涉及架构变更 |
| 重命名变量 | 除非是错误原因 |
| 添加新功能 | 只修复现有问题 |
| 改变逻辑流 | 除非修复错误 |
| 优化性能/风格 | 不是构建修复的目标 |

### 停止条件

当以下情况发生时，停止并报告：

1. **同一错误在 3 次修复尝试后仍然存在**
2. **修复引入的错误比解决的更多**
3. **错误需要超出范围的架构变更**
4. **缺少需要用户决策的外部依赖**
5. **硬件/驱动不兼容（建议驱动更新）**

---

## 输出格式

### 标准输出格式

```text
[FIXED] src/handler/user.cpp:42
Error: undefined reference to `UserService::create`
Fix: Added missing method implementation in user_service.cpp
Remaining errors: 3
```

### 最终状态报告

```text
Build Status: SUCCESS/FAILED | Errors Fixed: N | Files Modified: list
```

---

## 典型调试流程示例

### 场景：TypeScript 构建失败

```
┌─────────────────────────────────────────────────────────────┐
│ npm run build 失败                                           │
│    → "Type 'string' is not assignable to type 'number'"      │
├─────────────────────────────────────────────────────────────┤
│ 1. 诊断                                                       │
│    npx tsc --noEmit --pretty                                  │
│    → 收集所有类型错误                                         │
├─────────────────────────────────────────────────────────────┤
│ 2. 定位                                                       │
│    Read src/utils/calculator.ts                               │
│    → 发现 line 42: price = quantity (string → number)        │
├─────────────────────────────────────────────────────────────┤
│ 3. 修复                                                       │
│    Edit: price = Number(quantity)                             │
│    → 最小化修改，只添加类型转换                                │
├─────────────────────────────────────────────────────────────┤
│ 4. 验证                                                       │
│    npx tsc --noEmit --pretty                                  │
│    → 错误数量从 5 减少到 4                                     │
├─────────────────────────────────────────────────────────────┤
│ 5. 测试                                                       │
│    npm test                                                   │
│    → 确保修复没有破坏现有测试                                  │
└─────────────────────────────────────────────────────────────┘
```

### 场景：Go 构建失败

```
┌─────────────────────────────────────────────────────────────┐
│ go build ./... 失败                                           │
│    → "undefined: UserService"                                 │
├─────────────────────────────────────────────────────────────┤
│ 1. 诊断                                                       │
│    go build ./... 2>&1                                        │
│    → 识别缺少导入的文件                                        │
├─────────────────────────────────────────────────────────────┤
│ 2. 定位                                                       │
│    Read internal/handler/user.go                              │
│    → 发现缺少 import "project/internal/service"              │
├─────────────────────────────────────────────────────────────┤
│ 3. 修复                                                       │
│    Edit: 添加 import "project/internal/service"               │
│    → 只添加缺失的导入                                         │
├─────────────────────────────────────────────────────────────┤
│ 4. 验证                                                       │
│    go build ./...                                             │
│    → 构建成功                                                 │
├─────────────────────────────────────────────────────────────┤
│ 5. 检查                                                       │
│    go vet ./...                                               │
│    go test ./...                                              │
│    → 确保 vet 和测试通过                                      │
└─────────────────────────────────────────────────────────────┘
```

### 场景：Rust 借用检查器错误

```
┌─────────────────────────────────────────────────────────────┐
│ cargo check 失败                                              │
│    → E0502: cannot borrow `map` as mutable                    │
│       because it is also borrowed as immutable                │
├─────────────────────────────────────────────────────────────┤
│ 1. 诊断                                                       │
│    cargo check 2>&1                                           │
│    → 识别借用冲突位置                                         │
├─────────────────────────────────────────────────────────────┤
│ 2. 定位                                                       │
│    Read src/handler/user.rs                                   │
│    → 发现 line 42: map.get() 和 map.insert() 同时存在        │
├─────────────────────────────────────────────────────────────┤
│ 3. 修复                                                       │
│    Edit: let value = map.get("key").cloned();                 │
│          if value.is_none() { map.insert(...); }              │
│    → Clone 结束不可变借用后再进行可变操作                      │
├─────────────────────────────────────────────────────────────┤
│ 4. 验证                                                       │
│    cargo check                                                │
│    → 借用检查器错误解决                                        │
├─────────────────────────────────────────────────────────────┤
│ 5. 测试                                                       │
│    cargo test                                                 │
│    → 确保测试通过                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 相关 Skills

| Skill | 用途 |
|-------|------|
| `golang-patterns` | Go 语言最佳实践 |
| `rust-patterns` | Rust 模式和错误处理 |
| `cpp-coding-standards` | C++ 编码规范 |
| `springboot-patterns` | Spring Boot 模式 |
| `quarkus-patterns` | Quarkus 模式 |
| `kotlin-patterns` | Kotlin 模式 |
| `swift-concurrency-6-2` | Swift 并发模式 |
| `flutter-dart-code-review` | Flutter/Dart 审查 |
| `pytorch-patterns` | PyTorch 最佳实践 |
| `django-patterns` | Django 架构模式 |

---

## 故障排除参考

详细故障排除指南请参考原项目 [TROUBLESHOOTING.md](https://github.com/affaan-m/everything-claude-code/blob/main/TROUBLESHOOTING.md)

---

## 相关链接

- **GitHub 仓库：** https://github.com/affaan-m/everything-claude-code
- **官网：** https://ecc.tools
- **作者：** [@affaanmustafa](https://x.com/affaanmustafa)

---

**Star 这个项目如果有帮助。构建伟大的东西。**