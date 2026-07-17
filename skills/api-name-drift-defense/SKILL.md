---
name: api-name-drift-defense
description: "防御第三方依赖库在版本升级过程中的 API 重命名、移除或签名变更问题。TRIGGER when: 运行时出现导入或调用报错、之前正常工作的 API 出现类型错误、升级依赖大版本后排查破坏性变更。SKIP: 无关的普通语法错误或纯业务逻辑 Bug。"
version: "1.1.0"
author: xiaoyu
---

> **生命周期阶段**：稳定

## 🎯 触发条件

- **TRIGGER when**: 
  - 运行时出现 `X is not exported` / `X is not defined` / `X is not a function` 等导入或调用报错
  - 编译报错 `Module has no exported member 'X'` 或类似信息
  - 之前正常工作的第三方 API 出现类型错误或签名不匹配
  - 升级依赖版本后，原有代码或测试标红，需排查破坏性变更
- **SKIP**: 普通的独立业务逻辑 Bug 或显而易见的本地拼写/语法错误

## ⚙️ 依赖与先决条件

不限定具体技术栈。不同生态系统中 API 漂移的表现形式各异，但排查思路一致，可通过以下命令探测对应工具链环境：

| 生态系统 | 包管理器 / 依赖目录 | 校验/诊断探测命令 |
|----------|----------|---------------|
| Node.js | npm / pnpm / yarn | `node -v` 且 `cat package.json` |
| Python | pip / poetry / uv | `python --version` 且 `pip --version` |
| Java | Maven / Gradle | `mvn -v` 或 `gradle -v` |
| Rust | Cargo | `cargo --version` |
| Go | go mod | `go version` |

## 📖 标准工作流

### 阶段 0：识别项目类型与依赖位置

在执行排查前，先确定当前项目的技术栈和依赖存放位置：

1. 查看项目根目录的构建配置文件（`package.json` / `pyproject.toml` / `pom.xml` / `Cargo.toml` / `go.mod`）
2. 在命令行查看对应依赖的已安装版本，例如：
   - Node: `cat node_modules/<pkg>/package.json | grep '"version"'`
   - Python: `pip show <pkg> | grep Version`
   - Java: `mvn dependency:tree -Dincludes=<group>:<artifact>` 或查看 `pom.xml` 中的版本声明
   - Rust: `cargo tree -p <crate> --depth 0`
   - Go: `go list -m <module>`
3. 自查该依赖的 CHANGELOG、Release Notes 或 Migration Guide，寻找 Breaking Changes 线索

### 阶段 1：定位正确的 API

绝不依赖记忆！必须通过实证手段确认真实的 API 名称：

- **静态语言/有类型声明**: 查看依赖的类型声明文件或源码，确认导出名称和签名
  - Node: `grep -r "export.*function\|export.*const" node_modules/<pkg>/` 或在 `.d.ts` 中搜索
  - Python: 使用 `python -c "import <pkg>; print(dir(<pkg>))"` 列出可用符号，或 `help(<pkg>.<func>)`
  - Go / Rust: 直接用 Read 查看依赖源码或文档
- **查阅文档**: 使用 WebFetch 获取该依赖对应版本的官方文档，确认 API 的正确用法

### 阶段 2：修复代码

1. 确认变更后的实际 API 名称和签名后，修正所有调用点
2. 如果原先依赖了 auto-import 或隐式全局注入，改为显式导入语句，确保可追溯

### 阶段 3：回归验证

修复后通过技能调用执行 `self-check-trinity`，确保代码通过 Lint、类型/编译检查、测试全部通过

## ⛔ 行为护栏

- **实证优先原则**: 禁止在未查看依赖实际导出和文档的情况下凭记忆编写修复代码，避免次生破坏
- **保持引用一致性**: 所有 API 调用必须通过显式导入，禁止依赖隐式的全局注入或 auto-import
- **沉淀踩坑记录**: 确认第三方改名事件后，建议在当前项目或团队知识库中记录该变更，避免团队成员重复踩坑
- **禁止盲目升级**: 排查 API 漂移时，禁止在执行根因调查之前直接执行 `npm update` / `pip install --upgrade` / `cargo update` 等升级命令。应优先修复调用点以适配当前版本，升级依赖版本需要用户明确确认

## 📝 模板与范例

### <Bad>

```ts
// 错误：依赖了已被重命名或移除的旧 API
// Node (h3) 示例
export default defineEventHandler(async (event) => {
  const data = await readMultipartData(event) // 实际已改为 readMultipartFormData
})
```

```python
# 错误：使用了已废弃的 API
# Python 示例
from sklearn.cross_validation import train_test_split  # 已移至 sklearn.model_selection
```

### <Good>

```ts
// 正确：查明变更后的实际方法名并显式导入
// Node (h3)
import { readMultipartFormData } from 'h3'

export default defineEventHandler(async (event) => {
  const data = await readMultipartFormData(event)
})
```

```python
# 正确：使用迁移后的 API
# Python
from sklearn.model_selection import train_test_split
```

```java
// 正确：Java Spring Boot 升级后 API 变更
// 旧: WebSecurityConfigurerAdapter（Spring Security 5）
// 新: SecurityFilterChain Bean（Spring Security 6）
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    return http.build();
}
```
