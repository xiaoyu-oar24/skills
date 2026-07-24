---
name: xy-feat
description: "端到端功能开发工作流（需求澄清、方案设计、规划分解、TDD/VDD 执行、验证、代码审查收尾、指南与合规整理）。优先调用 superpowers 技能链，未安装时按需降级为 references/ 内联流程。TRIGGER when: 用户显式输入 '/xy-feat' 时激活。SKIP: 其他任何场景均不触发。"
version: "5.2.0"
author: xiaoyu
---

# xy-feat

> **生命周期阶段**：稳定 (v5.2.0)
> **定位**：将 **需求澄清 → 方案设计 → 规划分解 → TDD/VDD 执行 → 验证完成 → 审查收尾 → 指南与归档** 串成一条端到端流水线。
> **双模执行**：每个阶段**优先通过技能调用机制调用 superpowers 对应技能**；若 superpowers 未安装（技能调用返回不可用），则按需读取 `references/` 下的对应降级文件执行内联流程（对应关系见"Superpowers 技能依赖"表）。
> **核心思想**：文档是知识图谱，不是 Git Commit 历史。每个阶段的产出文档随代码演进而演进，而非每次修改新建文件。

## 🎯 触发条件

- 仅在用户显式输入 `/xy-feat [需求]` 时触发。其他自然语言描述（如"开发一个新功能"、"写一个 xxx 特性"）**不触发**本工作流。
- 由本工作流统一调度，**不允许跳过阶段**（除非用户明确授权）。

## ⚙️ 依赖与先决条件

- **命令行工具环境**：支持 `mkdir`、`date` 以及常规脚本操作的 Unix-like 终端。可通过在终端运行 `date` 确认环境就绪。
- **项目包管理器/测试运行器**：由 AI 运行时根据项目锁文件判定（如包含 `pnpm-lock.yaml` -> 使用 `pnpm`；包含 `Cargo.toml` -> 使用 `cargo`；包含 `pyproject.toml` -> 使用 `poetry`/`pytest`），无需预先假设。

### Superpowers 技能依赖（优先模式）

> 以下技能为本工作流各阶段的**首选执行载体**。可用性探测方式：优先尝试技能调用，返回不可用即自动降级为 `references/` 内联流程。
> **阶段要求**：“阶段必选”指该**阶段**不可跳过；执行载体不可用时必须走降级流程。

| 技能 | 阶段 | 阶段要求 | 降级策略 |
| :--- | :--- | :--- | :--- |
| `brainstorming` | 阶段 1 | 阶段必选 | 读取 `references/fallback-init-design.md` 执行一站式 Draft 呈现流程 |
| `writing-plans` | 阶段 2 | 推荐 | 读取 `references/fallback-plan.md` 执行内联流程（含批量预授权） |
| `test-driven-development` | 阶段 3 | 推荐 | 读取 `references/fallback-tdd.md` 执行内联 TDD/VDD 循环 |
| `dispatching-parallel-agents` | 阶段 3 | 可选 | 读取 `references/fallback-tdd.md` 执行内联并行策略（子代理禁写 tracking.md） |
| `subagent-driven-development` | 阶段 3 | 可选 | 读取 `references/fallback-tdd.md` 执行内联子代理策略 |
| `systematic-debugging` | 阶段 3 | 推荐 | 读取 `references/fallback-tdd.md` 执行内联四阶段调试法 |
| `verification-before-completion` | 阶段 4 | 阶段必选 | 读取 `references/fallback-verify-finish.md` 执行内联校验 |
| `requesting-code-review` | 阶段 5 | 推荐 | 读取 `references/fallback-verify-finish.md` 执行内联审查（代码归档前审查） |
| `finishing-a-development-branch` | 阶段 5 | 推荐 | 读取 `references/fallback-verify-finish.md` 执行内联收尾（含 Git Diff 测试去重） |
| `docs-layout-quadrant` | 阶段 6 | 阶段必选 | 本地技能，负责使用指南产出、五维整理、收尾归档与规则文件注入 |
| `using-git-worktrees` | 阶段 0 | 可选 | 读取 `references/fallback-init-design.md` 执行内联隔离策略 |

### 本地技能依赖

> 以下技能为本仓库自维护，不受外部安装状态影响。

