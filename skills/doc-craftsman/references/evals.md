# doc-craftsman 触发评测集

> 评估驱动开发：修改 `description` 或触发条件后，用以下提示词回归验证路由行为。本技能与 `docs-layout-quadrant`（结构/生命周期）、`lean-docs`（内容冗余治理）存在相邻边界，"不应触发"用例覆盖三技能分流。

## 应触发 (Expected: 激活)

1. **显式命令调用**：
   - 提示词：`/doc-craftsman 帮我为用户中心新增一个查询用户详情的接口文档`
   - 预期结果：激活 `doc-craftsman`，定位到 `specs/接口文档/user/` 并生成原子接口契约文件（七节结构）及更新三级索引。

2. **原子化接口契约编写**：
   - 提示词：`帮我编写 POST /admin-api/order/refund-apply 接口文档，并登记到订单模块索引中`
   - 预期结果：激活 `doc-craftsman`，遵循单接口原子化规范输出 `order-refund-apply.md`，更新 `order/INDEX.md` 接口清单与共享约定下沉节。

3. **巨石文档解耦重构**：
   - 提示词：`我们的 aidocs/specs/auth.md 文件里面混了 15 个登录认证接口和设计说明，太长了，帮我按原子化标准拆分并建立索引`
   - 预期结果：激活 `doc-craftsman`，执行阶段 0 判定（结构混杂）→ 阶段 2 拆解 → 阶段 3 三级索引联动。

4. **架构决策/规范文档创建**：
   - 提示词：`我们需要在 aidocs/specs/ 下新建一份关于分布式事务一致性方案的 ADR 技术规范文档`
   - 预期结果：激活 `doc-craftsman`，生成结构化 ADR（文档头遵循 docs-layout-quadrant 最小头）。

5. **共享约定下沉**：
   - 提示词：`接口文档里每个文件都重复写了一遍 URL 前缀和数据模型，太啰嗦了，帮我收敛`
   - 预期结果：激活 `doc-craftsman`，将共享约定下沉至模块子索引「架构与业务约定」节，原子文件去重。

6. **接口文档索引维护**：
   - 提示词：`新增了 blacklist 黑名单模块的 7 个接口，帮我建模块子索引和域总索引入口`
   - 预期结果：激活 `doc-craftsman`，创建 `blacklist/INDEX.md` 与域总索引分组行，并在 `aidocs/INDEX.md` 维护接口文档域入口。

## 不应触发 (Expected: 跳过/分流)

1. **端到端业务功能开发**：
   - 提示词：`/xy-feat 实现商品分类增删改查的前后端逻辑`
   - 预期结果：跳过 `doc-craftsman`，由 `xy-feat` 处理（其阶段 6 按需调用本技能）。

2. **五维目录归类与生命周期归档**：
   - 提示词：`/docs-layout-quadrant 检查当前文档目录是否符合五维象限规范，并把已交付的 plan 归档`
   - 预期结果：跳过 `doc-craftsman`，由 `docs-layout-quadrant` 执行生命周期管理。

3. **内容冗余精炼（非结构混杂）**：
   - 提示词：`specs 里的设计文档历史讨论过程太啰嗦了，帮我精简掉流水账`
   - 预期结果：跳过 `doc-craftsman`，由 `lean-docs` 专职处理（本技能只处理多主题结构混杂拆解）。

4. **文档瘦身与检索噪音阻断**：
   - 提示词：`/lean-docs 精简历史冗余文档并更新各 AI 工具的 ignore 配置以降低 Token 消耗`
   - 预期结果：跳过 `doc-craftsman`，由 `lean-docs` 专职处理。

5. **档位判定与索引全局重构**：
   - 提示词：`这次功能要不要写文档？帮我判定文档档位并重构 aidocs/INDEX.md 的全局索引`
   - 预期结果：跳过 `doc-craftsman`，档位判定与全局索引由 `docs-layout-quadrant` 执行。

6. **目录结构整理（非文档内容编写）**：
   - 提示词：`帮我整理一下混乱的 aidocs/ 目录结构，把散落的文档归类到五个子目录`
   - 预期结果：跳过 `doc-craftsman`，五维归类与结构整理由 `docs-layout-quadrant` 执行（本技能只做单篇文档内容契约与多主题拆解）。
