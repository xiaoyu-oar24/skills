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
| `xy-feat` | 端到端功能开发工作流（brainstorming → writing-plans → TDD → verification → review），串联 superpowers 技能链，自包含、跨项目复用 |
| `unicv` | 基于 UniApp + Vue 3 + TypeScript + Pinia + uv-ui 的微信小程序脚手架生成器 |
| `self-check-trinity` | 强制在交付代码前执行 lint → typecheck → test 三道质量检查 |
| `unified-api-response` | 强制所有 JSON API 返回统一的 `{ code, message, data }` 响应结构 |
| `api-name-drift-defense` | 防御第三方库版本升级导致的 API 重命名/移除问题 |
| `docs-layout-quadrant` | 强制文档四象限布局（specs / plan / tracking / guide），规整 docs/ 目录 |
| `health-probe-discipline` | Node 服务探针端点（health/readiness/liveness）规范约束 |
| `z-paging-best-practices` | z-paging 分页组件的最佳实践指南，包含下拉刷新、上滑加载及布局防踩坑指南 |

## 目录结构

```
skills/
├── docs/                          # 规范指南、设计文档（四象限：specs/plan/tracking/guide）
│   ├── 自定义Skills规范指南.md    # Skills 开发规范
│   ├── 任务边界自律守则.md        # AI Agent 行为约束
│   └── superpowers.md             # Superpowers 技能速查
├── skills/                        # 技能容器目录
│   ├── xy-feat/                   # 功能开发工作流
│   ├── unicv/                     # UniApp 脚手架
│   ├── self-check-trinity/        # 三合一质量检查
│   ├── unified-api-response/      # 统一 API 响应
│   ├── api-name-drift-defense/    # API 漂移防御
│   ├── docs-layout-quadrant/      # 文档四象限
│   ├── health-probe-discipline/   # 探针规范
│   └── z-paging-best-practices/   # z-paging 分页最佳实践
├── pack.sh                        # 打包脚本（每技能单独打 zip）
└── README.md
```

## 开发

所有技能严格遵循 `docs/自定义Skills规范指南.md`，每个 `SKILL.md` 必须包含：

- **🎯 触发条件** — 何时激活
- **⚙️ 依赖环境** — 所需工具
- **📖 核心工作流** — 执行步骤
- **⛔ 行为限制与护栏** — 安全约束
- **📝 模板与范例** — 代码参考

### 打包

为每个技能单独生成可直接导入 [cc-switch](https://ccswitch.io) 的 zip 包。cc-switch 要求「一个 zip = 一个技能」且 `SKILL.md` 位于 zip 根目录，因此脚本会进入各技能目录再打包：

```bash
bash pack.sh   # 在 dist/ 下为每个技能生成独立 zip
```

产物在 `dist/`，到 cc-switch 的「Skills → 从 ZIP 安装」中逐个导入即可。
