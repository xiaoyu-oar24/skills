# 技能体系深度审计报告（第 2 版）

> **历史状态**：本报告为 2026-07-21 基于内部规范审计的历史快照，部分内容（如 `unicv` 命名、版本号、问题定级）已被后续变更覆盖。
> **现役答案**：最新审计结论与修复记录请见 `docs/specs/技能审计报告-通用标准.md` 与 `docs/specs/技能修复报告.md`。

> 审计日期：2026-07-21
> 审计范围：`skills/` 下全部 8 个技能
> 审计方法：逐文件审阅 Frontmatter、五段式结构、工作流逻辑、行为护栏、依赖关系、章节合规性
> 合规基准：《AI规则与技能统一编写规范.md》（下称《规范》）

---

## 一、概况总览

| 技能 | 版本 | 行数 | 目录 | 定位 |
|:---|:---:|:---:|:---|:---|
| xy-feat | v4.2.0 | 577 | `skills/xy-feat/` | 端到端功能开发工作流 |
| unicv | v1.0.1 | 80 | `skills/unicv/` | UniApp 微信小程序脚手架 |
| unified-api-response | v1.1.1 | 129 | `skills/unified-api-response/` | API 统一响应规范 |
| api-name-drift-defense | v1.1.1 | 116 | `skills/api-name-drift-defense/` | 第三方 API 漂移防御 |
| z-paging-best-practices | v1.0.1 | 247 | `skills/z-paging-best-practices/` | z-paging 分页组件最佳实践 |
| health-probe-discipline | v1.1.1 | 119 | `skills/health-probe-discipline/` | 健康探针端点规范 |
| docs-layout-quadrant | v2.2.1 | 171 | `skills/docs-layout-quadrant/` | 文档四象限布局规范 |
| self-check-trinity | v1.0.2 | 95 | `skills/self-check-trinity/` | 质量三连闸（Lint → Typecheck → Test） |

**基础合规状况**：
- 8/8 目录使用 `kebab-case` 命名 ✅
- 8/8 Frontmatter 包含 `name` 和 `description` ✅
- 8/8 标注了版本号和作者 ✅
- 8/8 遵循五段式章节布局（🎯 → ⚙️ → 📖 → ⛔ → 📝）⚠️（2 个技能存在额外非标准章节，见 3.6）
- 仅 `unicv/` 包含 `references/` 子目录和 `CLAUDE.md` 文件，其余 7 个技能均为单文件 `SKILL.md`

---

## 二、🔴 严重问题

### 2.1 跨技能调用存在循环依赖路径

**涉及技能**：`self-check-trinity` ↔ `api-name-drift-defense`

**引用链**：

```
self-check-trinity (阶段 2：类型检查)
  └── typecheck 报错涉及第三方 API 改名
       └── 调用 api-name-drift-defense 辅助排查    ← "可通过技能调用执行"
              │
              └── api-name-drift-defense (阶段 3：回归验证)
                     └── 调用 self-check-trinity 做全量验证    ← "通过技能调用执行"
                            │
                            └── 若 typecheck 再次失败（API 漂移未彻底修复）
                                 → 再次触发 api-name-drift-defense → 循环
```

**具体代码位置**：
- `self-check-trinity/SKILL.md:43`：`"如果报错涉及第三方 API 改名，可通过技能调用执行 api-name-drift-defense 辅助排查。"`
- `api-name-drift-defense/SKILL.md:63`：`"修复后通过技能调用执行 self-check-trinity，确保代码通过 Lint、类型/编译检查、测试全部通过"`

**与 `unified-api-response` 的对比**：`unified-api-response/SKILL.md:56-59` 对 `self-check-trinity` 的调用**有内联降级方案**：
```
若本地支持 self-check-trinity 技能，通过技能调用执行...若该技能不可用，则执行以下内联验证：
- 运行项目对应的 lint 命令
- 运行类型/编译检查
- 运行测试命令
```
而 `api-name-drift-defense` **没有**降级方案，直接写了"通过技能调用执行"。

**风险评估**：
- 循环触发条件较苛刻（需要 typecheck 报错恰好是 API 改名 + 修复后再次出现同类报错），但**一旦触发理论上可无限循环**
- `api-name-drift-defense` 缺少对 `self-check-trinity` 的降级方案，技能独立加载时验证阶段可能静默失效

