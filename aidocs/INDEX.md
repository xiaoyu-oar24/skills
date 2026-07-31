# 📚 项目知识索引

> 本文件是 aidocs/ 目录的导航入口，按业务领域组织。新增文档时必须同步更新此索引。

## 审计报告
| 文档 | 象限 | 文件 |
| :--- | :--- | :--- |
| 技能体系深度综合审计报告（第3版·现役） | specs | [技能审计报告-深度综合版.md](specs/技能审计报告-深度综合版.md) |
| xy-feat 工作流深度逻辑与流程优化方案 | specs | [xy-feat工作流优化方案.md](specs/xy-feat工作流优化方案.md) |
| 技能审计历史记录汇总（第2版 + 通用标准版 + 修复 + 快照合并） | specs | [技能审计历史记录汇总.md](specs/技能审计历史记录汇总.md) |

## 架构设计
| 文档 | 象限 | 文件 |
| :--- | :--- | :--- |
| 项目结构重构设计 | specs | [project-refactor-design.md](specs/project-refactor-design.md) |

## Agent 行为规范
| 文档 | 象限 | 文件 |
| :--- | :--- | :--- |
| 任务边界自律守则 | specs | [任务边界自律守则.md](specs/任务边界自律守则.md) |

## 技能开发与编写指南
| 文档 | 象限 | 文件 |
| :--- | :--- | :--- |
| AI 规则与技能统一编写规范 | guide | [AI规则与技能统一编写规范.md](guide/AI规则与技能统一编写规范.md) |
| Superpowers 技能速查 | guide | [superpowers.md](guide/superpowers.md) |

## 📦 归档文档
> 仅列出仍有参考价值的归档文档（如被替代的旧版设计），纯 tracking 清单不列入。
| 文档 | 原象限 | 归档原因 | 文件 |
| :--- | :--- | :--- | :--- |
| 第 2 版技能审计报告 | specs | 已合并入《技能审计历史记录汇总》 | [.archive/specs/skills-audit-report-archived.md](.archive/specs/skills-audit-report-archived.md) |
| 通用标准版审计报告 | specs | 已合并入《技能审计历史记录汇总》 | [.archive/specs/技能审计报告-通用标准-archived.md](.archive/specs/技能审计报告-通用标准-archived.md) |
| 技能修复报告 | specs | 已合并入《技能审计历史记录汇总》 | [.archive/specs/技能修复报告-archived.md](.archive/specs/技能修复报告-archived.md) |
| 文档架构诊断与重构建议 | specs | 历史诊断，四象限布局已落地 | [.archive/specs/文档架构诊断与重构建议-archived.md](.archive/specs/文档架构诊断与重构建议-archived.md) |
| 项目结构重构执行计划 | plan | 项目已交付 | [.archive/plan/project-refactor-plan-archived.md](.archive/plan/project-refactor-plan-archived.md) |
| 全量快照审计报告 | tracking | 历史快照（数据已被第 3 版校准） | [.archive/tracking/skill-audit-report-archived.md](.archive/tracking/skill-audit-report-archived.md) |
| 项目重构追踪 | tracking | 任务已验收 | [.archive/tracking/project-refactor-archived.md](.archive/tracking/project-refactor-archived.md) |
| Git Diff 深度审查报告 | tracking | 历史审查（P0/P1/P2 已全部修复） | [.archive/tracking/git-diff-review-report-archived.md](.archive/tracking/git-diff-review-report-archived.md) |
| 技能 AI 兼容性审计报告 | tracking | 历史审计（P0/P1/P2 已全部修复） | [.archive/tracking/技能AI兼容性审计报告-archived.md](.archive/tracking/技能AI兼容性审计报告-archived.md) |
| 当前修改检查报告 | specs | 历史检查（引用状态已过期） | [.archive/specs/当前修改检查报告-archived.md](.archive/specs/当前修改检查报告-archived.md) |
| 五维文档布局设计方案 | specs | 已落地实现 | [.archive/specs/docs-layout-design-archived.md](.archive/specs/docs-layout-design-archived.md) |

---

## 象限说明

| 象限 | 路径 | 定位 | 生命周期 |
| :--- | :--- | :--- | :--- |
| 需求文档 | `reqs/` | 产品需求描述（PRD），记录 What——页面功能、交互流程、业务规则 [按需-决策树] | 中长周期 — 验收后冻结保留 |
| 设计规范 | `specs/` | 技术架构决策（ADR），记录 Why——架构选型、接口设计、数据模型 [核心必选] | 长周期 |
| 执行计划 | `plan/` | 实施计划与任务拆解 [核心必选] | 短周期 — 交付后追加 `-archived` 归档至 `.archive/plan/` |
| 进度跟踪 | `tracking/` | 任务 Checklist [xy-feat必选/单任务按需] | 超短周期 — 验收后归档至 `.archive/tracking/` 或删除 |
| 使用指南 | `guide/` | 面向开发者的 How-to 操作手册 [核心必选] | 长周期 |
| 归档 | `.archive/{象限}/` | 统一归档目录 | 历史版本、已完成计划、废弃文档 |
