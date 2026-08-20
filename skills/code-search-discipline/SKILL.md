---
name: "code-search-discipline"
description: "代码搜索纪律：强制 grep 执行前限定范围（模块级精准子目录 path 或跨模块 include 类型过滤器），禁止无过滤全量裸搜；自动绕开 uni_modules、node_modules、static、dist、build 等大体积与构建产物目录；大型源码优先 codegraph 语义探索，grep 仅用于定位行号等窄查询，避免全仓搜索导致的超时、乱码与上下文污染。TRIGGER when: 需要搜索、查找、定位代码、符号、函数、字符串、报错文案所在文件，理解大型源码结构或调用关系，或即将调用 grep 和文件搜索工具。SKIP: 仅读取已知路径的单个文件（用 Read）；纯文档瘦身与检索治理（用 lean-docs）。"
version: "1.1.1"
author: "xiaoyu"
---

# 代码搜索纪律 (code-search-discipline)

> **核心思想**：搜索是手段不是目的。先选对工具、再限定范围、最后才动手——一次精确的窄查询胜过十次全仓裸搜。全仓 grep 的三大代价：超时（目录过大）、乱码（二进制/压缩文件）、上下文污染（vendor 副本与构建产物噪音）。

## 🎯 触发条件

- **TRIGGER when**（任一命中即激活，本技能是**前置约束**，先于任何搜索工具调用生效）:
  - 任务需要搜索、查找、定位代码：符号、函数、变量、字符串、报错文案、配置值出现在哪个文件哪一行
  - 需要理解大型源码：某个功能如何实现、某个函数被谁调用、改动会影响哪些模块
  - 即将调用任何内容搜索工具（grep）或文件枚举工具（glob / codegraph 系列）
- **SKIP**:
  - 已知文件路径，仅需读取单个文件内容 → 直接用 Read，无需搜索
  - 纯文档瘦身、文档检索治理、降低文档 Token 消耗 → 用 `lean-docs`
  - 纯日志检索（非代码文件的内容搜索）→ 不强制 codegraph；但范围限定与风险目录绕开是**全局铁律**，独立于本技能激活状态，任何 grep 调用（含日志检索）均须遵守

## ⚙️ 依赖与先决条件

- **专用搜索工具（Grep）**：支持 `path`（限定搜索目录）与 `include`（限定文件后缀/Glob 模式）参数。
- **语义探索工具（codegraph 系列）**：`codegraph_explore`、`codegraph_search`、`codegraph_node`、`codegraph_callers`、`codegraph_callees`、`codegraph_impact`、`codegraph_files`。需项目已建立 `.codegraph/` 索引；未索引时按阶段 1 降级。
- **文件枚举工具（glob / codegraph_files）**：在目录结构不明确时，先探测 1 级目录树确认分布，再决定进入哪个子目录。
- **命令行工具（若使用 Bash rg/grep）**：类 Unix 终端环境。参数按工具区分——ripgrep 用 `-g '*.ts'` 过滤文件类型、`-g '!node_modules/**'` 排除目录；GNU grep 用 `--include='*.ts'` 过滤、`--exclude-dir=node_modules` 排除（**rg 不支持 `--exclude-dir`**，照抄会报 `unrecognized flag`）。

## 📖 标准工作流

### 阶段 1：先选工具，再动手（工具分层）

搜索前先判断“要找的是哪一类信息”，严格按优先级选择工具：

| 信息类型 | 典型问题 | 首选工具 | 降级路径 |
| :--- | :--- | :--- | :--- |
| **语义与流程** | 登录流程如何实现、AuthService 被谁调用、修改 X 影响哪些模块 | `codegraph_explore`、`codegraph_callers`、`codegraph_impact` | 无索引 → 先用 glob 摸清目录，再按阶段 2 进行 grep 窄查询 + Read |
| **符号定义** | getUserInfo 定义在哪个文件 | `codegraph_search`、`codegraph_node` | 无索引 → grep 精确符号 + include 源码类型 |
| **字面量/报错** | 报错文案在哪行、配置常量写在哪 | grep（必须执行阶段 2 限定与阶段 3 绕开） | — |
| **文件分布** | src 下有哪些 .vue 文件、模块目录结构 | `codegraph_files`（已索引）或 glob | — |

