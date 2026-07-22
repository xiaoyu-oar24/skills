# uniapp-wechat-scaffold 触发评测集

> 评估驱动开发：修改 `description` 或触发条件后，用以下提示词回归验证路由行为。

## 应触发（Expected：激活）

1. `/uniapp-wechat-scaffold`
2. "初始化一个基于 uniapp 的微信小程序前端项目"
3. "新建一个空白的 uniapp 项目脚手架"

## 不应触发（Expected：跳过）

1. "在现有 uniapp 项目里加一个我的页面"（已有项目结构，属业务开发）
2. "帮我改一下这个项目的 `@/utils/request` 封装"
3. "这个 uniapp 页面的样式为什么错乱？"（现有项目排障）
