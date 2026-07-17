---
name: self-check-trinity
description: "强制在交付任何代码变更前依次执行 lint、typecheck、test 三道质量检查。TRIGGER when: 完成功能开发、修复 Bug、准备提交或发起 PR、被要求验证/自检/交付/收尾。SKIP: 仅修改文档、注释或 README，不涉及运行时代码。"
version: "1.0.1"
author: xiaoyu
---

> **生命周期阶段**：稳定

## 🎯 触发条件

- **TRIGGER when**:
  - 完成代码修改，准备提交或发起 PR 时
  - 被要求执行"验证"、"自检"、"交付"或"收尾"时
  - 修复 Bug 并准备标记为完成时
- **SKIP**: 仅修改文档、注释或 README，不涉及运行时代码时

## ⚙️ 依赖与先决条件

不限定具体技术栈。核心要求是当前项目已配置 Lint、Typecheck 及 Test 的一项或多项检查能力。

**校验与诊断命令**：
- 包管理器探测：可通过运行 `ls pnpm-lock.yaml || ls package-lock.json || ls Cargo.toml || ls pyproject.toml` 等确认项目环境。
- 可通过执行 `git status` 确认当前有需要验证的代码变更。

## 📖 标准工作流

所有代码在交付前必须依次通过三道检查，任一步骤失败则必须修复后重试。

### 阶段 0：识别项目类型与命令

在执行检查前，通过以下方式确定具体命令：

1. 查看项目根目录的构建配置文件，识别技术栈与对应包管理器，确定可执行指令。
2. 查看对应配置文件，确认脚本名称是否与常规配置一致；如有差异，以实际配置为准。若项目中未找到任何相关脚本，向用户询问。

### 阶段 1：Lint 检查

执行项目对应的 lint 命令。如有报错，逐一修复后重试。

### 阶段 2：类型 / 编译检查

执行项目对应的类型检查命令。Lint 无法检测出类型错误、缺失的 import、编译失败等问题，必须通过此步骤排查。如果报错涉及第三方 API 改名，可通过技能调用执行 `api-name-drift-defense` 辅助排查。

### 阶段 3：测试

执行测试命令。确保所有测试通过，代码没有引入回归问题。

## ⛔ 行为护栏

- **禁止跳过检查**: 三道检查任一未通过，禁止提交代码。尤其禁止使用 `--no-verify` 或等效手段跳过 Git hooks
- **Lint 通过不代表代码正确**: Lint 只检查风格和基础语法，无法保证类型安全和逻辑正确。必须完成全部三步才算真正通过
- **环境问题如实上报**: 如果因本地网络、依赖安装等环境原因导致检查无法通过，不得伪造通过或跳过。应在最终报告中向开发者说明未通过步骤及原因
- **三次重试上限**: 如果修复同一问题超过三次仍未通过，应停止并询问用户，而不是反复尝试
- **先检测再执行**: 禁止不经检测就假定命令名称，必须按照阶段 0 的流程确认后再执行

## 📝 模板与范例

### <Bad>

```bash
# 错误：只跑 Lint 就跳过后续检查直接提交
npm run lint
git commit -m "fix: something" --no-verify
```

### <Good>

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