### 阶段 2：grep 必须限定范围（双合法模式）

grep 必须采取以下两种合法模式之一，**绝对禁止无任何过滤的全量裸搜**：

1. **模式 A · 模块级精准搜索（首选）**：
   - 目标模块明确时，显式指定具体子目录 `path`（如 `src/modules/auth`、`src/utils`）。
   - 配合 `include`（如 `*.ts`）锁定文件扩展名。
2. **模式 B · 跨模块广域搜索（备选）**：
   - 跨模块或模块未知时，`path` 允许指向源码根（`src/` 或项目对应根目录，如 uni-app 的 `pages/`），但**必须强制指定 `include`**（如 `*.{ts,vue}`），把范围钉死在目标语言，严禁不带 include 裸扫源码根。
3. **搜索词精准化（防泛词）**：
   - 使用完整标识符（如 `getUserInfo`）、带调用特征（如 `getUserInfo\(`，或直接用 `-F` 固定字符串模式免去正则转义）或词边界（如 `\bauth\b`；工具不支持 `\b` 时改用 `-w`）。
   - **禁止单字母或通用泛词**（如 `data`、`user`、`value`、`error`、`config`）进行无修饰模糊搜索。
4. **结果闭环处理机制**：
   - **结果过载（> 50 行）或超时**：立即停止，下钻子目录或追加更精确的搜索词/include，**禁止无序扩大范围或全仓扩散重试**。
   - **结果未命中（0 结果）**：先用 `glob` 或 `codegraph_files` 探测 1 级目录树确认是否猜错目录，再按模式 B（在 `src/` 下带 `include`）扩大一阶搜索，**禁止直接去除所有过滤全仓扫荡**。

### 阶段 3：绕开风险目录

以下目录**绝对禁止进入搜索范围**（除非用户显式要求）：

| 类别 | 风险目录/文件 | 潜在灾难 |
| :--- | :--- | :--- |
| **依赖安装** | `node_modules/`、`uni_modules/`、`vendor/`、`.pnpm/` | 体积巨大导致超时；vendor 副本与业务代码同名污染上下文 |
| **构建产物** | `dist/`、`build/`、`.turbo/`、`unpackage/`、`miniprogram_npm/` | 压缩代码与 SourceMap（`.min.js`/`.map`），命中即大量噪音 |
| **静态资源** | `static/`、`assets/`（含压缩包、图片、字体、大 JSON） | 二进制文件输出乱码，大文件导致搜索卡死 |
| **版本控制/IDE** | `.git/`、`.idea/`、`.vscode/` | 内部元数据噪音 |

**绕开手段**（按优先级）：
1. `path` 直达源码业务目录（如 `src/pages/`），天然隔绝风险目录；
2. `include` 限定源码扩展名（如 `*.{ts,vue,js}`），自动过滤二进制与构建产物；
3. Bash 命令行下显式追加排除参数（如 ripgrep `rg -g '!node_modules/**' -g '!uni_modules/**'`；GNU grep `--exclude-dir=node_modules`）。

### 阶段 4：执行与结果消化

1. grep 定位到文件与行号后，**使用 Read 工具读取该位置上下文**（配合 offset/limit），理解完毕即停止，禁止反复全量 grep。
2. codegraph 无索引时按阶段 1 降级表执行，降级路径同样严格遵守阶段 2/3 约束。

## ⛔ 行为护栏

