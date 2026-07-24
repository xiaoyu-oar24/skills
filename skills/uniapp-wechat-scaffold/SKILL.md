---
name: uniapp-wechat-scaffold
description: "生成基于 UniApp + Vue 3 + TypeScript + Pinia + uv-ui 的微信小程序项目脚手架。TRIGGER when: 用户要求'初始化前端项目'、'新建空白 uniapp 项目'、'创建一个微信小程序脚手架'或输入了 '/uniapp-wechat-scaffold' 时激活。SKIP: 用户正在讨论现有业务逻辑，或当前目录已存在完整的 uniapp 项目结构。"
version: "1.0.2"
author: xiaoyu
---

# uniapp-wechat-scaffold

> **生命周期阶段**：稳定
> 基于 UniApp 3.0 + Vue 3.5 + TypeScript + Pinia + uv-ui 的微信小程序项目模版标准操作手册（SOP）。
> 目标平台：微信小程序（mp-weixin），兼容 H5。
>
> **注意**: 本技能目录下的 `references/` 包含生成项目所需的全部模板文件，核心工作流中所有 `references/xxx.md` 路径均相对于本 SKILL.md 所在目录。

## 🎯 触发条件

- **TRIGGER when**:
  - 用户要求创建基于 UniApp 或微信小程序的初始前端项目架构时
  - 输入 `/uniapp-wechat-scaffold` 或要求"初始化前端项目"、"新建空白 uniapp 项目"、"创建微信小程序脚手架"时
- **SKIP**: 用户正在讨论现有业务逻辑，或当前目录已存在完整的 uniapp 项目结构（避免误判与覆盖）。

## ⚙️ 依赖与先决条件

- Node 环境（`>= 18`，推荐 20.x LTS），可通过运行 `node -v` 检查。执行前先确认项目实际版本。
- 全局安装/项目可用的包管理工具 `pnpm`（失败时可以回退到 `npm`），可通过运行 `pnpm -v` 校验探测。
- 一个全新的空白项目目录。可通过运行 `ls -A` 确认是否为空目录。
- 运行环境假设为类 Unix 终端（示例命令使用 `ls`）。

## 📖 标准工作流

严格按照以下顺序及工具执行项目生成，不可跳过或自行臆造代码：

1. **阶段 1：环境检查** —— 检查当前目录是否已有内容。若目录非空，向用户确认是否在当前目录下创建子目录，未得到确认前不允许动工。同时确认本技能 `references/` 下 5 个模板文件（`package-json.md`、`project-structure.md`、`core-configs.md`、`build-scripts.md`、`code-templates.md`）齐全可读，缺失时立即中止并向用户报告。
2. **阶段 2：读取参考库文件** —— 分批读取本技能目录下的参考文件，读取后立即写入：
   - 读取 `references/package-json.md` → 生成项目中的 `package.json`。
   - 读取 `references/project-structure.md` → 执行 `mkdir -p` 建立所需的目录骨架。
3. **阶段 3：注入核心配置与模板源码** —— 继续读取 + 写入文件推进：
   - 读取 `references/core-configs.md` → 将配置文件写入对应磁盘路径：`vite.config.ts`、`tsconfig.json`、`.prettierrc`、各环境 `.env`、`src/main.ts`、`src/App.vue`、`src/config/index.ts`。
   - 读取 `references/build-scripts.md` → 写入构建文件：`build.mjs`、`build.lib.mjs`。
   - 读取 `references/code-templates.md` → 将页面模板、组件、Store、`src/utils/request.ts` 等写入 `src/` 目录。
4. **阶段 4：安装依赖与验证** —— 支持包管理器中立判定，优先根据锁文件（如无则在此处默认使用 `pnpm`）在命令行依次执行：
   - `[package-manager] install`（如 `pnpm install`）安装依赖。
   - `[package-manager] run format`（如 `pnpm format`）格式化代码。
   - 完成后告知用户项目搭建成功。

## ⛔ 行为护栏

- **冲突检查**: 禁止在已有源码的非空目录中直接覆盖同名文件。遇到冲突时立即暂停并向用户确认
- **技术栈锁定**: 只使用 TypeScript（`.ts` / `<script setup lang="ts">`），禁止生成 `.js` 文件（配置文件除外）。严格使用 Composition API
- **请求规范**: 所有后端通讯必须使用 `import { request } from '@/utils/request'`，禁止直接调用 `wx.request` 或 `uni.request`
- **组件选型**: 优先使用 `uv-ui` 组件库
- **Git 控制**: 禁止自动执行 `git init` 或 `git commit`，除非用户明确要求

## 📝 模板与范例

### <Bad>

```typescript
// 错误1：未使用 TS 或 Composition API
export default {
  data() { return { test: 1 } }
}

// 错误2：原生直接请求
uni.request({ url: '/api/test' })
```

### <Good>

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