| 技能 | 阶段 | 角色 |
| :--- | :--- | :--- |
| `self-check-trinity` | 阶段 4 | Lint → Typecheck → Test 三道质量关卡 |
| `docs-layout-quadrant` | 阶段 6 | 使用指南产出、文档合规整理与归档、规则文件注入（AGENTS.md/CLAUDE.md）— 含弹性降级决策树 |

- **工具授权**：用户调用本工作流即视为明确授权进行文件读写、目录创建。

## 📐 文档命名规范

> **核心原则**：文档是知识图谱，不是 Git Commit 历史。文件名仅描述模块/领域，**禁止任何日期/时间戳前缀**。
> 详细规范定义在 `skills/docs-layout-quadrant/SKILL.md`，此处列出本工作流直接使用的约定：

| 象限 | 路径 | 命名模式 | 示例 | 说明/约束 |
| :--- | :--- | :--- | :--- | :--- |
| 需求文档 | `docs/reqs/` | `<功能名>-req.md` | `gateway-proxy-req.md` | 按决策树判定 |
| 设计规范 | `docs/specs/` | `<功能名>-design.md` | `gateway-proxy-design.md` | 核心必选 |
| 执行计划 | `docs/plan/` | `<功能名>-plan.md` | `gateway-proxy-plan.md` | 核心必选 |
| 进度跟踪 | `docs/tracking/` | `<功能名>.md` | `gateway-proxy.md` | xy-feat 上下文中强制必选 |
| 使用指南 | `docs/guide/` | `<功能名>-guide.md` | `gateway-proxy-guide.md` | 核心必选 |

- `<功能名>` 使用英文 kebab-case 或简明中文，描述功能模块本身，**不含日期**。
- 同一功能的五维文档共享相同的 `<功能名>` 前缀。
- **活文档策略**：若同名文档已存在，优先**原地更新**（文档随代码演进）。若必须保留旧版，将旧版追加 `-archived` 后缀后移入统一归档目录 `docs/.archive/{象限}/`，保持主文件名不变。

## 📖 标准工作流

> **总览**：`阶段 0 初始化` → `阶段 1 方案设计` → `阶段 2 规划分解` → `阶段 3 TDD/VDD 执行` → `阶段 4 验证完成` → `阶段 5 代码审查与收尾` → `阶段 6 指南与合规整理`
> 本文件只定义各阶段的调度逻辑、产出物与出入口条件；各阶段的**完整操作细节**在降级时按需从 `references/` 加载。

---

### 阶段 0：需求获取与初始化

**使用工具**：命令行/终端、向用户提问

1. **需求确认**：若用户输入为空或不清晰，向用户提问："你要做什么功能？"
2. **功能名确定**：从需求中提炼出 `<功能名>`（kebab-case 或简明中文），用于全流程文档命名。提炼结果**必须经用户确认**（或用户明确授权自动定名）后方可使用。
3. **目录准备**：确保 `docs/specs`、`docs/plan`、`docs/guide` 三个核心目录及 `docs/tracking` 强约束目录存在（`mkdir -p docs/specs docs/plan docs/guide docs/tracking`）。`docs/tracking/` **在 `/xy-feat` 上下文强制创建**（保障阶段 3 多 Agent 并发安全）；`docs/reqs/` 按决策树判定（见 `docs-layout-quadrant` 阶段 2）。
4. **冲突检测（活文档策略）**：

   ```
   若 docs/specs/<功能名>-design.md 已存在：
     默认行为：原地更新该文档（活文档理念，文档随代码演进）
     若用户明确要求保留旧版：
       1. 将旧文件重命名为 <功能名>-archived.md，移入 docs/.archive/specs/
       2. 新版本使用原始名称 <功能名>-design.md
   ```

   **禁止**自动追加 `-v2`/`-v3` 后缀——这会制造多版本歧义，违背知识图谱理念。
5. *(可选)* 若用户提到"隔离开发"或当前工作区已有未提交变更：
   > **优先**：通过技能调用 `using-git-worktrees`
   > **降级**：读取 `references/fallback-init-design.md` 的"阶段 0 降级"节执行
   > 若平台已有原生隔离工具，优先使用原生工具。

---

### 阶段 1：方案设计

**使用工具**：读取/写入文件、向用户提问

