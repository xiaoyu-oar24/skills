# Agent-Friendly 文档头规范

> 执行 docs-layout-quadrant 阶段 3、新建或修改 aidocs 功能文档时读取本文件。

## 最小模板

```markdown
---
status: draft        # draft | approved | delivered | archived
tier: L1             # L0 不落盘；L1 | L2 必填
domain: 网关代理转发   # 与文件名所属领域保持一致
updated_at: 2026-08-25
source_workflow: xy-feat  # xy-feat 或独立调用来源；无来源时可省略
---
```

## 状态语义

- `draft`：内容未批准。
- `approved`：设计或计划已获用户批准。
- `delivered`：代码交付且验证通过。
- `archived`：文档已归档；重命名时同步更新，禁止仅依赖目录位置推断生命周期。
