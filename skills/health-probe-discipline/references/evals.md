# health-probe-discipline 触发评测集

> 评估驱动开发：修改 `description` 或触发条件后，用以下提示词回归验证路由行为。

## 应触发（Expected：激活）

1. "给服务加一个 `/api/health` 健康检查端点"
2. "K8s 的 liveness 探针一直误报导致容器重启，帮我排查"
3. "为什么 `health.get.ts` 和 `health.head.ts` 要分成两个文件？帮我合并"

## 不应触发（Expected：跳过）

1. "写一个用户注册接口"（普通业务 API）
2. "帮我把所有业务接口的返回格式统一成 code-message-data"（属 unified-api-response 领域）
3. "这个登录接口报 500，帮我看看"
