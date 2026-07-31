# 技能审计报告（市面通用标准版）

> **审计日期**：2026-07-22
> **审计范围**：`skills/` 目录下全部 8 个技能（仅 `SKILL.md` 及引用关系，不含 `uniapp-wechat-scaffold/references/` 模板内容）
> **审计基准**：市面通用的 Agent Skills 编写标准（见下方来源），**不采用**本仓库内部《AI规则与技能统一编写规范.md》
> **修复状态**：2026-07-22 已全部修复（P1–P3，含规范文档同步改写），当前以各技能最新版本为准。
> **与既有报告的关系**：`docs/specs/skills-audit-report.md`（2026-07-21，第 2 版）基于内部规范审计，本报告换用外部通用标准独立复核，两者互补、互不替代。其中 `unicv` 已按本次修复重命名为 `uniapp-wechat-scaffold`，原命名问题已解决。

## 审计标准来源

| 标准 | 关键要求 | 来源 |
|:---|:---|:---|
| Anthropic 官方技能编写最佳实践 | 正文 < 500 行；description 用第三人称、同时说明做什么（WHAT）与何时用（WHEN）；渐进式披露；示例必须正确可运行；评估驱动开发（先建 3+ 触发评测再完善文档） | https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills/best-practices |
| 开放 Agent Skills 规范 | `name` ≤ 64 字符，仅小写字母/数字/连字符，不得含 "anthropic"/"claude"；`description` ≤ 1024 字符；三级渐进披露模型（元数据 ~100 tokens → 正文 → 资源文件） | https://agentskills.io/specification |
| Anthropic 工程博客（2025-10） | 技能正文建议 < 5000 tokens；资源按需加载，未触发的技能几乎零成本 | https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills |
| 通用 Agent 工程惯例 | 技能共存于同一元数据空间，规则不得互相矛盾；破坏性操作需用户确认；时效性信息需标注或避免；声明平台假设 | 行业通行做法 |

---

## 一、总览与评级

| 技能 | 行数 | 评级 | 主要短板（按通用标准） |
|:---|:---:|:---:|:---|
| self-check-trinity | 95 | ✅ 优 | 无明显问题；命名带隐喻色彩（可接受） |
| uniapp-wechat-scaffold | 80 | ✅ 优 | references 拆分是渐进式披露的正面示范；缺模板完整性自检 |
| api-name-drift-defense | 116 | 🟡 良 | 与 self-check-trinity 存在互相调用环（有界） |
| unified-api-response | 129 | 🟡 良 | 与 health-probe-discipline 规则冲突（见 P2-2） |
| docs-layout-quadrant | 171 | 🟡 良 | 允许无确认删除/移动文件（见 P2-3） |
| z-paging-best-practices | 247 | 🟡 良 | 示例含未定义符号；版本号钉死（时效性） |
| health-probe-discipline | 119 | 🔴 待改进 | 示例代码缺 import、验证命令自相矛盾、规则冲突 |
| xy-feat | 577 | 🔴 待改进 | 正文超 500 行红线，约 1.5–2 万 tokens，远超 < 5000 tokens 建议 |

**基础合规（全部通过）**：8/8 的 `name` 符合 ≤64 字符 kebab-case、不含保留词；8/8 的 `description` 在 1024 字符内且同时包含 WHAT + WHEN（中文 `TRIGGER when:` / `SKIP:` 写法与英文 "Use when... / Do not use for..." 语义等价，路由兼容性良好）；8/8 无 README/CHANGELOG 等多余文件；`xy-feat` 限定"仅显式 `/xy-feat` 触发"的写法是抑制误触发的优秀实践。

---

## 二、🔴 P1 问题（严重，建议优先修复）

### P1-1 `xy-feat` 正文体积违反渐进式披露红线

- **事实**：`skills/xy-feat/SKILL.md` 共 577 行，超过 Anthropic 官方建议的 500 行上限；按中文密度估算正文约 1.5–2 万 tokens，是"正文 < 5000 tokens"建议值的 3–4 倍。
- **影响**：该技能一旦被触发，全文注入上下文，挤占工作窗口；且其内联降级流程（阶段 1.1–1.5、2.2–2.6、3.5 调试法等约 300 行）是对 superpowers 技能的**内容复制**，形成双源维护负担——superpowers 升级后内联副本必然腐化。
- **建议**：
  1. 将各阶段的内联降级流程拆到 `skills/xy-feat/references/fallback-stage-1.md` 等按需加载文件，`SKILL.md` 正文只保留调度逻辑与护栏，压缩到 300 行以内；
  2. 降级流程改为"读取对应 references 文件执行"，正文不再内联。

## 三、🔴 P2 问题（内容正确性与冲突）

通用标准的底线是"技能给出的代码与命令必须可直接运行"。以下示例在被 AI 原样采用时会产生错误或歧义行为：

### P2-1 三处示例/命令硬错误

