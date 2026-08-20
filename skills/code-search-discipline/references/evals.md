# code-search-discipline 触发评测集

> 评估驱动开发：修改 `description` 或触发条件后，用以下提示词回归验证路由行为。

## 应触发（Expected：激活）

1. "帮我搜一下 getUserInfo 在哪些文件里"（符号定义定位）
2. "这个报错文案 'Network timeout' 出现在源码哪一行"（字面量精确查找）
3. "登录流程是怎么实现的，调用了哪些服务"（大型源码语义理解）
4. "AuthService 被哪些地方调用了，修改它有什么影响"（调用关系与影响面分析）
5. "src 目录下有哪些 .vue 页面组件"（文件类型枚举）
6. "帮我查一下项目里 z-paging 组件都用在哪些页面了"（跨页面特定组件使用排查）
7. "用 grep 找一下 VITE_API_BASE 这个配置常量在哪定义"（配置键查找）
8. "全局搜索一下 payOrder 函数在哪些业务模块被调用"（跨模块广域搜索）

## 不应触发（Expected：跳过）

1. "帮我读一下 src/utils/format.ts 的内容"（已知路径单文件 → 直接 Read）
2. "这个项目的 aidocs 文档太多，帮我瘦身、降低 Token 消耗"（文档治理 → lean-docs）
3. "帮我开发一个新功能"（端到端工作流 → xy-feat）
4. "把 aidocs 目录按五维象限归类整理"（文档布局维护 → docs-layout-quadrant）
5. "帮我改一下 src/pages/index.vue 的按钮样式"（单文件局部修改，不涉及搜索）

