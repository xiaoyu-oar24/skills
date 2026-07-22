---
name: xy-feat
description: "端到端功能开发工作流（需求澄清、方案设计、规划分解、TDD 执行、验证、审查收尾），优先调用 superpowers 技能链，未安装时降级为 references/ 内联流程。TRIGGER when: 用户显式输入 '/xy-feat' 时激活。SKIP: 其他任何场景均不触发——包括说'开发新功能'、'写xxx特性'等自然语言描述，以及物理路径匹配等。"
version: "4.3.0"
author: xiaoyu
---

# xy-feat 工作流

> **生命周期阶段**：稳定
> **定位**：将 **需求澄清 → 方案设计 → 规划分解 → TDD 执行 → 验证完成 → 审查收尾** 串成一条端到端流水线。
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
> **优先级说明**："强制"指该**阶段**不可跳过；执行载体不可用时必须走降级流程。

| 技能 | 阶段 | 优先级 | 降级策略 |
| :--- | :--- | :--- | :--- |
| `brainstorming` | 阶段 1 | **强制** | 读取 `references/fallback-init-design.md` 执行内联流程 |
| `writing-plans` | 阶段 2 | 推荐 | 读取 `references/fallback-plan.md` 执行内联流程 |
| `test-driven-development` | 阶段 3 | 推荐 | 读取 `references/fallback-tdd.md` 执行内联 TDD 循环 |
| `dispatching-parallel-agents` | 阶段 3 | 可选 | 读取 `references/fallback-tdd.md` 执行内联并行策略 |
| `subagent-driven-development` | 阶段 3 | 可选 | 读取 `references/fallback-tdd.md` 执行内联子代理策略 |
| `systematic-debugging` | 阶段 3 | 推荐 | 读取 `references/fallback-tdd.md` 执行内联四阶段调试法 |
| `verification-before-completion` | 阶段 4 | **强制** | 读取 `references/fallback-verify-finish.md` 执行内联校验 |
| `requesting-code-review` | 阶段 6 | 推荐 | 读取 `references/fallback-verify-finish.md` 执行内联审查 |
| `finishing-a-development-branch` | 阶段 6 | 推荐 | 读取 `references/fallback-verify-finish.md` 执行内联收尾 |
| `using-git-worktrees` | 阶段 0 | 可选 | 读取 `references/fallback-init-design.md` 执行内联隔离策略 |

### 本地技能依赖

> 以下技能为本仓库自维护，不受外部安装状态影响。

| 技能 | 阶段 | 角色 |
| :--- | :--- | :--- |
| `self-check-trinity` | 阶段 4 | Lint → Typecheck → Test |
| `docs-layout-quadrant` | 阶段 5 | 文档合规整理 |

- **工具授权**：用户调用本工作流即视为明确授权进行文件读写、目录创建。

## 📐 文档命名规范

> **核心原则**：文档是知识图谱，不是 Git Commit 历史。文件名仅描述模块/领域，**禁止任何日期/时间戳前缀**。
> 详细规范定义在 `skills/docs-layout-quadrant/SKILL.md`，此处列出本工作流直接使用的约定：

| 象限 | 路径 | 命名模式 | 示例 |
| :--- | :--- | :--- | :--- |
| 设计规范 | `docs/specs/` | `<功能名>-design.md` | `gateway-proxy-design.md` |
| 执行计划 | `docs/plan/` | `<功能名>-plan.md` | `gateway-proxy-plan.md` |
| 进度跟踪 | `docs/tracking/` | `<功能名>.md` | `gateway-proxy.md` |
| 使用指南 | `docs/guide/` | `<功能名>-guide.md` | `gateway-proxy-guide.md` |

- `<功能名>` 使用英文 kebab-case 或简明中文，描述功能模块本身，**不含日期**。
- 同一功能的四象限文档共享相同的 `<功能名>` 前缀。
- **活文档策略**：若同名文档已存在，优先**原地更新**（文档随代码演进）。若必须保留旧版，将旧版追加 `-archived` 后缀后移入对应 `.archive/` 子目录，保持主文件名不变。

