# audit-vue-biz-component 触发评测集

> 本文件仅在修改 `description` 或触发条件后按需读取，用于回归验证触发精度（见编写规范 §4.5）。

## 应触发（Expected：激活）

1. "帮我审计一下 src/components/BizForm.vue 这个组件"
2. "审查 components/table/ 下的业务组件"
3. "review 一下 src/components/UploadBiz.vue，检查 EP 透传是否规范"
4. "审计这个 Element Plus 二次封装组件，路径是 components/form/SelectWithLabel.vue"

## 不应触发（Expected：跳过）

1. "审计一下这个代码库有没有过度设计"（未指定组件路径，且为全仓审计 → `ponytail-audit`）
2. "帮我 review 一下这次的提交，准备合并"（代码审查收尾 → `requesting-code-review`）
3. "这个组件怎么用？"（无审计意图）
4. "审查一下 src/api/request.ts"（非 Vue 组件，不符合本技能审计对象）

## 回归清单

修改 description 或触发条件后，逐条确认上述用例的激活/跳过行为符合预期。
