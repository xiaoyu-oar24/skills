# lean-docs 触发评测集

> 评估驱动开发：修改 `description` 或触发条件后，用以下提示词回归验证路由行为。

## 应触发（Expected：激活）

1. "/lean-docs"
2. "这个项目的 aidocs 文档越来越多，帮我瘦身、降低 Token 消耗"
3. "aidocs 里的 plan 和 tracking 堆了一大堆已完成的，帮我清理归档"
4. "specs 里的设计文档太啰嗦了，把历史讨论过程精简掉"
5. "重构一下 INDEX.md，做成紧凑版，别把归档文件全列出来"
6. "帮我把 aidocs/.archive 加到检索忽略里，别再被全局扫描扫到"

## 不应触发（Expected：跳过）

1. "帮我开发一个新功能"（→ xy-feat）
2. "aidocs 目录结构乱了，帮我按五维归类"（→ docs-layout-quadrant）
3. "这些文档命名有日期前缀，帮我规范一下"（→ docs-layout-quadrant 阶段 1）
4. "把 aidocs 目录结构说明注入到 AGENTS.md"（→ docs-layout-quadrant 阶段 7）
5. "帮我改一下 src/ 里的业务代码"（代码修改，非文档治理）