**建议措施**：
1. `api-name-drift-defense` 阶段 3 增加内联降级代码（参照 `unified-api-response` 的做法）
2. 在 `self-check-trinity` 阶段 2 的跨技能调用点增加约束："仅调用一次，不得在 api-name-drift-defense 回调 self-check-trinity 后再次触发 api-name-drift-defense"

---

### 2.2 `api-name-drift-defense` 对 `self-check-trinity` 调用无降级方案

**涉及文件**：`skills/api-name-drift-defense/SKILL.md` 第 63 行

此问题与 2.1 相关但独立。对比同仓库内其他技能对 `self-check-trinity` 的调用方式：

| 调用方 | 文件:行 | 有降级方案？ |
|:---|:---|:---:|
| `xy-feat` | SKILL.md:418 | ✅ 内联在阶段 4 中详述 |
| `unified-api-response` | SKILL.md:56-59 | ✅ 内联 Lint/Typecheck/Test |
| `api-name-drift-defense` | SKILL.md:63 | ❌ 仅有"通过技能调用执行" |

当 AI 独立加载 `api-name-drift-defense`（不通过 `xy-feat` 串起）时，`self-check-trinity` 若不可用，阶段 3 的回归验证将静默跳过，无法保证修复质量。

**建议措施**：参照 `unified-api-response` 阶段 3 的格式，为 `api-name-drift-defense` 阶段 3 补充内联降级验证命令。

---

## 三、🟡 值得关注的问题

### 3.1 `unicv` 存在 CLAUDE.md 与 SKILL.md 的知识重复

**涉及目录**：`skills/unicv/`

两份文件的重叠内容：

| 知识域 | SKILL.md 中的位置 | CLAUDE.md 中的位置 | 重复程度 |
|:---|:---|:---|:---|
| 技术栈版本约束 | 阶段 2/3 内联（Node 20.19.0, pnpm） | 独立表格（9 行 × 3 列） | 中 |
| 开发命令 | 阶段 4（`pnpm install` 等） | 独立代码块（10 条命令） | 高 |
| references 文件索引 | 阶段 2/3 步骤中按流程引用 | 末尾独立索引表（5 行） | 高 |
| 架构概览 | 无 | 请求层/状态管理/分包策略（共 45 行） | 无（CLAUDE.md 独有） |

`CLAUDE.md` 中"架构概览"（请求层、状态管理、分包策略、构建脚本、编码规范、新增页面流程）共约 60 行内容是 SKILL.md 没有的，属于有效补充。但"技术栈约束"、"开发命令"、"参考文件索引"三部分与 SKILL.md 重叠，存在双维护风险。

**建议措施**：
- 保留 CLAUDE.md 中的架构概览、编码规范、新增页面流程（SKILL.md 未覆盖的有效内容）
- 将技术栈约束表、开发命令、参考文件索引从 CLAUDE.md 中移除，改为指向 SKILL.md 的引用（如 `"详细技术栈约束和开发命令见 SKILL.md"`）

---

### 3.2 `xy-feat` 行为护栏分类：部分条目是工作流规则而非护栏

**涉及文件**：`skills/xy-feat/SKILL.md` 第 537-546 行

当前 7 条护栏：

| # | 护栏 | 性质分析 |
|:---:|:---|:---|
| 1 | 🚫 禁止日期前缀 | ✅ 护栏 — 机械可判断的禁令 |
| 2 | 阶段不可跳跃 | ⚠️ 工作流规则 — 描述的是"如何执行"，而非"禁止做什么" |
| 3 | 防止多版本命名冲突 | ✅ 护栏 — 明确的禁令（禁止 `-v2`/`-v3`） |
| 4 | Git 写操作需授权 | ✅ 护栏 — 安全边界 |
| 5 | 上下文对齐 | ⚠️ 工作流规则 — 阶段间传递条件，非行为禁令 |
| 6 | 证据优先 | ✅ 护栏 — 禁止空口断言 |
| 7 | 调试纪律 + 测试铁律 | ⚠️ 工作流规则 — 已在阶段 3 详述，此处是重复 |

《规范》§3.4 定义护栏为"列出 AI **绝对不能做**的操作"，§5.2 强调"护栏超过 8 条后，AI 的注意力集中在前 3-5 条"。当前 7 条在数量上合规，但混合了两种性质的规则，弱化了真正致命约束的注意力权重。