| # | 位置 | 问题 | 修复建议 |
|:--|:---|:---|:---|
| 1 | `health-probe-discipline/SKILL.md:107,117` | FastAPI 示例只导入 `APIRouter, Request`，却使用未导入的 `Response`，直接运行即 `NameError` | 补 `from fastapi import Response` |
| 2 | `health-probe-discipline/SKILL.md:54-55` | `curl -I -X GET` 中 `-I` 隐含 HEAD，与 `-X GET` 自相矛盾；`curl -I -X HEAD` 完全冗余 | 改为 `curl -s -o /dev/null -w "%{http_code}" -X GET <url>` 与 `curl -I <url>` |
| 3 | `z-paging-best-practices/SKILL.md:116` | `<script setup>` 语境下使用未定义的 `proxy.$http`，AI 照抄必报错 | 改用 `@/utils/request` 实景演示，或显式写出 `proxy` 的来源 |

### P2-2 `unified-api-response` 与 `health-probe-discipline` 规则互相矛盾

- `unified-api-response` 护栏要求"**所有** JSON API Handler 必须返回统一 code-message-data 格式"（`SKILL.md:63`）；
- `health-probe-discipline` 的 Good 范例却让 `/api/health` 返回裸 `{ status: 'ok' }`（`SKILL.md:85`）。
- 在同时启用统一响应的项目里新增探针端点时，两条规则给出相反指令，且双方的 SKIP 均未豁免对方场景。所有技能共存于同一元数据空间，矛盾规则会使路由结果不确定。
- **建议**：在 `health-probe-discipline` 的 SKIP 或护栏中显式声明"探针端点豁免统一响应封装（需兼容 LB/裸探测）"，或反向在 `unified-api-response` 中声明探针为例外。二选一，但必须有一处明说。

### P2-3 破坏性操作缺少用户确认闸

- `docs-layout-quadrant/SKILL.md:80` 允许对 tracking 文档"移入 `.archive/` **或直接删除**"，`:41` 允许发现日期前缀时"**自动重命名**"，该技能 6 条护栏均未要求确认；
- `health-probe-discipline/SKILL.md:40` "成功合并后，删除多余的按动词拆分的文件"，同样无确认要求。
- 对照通用 Agent 安全惯例（不可逆操作需明确授权）以及本仓库其他技能的既有做法（`xy-feat` 放弃分支需输入 `discard`、`uniapp-wechat-scaffold` 冲突时暂停确认），这两处门槛偏低。
- **建议**：为两个技能各补一条护栏——"删除或批量移动任何已存在文件前，必须先列出清单并获得用户确认"。

## 四、🟡 P3 问题（结构性改进项）

1. **技能调用环未显式声明边界**：`self-check-trinity:43` → `api-name-drift-defense` → 其 `:63` 又调回 `self-check-trinity`。循环目前靠 trinity 的"三次重试上限"隐式终止，可运行但不可自证。建议在一方注明"回归验证仅执行一次，不再级联触发 drift"。
2. **全库无触发评测集**：Anthropic 官方主张"评估驱动开发"——先为每个技能准备 3 条以上"应触发 / 不应触发"的测试提示词，再迭代 description。当前 8 个技能均无评测资产，description 质量只能靠人工评审兜底。这是通用标准下全库层面的最大改进空间。
3. **时效性信息**：`z-paging-best-practices:12,26` 钉死 `v2.8.6`，组件升级后技能静默过期。建议改为"v2.8+（执行前先确认项目实际版本）"。
4. **平台假设未声明**：全部技能默认 Unix-like 环境（`ls`/`grep`/`mkdir -p`/`cat`），仅 `xy-feat` 在依赖中声明了 Unix-like 前提。按通用标准建议在其余技能的"依赖与先决条件"中补一句平台假设。
5. **`xy-feat` 措辞矛盾**：superpowers 依赖表中 `brainstorming`、`verification-before-completion` 优先级标"强制"却同时给出降级策略（`:31,:37`）。实际语义是"阶段强制、载体可降级"，建议改写避免歧义。
6. **`xy-feat` 临时文件位置未定义**：`:328` 的 `task-N-brief.md` / `task-N-report.md` 未说明存放路径，若落入 `docs/` 会违反本项目自己的四象限规则。建议明确为会话内临时产物或指定存放目录。

## 五、已识别但按既定决策不重审的事项

- **`uniapp-wechat-scaffold` 原名为 `unicv`**，名称不透明（缩写无语义，不符合通用标准的"见名知意"）：已按本次修复重命名为 `uniapp-wechat-scaffold`，问题已解决。
- **`self-check-trinity` 命名隐喻**（"trinity" 需内部知识才能理解）：description 已充分补偿路由可发现性，评级为可接受偏差。

## 六、修复优先级汇总

| 优先级 | 事项 | 涉及技能 |
|:---|:---|:---|
| P1 | 正文拆分至 references，压到 500 行 / 5000 tokens 内 | xy-feat |
| P2 | 修正 3 处示例/命令硬错误 | health-probe-discipline、z-paging-best-practices |
| P2 | 裁定探针端点是否豁免统一响应 | unified-api-response × health-probe-discipline |
| P2 | 补"删除/移动前需确认"护栏 | docs-layout-quadrant、health-probe-discipline |
| P3 | 声明调用环边界、建触发评测集、解钉版本号、补平台假设、修措辞 | 见 §四 |
