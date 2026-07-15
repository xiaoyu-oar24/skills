---
name: xy-feat
description: "端到端功能开发工作流（brainstorming → writing-plans → TDD → verification → review），串联 superpowers 技能链，自包含、跨项目复用。TRIGGER when: 用户显式输入 '/xy-feat' 时激活。SKIP: 其他任何场景均不触发——包括说'开发新功能'、'写xxx特性'等自然语言描述，以及物理路径匹配等。"
version: "3.0.0"
author: xiaoyu
---

# xy-feat 工作流

> **定位**：将 superpowers 技能链——**需求澄清 → 方案设计 → 规划分解 → TDD 执行 → 验证 → 审查收尾**——串成一条端到端流水线（编排自包含，各阶段委托给对应 superpowers 技能执行）。
> **核心思想**：每个阶段不再自行发明流程，而是委托给对应的 superpowers Skill，保持各环节职责清晰、可替换。
> **跨项目复用**：本文件与 superpowers 解耦；复制到任意项目后，只要 superpowers 可用即可复用。

## 🎯 触发条件
- 仅在用户显式输入 `/xy-feat [需求]` 时触发。其他自然语言描述（如"开发一个新功能"、"写一个 xxx 特性"）**不触发**本工作流。
- 由本工作流统一调度，**不允许跳过阶段**（除非用户明确授权）。

## ⚙️ 依赖环境
- **Bash 环境**：支持 `mkdir`、`date` 等基础命令。
- **Superpowers Skills 依赖**：以下技能为本工作流各阶段的执行载体，均为按需调用（缺失时降级而非阻断，详见各阶段说明）：

| 技能 | 阶段 | 必需性 |
| :--- | :--- | :--- |
| `brainstorming` | 阶段 1 | **强制**——缺失时必须询问用户 |
| `writing-plans` | 阶段 2 | 推荐——缺失时降级为手工规划 |
| `docs-layout-quadrant` | 阶段 5 | 推荐——用以强约束命名规范、内部相对路径引用和 docs/ 目录根部的清洁 |
| `test-driven-development` | 阶段 3 | 推荐——缺失时降级为先写代码后补测试 |
| `dispatching-parallel-agents` | 阶段 3 | 可选——仅独立任务并行时使用 |
| `subagent-driven-development` | 阶段 3 | 可选——复杂多步任务拆分时使用 |
| `systematic-debugging` | 阶段 3 | 推荐——遇到 bug/测试失败时必须调用 |
| `self-check-trinity` | 阶段 4 | **强制**——Lint → Typecheck → Test 三道必须全部通过 |
| `verification-before-completion` | 阶段 4 | **强制**——必须有命令输出为证，禁止空口断言"已通过" |
| `requesting-code-review` | 阶段 6 | 推荐——合并/交付前使用 |
| `finishing-a-development-branch` | 阶段 6 | 推荐——决定合并/PR/清理策略 |
| `using-git-worktrees` | 阶段 0 | 可选——需要工作区隔离时使用 |

- **工具授权**：用户调用本工作流即视为明确授权进行文件读写、目录创建及所需的子技能调用。

## 📐 文档命名规范（全局适用）

> **核心原则**：文档是知识图谱，不是 Git Commit 历史。文件名仅描述模块/领域，**禁止任何日期/时间戳前缀**。

所有产出文档统一使用 **领域主体命名法**：

| 象限 | 路径 | 命名模式 | 示例 |
| :--- | :--- | :--- | :--- |
| 设计规范 | `docs/specs/` | `<功能名>-design.md` | `gateway-proxy-design.md` |
| 执行计划 | `docs/plan/` | `<功能名>-plan.md` | `gateway-proxy-plan.md` |
| 进度跟踪 | `docs/tracking/` | `<功能名>.md` | `gateway-proxy.md` |
| 使用指南 | `docs/guide/` | `<功能名>-guide.md` | `gateway-proxy-guide.md` |

- `<功能名>` 使用英文 kebab-case 或简明中文，描述功能模块本身，**不含日期**。
- 同一功能的四象限文档共享相同的 `<功能名>` 前缀，形成关联。

## 📖 核心工作流

> **总览**：`阶段 0 初始化` → `阶段 1 方案设计` → `阶段 2 规划分解` → `阶段 3 TDD 执行` → `阶段 4 验证完成` → `阶段 5 产出指南` → `阶段 6 审查收尾`

---

### 阶段 0：需求获取与防冲突初始化
**使用工具**：`Bash`、`AskUserQuestion`
**可选技能**：`using-git-worktrees`

