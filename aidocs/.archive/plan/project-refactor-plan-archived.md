# AI Skills 集合库项目结构重构执行计划

> ⚠️ **注**：本文中的 `unicv` 已于 2026-07-22 重命名为 `uniapp-wechat-scaffold`。

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将现有所有扁平的技能子目录迁移至统一的容器目录 `skills/`，并修改打包脚本与说明文档，以使项目结构更清晰合理。

**Architecture:** 物理层级迁移，通过 bash 脚本调整，同步更新打 zip 包匹配规则和静态文档引用。

**Tech Stack:** Bash shell, Markdown docs.

## Global Constraints
- 禁止使用破坏性的 git 控制命令操作，除非用户明确授权。
- 迁移后不得遗漏任何原有的技能文件，必须保留全部 8 个技能的完整性。
- 代码注释与中英文对齐，文档与现有格式保持一致。

---

### Task 1: 创建 skills 容器文件夹并迁移技能子目录

**Files:**
- Create: `skills/` (文件夹)
- Modify: 移动以下目录：
  - `api-name-drift-defense` -> `skills/api-name-drift-defense`
  - `docs-layout-quadrant` -> `skills/docs-layout-quadrant`
  - `health-probe-discipline` -> `skills/health-probe-discipline`
  - `self-check-trinity` -> `skills/self-check-trinity`
  - `unicv` -> `skills/unicv`
  - `unified-api-response` -> `skills/unified-api-response`
  - `xy-feat` -> `skills/xy-feat`
  - `z-paging-best-practices` -> `skills/z-paging-best-practices`

**Interfaces:**
- Consumes: 根目录下的 8 个旧技能目录。
- Produces: 迁移到 `skills/` 目录下的新位置。

- [x] **Step 1: 创建 skills 文件夹**
  - 执行: `mkdir -p skills`
- [x] **Step 2: 移动技能子目录**
  - 执行:
    ```bash
    mv api-name-drift-defense docs-layout-quadrant health-probe-discipline self-check-trinity unicv unified-api-response xy-feat z-paging-best-practices skills/
    ```
- [x] **Step 3: 检查迁移后的状态**
  - 执行: `ls -la skills/`
  - 预期: 必须打印出已迁移的 8 个子目录，且保证各自的 `SKILL.md` 存在。例如 `ls skills/xy-feat/SKILL.md` 确认存在。

---

### Task 2: 改造打包脚本 `pack.sh`

**Files:**
- Modify: `/Users/xiaoyu/Documents/code/skills/pack.sh`

**Interfaces:**
- Consumes: `skills/` 目录下的各技能子目录。
- Produces: `dist/` 下的独立 `zip` 包。

- [x] **Step 1: 修改扫描与打包代码**
  - 修改 `pack.sh` 的第 11-21 行，以及第 37-44 行。

  **原代码片段 1:**
  ```bash
  # 自动扫描当前目录下包含 SKILL.md 的所有直接子目录（排除 dist 和 docs 目录）
  SKILLS=()
  for dir in */; do
    dir="${dir%/}" # 移除尾部斜杠
    if [[ "$dir" =~ ^\..* ]] || [ "$dir" = "dist" ] || [ "$dir" = "docs" ]; then
      continue
    fi
    if [ -f "$dir/SKILL.md" ]; then
      SKILLS+=("$dir")
    fi
  done
  ```

  **替换为新代码片段 1:**
  ```bash
  # 自动扫描 skills/ 目录下包含 SKILL.md 的所有直接子目录
  SKILLS=()
  for dir in skills/*/; do
    [ -d "$dir" ] || continue
    dir="${dir%/}" # 移除尾部斜杠
    skill_name=$(basename "$dir")
    if [ -f "$dir/SKILL.md" ]; then
      SKILLS+=("$skill_name")
    fi
  done
  ```

  **原代码片段 2:**
  ```bash
  for skill in "${SKILLS[@]}"; do
    if [ ! -d "$SCRIPT_DIR/$skill" ]; then
      echo "⚠️  跳过：目录不存在 $skill"
      continue
    fi
    ( cd "$SCRIPT_DIR/$skill" && zip -r -q "$DIST/${skill}.zip" . )
    echo "✓ ${skill}.zip ($(du -h "$DIST/${skill}.zip" | cut -f1))"
  done
  ```

  **替换为新代码片段 2:**
  ```bash
  for skill in "${SKILLS[@]}"; do
    skill_dir="skills/$skill"
    if [ ! -d "$SCRIPT_DIR/$skill_dir" ]; then
      echo "⚠️  跳过：目录不存在 $skill_dir"
      continue
    fi
    ( cd "$SCRIPT_DIR/$skill_dir" && zip -r -q "$DIST/${skill}.zip" . )
    echo "✓ ${skill}.zip ($(du -h "$DIST/${skill}.zip" | cut -f1))"
  done
  ```

- [x] **Step 2: 验证打包**
  - 执行: `bash pack.sh`
  - 预期输出包括以下 8 个技能打包成功信息：
    ```
    打包以下技能到 dist/:
      - api-name-drift-defense
      - docs-layout-quadrant
      - health-probe-discipline
      - self-check-trinity
      - unicv
      - unified-api-response
      - xy-feat
      - z-paging-best-practices
    ```
    并且 `dist/` 下能生成 8 个体积有效的 `.zip` 文件。

---

### Task 3: 优化 README.md、CLAUDE.md 与规范文档

**Files:**
- Modify: `README.md`, `CLAUDE.md`, `docs/guide/AI规则与技能统一编写规范.md`

**Interfaces:**
- Consumes: 重构后的新路径名。
- Produces: 更新后的指南。

- [x] **Step 1: 更新 README.md 中的结构树与描述**
  - 将 `README.md` 的 `## 目录结构` 更新为以 `skills/` 作为各技能父目录的结构。
  - 调整 `Claude Code` 与 `opencode` 安装路径示例。
  - 补充 `z-paging-best-practices` 技能简介。
- [x] **Step 2: 更新 CLAUDE.md 的项目结构和开发规范提及路径**
  - 将 `CLAUDE.md` 内各技能路径（如 `xy-feat/`, `unicv/`）更新为 `skills/xy-feat/`, `skills/unicv/`。
- [x] **Step 3: 更新 docs/guide/AI规则与技能统一编写规范.md 中的提及路径**
  - 将所有涉及技能位置的路径从相对于根目录更改为相对于 `skills/` 目录。
