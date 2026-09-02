# AI Skills 集合库

自定义 AI 代理技能（Skills）的集合仓库，专为现代 AI 辅助编程工具设计。每个技能聚焦一个具体的工程场景，通过标准化的 `SKILL.md` 文件指导 AI 完成任务。

## 技能列表

| 技能 | 描述 |
|------|------|
| `xy-feat` | 端到端功能开发工作流（需求澄清 → 方案设计 → 规划分解 → TDD/VDD 执行 → 验证 → 审查收尾 → 指南与合规整理），优先调用 superpowers，未安装时降级为内联流程；另整合 ponytail 可选增强（不可用时跳过） |
| `docs-layout-quadrant` | 文档五维布局（reqs / specs / plan / tracking / guide）与产出分级（L0/L1/L2 Doc-Tier Gate），以 aidocs/ 专属目录规整 AI 过程文档，支持旧版 docs/ 自动迁移、弹性降级与规则文件注入 |
| `lean-docs` | 文档瘦身与检索治理：归档过期 plan/tracking、特性群 SSOT 聚合、精简活文档流水账、紧凑 INDEX 索引、多 AI 工具检索阻断，降低 Token 消耗 |
| `doc-craftsman` | 结构化文档内容契约与接口文档原子化：One API One File 原子文件 + Hub-and-Spoke 三级索引（域总索引、模块子索引、原子文件）与共享约定下沉，支持编写接口契约、架构规范、业务需求、操作指南，目录级扫描诊断并批量拆解混杂多主题的巨石文档（诊断清单经确认后强制落盘执行）、自愈相对路径链接 |
| `self-check-trinity` | 强制在交付代码前执行 lint → typecheck → test 三道质量检查 |
| `uniapp-wechat-scaffold` | 基于 UniApp + Vue 3 + TypeScript + Pinia + uv-ui 的微信小程序脚手架生成器 |
| `audit-vue-biz-component` | 审计基于 Element Plus 二次封装的 Vue3 B端业务组件（四维评分报告，只读不改码） |
| `unified-api-response` | 强制所有 JSON API 返回统一的 `{ code, message, data }` 响应结构 |
| `api-name-drift-defense` | 防御第三方库版本升级导致的 API 重命名/移除问题 |
| `health-probe-discipline` | 服务探针端点（health/readiness/liveness）规范约束，支持 Node/Java/Python 多技术栈 |
| `z-paging-best-practices` | z-paging 分页组件的最佳实践指南，包含下拉刷新、上滑加载及布局防踩坑指南 |
| `code-search-discipline` | 代码搜索纪律：grep 必须限定目录级 path 或 include 文件类型过滤器（禁止裸搜 src/ 根）、绕开 node_modules/uni_modules/static 等风险目录、大型源码优先 codegraph 语义探索（grep 仅做定位行号等窄查询） |

## 文档技能组（四技能协同）

`xy-feat`、`docs-layout-quadrant`、`doc-craftsman`、`lean-docs` 构成一个互补技能组，围绕 AI 过程文档全生命周期分工协作，任一技能不重复实现其余技能的职责：

| 技能 | 分工 | 职责 |
|------|------|------|
| `xy-feat` | 工作流入口 | 端到端功能开发全流程；阶段 6 指南产出与文档合规整理时调用 `docs-layout-quadrant`，并按需调用 `doc-craftsman` 拆分接口契约 |
| `docs-layout-quadrant` | 五维骨架 | 目录结构、档位判定（Doc-Tier Gate L0/L1/L2）、生命周期归档、`aidocs/INDEX.md` 全局索引、规则文件标记块注入；放行 `specs/接口文档/` 子体系 |
| `doc-craftsman` | 内容契约 | 单篇结构化文档编写（接口契约七节结构）、巨石文档原子化拆解（含目录级扫描诊断）、接口文档三级索引与共享约定下沉；遵循五维骨架不重复实现 |
| `lean-docs` | 治理收敛 | 冗余流水账精炼、特性群 SSOT 聚合、INDEX 紧凑化、检索噪音阻断；结构混杂拆解让位 `doc-craftsman` |

**路由速查**：端到端开发功能 → `xy-feat`；文档放哪 / 要不要写文档 / 生命周期归档 → `docs-layout-quadrant`；文档内容怎么写 / 拆巨石 / 接口文档索引 → `doc-craftsman`；文档太啰嗦 / 堆积 / 降 Token → `lean-docs`。

## 目录结构

```
/
├── aidocs/                        # AI 过程文档（五维：reqs/specs/plan/tracking/guide，含文档产出分级 Doc-Tier Gate）
│   ├── INDEX.md                   # 知识索引导航
│   ├── reqs/                      # 需求文档 [按需-决策树]（中长周期，验收后冻结保留）
│   ├── specs/                     # 设计规范 [核心资产]（长周期，契约防腐）
│   │   ├── 任务边界自律守则.md
│   │   ├── 技能审计报告-深度综合版.md
│   │   ├── 文档产出分级判定.md
│   │   └── ...
│   ├── plan/                      # 执行计划 [L2 全量档落盘]（短周期，脚手架）
│   ├── tracking/                  # 进度跟踪 [L2 全量档落盘/多Agent强制]（超短周期，脚手架）
│   └── guide/                     # 使用指南 [核心资产/按需]（长周期，团队资产）
│       ├── AI规则与技能统一编写规范.md
│       ├── superpowers.md
│       └── ...
├── skills/                        # 技能容器目录
│   ├── xy-feat/                   # 功能开发工作流 (v5.7.0)
│   ├── uniapp-wechat-scaffold/    # UniApp 脚手架
│   ├── audit-vue-biz-component/   # Vue3 B端业务组件审计
│   ├── self-check-trinity/        # 三合一质量检查
│   ├── unified-api-response/      # 统一 API 响应
│   ├── api-name-drift-defense/    # API 漂移防御
│   ├── docs-layout-quadrant/      # 文档五维布局 (v3.4.0)
│   ├── lean-docs/                 # 文档瘦身与检索治理 (v1.4.1)
│   ├── doc-craftsman/             # 结构化文档契约与接口文档原子化 (v1.1.0)
│   ├── health-probe-discipline/   # 探针规范
│   ├── z-paging-best-practices/   # z-paging 分页最佳实践
│   └── code-search-discipline/    # 代码搜索纪律
├── superpowers-main/              # superpowers 技能集源码副本（xy-feat 优先调用链来源）
├── README.md
├── CLAUDE.md
└── AGENTS.md
```

## 开发

所有技能严格遵循 `aidocs/guide/AI规则与技能统一编写规范.md`，每个 `SKILL.md` 必须包含：

- **🎯 触发条件** — 何时激活
- **⚙️ 依赖环境** — 所需工具
- **📖 核心工作流** — 执行步骤
- **⛔ 行为护栏** — 安全约束
- **📝 模板与范例** — 代码参考