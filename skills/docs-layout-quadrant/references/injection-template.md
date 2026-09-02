# 注入内容模板（阶段 7 专用）

> 本文件是 `docs-layout-quadrant` 阶段 7 的注入内容模板，**仅在执行阶段 7 注入时读取**，平时无需加载。注入时将模板原样写入目标规则文件，仅将 `start` 标记行中的版本号刷新为当前技能 version 的主.次版本号（如 `3.4.0` → `v3.4`）。

````markdown
<!-- docs-layout-quadrant:start (v3.4) -->
## 📁 aidocs/ 目录结构

本项目采用五维文档布局，由 `docs-layout-quadrant` 技能维护。AI 过程文档统一存放在 `aidocs/` 专属目录，与 `docs/`（文档站/产品文档等其他用途）隔离。AI 助手在读取/创建/修改文档时请遵循此结构。

| 目录 | 用途 | 存放规则 |
| :--- | :--- | :--- |
| `aidocs/reqs/` | 产品需求描述（PRD），记录 What——页面功能、交互流程、业务规则 [按需-决策树] | 中长周期，验收后冻结保留；废弃需求归档至 `aidocs/.archive/reqs/` |
| `aidocs/specs/` | 技术架构决策（ADR），记录 Why——架构选型理由、接口设计、数据模型 [核心资产] | 长周期，随代码演进持续更新 |
| `aidocs/plan/` | 实施计划与任务拆解 [L2 全量档落盘] | 短周期，交付后归档至 `aidocs/.archive/plan/` |
| `aidocs/tracking/` | 任务进度追踪清单 [L2 全量档落盘/多Agent强制] | 超短周期，验收后归档或删除 |
| `aidocs/guide/` | 面向开发者的 How-to 操作手册 [核心资产/按需] | 长周期，随功能迭代更新 |
| `aidocs/.archive/` | 统一归档目录 | 历史版本、已完成计划、废弃文档 |
| `aidocs/INDEX.md` | 知识导航入口 | 新增/删除/重命名文档后必须同步更新 |

### 文档产出分级（Doc-Tier Gate）
- **L0 轻量（零文档）**：Bug 修复、单文件小改动、配置/样式微调 → 不建任何象限文件，决策内联 commit/注释
- **L1 精简（核心资产）**：常规单模块功能 → `specs/`（必产）+ `guide/`（按需）；`plan/`、`tracking/` 对话内维护不落盘
- **L2 全量（五维全出）**：跨模块/多 Agent 并行/架构决策/新页面新交互/用户明确要求 → 五维全出
- `reqs/` 为按需维度：纯技术重构可跳过（需求背景内联至 specs/）；任何档位禁止产出空壳文档

### 命名规范
- **禁止日期前缀**：文件名使用领域主体命名（如 `网关代理转发.md`），不使用 `2026-07-15-xxx.md`
- **优先中文命名**：优先使用简明中文命名文件，便于人类快速识别；英文 kebab-case 作为备选
- 文档间引用使用相对路径（如 `../specs/xxx.md`），禁止绝对路径
- 归档文件统一追加 `-archived` 后缀，移入 `aidocs/.archive/{象限}/`

### Agent-Friendly 文档头
- 新建功能文档必须在 H1 前携带最小 YAML frontmatter：`status`、`tier`、`domain`、`updated_at`；有来源时加 `source_workflow`
- `status` 使用 `draft / approved / delivered / archived`；归档后必须显式更新为 `archived`
- `INDEX.md` 必须校验相对链接有效，且覆盖五个象限的全部非归档功能文档

### 接口文档子体系（可选）
- `aidocs/specs/接口文档/{domain}/` 存放原子接口契约与模块子索引，由 `doc-craftsman` 维护三级索引；全局 `INDEX.md` 只列「接口文档」域入口一行，原子文件允许元信息表头（以项目接口文档规范为准）

### 与 docs/ 的边界
- `docs/` 若存在，归项目自由使用（文档站、产品文档等），本布局不管理、不移动其中内容
- 旧版五维 `docs/` 由 `docs-layout-quadrant` 技能的自动迁移规则（阶段 0）一次性迁移至 `aidocs/`
<!-- docs-layout-quadrant:end -->
````
