---
name: self-check-trinity
description: 强制在交付任何代码变更前依次执行 lint → typecheck → test 三道质量检查。TRIGGER when: 完成功能开发、修复 Bug、准备提交或发起 PR、被要求"验证"/"自检"/"交付"/"收尾"。SKIP: 仅修改文档、注释或 README，不涉及运行时代码。
version: "1.0.1"
author: xiaoyu
---

## 🎯 触发条件

- **TRIGGER when**:
  - 完成代码修改，准备提交或发起 PR 时
  - 被要求执行"验证"、"自检"、"交付"或"收尾"时
  - 修复 Bug 并准备标记为完成时
- **SKIP**: 仅修改文档、注释或 README，不涉及运行时代码时

## ⚙️ 依赖环境

不限定具体技术栈。核心要求是当前项目已配置以下三类检查能力（形式不限）：

- **Lint / 静态分析**：代码风格和基础语法检查
- **类型 / 编译检查**：编译或类型系统层面的正确性验证
- **测试**：单元测试或集成测试框架

## 📖 核心工作流

所有代码在交付前必须依次通过三道检查，任一步骤失败则必须修复后重试。

### 阶段 0：识别项目类型与命令

在执行检查前，先通过以下方式确定具体命令：

1. 使用 Glob 或 LS 工具查看项目根目录的构建配置文件，识别技术栈：

| 标志文件 | 技术栈 | Lint | Typecheck | Test |
|----------|--------|------|-----------|------|
| `package.json` | Node/前端 | `lint` 或 `eslint`/`prettier` 脚本 | `typecheck` 或 `tsc --noEmit` | `test` 脚本 |
| `pom.xml` | Java Maven | `checkstyle:check` 或 `spotless:check` | `compile` | `test` |
| `build.gradle` | Java Gradle | `checkstyleMain` 或 `spotlessCheck` | `compileJava` | `test` |
| `Cargo.toml` | Rust | `clippy` | `check` | `test` |
| `go.mod` | Go | `vet` | `build ./...` | `test ./...` |
| `pyproject.toml` / `setup.cfg` | Python | `flake8` / `ruff check` | `mypy` / `pyright` | `pytest` |
| `Makefile` / `CMakeLists.txt` | C/C++ | `lint` / `cppcheck` | `build` / `make` | `test` |

2. 使用 Read 工具查看对应配置文件，确认脚本名称是否与上表一致；如有差异，以实际配置为准。若项目中未找到任何相关脚本，使用 AskUserQuestion 工具询问用户

### 阶段 1：Lint 检查

使用 Bash 执行项目对应的 lint 命令。如有报错，使用 Edit 工具逐一修复后重试

### 阶段 2：类型 / 编译检查

使用 Bash 执行类型或编译检查命令。Lint 无法检测出类型错误、缺失的 import、编译失败等问题，必须通过此步骤排查。如果报错涉及第三方 API 改名，可调用 Skill 工具执行 `api-name-drift-defense` 辅助排查

### 阶段 3：测试

使用 Bash 执行测试命令。确保所有测试通过，代码没有引入回归问题

## ⛔ 行为限制与护栏

- **禁止跳过检查**: 三道检查任一未通过，禁止提交代码。尤其禁止使用 `--no-verify` 或等效手段跳过 Git hooks
- **Lint 通过不代表代码正确**: Lint 只检查风格和基础语法，无法保证类型安全和逻辑正确。必须完成全部三步才算真正通过
- **环境问题如实上报**: 如果因本地网络、依赖安装等环境原因导致检查无法通过，不得伪造通过或跳过。应在最终报告中向开发者说明未通过步骤及原因
- **三次重试上限**: 如果修复同一问题超过三次仍未通过，应停止并询问用户，而不是反复尝试
- **先检测再执行**: 禁止不经检测就假定命令名称，必须按照阶段 0 的流程确认后再执行

## 📝 模板与范例

<Bad>

```bash
# 错误：只跑 Lint 就跳过后续检查直接提交
npm run lint
git commit -m "fix: something" --no-verify
```

</Bad>

<Good>

```bash
# Node 项目示例
pnpm run lint
pnpm run typecheck
pnpm run test
```

```bash
# Python 项目示例
ruff check .
mypy src/
pytest
```

```bash
# Rust 项目示例
cargo clippy -- -D warnings
cargo check
cargo test
```

```bash
# Java Maven 项目示例
mvn checkstyle:check
mvn compile
mvn test
```

</Good>