> **优先**：通过技能调用 `brainstorming`。
> **降级**：读取 `references/fallback-init-design.md`，执行一站式 Draft 呈现审批流程（需求澄清 → 方案对比 → 结构化 Draft 一次性提交审批 → 撰写设计文档 → 自审）。

**必须产出**：`docs/specs/<功能名>-design.md`（背景、方案选择理由、架构设计、接口定义、数据模型、测试策略）。

> **reqs/ 内联降级**：若经决策树判定（纯技术重构/优化）跳过独立 `reqs/`，必须在 design.md 头部包含【`## 1. 需求背景与改造目标`】章节作为降级替代（模板见 `docs-layout-quadrant` 范例）。

**阶段出口**：设计文档完成自审（无占位符、内部一致、范围聚焦、无歧义）并获用户批准。

---

### 阶段 2：规划分解

**使用工具**：读取/写入文件

> **优先**：通过技能调用 `writing-plans`。
> **降级**：读取 `references/fallback-plan.md`，执行内联流程（任务拆解原则、任务卡片四要素、规划文档模板、批量 Git 原子提交预授权、自审清单）。

**必须产出**：
- `docs/plan/<功能名>-plan.md`：任务拆解到 2-5 分钟粒度，每个任务卡片含 Files / Interfaces / Steps / Expected，**禁止 TBD/TODO**。
- `docs/tracking/<功能名>.md`：**在 `/xy-feat` 上下文强制创建**，初始化全部任务卡片状态为 `⏳`（格式模板见 `references/fallback-plan.md`）。`tracking/` 的强制创建保障阶段 3 多 Agent 并发写入安全（子代理禁写、主编排代理集中更新）。

**阶段出口**：规划自审通过（Spec 全覆盖、无占位符、跨任务类型/签名一致）并获用户确认。在送审时声明：“批准本计划即授权 AI 在阶段 3 执行本地 Git 原子提交 (`git commit -m ...`)”。

---

### 阶段 3：TDD / VDD 执行

**使用工具**：命令行/终端、文件读取/编辑、子代理调度

> **优先**：执行过程中遇到下列场景时，优先通过技能调用对应 superpowers 技能：
> - 编写实现代码前 → `test-driven-development`
> - 遇到 Bug / 测试失败 → `systematic-debugging`
> - 多个独立任务可并行 → `dispatching-parallel-agents`
> - 单个复杂任务 → `subagent-driven-development`
>
> **降级**：读取 `references/fallback-tdd.md`（含 TDD/VDD 双模判定、单任务循环、任务调度决策树、四阶段调试法）。

**模式判定与 Iron Law**：

```
前置判断：当前任务是否属于纯 UI/CSS 样式、配置文件 (Dockerfile, CI) 或无单测框架环境？
├── 否 → 严格执行 TDD Iron Law: NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
│        （先写测试 → 看它失败 → 写最小实现 → 看它通过 → 重构）
└── 是 → 自动切换为 VDD (Validation-Driven Development)
         （RED 阶段运行静态检查 tsc/stylelint 或断言状态 → GREEN 阶段写入最小修改通过检查）
```

**执行要求（任何模式下都必须遵守）**：
- **子代理并发安全**：**禁止**并行子代理直接使用 Edit 修改全局 `docs/tracking/<功能名>.md`。子代理只需在返回消息中包含 `Task N Complete`，由**主编排代理**集中统一更新跟踪文件。
- **实时追踪**：主编排代理每确认完成一个任务卡片，立即将 `docs/tracking/<功能名>.md` 对应条目标记为 `✅`，并记录关键提交 hash。
- **调试纪律**：遇到 Bug 先走根因调查（四阶段调试法），最多 3 次不同假设的重试，禁止猜测式修复。
- **跨任务对齐**：开始下一个任务卡片前，重新查看 plan 文档的设计上下文，严防"文档写一套、代码写另一套"。

---

### 阶段 4：验证完成

**使用工具**：命令行/终端、技能调用

> **优先**：通过技能调用 `verification-before-completion`；三道质量闸门统一通过本地 `self-check-trinity` 技能执行（不依赖 superpowers）。
> **降级**：读取 `references/fallback-verify-finish.md` 执行内联校验。

**The Iron Law**：

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

