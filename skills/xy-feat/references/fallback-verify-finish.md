# xy-feat 降级流程：阶段 4（验证完成）与阶段 5（审查与收尾）

> 本文件仅在 superpowers 对应技能（`verification-before-completion` / `requesting-code-review` / `finishing-a-development-branch`）不可用时按需读取，正文为完整内联替代流程。三道质量闸门始终通过本地 `self-check-trinity` 技能执行，不依赖 superpowers。

## 阶段 4 内联流程：验证完成

### The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

在没有运行验证命令并看到输出之前，**禁止**声称"通过"、"完成"、"没问题"。

### 验证流程

1. 所有任务卡片在 `docs/tracking/<功能名>.md` 中标记为 `✅` 后，通过技能调用 `self-check-trinity`：
   - 自动识别项目技术栈，确定对应的 lint / typecheck / test 命令
   - 依次执行三道检查，任一步骤失败则必须修复后重试（最高 3 次）
   - 修复策略遵循 `references/fallback-tdd.md` 3.5 的调试流程

2. **证据优先原则**：
   - 每个验证命令必须**在当前消息中运行**，输出必须是新鲜的
   - 禁止使用"应该通过了"、"看起来没问题"等模糊表述
   - 禁止信任之前运行的缓存结果
   - 禁止只跑 lint 就声称"质量检查通过"（lint ≠ typecheck ≠ test）

3. 验证未通过 → 回到阶段 3，走调试流程修复后重新走阶段 4

4. 验证全部通过 → 进入阶段 5 (代码审查与收尾)

---

## 阶段 5 内联流程：代码审查与收尾

> **路径安全说明**：阶段 5 在文档归档前执行，此时 `docs/plan/<功能名>-plan.md` 与 `docs/tracking/<功能名>.md` 仍处于标准目录中，不会引发 FileNotFound 报错。

### 5.1 代码审查

1. 获取 git SHAs：
   ```bash
   BASE_SHA=$(git merge-base main HEAD 2>/dev/null || git merge-base master HEAD 2>/dev/null)
   HEAD_SHA=$(git rev-parse HEAD)
   ```

2. 派发子代理进行审查，提示词包含：
   - **DESCRIPTION**：本次功能简述
   - **PLAN_OR_REQUIREMENTS**：对应的 `docs/plan/<功能名>-plan.md` 与 `docs/specs/<功能名>-design.md` 文件路径
   - **BASE_SHA / HEAD_SHA**：变更范围
   - 要求输出：Strengths、Critical/Important/Minor Issues、Assessment

3. **审查结果处理**：
   - Critical：立即修复 → 回到阶段 3 → 重新走阶段 4 → 进入阶段 5
   - Important：修复后再继续
   - Minor：记录，可以后续处理
   - 若审查者判断有误，可以用技术理由反驳

4. 审查通过后进入收尾测试与分支处理。

### 5.2 收尾与测试去重

1. **Git Diff 测试去重检测**：
   ```bash
   # 检查自阶段 4 验证完成后是否有新的源码改动
   git diff --stat HEAD_SHA..HEAD
   ```
   - 若 diff 为空（阶段 4 验证后源码无改动）：跳过重复的全量测试运行，输出：“代码自阶段 4 校验后无变更，沿用阶段 4 Trinity 校验凭证”。
   - 若 diff 不为空：重新根据项目锁文件运行全量测试（如 `npm test`、`cargo test`、`pytest` 等）确认通过。未通过则停止并修复。

2. **检测环境**：
   ```bash
   git rev-parse --git-dir
   git rev-parse --git-common-dir
   ```

3. **呈现结构化选项**：

   ```
   所有任务完成，验证与代码审查通过。如何处理此分支？

   1. 合并回主分支（本地）
   2. 推送并创建 Pull Request
   3. 保持分支不动（我稍后自己处理）
   4. 放弃本次开发

   请选择：
   ```

4. **选项执行**：

   **选项 1（本地合并）**：
   ```bash
   git checkout <base-branch> && git pull
   git merge feat/<功能名>
   <运行测试确认合并无回归>
   ```
   合并成功后清理 worktree（如有）并删除分支

   **选项 2（创建 PR）**：
   ```bash
   git push -u origin feat/<功能名>
   ```
   不清理 worktree（用户需要迭代 PR feedback）

   **选项 3（保持）**：不做任何操作

   **选项 4（放弃）**：
   - 必须获得用户输入 `discard` 确认
   - 确认后清理 worktree（如有）并强制删除分支

5. **Git 写操作授权确认**：若阶段 2 未批量预授权后续写操作，选项执行前的写命令必须先经用户明确确认。审查收尾完成后，进入阶段 6 进行指南产出与文档整理归档。