**建议措施**：
- 护栏精简为 4 条纯禁令：禁止日期前缀、禁止 `-v2`/`-v3` 后缀、Git 写操作需授权、禁止空口断言
- "上下文对齐"移入各阶段开头作为步骤 0
- "调试纪律"和"测试铁律"在阶段 3 已有详细版本，护栏处改为一句引用：`"调试与测试纪律：详见阶段 3.1 Iron Law 和阶段 3.5 重试策略"`

---

### 3.3 `self-check-trinity` 三次重试上限界定模糊

**涉及文件**：`skills/self-check-trinity/SKILL.md` 第 54 行

原文：
> "如果修复同一问题超过三次仍未通过，应停止并询问用户"

"同一问题"存在三种解释：

| 解释方式 | 示例场景 | 行为差异 |
|:---|:---|:---|
| A：每个阶段独立计次 | Lint 失败 3 次停止；Typecheck 从头计次 | AI 可能在三个阶段各重试 3 次（共 9 次） |
| B：全流程合计计次 | Lint 1 次 + Typecheck 1 次 + Test 1 次 = 3，停止 | 过早终止，一个阶段失败就几乎耗尽配额 |
| C：同类错误同阶段计次 | Typecheck 中同一文件的类型错误修复 3 次仍未通过 → 停止 | 精确但需明确"同类"判定标准 |

**与 `xy-feat` 阶段 3.5 重试策略的差异**：`xy-feat` 的重试策略是"3 次**不同假设**"，侧重调试方法的多样性；`self-check-trinity` 是"同一问题超过 3 次"，侧重修复尝试的次数。两者语义不同，当 `xy-feat` 调度 `self-check-trinity` 时，AI 可能混淆两套标准。

**建议措施**：
```
如果同一个检查阶段（lint / typecheck / test）内，对同一条错误信息或同一类编译/运行时报错
的修复尝试超过 3 次仍未通过，则停止该阶段并询问用户。不同错误信息的修复尝试不累计次数，
切换阶段后重新计次。
```

---

### 3.4 `xy-feat` 的 `<Bad>` / `<Good>` 呈现方式不对称

**涉及文件**：`skills/xy-feat/SKILL.md` 第 550-577 行

| 区块 | 格式 | 内容类型 |
|:---|:---|:---|
| `<Bad>` | 无序列表（纯文字条目） | 4 条反模式描述 |
| `<Good>` | Markdown 代码块 | 2 个具体的文档结构示例 |

**问题**：《规范》§3.5 要求 `<Bad>` / `<Good>` "成对出现"，且示范中两者均为代码块格式。当前 `<Bad>` 用文字描述错误行为（"编码前不先写失败的测试"），`<Good>` 用代码块展示正确输出（文档目录结构），两侧格式不对称，削弱了对比效果。

**解释**：`xy-feat` 是工作流技能，其 Bad/Good 侧重 AI 行为模式而非代码。但《规范》§3.5 对工作流技能的范例也有明确格式要求——Bad 也用结构化描述。当前 `<Bad>` 的无序列表形式可接受，但建议与 `<Good>` 保持格式对称。

**建议措施**：将 `<Bad>` 改为 Markdown 代码块格式呈现反模式：
```markdown
### <Bad>
```markdown
# ❌ 反模式
- 编码前不先写失败的测试，直接编写实现代码
- 验证阶段只口头声称"已通过"，不提供命令输出
- 文件名使用日期前缀：2026-07-15-上传功能-design.md
- 硬编码 npm test，忽略项目实际包管理器
```

---

### 3.5 `xy-feat` Superpowers 技能优先级标注语义矛盾

**涉及文件**：`skills/xy-feat/SKILL.md` 第 29-40 行

> 第 27 行引导语："优先尝试技能调用，返回不可用即自动降级为本文件内联的精简版本。"

| 技能 | 优先级标注 | 实际行为 |
|:---|:---|:---|
| `brainstorming` | **强制** | 不可用时降级为内联 1.1-1.5 |
| `verification-before-completion` | **强制** | 不可用时降级为内联校验 |

**矛盾**："强制"的语义是"不可跳过"，但两处都备有完整降级方案，意味着外部技能和内部降级**功能等价**。真正的"强制"应该是"无此技能则阶段阻塞"。现状是"工作流步骤强制（不可跳过），技能作为首选实现载体而非唯一实现方式"。

**建议措施**：
- 将优先级列改为**"首选"**（替代"强制"）、"推荐"、"可选" 三档
- 或保留"强制"但改为标注工作流步骤的强制性（而非外部技能的强制性），并在表头增加说明

---

### 3.6 两个技能存在非标准额外章节

《规范》§3 定义了五段式标准结构：🎯 → ⚙️ → 📖 → ⛔ → 📝。以下技能在此之外插入了额外章节：

| 技能 | 额外章节 | 插入位置 | 内容 |
|:---|:---|:---|:---|
| `xy-feat` | `## 📐 文档命名规范` | ⚙️ 与 📖 之间（第 53-67 行） | 文档命名约定、四象限路径映射、活文档策略 |
| `z-paging-best-practices` | `## 📌 核心概念` | ⚙️ 与 📖 之间（第 29-33 行） | z-paging 组件行为概述 |
| `z-paging-best-practices` | `## 常见问题排查` | 📝 之后（第 228-247 行） | 4 个 FAQ 及解决方案 |