## 📖 标准工作流

> **总览**：`阶段 0 初始化` → `阶段 1 方案设计` → `阶段 2 规划分解` → `阶段 3 TDD 执行` → `阶段 4 验证完成` → `阶段 5 产出指南` → `阶段 6 审查收尾`
> 本文件只定义各阶段的调度逻辑、产出物与出入口条件；各阶段的**完整操作细节**在降级时按需从 `references/` 加载。

---

### 阶段 0：需求获取与初始化

**使用工具**：命令行/终端、向用户提问

1. **需求确认**：若用户输入为空或不清晰，向用户提问："你要做什么功能？"
2. **功能名确定**：从需求中提炼出 `<功能名>`（kebab-case 或简明中文），用于全流程文档命名。提炼结果**必须经用户确认**（或用户明确授权自动定名）后方可使用。
3. **目录准备**：确保 `docs/specs`、`docs/plan`、`docs/tracking`、`docs/guide` 四个目录存在（`mkdir -p`）。
4. **冲突检测（活文档策略）**：

   ```
   若 docs/specs/<功能名>-design.md 已存在：
     默认行为：原地更新该文档（活文档理念，文档随代码演进）
     若用户明确要求保留旧版：
       1. 将旧文件重命名为 <功能名>-archived.md，移入 docs/specs/.archive/
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
> **降级**：读取 `references/fallback-init-design.md`，执行内联流程（需求澄清 → 方案对比 → 分节呈现审批 → 撰写设计文档 → 自审）。

**必须产出**：`docs/specs/<功能名>-design.md`（背景、方案选择理由、架构设计、接口定义、数据模型、测试策略）。

**阶段出口**：设计文档完成自审（无占位符、内部一致、范围聚焦、无歧义）并获用户批准。

---

### 阶段 2：规划分解

**使用工具**：读取/写入文件

> **优先**：通过技能调用 `writing-plans`。
> **降级**：读取 `references/fallback-plan.md`，执行内联流程（任务拆解原则、任务卡片四要素、规划文档模板、自审清单）。

**必须产出**：
- `docs/plan/<功能名>-plan.md`：任务拆解到 2-5 分钟粒度，每个任务卡片含 Files / Interfaces / Steps / Expected，**禁止 TBD/TODO**。
- `docs/tracking/<功能名>.md`：初始化全部任务卡片状态为 `⏳`（格式模板见 `references/fallback-plan.md`）。

**阶段出口**：规划自审通过（Spec 全覆盖、无占位符、跨任务类型/签名一致）并获用户确认。

---

### 阶段 3：TDD 执行

**使用工具**：命令行/终端、文件读取/编辑、子代理调度

> **优先**：执行过程中遇到下列场景时，优先通过技能调用对应 superpowers 技能：
> - 编写实现代码前 → `test-driven-development`
> - 遇到 Bug / 测试失败 → `systematic-debugging`
> - 多个独立任务可并行 → `dispatching-parallel-agents`
> - 单个复杂任务 → `subagent-driven-development`
>
> **降级**：读取 `references/fallback-tdd.md`（含单任务 TDD 循环、任务调度决策树、四阶段调试法）。

**Iron Law**：

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

先写测试 → 看它失败 → 写最小实现 → 看它通过 → 重构。在测试失败之前编写的任何实现代码，一律删除重来。

**执行要求（任何模式下都必须遵守）**：
- **实时追踪**：每完成一个任务卡片，立即将 `docs/tracking/<功能名>.md` 对应条目标记为 `✅`，并记录关键提交 hash。
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

### 阶段 5：使用指南产出与文档合规整理

**使用工具**：写入文件、技能调用（调用文档布局整理技能）

1. 基于 plan 文档和实际实现，生成使用指南至 `docs/guide/<功能名>-guide.md`：
   - 功能概述、使用方式、API 接口说明（如有）、注意事项

2. 通过技能调用 `docs-layout-quadrant` 完成文档合规整理，覆盖：命名规范校验（无日期前缀）、四象限归类、生命周期管理（plan 标记 `[DONE]`、已验收 tracking 归档）、`docs/INDEX.md` 索引更新、移动文件与修复链接、残留验证。具体步骤与判定标准以该技能定义为准。

---

### 阶段 6：审查与收尾

**使用工具**：命令行/终端、技能调用、子代理调度、向用户提问

> **优先**：
> - 代码审查 → 通过技能调用 `requesting-code-review`
> - 收尾 → 通过技能调用 `finishing-a-development-branch`
>
> **降级**：读取 `references/fallback-verify-finish.md` 执行内联审查与收尾。

**安全要点（任何模式下都必须遵守）**：
- 审查范围用 `git merge-base` 确定 `BASE_SHA..HEAD_SHA`；审查问题按 Critical（立即修复并重走 3→4→5→6）/ Important（修复后继续）/ Minor（记录）分级处理。
- 收尾前必须**重新运行测试**确认通过，未通过则停止。
- 收尾四选项：`1 本地合并` / `2 推送并创建 PR` / `3 保持分支` / `4 放弃`；选项 4 必须获得用户输入 `discard` 确认后才可执行。
- **Git 写操作必须由用户明确授权**，禁止自动提交或推送。

---

## ⛔ 行为护栏

- **🚫 禁止日期前缀**：所有产出文档必须使用领域主体命名（`<功能名>-design.md`），**禁止** `YYYY-MM-DD-<功能名>-design.md` 格式。文档是知识图谱，不是流水账。
- **阶段不可跳跃**：默认必须按 0→1→2→3→4→5→6 顺序执行。若用户要求跳过某阶段，需明确告知风险并获得确认后方可跳过。
- **防止多版本命名冲突**：同名文档已存在时，默认原地更新（活文档）；需归档旧版时，追加 `-archived` 并移入对应 `.archive/`，**禁止**自动追加 `-v2`/`-v3` 后缀。
- **Git 写操作需授权**：`git add`/`git commit`/`git push`/`git reset`/`git checkout --` 等写操作必须先获得用户明确授权（逐次授权，或在阶段 2 计划获批时对全流程提交节点整段授权）；未授权时禁止执行。
- **上下文对齐**：每进入新阶段或切换任务卡片时，必须先读取上游产出文档。
- **证据优先**：阶段 4 的验证必须有命令输出为凭，阶段 6 的审查必须有 diff 结果为凭，禁止空口断言。
- **调试纪律**：阶段 3 遇到 Bug 必须先走根因调查，禁止猜测式修复。3 次不同假设均失败则汇报用户。
- **测试铁律**：禁止在编写失败测试之前编写任何实现代码。违反此规则——删代码，重来。

## 📝 模板与范例

### <Bad>

- 编码前不先写失败的测试，直接编写实现代码。
- 验证阶段只口头声称"已通过"，未在当前消息中提供真实的测试与代码质量校验命令输出。
- 使用日期前缀命名文档（`2026-07-15-上传功能-design.md`），或者同名文档冲突时自动追加 `-v2` 后缀制造多版本歧义。
- 没有通过项目锁文件检测包管理器，直接硬编码 `npm test`，导致在 Python/Rust/Go 项目中运行失败。

### <Good>

```markdown
# 领域主体命名 + 四象限（活文档）
docs/specs/file-upload-design.md      ← 原地更新，随代码演进的长周期文档
docs/plan/file-upload-plan.md         ← 交付后标记 [DONE]
docs/tracking/file-upload.md          ← 验收后归档/删除
docs/guide/file-upload-guide.md       ← 面向开发者的 How-to
```

```markdown
# 标准追踪文件格式
# 状态总览
创建日期：2026-07-15 | 当前阶段：阶段 3 | 模式：TDD

# 任务卡片
- ✅ [组件/模块] 创建 API 路由（commits abc123..def456, 测试 5/5, review clean）
- ✅ [组件/模块] 文件存储与校验逻辑（commits def456..ghi789, 测试 8/8）
- 🔄 [组件/模块] 上传组件 UI（RED 已写，等待 GREEN 实现）
- ⏳ [组件/模块] 组件联调
```
