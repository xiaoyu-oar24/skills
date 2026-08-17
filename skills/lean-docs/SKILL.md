---
name: "lean-docs"
description: "文档体系瘦身与检索治理技能：归档过期的 plan/tracking、精简活文档冗余流水账、重构紧凑版 INDEX.md 索引，并在各 AI 工具的忽略配置中阻断检索噪音，极致降低 Token 消耗。TRIGGER when: 用户输入 /lean-docs、提及精简文档/文档瘦身/降低文档 Token 消耗/收敛已完成的计划与追踪堆积。SKIP: 新建功能或进行中的需求开发（用 xy-feat）；仅做五维结构归类、命名校验、规则文件注入（用 docs-layout-quadrant）。"
version: "1.2.0"
author: "xiaoyu"
---

# 文档瘦身与检索治理 (lean-docs)

> **核心思想**：文档是随代码演进的知识图谱，不是流水账日志。AI 读取文档只需要**“最新的事实与结论”**，不需要**“历史怎么讨论的过程”**。
> **核心原则**：高信噪比（保留结论，剥离过程）、生命周期对齐（过期归档，活文档精炼）、安全无损（精简活文档需确认）、检索阻断（在各 AI 工具的忽略配置中物理隔离噪音）。

## 🎯 触发条件

- **TRIGGER when**:
  - 用户输入 `/lean-docs`
  - 用户提及“精简文档”、“文档瘦身”、“降低文档 Token 消耗”、“收敛已完成的计划与追踪堆积”
  - 文档库经过多轮迭代后出现大量已完成的 plan/tracking 堆积或活文档过长
- **SKIP**（与相邻技能硬性分流）:
  - 正在进行中的业务功能开发任务（应使用 `xy-feat`）
  - 仅做五维结构归类、命名规范校验、规则文件注入（应使用 `docs-layout-quadrant`）
  - 对代码源码或配置文件的直接修改

## ⚙️ 依赖与先决条件

- 项目存在 `aidocs/` 目录且遵循五维布局规范（若仍为旧版 `docs/`，先调用 `docs-layout-quadrant` 执行阶段 0 迁移）。
- **版本对齐**：依赖 `docs-layout-quadrant ≥ v3.0.0`（五维 `aidocs/` 布局、统一归档目录 `aidocs/.archive/{象限}/`、弹性降级决策树）。`docs-layout-quadrant` 升级主版本时，需同步复核本技能对生命周期与归档规则的引用。
- Git 版本控制正常工作（运行 `git --version` 确认，用于安全追踪与回滚）。
- 运行环境假设为类 Unix 终端（支持 `git mv`、文件搜索工具）。

## 📖 标准工作流

### 阶段 1：过期生命周期归档与清理（委托执行，最大收益点）

> **职责归属**：生命周期归档（plan/tracking/reqs 的过期判定、`-archived` 后缀、移入 `aidocs/.archive/{象限}/`）由 `docs-layout-quadrant` 阶段 3 统一定义与执行，本技能**不重复实现**，避免两处规则漂移。

**使用工具**：技能调用（调用 `docs-layout-quadrant`）、命令行/终端

1. 通过技能调用 `docs-layout-quadrant`，执行其**阶段 3（象限生命周期管理）**：
   - 归档已交付的 `aidocs/plan/` → `aidocs/.archive/plan/`
   - 处理已验收的 `aidocs/tracking/`（含决策记录→归档；纯勾选清单→经确认后删除）
   - 归档废弃的 `aidocs/reqs/` → `aidocs/.archive/reqs/`
2. 保护活跃任务：正在进行中或未交付的 plan/tracking 必须保持原位，严禁误归档。

### 阶段 2：检索阻断与噪音隔离（多 AI 工具通用）

**使用工具**：读取文件、写入文件/编辑文件

目标：将高噪音目录写入当前 AI 工具的忽略配置，阻断全局扫描消耗。**不局限于某一种工具**——按实际使用的 AI 助手选择对应机制：

| AI 工具 | 忽略/检索阻断配置 | 写法说明 |
| :--- | :--- | :--- |
| Claude Code | `.claudeignore` | 项目根目录文件，每行一条 glob/路径（gitignore 风格） |
| Cursor | `.cursorignore` | 项目根目录文件，每行一条 glob/路径（gitignore 风格） |
| opencode | `opencode.json` 的 `ignore` 字段 | 项目/全局配置中声明字符串数组（glob 模式） |
| Codex CLI | `.codexignore` | 项目根目录文件，每行一条 glob/路径 |

**排除清单（各工具一致）**：
- 历史归档目录：`aidocs/.archive/`
- 大体积外部依赖与文档附件（如 `aidocs/vendor/` 等）
- 常见的构建与缓存产物（`dist/`, `node_modules/`, `.turbo/`, `build/` 等）

执行步骤：
1. 检测当前项目实际使用的 AI 工具：根据已存在的配置文件（`.claudeignore` / `.cursorignore` / `opencode.json` / `.codexignore`）或规则文件（`CLAUDE.md` / `AGENTS.md`）判定。
2. 创建或更新对应的忽略配置，写入上述排除清单；若多个工具的配置同时存在，全部补齐以保证跨工具一致。
3. 若当前工具尚无等价忽略机制，跳过该项并在交付总结中说明。

### 阶段 3：长期活文档精炼 (specs/ & guide/)

**使用工具**：读取文件、编辑文件

针对 `aidocs/specs/`（技术设计规范）与 `aidocs/guide/`（开发者指南）执行精炼：

1. **识别冗余信息**：
   - 多轮讨论与调研历史、已废弃的旧版本方案对比；
   - 详尽的旧代码调用点流水账与过程日志；
   - 已解决并闭环的临时排错记录。
