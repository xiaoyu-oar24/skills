---
name: unicv
description: "生成基于 UniApp + Vue 3 + TypeScript + Pinia + uv-ui 的微信小程序项目脚手架。TRIGGER when: 用户要求'初始化前端项目'、'新建空白 uniapp 项目'、'创建一个微信小程序脚手架'或输入了 '/unicv' 时激活。SKIP: 用户正在讨论现有业务逻辑，或当前目录已存在完整的 uniapp 项目结构。"
version: "2026.05.28"
author: xiaoyu
---

# unicv

> 基于 UniApp 3.0 + Vue 3.5 + TypeScript + Pinia + uv-ui 的微信小程序项目模版标准操作手册（SOP）。
> 目标平台：微信小程序（mp-weixin），兼容 H5。
>
> **注意**: 本技能目录下的 `references/` 包含生成项目所需的全部模板文件，核心工作流中所有 `references/xxx.md` 路径均相对于本 SKILL.md 所在目录。

## 🎯 触发条件

- 当用户要求创建基于 Uniapp 或微信小程序的初始前端项目架构时
- 当用户输入 `/unicv` 或要求"初始化前端项目"、"新建空白 uniapp 项目"、"创建微信小程序脚手架"时

## ⚙️ 依赖环境

- Node 环境（推荐 `20.19.0`，最低 18）
- 全局安装的包管理工具 `pnpm`（失败时回退到 `npm`）
- 一个全新的空白项目目录

## 📖 核心工作流

严格按照以下顺序及工具执行项目生成，不可跳过或自行臆造代码：

1. **阶段 1：环境检查** —— 使用 LS 工具检查当前目录是否已有内容。若目录非空，使用 AskUserQuestion 工具向用户确认是否在当前目录下创建子目录。未得到确认前不允许动工
2. **阶段 2：读取参考库文件** —— 使用 Read 工具分批读取本技能目录下的参考文件：
   - 读取 `references/package-json.md` → 使用 Write 工具生成项目中的 `package.json`
   - 读取 `references/project-structure.md` → 使用 Bash 执行 `mkdir -p` 建立所需的目录骨架
3. **阶段 3：注入核心配置与模板源码** —— 使用 Read 工具继续读取：
   - 读取 `references/core-configs.md` → 提取配置内容
   - 读取 `references/build-scripts.md` → 提取构建脚本
   - 使用 Write 工具落地的配置文件：`vite.config.ts`、`tsconfig.json`、`.prettierrc`、各环境 `.env`、`src/main.ts`、`src/App.vue`、`src/config/index.ts`
   - 使用 Write 工具落地的构建文件：`build.mjs`、`build.lib.mjs`
   - 读取 `references/code-templates.md` → 使用 Write 工具将页面模板、组件、Store、`src/utils/request.ts` 等写入 `src/` 目录
4. **阶段 4：安装依赖与验证** —— 使用 Bash 依次执行：
   - `pnpm install` 安装依赖
   - `pnpm format` 格式化代码
   - 完成后告知用户项目搭建成功

## ⛔ 行为限制与护栏

- **冲突检查**: 禁止在已有源码的非空目录中直接覆盖同名文件。遇到冲突时立即暂停并向用户确认
- **技术栈锁定**: 只使用 TypeScript（`.ts` / `<script setup lang="ts">`），禁止生成 `.js` 文件（配置文件除外）。严格使用 Composition API
- **请求规范**: 所有后端通讯必须使用 `import { request } from '@/utils/request'`，禁止直接调用 `wx.request` 或 `uni.request`
- **组件选型**: 优先使用 `uv-ui` 组件库
- **Git 控制**: 禁止自动执行 `git init` 或 `git commit`，除非用户明确要求

## 📝 模板与范例

### <Bad> 违规操作示范

```typescript
// 错误1：未使用 TS 或 Composition API
export default {
  data() { return { test: 1 } }
}

// 错误2：原生直接请求
uni.request({ url: '/api/test' })
```

### <Good> 标准化组件与请求

```vue
<script setup lang="ts">
import { request } from '@/utils/request';

const fetchData = async (data: Record<string, unknown>) => {
  return await request({ url: '/api/module/action', method: 'POST', data });
};
</script>

<template>
  <uv-button @click="fetchData">获取数据</uv-button>
</template>
```