在没有运行验证命令并看到输出之前，**禁止**声称"通过"、"完成"、"没问题"。

**执行要求**：
- 所有任务卡片标记 `✅` 后，调用 `self-check-trinity` 依次执行 lint → typecheck → test，任一失败则修复后重试（最高 3 次）。
- 验证命令必须**在当前消息中运行**，证据必须新鲜；禁止信任缓存结果；禁止只跑 lint 就声称"质量检查通过"。
- 验证未通过 → 回到阶段 3 走调试流程；全部通过 → 进入阶段 5。

---

### 阶段 5：代码审查与收尾

**使用工具**：命令行/终端、技能调用、子代理调度、向用户提问

> **顺序关键说明**：阶段 5 处于文档归档前，此时 `docs/plan/<功能名>-plan.md` 与 `docs/tracking/<功能名>.md` 仍处于标准路径下，保证 CR 子代理读取文件不会引发 FileNotFound 错误。
>
> **优先**：
> - 代码审查 → 通过技能调用 `requesting-code-review`
> - 收尾 → 通过技能调用 `finishing-a-development-branch`
>
> **降级**：读取 `references/fallback-verify-finish.md` 执行内联审查与收尾。

**核心流程**：
- 审查范围用 `git merge-base` 确定 `BASE_SHA..HEAD_SHA`；审查问题按 Critical（立即修复并重走 3→4→5）/ Important（修复后继续）/ Minor（记录）分级处理。
- 收尾前进行测试检测：若 `git diff BASE_SHA..HEAD_SHA` 自阶段 4 后无任何变动，跳过重复全量测试并沿用阶段 4 凭证；如有变动则重新运行全量测试确认通过。
- 收尾四选项：`1 本地合并` / `2 推送并创建 PR` / `3 保持分支` / `4 放弃`；选项 4 必须获得用户输入 `discard` 确认后才可执行。

---

### 阶段 6：使用指南产出与文档合规整理

**使用工具**：写入文件、技能调用（调用文档布局整理技能）

1. 基于 plan 文档和实际实现，生成使用指南至 `docs/guide/<功能名>-guide.md`：
   - 功能概述、使用方式、API 接口说明（如有）、注意事项

2. 通过技能调用 `docs-layout-quadrant` 完成文档合规整理与收尾归档，覆盖：
   - 命名规范校验（无日期前缀）
   - 五维归类（含弹性降级决策树判定）
   - **象限生命周期管理**：将已交付的 `docs/plan/<功能名>-plan.md` 追加 `-archived` 后缀移入 `docs/.archive/plan/`（若含内联 Checklist 则整体归档）；将已验收的 `docs/tracking/<功能名>.md` 按价值决策处理（含设计讨论/决策记录 → 追加 `-archived` 移入 `docs/.archive/tracking/`；纯勾选清单 → 经用户确认后删除）
   - `docs/INDEX.md` 索引更新
   - 相对路径校验与残留验证
   - **规则文件注入**：向项目根目录 `AGENTS.md` / `CLAUDE.md` 注入 docs 目录结构说明标记块（已存在则更新块内内容，文件不存在则跳过）

---

## ⛔ 行为护栏

- **🚫 禁止日期前缀**：所有产出文档必须使用领域主体命名（`<功能名>-design.md`），**禁止** `YYYY-MM-DD-<功能名>-design.md` 格式。文档是知识图谱，不是流水账。
- **阶段不可跳跃**：默认必须按 0→1→2→3→4→5→6 顺序执行。必须先审查收尾(阶段 5)后再归档文档(阶段 6)。若用户要求跳过某阶段，需明确告知风险并获得确认后方可跳过。
- **防止多版本命名冲突**：同名文档已存在时，默认原地更新（活文档）；需归档旧版时，追加 `-archived` 并移入 `docs/.archive/{象限}/`，**禁止**自动追加 `-v2`/`-v3` 后缀。
- **Git 写操作需授权**：`git add`/`git commit`/`git push`/`git reset`/`git checkout --` 等写操作必须先获得用户授权（阶段 2 计划送审时可批量预授权阶段 3 本地原子提交）；未授权时禁止自动操作。
- **证据优先与强校验**：阶段 4 的验证必须有新鲜命令输出为凭，阶段 5 的审查必须有 diff 结果为凭，禁止空口断言或信任过时缓存。

