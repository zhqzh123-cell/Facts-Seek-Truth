# doudou-perspective Skill 入库推送 Implementation Plan

## 仓库调研结论

**当前条件盘点：**

1. **Git 仓库 Facts-Seek-Truth**

   * 本地路径：`/workspace/`

   * 当前分支：`main`，与 `origin/main` 同步（working tree clean）

   * 远程地址：`https://github.com/zhqzh123-cell/Facts-Seek-Truth`

   * 仓库当前仅含 `README.md`（18 字节）

2. **doudou-perspective skill 目录**

   * 源路径：`/data/user/skills/doudou-perspective/`（⚠️ 位于 git 仓库之外，不在 `/workspace/` 内）

   * 文件结构：

     ```
     doudou-perspective/
     ├── SKILL.md                    (62 KB, 含 YAML front matter)
     ├── references/
     │   ├── research/               (6 个调研 md 文件)
     │   └── sources/transcripts/    (2 个一手语料 txt 文件)
     └── scripts/                    (空目录)
     ```

3. **SKILL.md YAML 头部语法初检（人工目测）**

   * 以 `---` 开头，第 10 行以 `---` 闭合，格式符合 YAML front matter 规范

   * 含 `name`（字符串）和 `description`（`|` 多行字面量块）两个字段

   * 缩进统一为 2 空格，description 每行缩进对齐，目测无语法错误

   * 待执行时用 Python 脚本（安装 pyyaml 后）做机器级校验确认

4. **敏感信息初检**

   * 用正则 `(api[_-]?key|secret|token|password|passwd|authorization|bearer|ghp_|github_pat_|sk-|xoxb|xoxp|xoxa|xoxs|AKIA|ASIA|AIza|private[_-]?key)` 对全目录 grep，结果：**No matches found（未检出）**

   * 执行时将再次交叉检查（含大小写变体及 `.env`/`.ini`/`.json` 等配置格式）

5. **关键前置条件（用户未提但必做）**

   * skill 目录在 `/data/user/skills/` 下，不在 git 工作区内

   * 必须先把整个 `doudou-perspective/` 目录复制到 `/workspace/` 根目录下，之后才能 `git add`

## 文件和模块

* `/workspace/doudou-perspective/`（新增目录，从 `/data/user/skills/doudou-perspective/` 复制）

  * `SKILL.md`：YAML 头部校验 + 敏感信息确认

  * `references/research/*.md`（6 个文件）：敏感信息确认

  * `references/sources/transcripts/*.txt`（2 个文件）：敏感信息确认

  * `scripts/`（空目录，保留）

## 实施步骤（依赖顺序）

### Step 1：复制 skill 目录进入 git 工作区

* 执行 `cp -a /data/user/skills/doudou-perspective /workspace/`

* 验证：`ls /workspace/doudou-perspective/` 显示 SKILL.md、references、scripts 齐全

### Step 2：机器级校验 SKILL.md YAML 头部语法

* 安装 pyyaml：`pip install pyyaml`（无交互）

* 运行 Python 脚本：

  * 抽取 `---` 之间的 front matter

  * `yaml.safe_load()` 解析，无异常即语法合法

  * 递归遍历解析结果中的 key/value，检查敏感字段名

### Step 3：全目录敏感信息二次扫描（交叉确认）

* 对 `/workspace/doudou-perspective/` 重新跑一次正则 grep（覆盖 Step 1 复制后的实际文件）

* 增加 `.env`、`*.ini`、`*.conf`、`config*.json` 等文件名关键词检测

* 预期结果：零命中

### Step 4：拉取远程 main 最新代码（防止远程有更新）

* 执行 `cd /workspace && git pull --rebase origin main`

* 若有冲突：

  * 检查冲突文件列表（`git status` 中 `both modified` 项）

  * 因本次只新增 `doudou-perspective/` 目录，不碰现有文件，理论上 README.md 单独修改可能冲突，若冲突则保留 `--theirs`（远程版 README），因为本次提交与 README 无关

  * `git add` 解决后继续 rebase：`git rebase --continue`

  * 若无法自动解决，停止并报告冲突详情给用户

### Step 5：git add 暂存整个目录

* `cd /workspace && git add doudou-perspective/`

* 验证：`git status` 显示 new file 列表包含该目录下所有文件

### Step 6：git commit 提交

* `cd /workspace && git commit -m "feat:新增skill doudou-perspective"`