**分析**：
- `xy-feat` 的"文档命名规范"在逻辑上是 `docs-layout-quadrant` 的摘要，与 ⚙️ 和 📖 都有重叠。建议合并到 ⚙️（作为约定）或 📖 阶段 0（作为初始化步骤）。
- `z-paging-best-practices` 的"核心概念"为理解工作流提供必要背景，放在 ⚙️ 和 📖 之间有合理性。但按《规范》应归入 ⚙️ 或 📖 阶段 1 的开头。
- "常见问题排查"是 FAQ，独立于五段式结构之外。可作为 📖 的附录或单独作为阶段。

**建议措施**：不强制删除（内容本身有价值），但建议在章节标题后加注 `（扩展）` 或在五段式对应位置内引用。若严格执行《规范》，则需合并到最近的规范章节中。

---

## 四、🟢 改进建议

### 4.1 多个技能的依赖探测命令缺乏判定标准

《规范》§3.2 要求"每个条件都附带验证命令，AI 可以先检查再执行而非假设一切就绪"。以下技能的探测命令未附带预期结果：

| 技能 | 当前命令 | 缺失 |
|:---|:---|:---|
| `unified-api-response` | `cat package.json` | 未说明检查 `dependencies` 中的哪个字段/包名 |
| `self-check-trinity` | `ls pnpm-lock.yaml ...` | 未说明匹配到不同锁文件后的包管理器选择逻辑 |
| `health-probe-discipline` | `cat package.json \| grep -i health` | 未说明 grep 无输出时的判定（是项目无探针 or 搜索方式不对） |

**对比良好实践**：`z-paging-best-practices` 的依赖探测写得很清晰：
> `ls src/uni_modules/` 并查找 `z-paging` 目录，或检查锁文件依赖确认

**建议措施**：为每个探测命令补充预期输出和判定逻辑（参照 4.1 表格格式）。

---

### 4.2 `health-probe-discipline` 验证阶段不完整

**涉及文件**：`skills/health-probe-discipline/SKILL.md` 第 53-55 行

当前验证仅覆盖两条 curl 命令（GET 和 HEAD 返回 200）。护栏要求"显式声明只接受 GET 和 HEAD"，但未验证拒绝行为。

缺失验证项：
- `curl -X POST http://localhost:PORT/api/health` → 应返回 405 Method Not Allowed
- 项目原有测试和类型检查未受影响（无回归）
- 若项目 CI 中有探针端点测试，确认一同通过

**建议措施**：在阶段 4 追加 POST 拒绝验证和回归测试检查。

---

### 4.3 `self-check-trinity` 阶段 0 缺少技术栈→命令映射表

**涉及文件**：`skills/self-check-trinity/SKILL.md` 第 30-35 行

阶段 0 的指引是"查看项目根目录的构建配置文件，识别技术栈与对应包管理器，确定可执行指令"。具体的命令映射**仅在 `<Good>` 示例中展示**（第 70-96 行），而非在工作流正文中。AI 在跟随阶段 0 指令时可能看不到示例区的命令表。

**对比**：`api-name-drift-defense` 和 `unified-api-response` 在 ⚙️ 章节中直接嵌入了技术栈→工具映射表。

**建议措施**：将 `<Good>` 中的四种技术栈命令示例提升到阶段 0 的工作流正文中。

---

### 4.4 `docs-layout-quadrant` 硬依赖 Git，无非 Git 环境降级

**涉及文件**：`skills/docs-layout-quadrant/SKILL.md`

全文 6 处出现 `git mv`：
- 阶段 1（命名规范校验 — 自动重命名使用 `git mv`）
- 阶段 2（四象限归类 — `git mv`）
- 阶段 3（象限生命周期管理 — `git mv`）
- 阶段 5（移动文件与修复链接 — `git mv`）
- 护栏（保留 Git 历史 — 使用 `git mv`）

