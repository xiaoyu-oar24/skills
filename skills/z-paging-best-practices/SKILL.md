---
name: z-paging-best-practices
description: "z-paging 分页组件的最佳实践指南，强制使用 fixed 模式 + slot 插槽布局。TRIGGER when: 用户需要在 uni-app 项目中新建分页列表页面、实现下拉刷新/上滑加载、或排查 z-paging 相关布局问题（如导航栏遮挡、底部按钮越界）。SKIP: 非 uni-app 项目，或页面不涉及分页列表。"
version: "1.0.1"
author: xiaoyu
---

# z-paging 最佳实践

> **生命周期阶段**：稳定

本指南假设目标 uni-app 项目已通过 `uni_modules` 引入 z-paging (v2.8.6)，典型路径为 `src/uni_modules/z-paging/`（实际路径以项目为准）。

## 🎯 触发条件

- **TRIGGER when**:
  - 在 uni-app 项目中新建分页列表页面时
  - 需要实现下拉刷新、上滑加载更多功能时
  - 排查 z-paging 布局问题（导航栏遮挡列表、底部按钮越界、弹窗被覆盖等）
  - 将旧版手动计算高度的 z-paging 页面迁移到 fixed 模式时
- **SKIP**: 非 uni-app 项目，或页面不涉及分页列表

## ⚙️ 依赖与先决条件

- uni-app 项目（微信小程序 / H5）。可通过检查项目根目录下是否存在 `manifest.json` 文件（`ls manifest.json`）来确认。
- `z-paging` v2.8.6。可通过运行 `ls src/uni_modules/` 并查找 `z-paging` 目录，或检查锁文件依赖确认。
- `uv-ui` 组件库（用于 `uv-navbar` 等 UI 组件）。可通过运行 `ls src/uni_modules/` 并查找 `uv-ui` 确认。

## 📌 核心概念

z-paging 维护内部 scroll-view，通过 `@query` 回调自动计算 `pageNo` 和 `pageSize`。获取数据后调用 `paging.value.completeByTotal(list, total)` 或 `paging.value.complete(list)` 告知组件。

组件默认开启 `fixed` 模式，`position: fixed; top: 0; bottom: 0` 铺满屏幕，内部使用 flex 纵向布局。

## 📖 标准工作流

### 阶段 1：诊断 — 读取页面结构，确认分页场景

**使用工具**：读取文件

读取目标页面文件，理解现有布局结构，确认是否满足分页列表场景。关注以下关键点：
- 当前是否已使用 z-paging，是否在 fixed 模式
- 导航栏和底部按钮的定位方式
- 是否存在手动高度计算、`paging-style` 等反模式代码

### 阶段 2：执行 — 采用 fixed 模式 + slot 插槽布局

**使用工具**：编辑文件

这是 z-paging 源码设计的原生用法，代码最简洁，无手动高度计算。编写或重构布局，确保以下结构：

**模板结构**:

```vue
<template>
  <view class="page-container">
    <z-paging
      ref="paging"
      v-model="dataList"
      :fixed="true"
      :safe-area-inset-bottom="true"
      :empty-view-text="emptyView.text"
      @onRefresh="onRefresh"
      @query="queryList"
    >
      <!-- 导航栏放入 slot="top"，使用 :fixed="false" -->
      <template #top>
        <uv-navbar :fixed="false" leftText="返回" title="页面标题" />
      </template>

      <!-- 列表内容：滚动区域 -->
      <view v-for="item in dataList" :key="item.id">
        ...
      </view>

      <!-- 底部固定按钮放入 slot="bottom" -->
      <template #bottom>
        <view class="footer-btn">
          <uv-button text="操作按钮" @click="handleClick" />
        </view>
      </template>
    </z-paging>

    <!-- 弹窗等浮层放在 z-paging 外面 -->
    <uv-popup ref="popup" mode="center">
      ...
    </uv-popup>
  </view>
</template>
```

**脚本要点**:

```ts
import { ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'

const paging = ref(null)  // z-paging 实例引用

// 空数据提示
const emptyView = { text: '暂无数据' }

// 下拉刷新
function onRefresh() {
  getListData(1)
}

// 查询回调：pageNo/pageSize 由 z-paging 自动计算
function queryList(pageNo: number, pageSize: number) {
  getListData(pageNo)
}

async function getListData(pageNo: number) {
  try {
    // 请求方式遵循项目自有封装（如 unicv 脚手架的 @/utils/request），此处仅示意
    const res = await proxy.$http.post(api.xxx, {
      pageInfo: { pageNum: pageNo, pageSize: 10 }
    })
    if (res.success) {
      // 关键：告知 z-paging 数据及总数
      paging.value.completeByTotal(res.data, res.total)
    } else {
      paging.value.complete([])
    }
  } catch {
    paging.value.complete([])
  }
}

// 增删后刷新列表
function reloadList() {
  paging.value.reload()
}
```