* 验证：`git log -1 --oneline` 显示本次 commit 与 message 一致

### Step 7：推送到远程 main 分支

* `cd /workspace && git push origin main`

* 若 push 被拒绝（remote 有新提交），回到 Step 4 重新 pull → 处理 → 再 push

### Step 8：输出提交网页链接 + 提醒

* 获取最新 commit hash：`COMMIT=$(git rev-parse HEAD)`

* 拼接 GitHub 网页链接：`https://github.com/zhqzh123-cell/Facts-Seek-Truth/commit/$COMMIT`

* 输出两点：

  1. 本次提交的 GitHub 网页访问链接
  2. 提醒：GitHub 仅存储 skill 源码，如需在 Trae Work 使用，需要复制 raw 原始链接导入技能面板

## 依赖与注意事项

* **依赖**：需要当前运行环境能访问 `https://github.com`（push 用），以及 pip 能装 pyyaml（校验用）。若 pip 装包失败，改用纯 Python 手写 YAML front matter 语法检查（正则 + 缩进规则），不阻塞主流程

* **远程分支名**：确认是 `main`（仓库分支列表已核实为 `remotes/origin/main`），不是 `master`

* **目录路径**：复制源路径 `/data/user/skills/doudou-perspective` 末尾**不要加** `/`，`cp -a` 时才能正确复制目录本身（含目录名）到 `/workspace/`

* **空目录 scripts/**：git 默认不跟踪空目录，若 scripts/ 需要入库，需加 `.gitkeep`。用户说"整个目录加入暂存区"，但 git 行为决定空目录不会被追踪——这是客观条件限制。计划内不额外加 `.gitkeep`，除非用户明确要求；如果执行时发现 scripts/ 未被 add，在最终回复里如实说明

* **push 鉴权**：当前仓库 remote 是 `https://` 协议，之前 `git status` 显示与 origin 同步说明已有可用凭据（git credential helper 或环境变量）。若 push 时 403 失败，需要用户介入

## 验证点（执行完每步要检查）

| Step | 验证命令 / 方法                                                                                                         | 通过条件                |
| ---- | ----------------------------------------------------------------------------------------------------------------- | ------------------- |
| 1    | `ls /workspace/doudou-perspective/SKILL.md`                                                                       | 文件存在                |
| 2    | Python 脚本 exit code = 0 且打印 "YAML语法校验: 通过"                                                                        | 无 YAMLError，无敏感字段告警 |
| 3    | grep 输出行数 = 0                                                                                                     | 零匹配                 |
| 4    | `git status` 显示 up to date 或 rebase 成功完成                                                                          | 无未解决冲突              |
| 5    | `git status` 中 Changes to be committed 包含 `doudou-perspective/` 前缀的所有文件                                           | 无遗漏                 |
| 6    | `git log -1 --format=%s` == `feat:新增skill doudou-perspective`                                                     | message 精确匹配        |
| 7    | `git status` 显示 "Your branch is ahead of 'origin/main'." 消失；或 `git rev-parse HEAD` == `git rev-parse origin/main` | 本地与远程对齐             |
| 8    | 链接用 curl 返回 200（可选）；提醒文案原文输出                                                                                      | 链接可访问、信息完整          |

## 风险与处理

| 风险                           | 概率         | 处理方案                                                                                                                                                       |
| ---------------------------- | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| pip install pyyaml 失败（网络/权限） | 中          | 降级：手写解析脚本，仅校验 YAML front matter 的 `---` 闭合、`name`/`description` key 存在、多行块 `\|` 下每行缩进一致、无 Tab 字符缩进                                                         |
| git pull rebase 出现冲突         | 低（本次只新增目录） | 如果是 README.md 冲突，用 `git checkout --theirs README.md && git add README.md && git rebase --continue` 保留远程版本；如果是 doudou-perspective 内的冲突（概率极低），停止并人工输出冲突片段给用户 |
| git push 403 鉴权失败            | 中          | 输出"Push 失败：GitHub 凭据缺失/失效"，提示用户配置 token 或用 SSH 协议，不阻塞计划终止                                                                                                  |
| git push 被拒（remote 比本地新）     | 中          | 自动回到 Step 4 再 pull → rebase → commit（若需要）→ push，最多重试 2 轮                                                                                                   |
| 复制目录后文件权限问题导致 diff 大         | 低          | `cp -a` 保留权限；若 git 显示 mode 变化，执行 `git config core.filemode false` 忽略权限差异                                                                                   |