2. **提炼核心知识**：
   - 仅保留当前有效架构设计、接口契约矩阵与核心领域规则；
   - 将历史闭环问题提炼为简练的“决策-落地代码”矩阵表格。
3. **安全确认执行**：
   - **严禁静默覆盖**！先向用户展示精炼前后的结构差异与删减要点；
   - 获得用户确认后，再执行修改写入。

### 阶段 4：紧凑型全局知识索引重构 (aidocs/INDEX.md)

**使用工具**：读取文件、写入文件

对齐 `docs-layout-quadrant` 阶段 4 的索引规范，对 `aidocs/INDEX.md` 做一次紧凑化重构：

1. **按业务领域聚合**：
   - 每个业务模块集中列出其当前活跃的核心文档（覆盖 `reqs/` 需求、`specs/` 设计、`guide/` 指南）；
   - **严禁遗漏 `reqs/`**，需求文档作为业务事实必须在索引中保留。
2. **归档收敛概括**：
   - `aidocs/.archive/` 下的文件**禁止展开长列表**；
   - 仅保留分类概括或极少数仍具重大参考价值的历史架构链接（归档小节总行数控制在 5 行内）。
3. **规模控制**：
   - 确保 `aidocs/INDEX.md` 总行数控制在 40~60 行以内（单次读取 Token 消耗 < 1k）。

### 阶段 5：交付总结与后续建议

- 向用户输出治理成果报告：
  - 归档与清理的 plan/tracking 数量；
  - 精炼的活文档列表与预估行数/Token 压缩比例；
  - 忽略配置与 `INDEX.md` 的更新状态。
- 建议用户在长会话后使用 `/compact`，或在开启新功能前使用 `/clear` 刷新上下文。

## ⛔ 行为护栏

- **🚫 禁止抹杀 `reqs/`**：需求文档（PRD）是系统业务规则的核心事实源，重构 `INDEX.md` 时必须保留活跃的 `reqs/` 链接，不得仅保留 specs/guide。
- **🚫 禁止静默精简活文档**：对 `specs/` 和 `guide/` 的内容精炼必须遵循“提案预览 -> 用户确认 -> 落地修改”，严禁未经确认直接删除历史技术决策细节。
- **🚫 禁止提前归档未交付任务**：必须核实功能交付状态，正在开发中的 plan/tracking 绝对不可归档。
- **归档规则单一来源**：生命周期归档的判定与执行统一由 `docs-layout-quadrant` 阶段 3 负责，本技能仅调用、不重复实现归档逻辑。
- **必须使用 `git mv` 保留历史**：移动文件至归档目录时，必须使用 `git mv` 保持 Git 版本追溯历史完整。
- **索引与归档强一致**：归档文件后必须同步更新 `aidocs/INDEX.md`，禁止产生死链或遗留孤岛文档。

## 📝 模板与范例

### <Bad> 反模式

```markdown
# ❌ 打碎五维结构 + 遗漏需求文档 + 归档长列表展开

| 文档类型 | 链接 |
| :--- | :--- |
| 设计文档 | [specs/auth.md](specs/auth.md) |
| 操作指南 | [guide/auth.md](guide/auth.md) |
<!-- 错误：丢失了 reqs/ 需求，且打散了业务领域内聚 -->

## 归档列表
- [.archive/plan/2026-01-01-auth-plan.md](.archive/plan/2026-01-01-auth-plan.md)
- [.archive/plan/2026-01-05-pay-plan.md](.archive/plan/2026-01-05-pay-plan.md)
<!-- 错误：展开几十行已归档文件，白白浪费检索 Token -->
```

### <Good> 规范示例

```markdown
# ✅ 紧凑型五维业务索引 (aidocs/INDEX.md)

# 📚 项目知识索引

> 本文件是 aidocs/ 目录的全局导航入口，按业务领域组织。总览最新事实，历史过程已归档。

## 用户与认证
| 需求 (What) | 设计 (Why) | 指南 (How) |
| :--- | :--- | :--- |
| [用户需求](reqs/user-req.md) | [认证设计](specs/auth-design.md) | [接入指南](guide/auth-guide.md) |

## 支付中心
| 需求 (What) | 设计 (Why) | 指南 (How) |
| :--- | :--- | :--- |
| [支付需求](reqs/pay-req.md) | [网关架构](specs/pay-gateway.md) | [对接手册](guide/pay-guide.md) |

## 📦 历史归档
- 已归档 12 篇已交付 Plan 与历史追踪（统一存放在 `aidocs/.archive/`，不占用主索引）。
```

## 📜 版本变更历史 (Changelog)

- **v1.2.0** (2026-07-31):
  - **去 AI 工具特化**：检索阻断从仅 `.claudeignore` 泛化为多 AI 工具通用（Claude Code `.claudeignore` / Cursor `.cursorignore` / opencode `opencode.json#ignore` / Codex CLI `.codexignore`），并新增“按实际工具检测补齐”步骤。
  - **生命周期归档改为委托**：阶段 1 不再重复实现归档逻辑，改为调用 `docs-layout-quadrant` 阶段 3，消除“双权威”与规则漂移风险。
  - **触发硬性分流**：description 与触发条件明确 SKIP `docs-layout-quadrant`（结构归类/命名校验/规则注入）与 `xy-feat`（开发中）。
  - **版本对齐声明**：先决条件声明依赖 `docs-layout-quadrant ≥ v3.0.0`。
  - **补齐规范**：新增 `references/evals.md` 触发评测集与版本变更历史章节。
- **v1.1.0**: 初始版本（`.claudeignore` 单工具检索阻断 + 归档/精炼/紧凑索引）。
