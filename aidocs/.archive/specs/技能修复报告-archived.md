# 技能问题修复报告

> **修复日期**：2026-07-22
> **修复范围**：`skills/` 目录下全部 8 个技能及配套规范文档
> **修复依据**：`docs/specs/技能审计报告-通用标准.md` 所列 P1–P3 问题
> **报告状态**：已完成 `unicv` 重命名为 `uniapp-wechat-scaffold`（见 §五）

## 一、修复总览

| 技能 | 原版本 | 新版本 | 修复类型 |
|:---|:---:|:---:|:---|
| `api-name-drift-defense` | 1.1.1 | 1.1.2 | 补 H1、显式声明调用环边界、补平台假设 |
| `docs-layout-quadrant` | 2.2.1 | 2.2.2 | 删除/移动前需用户确认、章节标题统一、补平台假设 |
| `health-probe-discipline` | 1.1.1 | 1.1.2 | 补 H1、FastAPI 示例补 `Response` import、修正 `curl` 命令、探针豁免统一响应、删除确认护栏 |
| `self-check-trinity` | 1.0.2 | 1.0.3 | 补 H1、声明调用环已闭合、补平台假设 |
| `uniapp-wechat-scaffold` | 1.0.2 | 1.0.2 | 内容修复（正文 SKIP、references 完整性检查、平台假设）；并从 `unicv` 重命名至此，目录/frontmatter/触发词/交叉引用已全部同步 |
| `unified-api-response` | 1.1.1 | 1.1.2 | 补 H1、SKIP 中声明探针端点豁免、补平台假设 |
| `xy-feat` | 4.2.0 | 4.3.0 | 正文从 577 行重构为 248 行，内联降级流程拆入 `references/fallback-*.md`，修正"强制/降级"措辞，子代理简报不落盘 |
| `z-paging-best-practices` | 1.0.1 | 1.0.2 | 去钉 `v2.8.6`、示例改用 `@/utils/request`、补平台假设 |

## 二、关键修复详情

### P1 `xy-feat` 正文规模超限

- **问题**：`SKILL.md` 577 行，远超通用标准 500 行/5000 tokens 红线，且内联降级流程是对 superpowers 的内容复制。
- **修复**：
  - 正文压缩为 248 行，仅保留调度骨架、阶段出入口、安全护栏、文档命名规范与范例
  - 新增 4 个 references 文件：
    - `references/fallback-init-design.md`
    - `references/fallback-plan.md`
    - `references/fallback-tdd.md`
    - `references/fallback-verify-finish.md`
  - `description` 同步更新为"未安装时降级为 references/ 内联流程"

### P2 内容正确性与规则冲突

| 位置 | 问题 | 修复 |
|:---|:---|:---|
| `health-probe-discipline/SKILL.md:111` | FastAPI 示例缺少 `Response` import | 改为 `from fastapi import APIRouter, Request, Response` |
| `health-probe-discipline/SKILL.md:90-91` | `curl -I -X GET` 与 `curl -I -X HEAD` 命令自相矛盾/冗余 | 改为 `curl -s -o /dev/null -w "%{http_code}" -X GET ...` 与 `curl -I ...` |
| `z-paging-best-practices/SKILL.md:98,116` | 示例使用未定义的 `proxy.$http` | 顶部导入 `@/utils/request`，函数体改用 `request({...})` |
| `unified-api-response` vs `health-probe-discipline` | 探针端点是否套统一响应格式存在规则死锁 | 双方在 SKIP 中互声明豁免，health-probe 护栏声明"规则冲突时以本技能为准" |
| `docs-layout-quadrant` / `health-probe-discipline` | 删除/移动文件无用户确认 | 两技能各补"先列出清单并获得用户确认"护栏 |

### P3 结构性改进

| 问题 | 修复 |
|:---|:---|
| `self-check-trinity` ↔ `api-name-drift-defense` 互相调用环无界 | 双方在阶段说明中显式声明"回归验证不再级联回调"，边界清晰 |
| 全库无触发评测集 | 8 个技能各新增 `references/evals.md`，含 3+ 应触发 + 2+ 不应触发用例 |
| `z-paging` 钉死 `v2.8.6` | 改为 `v2.8+` 并提示"执行前先确认项目实际版本" |
| 各技能未声明平台假设 | 在"依赖与先决条件"中统一补充"类 Unix 终端"假设 |
| `xy-feat` 子代理简报/汇报文件位置未定义 | 改为"直接在提示词和返回结果中传递，不落盘为文件" |

## 三、规范文档与配置同步

- **改写** `docs/guide/AI规则与技能统一编写规范.md`（559 行），融合市面通用标准：
  - 正文 500 行/5000 tokens 红线与渐进式披露拆分模式
  - `references/evals.md` 评估驱动开发要求
  - 技能组合纪律（无矛盾、调用环有界、跨技能降级）
  - 破坏性操作确认闸、平台假设、示例可运行等硬性要求
- **更新** `AGENTS.md`：修正 `uniapp-wechat-scaffold`（原 `unicv`）分类错误，补充触发评测集与平台假设说明。
- **更新** `docs/specs/技能审计报告-通用标准.md`：顶部标注"2026-07-22 已全部修复"。
- **登记** 本报告至 `docs/INDEX.md`。

## 四、验证结果

```
SKILL.md 行数：
     120 api-name-drift-defense
     173 docs-layout-quadrant
     123 health-probe-discipline
      98 self-check-trinity
      82 uniapp-wechat-scaffold
     132 unified-api-response
     248 xy-feat
     250 z-paging-best-practices

残留问题扫描：
- proxy.$http          无
- curl -I -X           无
- Response import      已修正
- 删除/批量移动确认闸  已补充
- 探针豁免声明         已补充
- 调用环边界           已显式声明
```

## 五、已完成：unicv 重命名为 uniapp-wechat-scaffold

`unicv` 名称不透明，不符合"见名知意"原则。已将其目录及所有引用重命名为 `uniapp-wechat-scaffold`：

- 目录：`skills/unicv/` → `skills/uniapp-wechat-scaffold/`
- `SKILL.md`：`name`、H1、`description`、触发词 `/unicv` → `/uniapp-wechat-scaffold`
- `references/evals.md`：标题与 `/unicv` 触发用例同步更新
- `README.md`：技能表与目录树同步更新
- `AGENTS.md`、`CLAUDE.md`：分类描述与路径同步更新
- `z-paging-best-practices/SKILL.md`：示例注释中的"unicv 脚手架约定"同步更新
- 本报告与 `docs/specs/技能审计报告-通用标准.md` 中的 `unicv` 字样已同步替换

重命名后仍满足 kebab-case、≤64 字符、无保留词等通用标准。
