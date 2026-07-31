# 当前项目修改深度检查报告（合并终版）

> **初版**：2026-07-23 | **二次审计**：2026-07-23 | **修复验证**：2026-07-24  
> **检查范围**：当前工作区未提交修改 + 历史问题修复验证  
> **检查对象**：5 个已修改文件、注入块（AGENTS.md/CLAUDE.md）、目录结构、历史报告问题项

---

## 1. 检查概览

### 1.1 当前工作区变更

| 文件 | 状态 | 主要变更 |
| :--- | :--- | :--- |
| `README.md` | M | 四象限→五维布局，目录树增加 reqs/ |
| `skills/docs-layout-quadrant/SKILL.md` | M | v2.3.0→v2.5.1，新增 reqs/、弹性降级决策树、中文命名优先 |
| `skills/docs-layout-quadrant/references/evals.md` | M | 新增 3 条触发用例 |
| `skills/xy-feat/SKILL.md` | M | v5.1.0→v5.2.0，五维/弹性降级对齐，P0-2 已修复 |
| `skills/xy-feat/references/fallback-init-design.md` | M | P1-3 已修复：同步 reqs/ 内联降级要求 |

### 1.2 前序报告问题修复验证

基于 `docs/specs/当前修改检查报告.md`（2026-07-23）的 11 项问题，逐一验证修复状态：

| 序号 | 级别 | 问题简述 | 状态 |
| :--- | :--- | :--- | :--- |
| P0-1 | 严重 | 根目录散落业务 PRD | ✅ **已修复** — 文件已移除，根目录仅余 AGENTS.md/CLAUDE.md/README.md |
| P0-2 | 严重 | xy-feat 阶段 0 mkdir 遗漏 tracking | ✅ **已修复** — 命令已补全为 `mkdir -p docs/specs docs/plan docs/guide docs/tracking` |
| P0-3 | 严重 | CLAUDE.md/AGENTS.md 注入块 `xy-feet` 拼写 | ✅ **已修复** — 两处均修正为 `xy-feat` |
| P0-4 | 严重 | `docs/.archive/reqs/` 目录缺失 | ✅ **已修复** — 目录已创建 |
| P1-1 | 高 | 术语"五象限"混用且设计文档自我矛盾 | ❌ **未修复** — SKILL.md 中"五象限"出现 6 次，与设计文档批判矛盾 |
| P1-2 | 高 | Changelog 日期异常（v5.1.0 历史篡改） | ❌ **未修复** — v5.1.0 日期仍为 `2026-07-24`，对齐版本仍为 `v2.4.0` |
| P1-3 | 高 | fallback-init-design.md 未同步 reqs 降级 | ✅ **已修复** — 已追加 reqs/ 内联降级要求 |
| P1-4 | 高 | docs-layout-design.md 版本状态脱节 | ✅ **已修复** — 状态已更新为"✅ 已落地" |
| P2-1 | 中 | 中文命名规则与示例脱节 | ❌ **未修复** — 示例仍全用英文 kebab-case |
| P2-2 | 中 | xy-feat changelog 版本对齐号不精确 | ❌ **未修复** — 仍称"与 v2.5.0 对齐"，当前为 v2.5.1 |
| P2-3 | 中 | INDEX.md 缺空 reqs/ 域说明 | ✅ **已修复** — INDEX 已含 reqs/ 象限说明 |

**修复率：7/11（64%）。4 项 P0 全部修复，2 项 P1 已修复，4 项未修复（2 项 P1 + 2 项 P2）。**

---

## 2. 新发现问题

| 序号 | 级别 | 文件 / 位置 | 问题 | 建议 |
| :--- | :--- | :--- | :--- | :--- |
| N1 | 轻微 | `skills/xy-feat/SKILL.md` 阶段 0 | 目录准备称"四个核心目录"（specs/plan/guide/tracking），与 `docs-layout-quadrant` 的"核心三支撑"（specs+plan+guide）措辞不一致。changelog 中正确表述为"三核心必选 + tracking 强约束"，但代码正文简化为"四个核心" | 改为"三个核心必选目录（specs/plan/guide）+ tracking 强制创建"或使用 changelog 的一致表述 |
| N2 | 轻微 | `skills/xy-feat/references/fallback-init-design.md` L56 | 新增行存在"必须...必须"重复："必须在 `docs/specs/<功能名>-design.md` 的头部必须包含【…】" | 删除冗余的第二个"必须" |

