# 验证证据与复用协议

> 阶段 4 首次验证前读取；阶段 5 修复后、阶段 7 最终核对时复用本协议。优先技能与降级流程共用此处规则，禁止另以提交区间判断测试是否过期。

## 1. 记录验证输入

- 在验证命令运行前、全部适用检查结束后各采集一次快照。输入一致且检查有效通过时，保存后快照、命令、工作目录、结果和环境说明作为证据。输入在检查期间变化时，先查明原因并重验受影响项，不能把后快照冒充已验证状态。
- 必须覆盖已跟踪文件的工作区内容、暂存区以及未跟踪文件内容，包含路径、删除、重命名和文件模式变化；只比较 `HEAD`、`git status` 或文件名列表不够。
- 同时记录实际运行时/包管理器版本、检查命令及配置、依赖安装状态。环境或外部服务状态变化、缓存污染、会话恢复后无法核实环境时，不复用旧结果。
- 快照只存路径与摘要，不保存文件正文、凭据或完整环境变量。放在系统临时目录，不写入仓库，不加入提交。Git 忽略的输入（如本地配置、生成源码）、子模块、外部符号链接目标不由下面示例完整覆盖；有这些依赖时使用项目已有可靠快照能力，否则重新验证，不声称已证明一致。

## 2. 可执行采集示例

在项目目录运行。需要 Python 3 与 Git；先用 `python3 --version`、`git --version` 检查。例子读取 Git 可见文件，输出临时证据路径与总摘要；大型仓库可使用覆盖相同输入的项目工具，不能为节省成本漏掉测试、配置或锁文件。

```bash
python3 - <<'PY'
import hashlib
import json
import os
import stat
import subprocess
import tempfile
from pathlib import Path

def git(*args):
    return subprocess.check_output(["git", *args])

root = Path(os.fsdecode(git("rev-parse", "--show-toplevel").rstrip(b"\n")))
os.chdir(root)
head = subprocess.run(["git", "rev-parse", "--verify", "HEAD"],
                      capture_output=True, check=False)
paths = sorted(set(git("ls-files", "-z", "--cached", "--others",
                       "--exclude-standard").split(b"\0")) - {b""})
files = {}
incomplete = []
for raw in paths:
    name = os.fsdecode(raw)
    path = root / name
    try:
        info = path.lstat()
    except FileNotFoundError:
        files[name] = {"kind": "deleted"}
        continue
    if stat.S_ISLNK(info.st_mode):
        data = os.fsencode(os.readlink(path))
        digest = hashlib.sha256(data).hexdigest()
        kind = "symlink"
        incomplete.append(name)  # 仅记录链接本身，未证明目标内容一致
    elif stat.S_ISREG(info.st_mode):
        hasher = hashlib.sha256()
        with path.open("rb") as stream:
            for chunk in iter(lambda: stream.read(1024 * 1024), b""):
                hasher.update(chunk)
        digest, kind = hasher.hexdigest(), "file"
    else:
        incomplete.append(name)  # 目录/gitlink 等需项目专用验证
        files[name] = {"kind": "unsupported"}
        continue
    files[name] = {"kind": kind, "mode": stat.S_IMODE(info.st_mode),
                   "sha256": digest}
snapshot = {
    "root": str(root),
    "head": head.stdout.decode().strip() if head.returncode == 0 else None,
    "index_sha256": hashlib.sha256(git("ls-files", "--stage", "-z")).hexdigest(),
    "files": files,
    "incomplete": incomplete,
}
payload = json.dumps(snapshot, sort_keys=True, ensure_ascii=True).encode()
with tempfile.NamedTemporaryFile(prefix="xy-feat-evidence-", suffix=".json",
                                 delete=False) as target:
    target.write(payload)
    evidence_path = target.name
print(json.dumps({"path": evidence_path,
                  "sha256": hashlib.sha256(payload).hexdigest(),
                  "complete_for_git_visible_files": not incomplete}))
PY
```

命令失败、采集期间有并发写入，或 `complete_for_git_visible_files` 为 false 时，这份快照不能用于直接放行。必须在工作区稳定时采集；必要时暂停本任务并行写入，不能停止用户的无关进程。

## 3. 收尾时的判定

1. 再采集当前快照，比较与验证证据关联的基线；证据不存在或无法读取时重新验证，不凭记忆补造。
2. 快照完全一致，且命令、环境、依赖和外部输入均可确认未变时，可复用该次检查结果，报告原命令与证据来源，不声称本轮重新运行过。
3. 快照不同：比较两个 JSON 中的 `files` 映射，按路径、类型、模式与内容摘要找出新增、修改、删除项，并核查暂存区和 HEAD 的变化。**新提交或仅重新暂存且实际验证内容未变**，经核实后可保留证据；有不同的待提交内容时不得把工作区通过等同于待提交树通过。
4. **仅说明性文档变化**且确认未参与构建、测试或运行行为时，只执行适用的链接/结构检查。不能仅凭目录或 `.md` 后缀判断：技能指令、MDX、生成输入、配置与锁文件都可能影响验证。
5. 源码、测试、配置、依赖、生成输入或运行环境发生变化时，重跑能覆盖影响的检查；无法确定影响范围时重跑全部适用检查。通过后重新建立匹配最终内容的证据。审查修复同样适用，不分 Critical/Important 豁免。

## 4. 提交与集成边界

- 验证前不强迫用户提交代码；未提交工作区同样可以有有效证据。
- 只暂存本任务已审查的内容；提交前核对暂存内容与已验证工作区一致，不能把工作区修复留在暂存区之外。部分暂存导致实际交付内容不同，须对预期提交树单独验证或调整暂存范围后重验。
- 阶段 6 的指南、索引、归档和规则更新都在最终交付清单中。阶段 3 可执行已授权的本地原子提交；最终交付提交/推送/合并留到阶段 7，之后不得再生成遗漏的交付文档。
- 合并或变基后的最终内容若不同于已验证内容，必须验证最终树；单纯更换提交 SHA 不代表内容变化，也不代表内容未变。