## 📝 模板与范例

### <Bad>

- 先归档 tracking/plan 文档再执行代码审查，导致 CR 阶段报 FileNotFound 文件不存在错误。
- 编码前不先写失败的测试（也不判定是否适用 VDD），直接编写实现代码。
- 并行派发多个子代理时让子代理直接编辑 `docs/tracking/<功能名>.md`，导致后完成者覆盖先完成者的进度。
- 验证阶段只口头声称"已通过"，未在当前消息中提供真实的测试与代码质量校验命令输出。
- 使用日期前缀命名文档（`2026-07-15-上传功能-design.md`），或者同名文档冲突时自动追加 `-v2` 后缀制造多版本歧义。

### <Good>

```markdown
# 领域主体命名 + 五维（活文档）
docs/reqs/file-upload-req.md           ← 需求文档，验收后冻结保留
docs/specs/file-upload-design.md      ← 原地更新，随代码演进的长周期文档
docs/plan/file-upload-plan.md         ← 阶段 5 审查通过后在阶段 6 追加 -archived 归档至 docs/.archive/plan/
docs/tracking/file-upload.md          ← 阶段 5 审查通过后在阶段 6 归档至 docs/.archive/tracking/ 或删除
docs/guide/file-upload-guide.md       ← 面向开发者的 How-to
```

```markdown
# 标准追踪文件格式（主编排代理统一维护）
# 状态总览
创建日期：2026-07-15 | 当前阶段：阶段 3 | 模式：TDD/VDD

# 任务卡片
- ✅ [组件/模块] 创建 API 路由（commits abc123..def456, 测试 5/5, review clean）
- ✅ [组件/模块] 文件存储与校验逻辑（commits def456..ghi789, 测试 8/8）
- 🔄 [组件/模块] 上传组件 UI（VDD 模式：RED 静态校验完成，等待 GREEN 实现）
- ⏳ [组件/模块] 组件联调
```

---

## 📜 版本变更历史 (Changelog)

- **v5.2.0** (2026-07-24):
  - **弹性降级对齐**：与 docs-layout-quadrant v2.5.1 对齐。阶段 0 目录准备改为"核心三支撑（specs/ + plan/ + guide/）+ tracking 强约束 + reqs 按决策树"；阶段 1 增加 reqs/ 内联降级说明；阶段 2 明确 tracking/ 在 `/xy-feat` 下强制创建；阶段 6 增加内联 Checklist 整体归档处理。
  - 文档命名表标注 `tracking/` 为 `[xy-feat强制必选]`。
- **v5.1.0** (2026-07-23):
  - **归档方案统一**：specs/plan/tracking 旧版统一追加 `-archived` 后缀并移入 `docs/.archive/{象限}/`，废弃 `[DONE]` 标记与象限内 `.archive/` 子目录方案，与 docs-layout-quadrant v2.3.0 对齐。
  - **规则文件注入并入阶段 6**：docs-layout-quadrant 的规则文件注入（向 AGENTS.md/CLAUDE.md 注入 docs 结构说明标记块）明确为阶段 6 职责，xy-feat 不新增独立阶段。
- **v5.0.0** (2026-07-23):
  - **打破逻辑死锁**：交换阶段 5 与阶段 6 顺序，调整为`验证 → 审查收尾(阶段 5) → 指南产出与归档(阶段 6)`，彻底解决 CR 子代理找不到已归档文件的死锁问题。
  - **TDD 逃生通道**：增加 VDD (Validation-Driven Development) 模式判定，解决纯 UI/CSS、配置文件和无单测场景下的流程死锁。
  - **测试去重**：阶段 5 收尾增加 Git Diff 检测，自阶段 4 后源码无变更时跳过重复全量测试。
  - **并发写安全**：约束并行子代理禁止写全局 tracking.md，统一由主编排代理集中更新。
  - **交互体验升级**：阶段 1 引入一站式 Draft 呈现，阶段 2 引入 Git 原子提交批量预授权。
  - **护栏精简**：从 8 条精简为 5 条硬边界约束。
- **v4.3.0** (2026-07-22): 规范 YAML Frontmatter 与 superpowers 依赖模式，抽取 references 降级流程。