---

## 3. xy-feet 拼写错误专项验证

> **问题**：前序报告 P0-3 标记的 `xy-feet` 拼写错误是否已修复？

### 3.1 验证命令

```bash
grep -n "xy-feat必选" AGENTS.md CLAUDE.md
```

### 3.2 验证结果

```
AGENTS.md:63:  [xy-feat必选/单任务按需]   ← ✅ 正确
CLAUDE.md:63:  [xy-feat必选/单任务按需]   ← ✅ 正确
```

全仓库扫描 `xy-feet` 仅出现在历史记忆文件（`.memsearch/`）和报告文档中，活跃文件已清零。

### 3.3 根因回顾

- 模板源头（`SKILL.md:188`）始终正确使用 `xy-feat`
- 注入产物（`AGENTS.md:63`、`CLAUDE.md:63`）曾错误使用 `xy-feet`
- 系阶段 7 注入时产生的笔误，非模板错误
- 已被 2 份文档（`当前修改检查报告.md`、`.memsearch/memory/2026-07-23.md`）记录为 P0，今已修复

---

## 4. 未修复问题详析

### P1-1：术语"五象限"仍在使用

`skills/docs-layout-quadrant/SKILL.md` 中"五象限"出现 6 处（description、阶段 2 标题、触发条件、护栏等）。`docs/specs/docs-layout-design.md` §一 明确批判该词"名不副实，缺乏工程严肃性"。技能文件与其自身设计规范存在自我矛盾。建议统一为"五维文档布局"。

### P1-2：Changelog 历史条目被修改

`skills/xy-feat/SKILL.md` v5.1.0 条目：
- 日期 `2026-07-23` → `2026-07-24`（篡改已发布版本日期）
- 对齐版本 `v2.3.0` → `v2.4.0`（v5.1.0 的"归档方案统一"对应的是 v2.3.0 的"统一归档目录"，非 v2.4.0 的"新增 reqs/"）

建议还原日期为 `2026-07-23`，对齐版本还原为 `v2.3.0`。

### P2-1：中文命名规则与示例脱节

规则明确"优先中文命名"，但所有 `<Good>` 示例（`gateway-proxy-design.md` 等）仍全用英文。建议在示例中至少给出 1 组中文文件名对比。

### P2-2：版本对齐表述不精确

`xy-feat` v5.2.0 changelog 称"与 docs-layout-quadrant v2.5.0 对齐"，实际版本为 v2.5.1。v2.5.1 仅增加中文命名优先级，不影响弹性降级，但表述可更精确（如"基于 v2.5.0，含 v2.5.1 中文命名"）。

---

## 5. 全局一致性验证

| 验证项 | 结果 |
| :--- | :--- |
| xy-feat ↔ docs-layout-quadrant tracking 强制创建一致 | ✅ |
| xy-feat ↔ docs-layout-quadrant reqs 决策树一致 | ✅ |
| xy-feat ↔ docs-layout-quadrant 内联归档处理一致 | ✅ |
| SKILL.md 注入模板 ↔ AGENTS.md/CLAUDE.md 注入块一致 | ✅（xy-feet 已修复） |
| SKILL.md version v2.5.1 ↔ 注入块标记 v2.5 一致 | ✅ |
| README 目录树顺序 ↔ 标准表格顺序 | ❌ guide/ 在 specs/ 前 |
| P0-2 修复后 xy-feat 措辞 ↔ docs-layout-quadrant 框架 | ⚠️ "四个核心" vs "核心三支撑" |

---

## 6. 结论

本次验证确认：前序报告的 **4 项 P0 严重问题已全部修复**（散落 PRD、mkdir 遗漏、xy-feet 拼写、.archive/reqs/ 缺失），**2 项 P1 已修复**（fallback 同步、设计文档状态），整体修复率 64%。

**无新引入的严重问题**。剩余 4 项未修复（P1-1 术语、P1-2 日期、P2-1 示例、P2-2 版本号）均为非阻塞性改进。新发现 2 个轻微措辞问题（N1、N2）。

建议优先级：先提交当前修复（7/11 已修），剩余 4 项可在后续迭代中处理。
