---
name: unified-api-response
description: "强制所有 JSON API Handler 返回统一的 code-message-data 响应结构。TRIGGER when: 设计或审查 BFF / REST API、添加新的服务端路由处理器、发现已有 API 返回格式不一致、挂载全局错误兜底插件。SKIP: 流式响应或非 JSON 的 SSR HTML 路由。"
version: "1.1.0"
author: xiaoyu
---

> **生命周期阶段**：稳定

## 🎯 触发条件

- **TRIGGER when**: 设计或审查 REST API，添加新的服务端路由处理器，发现现有 API 返回格式不一致，或挂载全局错误兜底插件时。代码评审中发现用异常抛出来表达正常业务错误路径、或 e2e 测试中断言响应体格式时亦可触发
- **SKIP**: 流式响应（SSE / WebSocket / 文件下载）或非 JSON 响应（SSR HTML、静态资源）

## ⚙️ 依赖与先决条件

不限定具体框架。核心要求是项目实现了统一的 JSON 响应格式封装，可通过包配置文件确认项目技术栈。常见模式：

| 框架 / 语言 | 响应工具 | 错误处理 | 全局兜底 |
|-------------|----------|----------|----------|
| Nuxt / Nitro (h3) | `server/utils/api-response.ts` | `createError` | `server/plugins/error.ts` |
| Express (Node) | 自定义 middleware | `next(err)` | error-handling middleware |
| Spring Boot (Java) | `@RestControllerAdvice` + `ResponseEntity` | 自定义 Exception | `@ExceptionHandler` |
| Flask / FastAPI (Python) | 自定义 decorator / dependency | HTTPException | `@app.errorhandler` |
| Gin (Go) | 自定义 middleware | `c.AbortWithStatusJSON()` | Recovery middleware |

**验证命令**：
- Node.js 项目：运行 `cat package.json` 确定所含框架。
- Maven 项目：运行 `cat pom.xml` 或 `mvn dependency:tree` 检查。
- Python 项目：运行 `cat pyproject.toml` 或 `pip show` 确认依赖。

## 📖 标准工作流

### 阶段 0：识别当前项目的 API 响应模式

1. 在项目中搜索是否存在已有的响应封装工具（如 `api-response`、`Result`、`ApiResult` 等类/文件）
2. 如果项目已有封装，查看其方法与数据结构，沿用现有模式；如果尚未建立，则创建统一工具

### 阶段 1：改造处理函数

1. **读取**: 读取目标 API Handler 文件
2. **成功响应**: 将裸返回数据重构为统一成功响应格式，以所发现的现有工具/封装承载
3. **业务错误**: 将参数错误、未授权等业务异常重构为统一错误响应，并设置正确的 HTTP 状态码。禁止直接用异常抛出表达正常业务错误路径
4. **创建资源**: 优先设置 `201` 状态码后再返回统一响应

### 阶段 2：全局错误兜底

如果没有全局兜底，创建全局错误处理器：
- 捕获未预料的异常，返回统一格式的错误响应（`{ code: 500, message: "Internal Server Error", data: null }`）
- 仅对 API 路由生效，不影响 SSR 页面或静态资源

### 阶段 3：验证

在完成修改后，必须进行本地验证以保证安全：
1. 自动判定项目所用包管理器，执行对应的测试与检查指令确保通过。
2. 若本地支持 `self-check-trinity` 技能，通过技能调用执行其全部质量闸门验证。若该技能不可用，则执行以下内联验证：
   - 运行项目对应的 lint 命令，确保无风格/语法错误
   - 运行类型/编译检查，确保无类型错误
   - 运行测试命令，确保无回归问题

## ⛔ 行为护栏

- **必须统一**: 所有 JSON API Handler 必须返回统一格式的响应。成功和失败通过 HTTP 状态码区分
- **禁止直接抛错**: 禁止用异常抛出表达正常业务错误（如"用户不存在"、"参数无效"），应返回统一错误响应
- **排除流式响应**: WebSocket、SSE、文件下载等流式响应路径不能套统一响应外壳
- **全局兜底仅限 API**: 全局错误处理器只拦截 `/api/*` 或 `Accept: application/json` 请求，不影响页面渲染

## 📝 模板与范例

### <Bad>

```ts
// 错误：裸返回和利用全局异常做普通业务中断
// Nuxt/h3 示例
export default defineEventHandler((event) => {
  if (!user) throw createError({ statusCode: 400, message: 'invalid user' })
  return { success: true, data }
})
```

```java
// 错误：业务异常直接抛，响应格式不统一
// Spring Boot 示例
@GetMapping("/user/{id}")
public User getUser(@PathVariable Long id) {
    User user = userService.findById(id);
    if (user == null) throw new RuntimeException("用户不存在");
    return user;  // 直接返回裸对象
}
```

### <Good>

```ts
// 正确：套统一外壳并使用对应工具返回
// Nuxt/h3 示例
import { ok, err } from '../../utils/api-response'

export default defineEventHandler((event) => {
  if (!user) return err(event, 400, 'invalid user')
  setResponseStatus(event, 201)
  return ok(data)
})
```

```java
// 正确：统一响应封装
// Spring Boot 示例
@GetMapping("/user/{id}")
public ApiResult<User> getUser(@PathVariable Long id) {
    User user = userService.findById(id);
    if (user == null) return ApiResult.error(404, "用户不存在");
    return ApiResult.ok(user);
}
```

```python
# 正确：FastAPI 统一响应，通过 HTTP 状态码区分成功/失败
# Python 示例
from fastapi import Response

@app.get("/user/{id}")
async def get_user(id: int, response: Response):
    user = await user_service.find_by_id(id)
    if not user:
        response.status_code = 404
        return {"code": 404, "message": "用户不存在", "data": None}
    return {"code": 200, "message": "ok", "data": user}
```