1. **需求确认**：若用户输入为空或不清晰，使用 `AskUserQuestion` 追问："你要做什么功能？"
2. **功能名确定**：从需求中提炼出 `<功能名>`（kebab-case 或简明中文），用于全流程文档命名。若无法自动确定，询问用户。
3. **目录准备**：确保 `docs/specs`、`docs/plan`、`docs/tracking`、`docs/guide` 四个目录存在（`mkdir -p`）。
4. **冲突检测**：检查是否存在同名的领域主体命名文档，如有则自动追加 `-v2`/`-v3` 等后缀，**严禁覆盖**。
5. *(可选)* 若用户提到"隔离开发"或当前工作区已有未提交变更，调用 `using-git-worktrees` Skill 创建干净的工作区。

---

### 阶段 1：方案设计（brainstorming）
**强制技能**：`brainstorming`

1. 通过 `Skill` 工具调用 `brainstorming`，将阶段 0 获取的需求完整传入。
2. **异常处理**：
   - 若 `brainstorming` 不存在或加载失败，必须使用 `AskUserQuestion` 询问："brainstorming 技能不可用，是否跳过设计阶段直接进入规划？"
   - 用户选"否" → **立即终止工作流**；选"是" → 跳过阶段 1，进入阶段 2。
3. `brainstorming` 输出后，将其内容写入 `docs/specs/<功能名>-design.md`。

---

### 阶段 2：规划分解（writing-plans）
**推荐技能**：`writing-plans`

1. 使用 `Read` 工具重新读取阶段 1 产出的 `docs/specs/<功能名>-design.md`，刷新上下文。
2. 通过 `Skill` 工具调用 `writing-plans`，将设计方案分解为可执行的任务卡片列表。
3. **降级策略**：若 `writing-plans` 不可用，则手工将 spec 分解为任务卡片，按"后端 → 前端 → 集成"的顺序排列。
4. 将规划结果写入 `docs/plan/<功能名>-plan.md`。
5. 同步创建进度跟踪文件 `docs/tracking/<功能名>.md`，初始化所有任务卡片状态为 `⏳`。

> **模板**：
> ```markdown
> # 状态总览
> 创建日期：YYYY-MM-DD | 当前阶段：阶段 2
> 
> # 任务卡片
> - ⏳ [后端] 创建 API 路由 `POST /api/upload`
> - ⏳ [后端] 文件存储与校验逻辑
> - ⏳ [前端] 上传组件 UI
> - ⏳ [前端] 与 API 联调
> - ⏳ [集成] 端到端流程验证
> ```

---

### 阶段 3：TDD 执行
**推荐技能**：`test-driven-development`、`dispatching-parallel-agents` / `subagent-driven-development`、`systematic-debugging`

> **执行原则**：先写测试，再写实现；独立任务可并行调度；遇到 Bug 必须走调试流程。

1. **调用 TDD 技能**：在开始编写任何实现代码前，先通过 `Skill` 工具调用 `test-driven-development`，按"红灯（写测试）→ 绿灯（最小实现）→ 重构"节奏推进。
   - 若 TDD 技能不可用，则降级为：先手工编写测试，通过 `Bash` 运行验证其失败（红灯），再编写实现代码。
2. **任务调度**：
   - 若任务卡片中存在 2+ 个无共享状态的独立任务，调用 `dispatching-parallel-agents` 并行推进。
   - 若某个任务卡片内部步骤多、复杂度高，调用 `subagent-driven-development` 委托子代理独立完成。
   - 若不满足上述条件，则按顺序逐个执行任务卡片。
3. **实时追踪**：每完成一个任务卡片，立即使用 `Edit` 工具将 `docs/tracking/<功能名>.md` 中对应条目标记为 `✅`。
4. **Bug 处理**：执行过程中若遇到任何测试失败、编译错误或非预期行为，**必须先调用 `systematic-debugging`**，走"假设 → 插桩 → 复现 → 分析 → 修复 → 验证"流程，禁止跳过调试直接猜测式修复。最高重试 3 次；3 次仍未修复则暂停并向用户汇报。
5. **跨卡片内存对齐**：在开始实现下一个任务卡片前，使用 `Read` 重新查看 plan 文档的设计上下文，严防"文档写一套、代码写另一套"。

---

### 阶段 4：验证完成（self-check-trinity + verification-before-completion）
**强制技能**：`self-check-trinity`、`verification-before-completion`

> 两技能互补：`self-check-trinity` 定义"查什么"（lint → typecheck → test），`verification-before-completion` 保证"怎么查"（证据优先，拒绝空口断言）。

1. 所有任务卡片标记为 `✅` 后，先通过 `Skill` 工具调用 `self-check-trinity`，该技能会：
   - 自动识别项目技术栈，确定对应的 lint / typecheck / test 命令
   - 依次执行三道检查，任一步骤失败则必须修复后重试（最高 3 次）
2. 三道检查全部通过后，调用 `verification-before-completion`，将 `self-check-trinity` 的命令输出作为证据提交。
3. 验证未通过 → 回到阶段 3，调用 `systematic-debugging` 修复后重新走阶段 4。
4. 验证全部通过 → 进入阶段 5。