⚙️ 章节已将 Git 列为前提条件（"运行 `git --version` 确认"），但若 `git` 不可用，整个工作流将阻塞。虽然文档整理场景中 Git 几乎是普遍存在的，但提供 `mv` 降级（附警告）可提升鲁棒性。

**建议措施**：在每个使用 `git mv` 的阶段增加一句：
> "若 `git mv` 不可用（非 Git 仓库），使用普通 `mv` 替代并提示用户：文件移动历史将不保留。"

---

### 4.5 `xy-feat` 的 `description` 字段超长

**涉及文件**：`skills/xy-feat/SKILL.md` 第 3 行

当前 `description` 值共 **322 字符**，单行。对比其他 7 个技能：

| 技能 | description 字符数 |
|:---|:---:|
| xy-feat | 322 |
| z-paging-best-practices | 211 |
| docs-layout-quadrant | 209 |
| unified-api-response | 175 |
| api-name-drift-defense | 168 |
| health-probe-discipline | 161 |
| unicv | 155 |
| self-check-trinity | 154 |

`xy-feat` 的 description 几乎是其他技能平均长度的 2 倍，在 YAML 编辑器中难以阅读。功能上无解析风险（中文字符在 YAML 双引号字符串中是安全的），纯属可维护性问题。

**建议措施**：使用 YAML 折叠块标量 `>` 改写为多行，或精简到 200 字符以内。

---

### 4.6 `z-paging-best-practices` 代码示例中存在未定义变量

**涉及文件**：`skills/z-paging-best-practices/SKILL.md` 第 116 行

```ts
const res = await proxy.$http.post(api.xxx, { ... })
```

代码中使用 `proxy.$http.post()` 和 `api.xxx`，但未说明：
- `proxy` 的来源（是 `getCurrentInstance().proxy`？是全局注入？）
- `api.xxx` 的定义位置

注释写的是"请求方式遵循项目自有封装（如 unicv 脚手架的 @/utils/request），此处仅示意"，但这让 AI 在非 unicv 项目中不知道如何适配。

**建议措施**：将代码中的 `proxy.$http.post(api.xxx, ...)` 替换为更通用的伪代码或增加 `// TODO: 替换为项目实际请求方法` 注释。

---

### 4.7 缺少版本变更日志

所有技能在 Frontmatter 中更新了 `version` 字段，但正文中没有对应的 CHANGELOG。对于已进入稳定期（全部 v1.0+）的技能，跨版本间的行为变化无迹可寻。最突出的是 `xy-feat` v4.2.0（已迭代 4 个大版本），无法从正文了解各版本间的差异。

**建议措施**：在 `SKILL.md` 末尾添加 CHANGELOG 区块：
```markdown
## 📋 变更日志
- **v4.2.0** (2026-07-21): 更新工具描述和操作说明
- **v4.1.0** (2026-07-17): ...
```

---

### 4.8 `unicv` references 文件角色不明确

**涉及文件**：`skills/unicv/SKILL.md` 阶段 2/3

工作流中的描述：
> "读取 `references/package-json.md` → 生成项目中的 `package.json`"

未明确 `references/package-json.md` 的角色：
- 是**直接复制**的模板文件？
- 是**带占位符需替换**的模板？
- 是**描述性指南**（AI 阅读后自己写）？

从文件名推断应该是完整模板，但使用动词"生成"而非"复制"或"写入"，留有歧义。

**建议措施**：明确区分"直接复制"和"参考后生成"两类 references，在阶段 2 中标注每类文件的操作方式。

---

## 五、✅ 良好实践（需保持）

