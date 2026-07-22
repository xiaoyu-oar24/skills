# xy-feat 降级流程：阶段 4（验证完成）与阶段 6（审查收尾）

> 本文件仅在 superpowers 对应技能（`verification-before-completion` / `requesting-code-review` / `finishing-a-development-branch`）不可用时按需读取，正文为完整内联替代流程。三道质量闸门始终通过本地 `self-check-trinity` 技能执行，不依赖 superpowers。

## 阶段 4 内联流程：验证完成

### The Iron Law

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

在没有运行验证命令并看到输出之前，**禁止**声称"通过"、"完成"、"没问题"。

### 验证流程

1. 所有任务卡片标记为 `✅` 后，通过技能调用 `self-check-trinity`：
   - 自动识别项目技术栈，确定对应的 lint / typecheck / test 命令
   - 依次执行三道检查，任一步骤失败则必须修复后重试（最高 3 次）
   - 修复策略遵循 `references/fallback-tdd.md` 3.5 的调试流程

2. **证据优先原则**：
   - 每个验证命令必须**在当前消息中运行**，输出必须是新鲜的
   - 禁止使用"应该通过了"、"看起来没问题"等模糊表述
   - 禁止信任之前运行的缓存结果
   - 禁止只跑 lint 就声称"质量检查通过"（lint ≠ typecheck ≠ test）

3. 验证未通过 → 回到阶段 3，走调试流程修复后重新走阶段 4

4. 验证全部通过 → 进入阶段 5

### 常见反模式（禁止）

| 声称 | 实际需要 |
| :--- | :--- |
| "测试通过了" | 运行测试命令，输出显示 0 failures |
| "代码没毛病" | Lint + Typecheck + Test 三者全部通过 |
| "应该没问题" | Run the verification commands NOW |
| "之前跑过是好的" | 代码可能已变更，必须重新运行 |

---

## 阶段 6 内联流程：审查与收尾

### 6.1 代码审查

1. 获取 git SHAs：
   ```bash
   BASE_SHA=$(git merge-base main HEAD 2>/dev/null || git merge-base master HEAD 2>/dev/null)
   HEAD_SHA=$(git rev-parse HEAD)
   ```

2. 派发子代理进行审查，提示词包含：
   - **DESCRIPTION**：本次功能简述
   - **PLAN_OR_REQUIREMENTS**：对应的 plan/spec 文件路径
   - **BASE_SHA / HEAD_SHA**：变更范围
   - 要求输出：Strengths、Critical/Important/Minor Issues、Assessment

3. **审查结果处理**：
   - Critical：立即修复 → 回到阶段 3 → 重新走阶段 4 → 5 → 6
   - Important：修复后再继续
   - Minor：记录，可以后续处理
   - 若审查者判断有误，可以用技术理由反驳

4. 审查通过后进入收尾

### 6.2 收尾

1. **验证测试**（阶段 6 仍须重新确认）：
   根据项目锁文件判定最适合的测试指令（如 `npm test`、`cargo test`、`pytest` 或 `go test ./...`），通过 Bash 执行。测试未通过则停止，不允许进入收尾选项。

2. **检测环境**：
   ```bash
   git rev-parse --git-dir
   git rev-parse --git-common-dir
   ```

3. **呈现结构化选项**：

   ```
   所有任务完成，验证通过。如何处理此分支？

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

5. **Git 操作必须由用户明确授权**，禁止自动提交或推送