---

### 阶段 5：使用指南产出与文档合规整理
**使用工具**：`Write`
**推荐技能**：`docs-layout-quadrant`

1. 基于 plan 文档和实际实现，通过 `Write` 工具生成最终使用指南至 `docs/guide/<功能名>-guide.md`。
2. 指南应包含：功能概述、使用方式、API 接口说明（如有）、注意事项。
3. **调用文档合规整理**：调用并执行 `docs-layout-quadrant` 技能，完成以下工作：
   - **命名校验**：扫描本次开发在 `docs/` 目录下产生的所有文档，确认全部使用领域主体命名（无日期前缀）
   - **路径修复**：强制校验并修正所有绝对路径链接为相对路径
   - **索引更新**：更新 `docs/INDEX.md`，将本次产出的 specs/plan/guide 文档登记到知识导航网中
   - **过期清理**：将已完成的 `plan` 文档标记为 `[DONE]`，将已验收的 `tracking` 文档归档到 `.archive/` 或删除

---

### 阶段 6：审查与收尾
**推荐技能**：`requesting-code-review`、`finishing-a-development-branch`

1. 通过 `Skill` 工具调用 `requesting-code-review`，对本次实现的完整 diff 进行自审，输出结构化的代码审查反馈。
2. 若审查发现需要修改的问题，回到阶段 3 修复，修复完成后重新走阶段 4 → 5 → 6。
3. 审查通过后，调用 `finishing-a-development-branch`，向用户呈现结构化的收尾选项：合并、PR、或清理。
4. **Git 操作必须由用户明确授权**，禁止自动提交或推送。

---

## ⛔ 行为限制与护栏

- **🚫 禁止日期前缀**：所有产出文档必须使用领域主体命名（`<功能名>-design.md`），**禁止** `YYYY-MM-DD-<功能名>-design.md` 格式。文档是知识图谱，不是流水账。
- **阶段不可跳跃**：默认必须按 0→1→2→3→4→5→6 顺序执行。若用户要求跳过某阶段，需明确告知风险并获得确认后方可跳过。
- **覆盖保护**：阶段 0 的冲突检测机制禁止对已有文档的强制覆盖，冲突时自动追加版本后缀。
- **Git 严格禁用**：默认情况下严禁任何 `git add`/`git commit`/`git push`/`git reset`/`git checkout --` 等命令。即使用户授权提交，也只能提交一次且需用户最终确认。
- **内存对齐**：每进入新阶段或切换任务卡片时，必须先 `Read` 上游产出文档，严防上下文漂移。
- **失败兜底**：任何子技能不可用时，本工作流必须有明确的降级策略（详见各阶段），不允许因缺失依赖而静默跳过关键环节。
- **证据优先**：阶段 4 的验证必须有命令输出为凭，阶段 6 的审查必须有 diff 分析为凭，禁止空口断言"没问题"。
- **强制索引同步**：阶段 5 完成后，必须确保 `docs/INDEX.md` 已更新，保持导航网最新。

## 📝 模板与范例

### <Bad> 不规范的操作行为
- 跳过 brainstorming 直接写代码，导致需求理解偏差。
- 编码时不查看 plan 文件，凭记忆实现。
- 测试失败后不调用 systematic-debugging，直接猜测式修改。
- 验证阶段只口头声称"已通过"，未提供 lint/test 命令输出。
- 未经用户允许执行 `git commit` 或 `git push`。
- **使用日期前缀命名文档**（`2026-07-15-上传功能-design.md`），让文档变成一次性产物。

### <Good> 规范的文档产出
```markdown
# 领域主体命名 + 四象限 + 知识索引

docs/specs/file-upload-design.md      ← 设计规范（长周期）
docs/plan/file-upload-plan.md         ← 执行计划（交付后标记 [DONE]）
docs/tracking/file-upload.md          ← 进度跟踪（验收后归档/删除）
docs/guide/file-upload-guide.md       ← 使用指南（长周期）

# docs/INDEX.md 同步更新
## 文件上传
| 文档 | 象限 | 文件 |
| :--- | :--- | :--- |
| 设计规范 | specs | [file-upload-design.md](specs/file-upload-design.md) |
| 使用指南 | guide | [file-upload-guide.md](guide/file-upload-guide.md) |
```

### <Good> 标准追踪文件格式
```markdown
# 状态总览
创建日期：2026-07-14 | 当前阶段：阶段 3 | 模式：TDD

# 任务卡片
- ✅ [后端] 创建 API 路由 `POST /api/upload`（systematic-debugging 修复 NPE 后通过）
- ✅ [后端] 文件存储与校验逻辑（并行代理 A 完成）
- 🔄 [前端] 上传组件 UI（TDD 红灯已写，等待绿灯实现）
- ⏳ [前端] 与 API 联调
- ⏳ [集成] 端到端流程验证
```