- **🚫 禁止全量裸搜**：禁止既不指定具体子目录 `path`、又不指定 `include` 类型的全局扫描；禁止裸搜 `src/` 根。
- **🚫 禁止搜索风险目录**：未经用户显式要求，`node_modules/`、`uni_modules/`、`static/`、`dist/`、`build/`、`.git/` 等一律不得作为搜索路径（与阶段 3 例外条款一致）。
- **🚫 禁止 grep 替代语义探索**：语义理解类与调用链路类问题，必须首选 codegraph 工具族，不可直接拿 grep 碰运气。
- **🚫 禁止泛词模糊搜索**：禁止使用 `data`、`user`、`value`、`res` 等超高频泛词进行无边界模糊搜索。
- **🚫 禁止重复全量 grep**：定位到行号后立即使用 Read 工具读上下文，命中即止。
- **🛑 异常收窄与有序上浮**：结果超过 50 行或超时立即停止并收窄；0 命中时先探目录再按模式 B 上浮一阶，禁止无序全仓扩散。

## 📝 模板与范例

### 场景 1：精准搜索 vs 全量裸搜

### <Bad>
```text
# ❌ 场景：找 getUserInfo 的定义
grep pattern: "getUserInfo"              # 未指定 path，默认扫全仓
grep pattern: "getUserInfo", path: src/  # 裸搜 src/ 根，未加 include 过滤
```

### <Good>
```text
# ✅ 模式 A：下钻到具体业务子目录，限定文件类型
grep pattern: "getUserInfo", path: src/modules/user, include: "*.ts"

# ✅ 模式 B：跨模块位置未知时，在 src/ 下强制限定文件扩展名
grep pattern: "getUserInfo", path: src, include: "*.{ts,vue}"
# 命中后使用 Read 工具读取对应文件行号上下文
```

### 场景 2：风险目录绕开

### <Bad>
```text
# ❌ 场景：uni-app 项目，想确认 z-paging 组件用在哪
grep pattern: "z-paging", path: .  # 搜出 uni_modules 里的组件实现源码，产生大量噪音
```

### <Good>
```text
# ✅ 只在业务页面目录内搜索，排除依赖目录
grep pattern: "z-paging", path: src/pages, include: "*.vue"
# 若需查阅组件实现本身，直接使用 Read 读取已知路径 uni_modules/z-paging/components/...
```

### 场景 3：语义问题选择专用工具

### <Bad>
```text
# ❌ 场景：理解"登录流程是怎么实现的"
grep pattern: "login", path: src, include: "*.ts"  # 命中几十处分散调用，无法拼装完整流程
```

### <Good>
```text
# ✅ 语义探索用 codegraph
codegraph_explore query: "登录流程 login auth 实现"    # 一次性返回关键符号与上下文源码
codegraph_callers symbol: "LoginService.login"         # 精确查看上游调用链路
```

## 📜 版本变更历史

- **v1.1.1** (2026-08-20): 修复审查发现的问题——按工具区分 rg 与 GNU grep 的过滤/排除参数（实测 `rg` 不支持 `--exclude-dir`）；搜索词示例补充正则转义与 `-F`/`-w` 用法；护栏与阶段 3 的风险目录例外条款对齐（补"未经用户显式要求"）；"禁止扩大范围"改为"禁止无序扩大"，消除与 0 命中上浮的字面矛盾；声明范围限定为全局铁律（不受 SKIP 影响）；模式 B 源码根泛化（支持 `pages/` 等根目录）。
- **v1.1.0** (2026-08-20): 消除规则逻辑冲突与表述歧义：
  1. 明确双合法模式（模式 A · 模块级精准 vs 模式 B · 跨模块带 include 广域）；
  2. 补充结果闭环机制（过载 > 50 行收窄 + 0 命中时探测目录并有序上一阶上浮）；
  3. 完善泛词防御与词边界修饰约束；
  4. 隔离专用 Grep 参数与 Bash 命令行参数。
- **v1.0.0** (2026-08-17): 初始版本。规范 grep 范围限定、风险目录绕开与 codegraph 优先工具分层。
