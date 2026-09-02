# lean-docs 触发评测集

> 评估驱动开发：修改 `description` 或触发条件后，用以下提示词回归验证路由行为。

## 应触发（Expected：激活）

1. "/lean-docs"
2. "这个项目的 aidocs 文档越来越多，帮我瘦身、降低 Token 消耗"
3. "aidocs 里的 plan 和 tracking 堆了一大堆已完成的，帮我清理归档"
4. "specs 里的设计文档太啰嗦了，把历史讨论过程精简掉"
5. "重构一下 INDEX.md，做成紧凑版，别把归档文件全列出来"
6. "帮我把 aidocs/.archive 加到检索忽略里，别再被全局扫描扫到"
7. "umc 迁移的 8 个子特性都交付完成了，帮我把散碎的 design 和 guide 收敛成一个单一真理源"
8. "pay 领域多个子任务的文档太散了，帮我聚合收敛并归档历史过程文档"

## 不应触发（Expected：跳过）

1. "帮我开发一个新功能"（→ xy-feat）
2. "aidocs 目录结构乱了，帮我按五维归类"（→ docs-layout-quadrant）
3. "这些文档命名有日期前缀，帮我规范一下"（→ docs-layout-quadrant 阶段 1）
4. "把 aidocs 目录结构说明注入到 AGENTS.md"（→ docs-layout-quadrant 阶段 7）
5. "帮我改一下 src/ 里的业务代码"（代码修改，非文档治理）
6. "帮我新建一个 umc 子特性的设计文档"（→ xy-feat，新特性产出而非收敛治理）
7. "auth.md 里混了 15 个独立接口和设计说明，帮我拆成原子文件并建索引"（→ doc-craftsman，结构混杂拆解，非内容冗余精炼）

## 与 doc-craftsman 分流专项评测（v1.4）

1. "specs 里的设计文档历史讨论过程太啰嗦了，帮我精简掉流水账" → 应触发：内容冗余，本技能精炼。
2. "这个 design 文档既有啰嗦的历史讨论，又混了 5 个独立接口，帮我处理" → 应触发但先精炼剥离过程，剩余结构混杂转 `doc-craftsman` 拆解。
3. "umc 领域聚合后的 SSOT 里接口细节和接口文档子体系重复了" → 应删除 SSOT 中复制内容，改为链接到 `specs/接口文档/{domain}/INDEX.md`。