### 阶段 3：理解 — 布局原理

z-paging `fixed` 模式 CSS 类 `.z-paging-content-fixed`：`position: fixed; top: 0; left: 0; bottom: 0; right: 0;`，内部 flex 纵向布局：

```
┌── .z-paging-content-fixed ───────────────────┐
│  flex-direction: column                       │
│  ┌──────────────────────────────────────────┐ │
│  │ slot="top" → uv-navbar (:fixed="false")  │ │
│  │  (uv-navbar 非 fixed 模式 = 状态栏+44px) │ │
│  └──────────────────────────────────────────┘ │
│  ┌──────────────────────────────────────────┐ │
│  │ .zp-scroll-view-super  (flex: 1)         │ │
│  │   自动填充剩余高度，scroll-view 滚动     │ │
│  └──────────────────────────────────────────┘ │
│  ┌──────────────────────────────────────────┐ │
│  │ slot="bottom" → 底部按钮                 │ │
│  └──────────────────────────────────────────┘ │
└──────────────────────────────────────────────┘
```

**关键**：`slot="top"` / `slot="bottom"` 内的元素会作为 flex 子项参与布局，自动占据空间，scroll-view 的 `flex: 1` 填充剩余区域。无需任何手动高度计算。

## ⛔ 行为护栏

- **禁止在 `slot="top"` 中使用 `position: fixed`** — z-paging 文档明确要求，会导致布局错乱
- **禁止在 `slot="bottom"` 中使用 `position: fixed`** — 同上
- **禁止手动计算 navBarHeight** — uv-navbar `:fixed="false"` 时自身包含状态栏 + 44px 内容高度，由 flex 自动处理
- **禁止设置 `:paging-style`** — fixed 模式下 z-paging 自动处理布局，手动设置会破坏内部计算
- **弹窗必须放在 `<z-paging>` 标签外部** — 否则 uv-popup 的 z-index 无法正确生效
- **底部不需要占位元素** — `slot="bottom"` 在 scroll-view 外部，不会遮挡列表内容，不需要 `.bottom-placeholder`

## 📝 模板与范例

### <Bad>

```vue
<!-- 错误：手动计算高度，代码冗余 -->
<template>
  <view>
    <uv-navbar :fixed="true" leftText="返回" title="标题" />
    <view :style="`height:${navBarHeight}px;`" />

    <z-paging
      ref="paging"
      v-model="dataList"
      :paging-style="styleZPage"
      @onRefresh="onRefresh"
      @query="queryList"
    >
      <view class="list-content">...</view>
    </z-paging>

    <view class="footer-btn" style="position: fixed; bottom: 0">...</view>
  </view>
</template>
```

### <Good>

```vue
<!-- 正确：fixed + slot 插槽 -->
<template>
  <view class="page-container">
    <z-paging
      ref="paging"
      v-model="dataList"
      :fixed="true"
      :safe-area-inset-bottom="true"
      @onRefresh="onRefresh"
      @query="queryList"
    >
      <template #top>
        <uv-navbar :fixed="false" leftText="返回" title="页面标题" />
      </template>

      <view v-for="item in dataList" :key="item.id">...</view>

      <template #bottom>
        <view class="footer-btn">
          <uv-button text="操作按钮" @click="handleClick" />
        </view>
      </template>
    </z-paging>

    <uv-popup ref="popup" mode="center">...</uv-popup>
  </view>
</template>
```

---

## 常见问题排查

### 导航栏遮挡列表内容

- **fixed 模式**：检查 `#top` 中的 navbar 是否遗漏 `:fixed="false"`
- **非 fixed 模式（旧方案）**：检查 `pagingStyle.paddingTop` 是否等于 `navBarHeight`（`44 + statusBarHeight`）

### 下拉刷新/上滑加载不触发

- 检查 `paging.value.completeByTotal()` 是否被正确调用
- 数据为空时改用 `paging.value.complete([])`，不会触发加载更多
- `:auto="false"` 会阻止初始加载

### 弹窗/浮层被遮挡

- 弹窗放在 `<z-paging>` 标签**外部**（如 `<uv-popup>` 的 z-index 才能正确生效）

### 底部安全区域

- 使用 `:safe-area-inset-bottom="true"` 让 z-paging 自动处理