| # | 实践 | 涉及技能 | 说明 |
|:---:|:---|:---|:---|
| 1 | 包管理器中立 | 全部 | 通过锁文件（`pnpm-lock.yaml` / `package-lock.json` / `Cargo.toml`）动态判定，不硬编码 |
| 2 | TRIGGER/SKIP 格式统一 | 全部 | `description` 统一包含触发/跳过条件，结构一致 |
| 3 | 多生态范例 | `unified-api-response`, `health-probe-discipline`, `api-name-drift-defense`, `self-check-trinity` | 提供了 Node/Python/Java/Go/Rust 多种技术栈的代码示例 |
| 4 | 证据优先原则 | `xy-feat`, `self-check-trinity` | 明确禁止空口断言，要求验证命令输出 |
| 5 | 活文档理念 | `docs-layout-quadrant`, `xy-feat` | 文档随代码演进，禁止日期前缀命名 |
| 6 | 护栏质量高 | 全部 | 护栏聚焦于可机械判断的操作禁令，无模糊的安全呼吁型条目 |
| 7 | 工作流工具有效指定 | `z-paging-best-practices`, `docs-layout-quadrant` | 每个阶段明确列出工具（"使用工具：读取文件"、"使用工具：编辑文件"），符合《规范》§5.5 |
| 8 | Bad/Good 代码块成对 | 除 `xy-feat` 外的 7 个技能 | 均使用代码块格式呈现反模式和推荐模式 |
| 9 | 自审机制 | `xy-feat` 阶段 1.5, 2.6 | 设计文档和规划文档均有结构化的自审清单 |
| 10 | 降级设计 | `xy-feat`, `unified-api-response` | 外部技能不可用时提供内联流程，保证核心功能不中断 |
| 11 | 调试纪律 | `xy-feat` 阶段 3.5 | 四阶段调试法 + 红牌警示 + 重试策略，系统性防猜测式修复 |
| 12 | 目录自律 | `docs/` 自身 | 仓库 `docs/` 遵循四象限布局，是技能"吃自己的狗粮"的良好范例 |
| 13 | ASCII 布局图 | `z-paging-best-practices` 阶段 3 | 使用 ASCII 图示解释 fixed 模式的 flex 布局原理，直观清晰 |

---

## 六、技能间依赖关系图

```
                          ┌─────────────────────────┐
                          │       xy-feat (编排器)    │
                          └──────┬────────┬─────────┘
                                 │        │
                    ┌────────────┘        └────────────┐
                    ▼                                  ▼
          ┌─────────────────┐              ┌─────────────────────┐
          │ self-check-     │              │ docs-layout-        │
          │ trinity         │              │ quadrant            │
          └────────┬────────┘              └─────────────────────┘
                   │
                   │ (typecheck 失败涉及 API 改名时)
                   ▼
          ┌─────────────────┐
          │ api-name-drift- │
          │ defense         │
          └────────┬────────┘
                   │
                   │ (回归验证)
                   ▼
          ┌─────────────────┐
          │ self-check-     │◀──── unified-api-response 也调用
          │ trinity         │     (阶段 3，有降级方案)
          └─────────────────┘
```

**图例**：
- 实线箭头 = 稳定调用关系（有降级方案或无循环风险）
- 虚线箭头 = 潜在循环路径（`self-check-trinity` ↔ `api-name-drift-defense`）
- `unicv`、`z-paging-best-practices`、`health-probe-discipline` 无跨技能依赖，为独立技能

---

## 七、优先级排序与行动建议

| 序号 | 问题 | 严重程度 | 修复复杂度 | 优先级 |
|:---:|:---|:---:|:---:|:---:|
| 2.2 | `api-name-drift-defense` 缺少 self-check-trinity 降级 | 🔴 | 低（复制 unified-api-response 的降级代码） | **P0** |
| 2.1 | 跨技能循环依赖路径 | 🔴 | 中（需双向修改 + 增加约束） | **P0** |
| 3.5 | Superpowers 优先级标注语义矛盾 | 🟡 | 低（改两个字） | **P1** |
| 3.4 | `<Bad>`/`<Good>` 格式不对称 | 🟡 | 低（格式统一） | **P1** |
| 3.3 | 三次重试逻辑歧义 | 🟡 | 低（替换一句话） | **P2** |
| 3.2 | 护栏混入工作流规则 | 🟡 | 中（需拆分 + 移动内容） | **P2** |
| 3.1 | `unicv` CLAUDE.md 与 SKILL.md 重复 | 🟡 | 中（需对比后删减） | **P2** |
| 3.6 | 非标准额外章节 | 🟡 | 低（加标注或合并） | **P3** |
| 4.3 | `self-check-trinity` 阶段 0 缺少命令映射表 | 🟢 | 低（从示例提升到正文） | **P3** |
| 4.2 | `health-probe-discipline` 验证不完整 | 🟢 | 低（追加两条命令） | **P3** |
| 4.1 | 依赖探测无判定标准（3 个技能） | 🟢 | 低（每处补充一句话） | **P3** |
| 4.6 | `z-paging` 代码示例未定义变量 | 🟢 | 低（改注释/伪代码） | **P3** |
| 4.4 | `docs-layout-quadrant` 非 Git 环境降级 | 🟢 | 低（4 处各加一句） | **P3** |
| 4.8 | `unicv` references 文件角色不明确 | 🟢 | 低（标注复制 vs 生成） | **P3** |
| 4.5 | `xy-feat` description 超长 | 🟢 | 低（YAML 多行改写） | **P3** |
| 4.7 | 缺少版本变更日志 | 🟢 | 中（8 个技能 × 补写历史） | **P4** |

