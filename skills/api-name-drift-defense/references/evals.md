# api-name-drift-defense 触发评测集

> 评估驱动开发：修改 `description` 或触发条件后，用以下提示词回归验证路由行为。

## 应触发（Expected：激活）

1. "升级 h3 之后 `readMultipartData is not a function`，帮我看看"
2. "编译报错 `Module has no exported member 'xxx'`，这个库昨天还好好的"
3. "刚把 sklearn 升到大版本，原来那行 import 挂了"

## 不应触发（Expected：跳过）

1. "这个折扣金额算错了，帮我调试业务逻辑"
2. "我把变量名拼错了，帮我全局改一下"
3. "帮我看看这个 SQL 为什么查不出数据"
