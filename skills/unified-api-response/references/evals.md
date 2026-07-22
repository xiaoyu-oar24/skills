# unified-api-response 触发评测集

> 评估驱动开发：修改 `description` 或触发条件后，用以下提示词回归验证路由行为。

## 应触发（Expected：激活）

1. "新增一个 `/api/users` 的 GET 接口"
2. "这几个 API 的返回格式不统一，帮我规整一下"
3. "给服务端加一个全局错误兜底，异常别直接抛 HTML 错误页"

## 不应触发（Expected：跳过）

1. "帮我写一个 SSE 实时推送接口"（流式响应豁免）
2. "这个 SSR 页面渲染出来的 HTML 不对"
3. "给服务加一个 `/api/health` 健康探针"（属 health-probe-discipline 领域，保持裸格式）