---

## 八、与第 1 版报告的关键差异

| 差异点 | 第 1 版 | 第 2 版 | 说明 |
|:---|:---|:---|:---|
| 2.2 严重程度 | 🔴 YAML 解析安全 | 🟢 降级为可维护性问题（见 4.5） | 中文全角引号不破坏 YAML 双引号字符串解析 |
| 3.2 护栏数量 | 声称 8 条 | 实际 7 条 | 逐条计数修正 |
| 2.2 (新版) | — | 🔴 新增 `api-name-drift-defense` 降级缺失 | `unified-api-response` 有降级而它没有，是对比后发现的独立严重问题 |
| 3.6 | — | 🟡 新增非标准章节问题 | 第 1 版未覆盖 |
| 4.6 | — | 🟢 新增 `z-paging` 代码示例未定义变量 | 第 1 版未覆盖 |
| 4.7 | 原 4.4 | 🟢 补充了 xy-feat 已迭代 4 个大版本的事实 | 增强了说服力 |
| 4.8 | — | 🟢 新增 `unicv` references 角色不明确 | 第 1 版未覆盖 |
| 4.3 (新版) | — | 🟢 新增 `self-check-trinity` 命令映射位置不当 | 第 1 版未覆盖 |
| 6 | — | 新增技能间依赖关系图 | 可视化 2.1 的循环路径 |
| 良好实践 | 9 条 | 13 条 | 补充了自审机制、降级设计、调试纪律、ASCII 布局图等 |

---

## 九、修复状态追踪（2026-07-21）

### 9.1 修复执行总览

| 状态 | 数量 | 说明 |
|:---|:---:|:---|
| ✅ 已修复 | 11 | 变更已应用到对应技能文件 |
| ⬜ 未修复 | 4 | 低优先级或需人工决策，暂缓 |
| ➖ 已过时 | 1 | `unicv` CLAUDE.md 已被外部清理，问题自然消除 |

### 9.2 逐项修复明细

#### 🔴 严重问题

| 编号 | 问题 | 状态 | 修复内容 | 涉及文件 |
|:---|:---|:---:|:---|:---|
| 2.1 | 跨技能循环依赖 | ✅ 已修复 | `self-check-trinity` 阶段 2 增加 **"单次调用，禁止递归"** 约束，阻断 `api-name-drift-defense` 回调后的二次触发 | `self-check-trinity/SKILL.md:54` |
| 2.2 | `api-name-drift-defense` 无降级方案 | ✅ 已修复 | 阶段 3 补充内联 Lint/Typecheck/Test 降级验证代码，与 `unified-api-response` 对齐 | `api-name-drift-defense/SKILL.md:61-68` |

#### 🟡 值得关注的问题

| 编号 | 问题 | 状态 | 修复内容 | 涉及文件 |
|:---|:---|:---:|:---|:---|
| 3.1 | `unicv` CLAUDE.md 与 SKILL.md 重复 | ➖ 已过时 | CLAUDE.md 已被清理（文件不存在），重复问题自然消除 | — |
| 3.2 | `xy-feat` 护栏混入工作流规则 | ✅ 已修复 | 护栏从 7 条精简为 4 条核心禁令（禁止日期前缀、禁止版本后缀、Git 授权、禁止空口断言）+ 1 条工作流引用；"上下文对齐"、"调试纪律"、"测试铁律"保留在工作流阶段正文中 | `xy-feat/SKILL.md:537-543` |
| 3.3 | `self-check-trinity` 三次重试逻辑歧义 | ✅ 已修复 | 护栏中的重试规则明确为"同阶段、同错误类型、独立计次，切换阶段重新计次" | `self-check-trinity/SKILL.md:65` |
| 3.4 | `xy-feat` `<Bad>`/`<Good>` 格式不对称 | ✅ 已修复 | `<Bad>` 从无序列表改为 `markdown` 代码块，与 `<Good>` 保持格式对称 | `xy-feat/SKILL.md:547-556` |
| 3.5 | Superpowers 优先级标注语义矛盾 | ✅ 已修复 | `brainstorming`、`verification-before-completion` 优先级从 "**强制**" 改为 "**首选**"，与引导语"优先尝试...降级"语义一致 | `xy-feat/SKILL.md:31,37` |
| 3.6 | 非标准额外章节 | ⬜ 未修复 | 报告建议"不强制删除"，`xy-feat` 的 📐 和 `z-paging` 的 📌 有独立价值，暂保留 | — |

