# 项目结构重构设计 (方案 A)

## 1. 目标与背景
本项目是一个自定义 AI 代理技能（Skills）的集合库，原有的技能目录（例如 `xy-feat/`, `uniapp-wechat-scaffold/` 等）直接扁平地存放在项目的根目录下。随着技能数量的增加，根目录包含了过多的技能文件夹，导致管理混乱、配置脚本分散，职责不清晰。
为了优化结构，我们决定将所有具体的技能子目录迁移至统一的容器目录 `skills/` 中。

## 2. 重构前后的目录结构对比

### 重构前
```
/
├── api-name-drift-defense/
├── docs-layout-quadrant/
├── health-probe-discipline/
├── self-check-trinity/
├── uniapp-wechat-scaffold/
├── unified-api-response/
├── xy-feat/
├── z-paging-best-practices/
├── docs/
├── dist/
├── pack.sh
├── README.md
└── CLAUDE.md
```

### 重构后
```
/
├── skills/                           # 统一存放技能的容器目录
│   ├── api-name-drift-defense/
│   ├── docs-layout-quadrant/
│   ├── health-probe-discipline/
│   ├── self-check-trinity/
│   ├── uniapp-wechat-scaffold/
│   ├── unified-api-response/
│   ├── xy-feat/
│   └── z-paging-best-practices/
├── docs/                             # 规范与设计文稿
│   ├── specs/                        # 本方案设计文档存放地
│   ├── plan/
│   ├── tracking/
│   └── guide/
├── dist/                             # 打包生成目录 (Git 忽略)
├── pack.sh                           # 兼容新路径的打包脚本
├── README.md                         # 更新安装与目录描述
└── CLAUDE.md                         # 调整 AI 助手路径说明
```

## 3. 具体修改设计

### 3.1 技能子目录文件移动
将以下 8 个技能子目录，整体移动至 `skills/` 文件夹下。
- `api-name-drift-defense`
- `docs-layout-quadrant`
- `health-probe-discipline`
- `self-check-trinity`
- `uniapp-wechat-scaffold`
- `unified-api-response`
- `xy-feat`
- `z-paging-best-practices`

如果目的文件夹 `skills` 不存在，需先创建之。

### 3.2 打包脚本 `pack.sh` 改造
需调整获取技能列表的逻辑。

**旧代码核心部分：**
```bash
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
...
for skill in "${SKILLS[@]}"; do
  ...
  ( cd "$SCRIPT_DIR/$skill" && zip -r -q "$DIST/${skill}.zip" . )
done
```

**新代码改造设计：**
```bash
SKILLS=()
# 自动扫描 skills 目录下的所有子目录
for dir in skills/*/; do
  # 确保有匹配的目录，防止无匹配时保留通配符
  [ -d "$dir" ] || continue
  dir="${dir%/}" # 移除尾部斜杠
  skill_name=$(basename "$dir")
  if [ -f "$dir/SKILL.md" ]; then
    SKILLS+=("$skill_name")
  fi
done
...
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

### 3.3 文档更新

#### 3.3.1 `README.md`
更新技能列表路径、克隆和指定技能位置章节，如：
- 从 `~/.config/opencode/skills/` 更新为 `~/.config/opencode/skills/skills/` （或者提示克隆到某个位置把子目录移过去）。
- 更新 `目录结构` 树，以体现引入了 `skills/` 父目录。
- 补齐 `z-paging-best-practices` 说明。

#### 3.3.2 `docs/guide/AI规则与技能统一编写规范.md`
- 更新所有涉及技能位置的路径。
- 如 `your-skill-name/` 指向变为 `skills/your-skill-name/`。

## 4. 交付与验证方式
1. 运行 `bash pack.sh`，验证是否在 `dist/` 下能成功产生 `api-name-drift-defense.zip` 等所有 8 个技能的压缩包。
2. 运行 `git status` 检查变动是否完全符合预期。
