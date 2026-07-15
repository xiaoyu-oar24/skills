---
name: health-probe-discipline
description: 强制执行服务 health/readiness/liveness 探针端点的规范约束。TRIGGER when: 新增或审查探针端点（/api/health、/api/live）、探针误报触发告警、发现探针端点被按 HTTP 动词拆分为多个文件。SKIP: 普通业务功能 API 接口。
version: "1.1.0"
author: xiaoyu
---

## 🎯 触发条件

- **TRIGGER when**:
  - 新增 `/api/health`、`/api/ready`、`/api/live` 等监控探针端点时
  - 代码审查时发现探针端点被拆分为多个文件（如 `health.get.ts` + `health.head.ts`）
  - K8s 等环境中探针误报导致容器重启，需要排查时
  - 框架升级后探针请求返回 404/405 时
- **SKIP**: 普通业务功能 API 接口

## ⚙️ 依赖环境

不限定具体框架。探针端点是云原生部署的通用需求。不同框架的 HTTP 方法限制方式：

| 框架 / 语言 | 方法限制方式 | 路由文件模式 |
|-------------|-------------|-------------|
| Nuxt / Nitro (h3) | `assertMethod(event, ['GET', 'HEAD'])` | `server/api/health.ts` |
| Express (Node) | `req.method` 判断或 middleware | `routes/health.js` |
| Spring Boot (Java) | `@RequestMapping(method = {GET, HEAD})` | `HealthController.java` |
| FastAPI (Python) | `@app.get()` / `@app.head()` 合并 | `routers/health.py` |
| Gin (Go) | `c.Request.Method` 判断 | `handler/health.go` |

## 📖 核心工作流

1. **检查探针是否拆分**: 使用 Glob 或 LS 工具扫描探针目录，检查是否存在按 HTTP 动词拆分的多个文件（如 `health.get.ts` + `health.head.ts` 并存）。不同框架的文件命名模式不同，但核心判断标准一致：同一个 URL 路径不应由多个文件分别处理不同 HTTP 方法

2. **合并为单一入口**: 如果有拆分的文件，使用 Edit 工具创建一个统一的探针入口文件（如 `health.ts`、`HealthController.java`、`health.py`），用一个处理函数/方法根据请求方法区分行为，然后用 Bash 执行 `rm` 删除多余的动词文件

3. **限制 HTTP 方法**: 在统一的探针端点中，显式声明只接受 `GET` 和 `HEAD` 请求，防止未授权的 POST/PUT/DELETE 等请求到达探针逻辑

4. **区分 GET 和 HEAD**:
   - **HEAD 请求**：部分负载均衡器 / CDN / 反向代理的健康检查使用 HEAD，属轻量级探测，返回 `null` 或空 body，仅验证服务存活（注意：Kubernetes 的 `httpGet` 探针只发 GET，不支持 HEAD）
   - **GET 请求**：容器编排探针（如 K8s liveness/readiness）、人工排查时使用，返回完整状态信息，如 `{ status: 'ok', uptime, timestamp }`

## ⛔ 行为限制与护栏

- **禁止按 HTTP 动词拆分文件**: 同一个探针 URL 的所有 HTTP 方法必须在单一文件中处理，禁止创建分别处理 GET/HEAD 的多个文件
- **必须限制 HTTP 方法**: 探针端点必须显式声明只接受 `GET` 和 `HEAD`，不允许任何其他 HTTP 方法
- **避免新增补丁文件**: 当需要支持新的 HTTP 方法时，在现有探针文件中追加逻辑，而不是新建补丁文件

## 📝 模板与范例

<Bad>

```ts
// 错误：按 HTTP 动词拆分文件，且未限制方法
// server/api/health.get.ts
export default defineEventHandler(() => {
  return { status: 'ok' }
})
```

</Bad>

<Good>

```ts
// Node (Nuxt/h3) 示例：统一入口，显式限制方法，区分 GET/HEAD
// server/api/health.ts
import { assertMethod, defineEventHandler } from 'h3'

export default defineEventHandler((event) => {
  assertMethod(event, ['GET', 'HEAD'])
  if (event.method === 'HEAD') return null
  return { status: 'ok', uptime: process.uptime(), timestamp: Date.now() }
})
```

```java
// Java (Spring Boot) 示例
@RestController
public class HealthController {
    @RequestMapping(value = "/api/health", method = {RequestMethod.GET, RequestMethod.HEAD})
    public ResponseEntity<Map<String, Object>> health(HttpServletRequest request) {
        if ("HEAD".equalsIgnoreCase(request.getMethod())) {
            return ResponseEntity.ok().build();
        }
        Map<String, Object> body = Map.of("status", "ok", "uptime", 
            ManagementFactory.getRuntimeMXBean().getUptime() / 1000.0);
        return ResponseEntity.ok(body);
    }
}
```

```python
# Python (FastAPI) 示例
from fastapi import APIRouter, Request
import time

router = APIRouter()
start_time = time.time()

@router.get("/api/health")
@router.head("/api/health")
async def health(request: Request):
    if request.method == "HEAD":
        return Response(status_code=200)
    return {"status": "ok", "uptime": time.time() - start_time, "timestamp": int(time.time())}
```

</Good>