#### 🟢 改进建议

| 编号 | 问题 | 状态 | 修复内容 | 涉及文件 |
|:---|:---|:---:|:---|:---|
| 4.1 | 依赖探测无判定标准（3 个技能） | ⬜ 未修复 | 涉及 `unified-api-response`、`self-check-trinity`、`health-probe-discipline` 的探测命令，改动分散，暂缓 | — |
| 4.2 | `health-probe-discipline` 验证不完整 | ✅ 已修复 | 阶段 4 追加 `curl -X POST` → 405 验证 + 项目回归测试检查 | `health-probe-discipline/SKILL.md:56-57` |
| 4.3 | `self-check-trinity` 缺少命令映射表 | ✅ 已修复 | 阶段 0 新增 5 技术栈（Node/Python/Rust/Java/Go）× 4 列（包管理器/Lint/Typecheck/Test）命令映射表 | `self-check-trinity/SKILL.md:36-43` |
| 4.4 | `docs-layout-quadrant` 非 Git 环境降级 | ✅ 已修复 | 3 处 `git mv` 添加非 Git 环境 `mv` 降级提示（阶段 1 命名校验、阶段 5 移动文件、护栏保留历史） | `docs-layout-quadrant/SKILL.md:41,107,126` |
| 4.5 | `xy-feat` description 超长（322 字符） | ⬜ 未修复 | 仅为可读性问题，非功能性缺陷，且涉及 YAML 格式变更需同步更新规范文档 | — |
| 4.6 | `z-paging` 代码示例未定义变量 | ✅ 已修复 | `proxy.$http.post(api.xxx, ...)` → 通用的 `request({ url, method, data })` 函数签名 + 明确注释"替换为项目实际请求方法" | `z-paging-best-practices/SKILL.md:115-117` |
| 4.7 | 缺少版本变更日志 | ⬜ 未修复 | 8 个技能需补写历史，成本较高。本次修复已升级版本号，后续迭代可逐步建立 | — |
| 4.8 | `unicv` references 文件角色不明确 | ✅ 已修复 | 阶段 2/3 标注：references 文件为"可直接复制的完整模板"，操作方式明确为"直接复制"/"直接写入" | `unicv/SKILL.md:32-38` |

### 9.3 版本变更汇总

| 技能 | 旧版本 → 新版本 | 变更项数 |
|:---|:---|:---:|
| `api-name-drift-defense` | v1.1.1 → **v1.2.0** | 1（降级方案补全） |
| `self-check-trinity` | v1.0.2 → **v1.1.0** | 3（循环约束 + 重试明确 + 命令映射表） |
| `xy-feat` | v4.2.0 → **v4.3.0** | 3（优先级标注 + Bad/Good 格式 + 护栏精简） |
| `unicv` | v1.0.1 → **v1.1.0** | 1（references 角色标注） |
| `health-probe-discipline` | v1.1.1 → **v1.2.0** | 1（验证补全） |
| `docs-layout-quadrant` | v2.2.1 → **v2.3.0** | 1（非 Git 降级） |
| `z-paging-best-practices` | v1.0.1 → **v1.1.0** | 1（示例变量明确化） |

### 9.4 未修复项说明

| 编号 | 问题 | 暂缓原因 |
|:---|:---|:---|
| 3.6 | 非标准额外章节 | 报告明确建议"不强制删除"，📐 文档命名规范和 📌 核心概念有独立参考价值 |
| 4.1 | 依赖探测无判定标准 | 涉及 3 个技能的分散改动，每个探测命令的预期输出依赖具体项目上下文，改动需逐一斟酌 |
| 4.5 | `xy-feat` description 超长 | 纯可读性问题。改用 YAML `>` 折叠标量需同步更新《规范》§2.4 的"必须用双引号包裹"要求 |
| 4.7 | 缺少版本变更日志 | 需补写 8 个技能的历史版本记录，建议后续迭代中逐步建立，本次暂不作为阻塞项 |
