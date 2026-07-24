# AI Skills 集合库

自定义 AI 代理技能（Skills）的集合仓库，专为 **opencode**、**Claude Code** 等现代 AI 辅助编程工具设计。每个技能聚焦一个具体的工程场景，通过标准化的 `SKILL.md` 文件指导 AI 完成任务。

## 安装

### opencode

将仓库克隆到 opencode 的全局技能目录：

```bash
git clone <repo-url> ~/.config/opencode/skills/
```

重启编辑器或打开新会话即可生效（opencode 有缓存机制）。

### Claude Code

将仓库克隆到本地后，在目标项目的 `CLAUDE.md` 中添加钩子：

```markdown
需要时，主动通过 Read 工具读取 ~/.config/opencode/skills/skills/<skill-name>/SKILL.md，
并严格遵循该文件中的工作流和行为护栏执行任务。
```

## 技能列表

| 技能 | 描述 |
|------|------|
| `xy-feat` | 端到端功能开发工作流（需求澄清 → 方案设计 → 规划分解 → TDD/VDD 执行 → 验证 → 审查收尾 → 指南与合规整理），优先调用 superpowers，未安装时降级为内联流程 |
| `docs-layout-quadrant` | 文档五维布局（reqs / specs / plan / tracking / guide），规整 docs/ 目录，支持弹性降级（按需判定 reqs/tracking），含决策树、内联模板与规则文件注入 |
| `self-check-trinity` | 强制在交付代码前执行 lint → typecheck → test 三道质量检查 |
| `uniapp-wechat-scaffold` | 基于 UniApp + Vue 3 + TypeScript + Pinia + uv-ui 的微信小程序脚手架生成器 |
| `unified-api-response` | 强制所有 JSON API 返回统一的 `{ code, message, data }` 响应结构 |
| `api-name-drift-defense` | 防御第三方库版本升级导致的 API 重命名/移除问题 |
| `health-probe-discipline` | 服务探针端点（health/readiness/liveness）规范约束，支持 Node/Java/Python 多技术栈 |
| `z-paging-best-practices` | z-paging 分页组件的最佳实践指南，包含下拉刷新、上滑加载及布局防踩坑指南 |

## 目录结构

```
/
├── docs/                          # 规范指南、设计文档（五维：reqs/specs/plan/tracking/guide，含弹性降级）
│   ├── INDEX.md                   # 知识索引导航
│   ├── reqs/                      # 需求文档 [按需-决策树]（中长周期，验收后冻结保留）
│   ├── specs/                     # 设计规范 [核心必选]（长周期）
│   │   ├── 任务边界自律守则.md
│   │   ├── 技能审计报告-深度综合版.md
│   │   ├── xy-feat工作流优化方案.md
│   │   └── ...
│   ├── plan/                      # 执行计划 [核心必选]（短周期）
│   ├── tracking/                  # 进度跟踪 [xy-feat必选/单任务按需]（超短周期）
│   └── guide/                     # 使用指南 [核心必选]（长周期）
│       ├── AI规则与技能统一编写规范.md
│       ├── superpowers.md
│       └── ...
├── skills/                        # 技能容器目录
│   ├── xy-feat/                   # 功能开发工作流 (v5.2.0)
│   ├── uniapp-wechat-scaffold/    # UniApp 脚手架
│   ├── self-check-trinity/        # 三合一质量检查
│   ├── unified-api-response/      # 统一 API 响应
│   ├── api-name-drift-defense/    # API 漂移防御
│   ├── docs-layout-quadrant/      # 文档五维布局 (v2.5.1)
│   ├── health-probe-discipline/   # 探针规范
│   └── z-paging-best-practices/   # z-paging 分页最佳实践
├── README.md
├── CLAUDE.md
└── AGENTS.md
```

## 开发

所有技能严格遵循 `docs/guide/AI规则与技能统一编写规范.md`，每个 `SKILL.md` 必须包含：

- **🎯 触发条件** — 何时激活
- **⚙️ 依赖环境** — 所需工具
- **📖 核心工作流** — 执行步骤
- **⛔ 行为护栏** — 安全约束
- **📝 模板与范例** — 代码参考

### 打包分发

> 若需将技能导入 [cc-switch](https://ccswitch.io) 等工具，请手动进入各技能目录打包，确保 `SKILL.md` 位于 zip 根目录，且一个 zip 对应一个技能。
