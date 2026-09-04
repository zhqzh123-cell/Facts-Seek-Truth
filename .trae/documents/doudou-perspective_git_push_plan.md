# doudou-perspective Skill 入库推送 Implementation Plan

## 仓库调研结论

**当前条件盘点：**

1. **Git 仓库 Facts-Seek-Truth**

   * 本地路径：`/workspace/`

   * 当前分支：`main`，与 `origin/main` 同步（working tree clean）

   * 远程地址：`https://github.com/zhqzh123-cell/Facts-Seek-Truth`

   * 仓库当前 `README.md` 内容仅一行 `# Facts-Seek-Truth`（18 字节，近乎空文件，需替换）

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

5. **关键前置条件**

   * skill 目录在 `/data/user/skills/` 下，不在 git 工作区内，必须先复制到 `/workspace/`

   * `README.md` 需要基于 SKILL.md 的核心内容完整重写，不能只写标题

## 文件和模块

* `/workspace/doudou-perspective/`（新增目录，从 `/data/user/skills/doudou-perspective/` 复制）

  * `SKILL.md`：YAML 头部校验 + 敏感信息确认

  * `references/research/*.md`（6 个文件）：敏感信息确认

  * `references/sources/transcripts/*.txt`（2 个文件）：敏感信息确认

  * `scripts/`（空目录，保留）

* `/workspace/README.md`（**覆盖重写**）：基于 skill 内容和场景功能的完整仓库说明文档

## 实施步骤（依赖顺序）

### Step 1：复制 skill 目录进入 git 工作区

* 执行 `cp -a /data/user/skills/doudou-perspective /workspace/`

* 验证：`ls /workspace/doudou-perspective/` 显示 SKILL.md、references、scripts 齐全

### Step 2：机器级校验 SKILL.md YAML 头部语法

* 安装 pyyaml：`pip install pyyaml --quiet`（无交互）

* 运行 Python 脚本：

  * 抽取 `---` 之间的 front matter

  * `yaml.safe_load()` 解析，无异常即语法合法

  * 递归遍历解析结果中的 key/value，检查敏感字段名

* 若 pyyaml 安装失败，降级为手写解析脚本（校验 `---` 闭合、`name`/`description` key 存在、缩进一致、无 Tab）

### Step 3：全目录敏感信息二次扫描（交叉确认）

* 对 `/workspace/doudou-perspective/` 重新跑一次正则 grep（覆盖复制后的实际文件）

* 增加 `.env`、`*.ini`、`*.conf`、`config*.json`、`id_rsa*`、`*.pem` 等文件名关键词检测

* 预期结果：零命中

### Step 4：基于 skill 内容撰写 README.md（用户新增要求，覆盖原有空文件）

**README.md 大纲（严格基于 SKILL.md 已有的内容提炼，不编造信息）：**

```markdown
# Facts-Seek-Truth

> 收录高质量 Trae Skills 的仓库。当前首个 Skill：**doudou-perspective**——以豆豆三部曲（《背叛》《遥远的救世主》《天幕红尘》）的思维框架作为视角，用于分析问题、审视决策、提供反馈。

---

## 📦 目录结构

```

Facts-Seek-Truth/
├── README.md                              # 本文件：仓库说明
└── doudou-perspective/                    # 豆豆思维操作系统 Skill
├── SKILL.md                           # Skill 主文件（含 YAML 头部 + 完整体系）
├── references/
│   ├── research/                      # 六维度深度调研文档
│   │   ├── 01-writings.md             # 著作分析
│   │   ├── 02-conversations.md        # 对话风格
│   │   ├── 03-expression-dna.md       # 表达DNA
│   │   ├── 04-external-views.md       # 外部评价
│   │   ├── 05-decisions.md            # 决策模式
│   │   └── 06-timeline.md             # 人物时间线
│   └── sources/transcripts/           # 一手语料（天幕红尘）
│       ├── 天幕红尘-200金句与核心思想.txt
│       └── 天幕红尘-叶子农全部对话场景.txt
└── scripts/                           # 脚本目录（预留）

```

---

## 🧠 doudou-perspective · 豆豆思维操作系统

> "见路不走就是实事求是，不住一法。"——叶子农《天幕红尘》

### 一句话简介
基于六维度深度调研及《天幕红尘》一手语料（200 金句 + 50 核心思想），提炼 9 个核心心智模型、18 条决策启发式和完整表达 DNA，以**宋一坤（狠辣算计）→ 丁元英（冷静智慧）→ 叶子农（条件法则）**三层递进构建认知操作系统。激活后默认以叶子农的全面视角回应。

### 三层觉悟体系

| 层次 | 主角 | 境界 | 核心特征 | 何时调用 |
|------|------|------|---------|---------|
| 第一层 | 宋一坤 | 执 / 有为 | 狠辣、算计、掌控——丛林法则，利用规则 | 利益博弈、竞争策略、商业算计 |
| 第二层 | 丁元英 | 退 / 顺势 | 冷静、通透、看透规律——文化属性，天道思维 | 规律洞察、文化分析、战略推演 |
| 第三层 | 叶子农 | 无 / 无为 | 如实、无执、承担因果——条件的可能，彻底实事求是 | 终极决策、条件判断、因果承担（**默认**） |

### 9 个核心心智模型
1. **文化属性** —— 命运底层代码：强势文化遵循规律，弱势文化等待救主
2. **见路不走** —— 条件的可能：不走别人的路也不刻意反着走，只走自身条件允许的路
3. **神即道，道法自然，如来** —— 神=道=事物规律=因果律，没有外在的救主
4. **因果观** —— 做了想做的，就受该受的，不能把因果链切断
5. **强势文化 vs 弱势文化** —— 靠自己 vs 等靠要
6. **如实观照** —— 按事物本来面目看问题，不加愿望和恐惧
7. **度** —— 凡事都有个度，过了度质就变
8. **场的世界** —— 众生是立场的、利益的、好恶的，出离立场的观点没有立足之地
9. **不住一法** —— 没有万能方法，方法随条件变化

### 18 条决策启发式（精选）
| # | 启发式 | 对应场景 |
|---|--------|---------|
| 1 | 看条件，不看路 | 面对选择时先盘点自身条件 |
| 2 | 忍人所不忍，能人所不能 | 生存竞争的定位 |
| 4 | 做了想做的，就受该受的 | 承担选择的后果 |
| 6 | 传统观念的死结在一个"靠"字 | 分析依赖心理 |
| 12 | 普通人改结果，明白人改条件 | 解决问题时改变前置条件 |
| 15 | 自知就是知道什么条件允许自己做 | 自我认知 |
| 16 | 大是大非，不能交给运气 | 原则性判断 |
| 17 | 当你一动就是损失的时候，不动就是最大的效益 | 面临行动困境 |
| 18 | 我不输出观点，我只解释因果 | 争议性话题 |

### 10 种对话思维技巧
先破后立 · 降维比喻 · 反问破题 · 接受标签翻转价值 · 实物演示 · 确认规则再行动 · 主动回避利益决策 · 果断拒绝 · 不拔高动机 · 条件变化即行为变化

### 8 个叶子农对话场景（快速匹配）

| 用户问题类型 | 匹配场景 | 核心思维 |
|-------------|---------|---------|
| 面临危机找方案 | 场景1 债务危机 | 厘清边界→找条件窗口→方案+兜底→正视法律 |
| 要不要走某条路 | 场景2 面馆创业 | 破经验执念→区分形式本质→盘点条件→提醒约束 |
| 面对争论/立场冲突 | 场景3 立场与因果 | 破客观神话→给替代标准→破阵营主义执念 |
| 面对理念输出/被拉拢 | 场景4 民主辩论 | 比喻降维→破普世执念→条件判断→用结果验证 |
| 面对逻辑陷阱/博弈 | 场景5 20万命题 | 确认规则→按确定性处理→用命题自身逻辑破题 |
| 面对生存困境/人生选择 | 场景6 戴梦岩 | 接受标签→破逃避→给生存哲学→定位自我 |
| 面对质疑/追问动机 | 场景7 被讯问 | 承认风险→不拔高→解释因果≠站队立场 |
| 需要落地方法 | 场景8 方迪之问 | 拆解条件→盘点→判断·不拿愿望替代现实 |

### 触发条件（何时自动激活此 Skill）
- 用户提到「用豆豆的视角」「丁元英会怎么看」「叶子农模式」「见路不走」「文化属性」
- 用户说「帮我用豆豆的角度想想」「如果丁元英会怎么做」「切换到叶子农」
- 涉及决策分析、利益博弈、规律洞察、条件判断、人生选择类问题

### 见路不走落地操作法（六步法）
1. **拆解必要条件** —— 这件事成立需要哪些必要条件？缺一个就不成立
2. **盘点自身条件** —— 你有什么资源、环境、性格、机遇？
3. **识别缺失条件** —— 缺什么？能不能补？补的代价？
4. **判断条件可能** —— 条件允许什么？允许就做，不允许就不贪
5. **不看路，只看条件** —— 不盲从老路，不刻意逆反，只服从因果
6. **改条件，不追结果** —— 你改变不了果，只能改变产生果的因

---

## 🚀 如何在 Trae Work 中使用

> **重要提醒**：GitHub 仅存储 Skill 源码，Trae Work 平台使用时需通过「原始链接（raw）」导入。

### 导入步骤
1. 打开本仓库 `doudou-perspective/SKILL.md` 文件页面
2. 点击「Raw」按钮获取原始文件的直链（URL 形如 `https://raw.githubusercontent.com/zhqzh123-cell/Facts-Seek-Truth/main/doudou-perspective/SKILL.md`）
3. 进入 **Trae Work → Skills 技能面板**
4. 选择「导入 Skill」→ 粘贴 raw 链接 → 确认导入
5. 触发词命中或手动启用即可使用

---

## ⚠️ 免责声明

本 Skill 基于豆豆三部曲（《背叛》2000、《遥远的救世主》2005、《天幕红尘》2013）公开作品及六维度调研提炼，存在以下边界：
- **非作者本人观点**：提炼的是作品中的思维框架，不代表豆豆本人立场
- **文学 vs 现实**：三部曲是文学创作，人物是理想化的，将小说哲学作为现实行动指南需谨慎
- **"文化属性"争议**：该概念被批评为"文化决定论"，使用时需补上制度、资源、历史起点等条件维度
- **精英主义倾向**：三部曲隐含精英主义立场，使用时需意识到此盲区
- **调研日期**：2026-09-04，此后变化未覆盖
- **引用来源**：招牌金句均附极简出处，可分辨「原话」与「框架推断」
```

* 撰写方式：直接用 Write 工具覆盖写入 `/workspace/README.md`

* 验证：`wc -l /workspace/README.md` 行数明显大于 1，且包含上述章节标题

### Step 5：拉取远程 main 最新代码（防止远程有更新）

* 执行 `cd /workspace && git pull --rebase origin main`

* 若有冲突：

  * 检查冲突文件列表（`git status` 中 `both modified` 项）

  * 若 `README.md` 冲突：**保留本地新写的版本**（用 `git checkout --ours README.md && git add README.md`），因为本次提交的 README 才是用户期望的最终版

  * 若 `doudou-perspective/` 内的文件冲突，停止并输出冲突详情给用户

  * 解决后 `git rebase --continue`

### Step 6：git add 暂存（skill 目录 + README.md）

* `cd /workspace && git add doudou-perspective/ README.md`

* 验证：`git status` 显示 new file / modified 列表包含 `doudou-perspective/` 前缀所有文件及 `README.md`

### Step 7：git commit 提交

* `cd /workspace && git commit -m "feat:新增skill doudou-perspective"`

* （注：README 重写属于本次 skill 新增的配套文档，归入同一 commit 合理）

* 验证：`git log -1 --format=%s` 精确等于 `feat:新增skill doudou-perspective`

### Step 8：推送到远程 main 分支

* `cd /workspace && git push origin main`

* 若 push 被拒绝（remote 有新提交），回到 Step 5 重新 pull → 处理冲突 → 再 push，最多重试 2 轮

### Step 9：输出提交网页链接 + 提醒

* 获取最新 commit hash：`COMMIT=$(git rev-parse HEAD)`

* 拼接 GitHub 网页链接：`https://github.com/zhqzh123-cell/Facts-Seek-Truth/commit/$COMMIT`

* 输出两点：

  1. **本次提交的 GitHub 网页访问链接**
  2. **提醒**：GitHub 仅存储 skill 源码，如需在 Trae Work 使用，需要复制 raw 原始链接导入技能面板

## 依赖与注意事项

* **依赖**：需要当前环境能访问 `https://github.com`（push 用），以及 pip 能装 pyyaml（校验用）

* **远程分支名**：`main`（仓库分支列表已核实），不是 `master`

* **复制路径**：`/data/user/skills/doudou-perspective` 末尾不要加 `/`，`cp -a` 才能复制目录本身

* **空目录 scripts/**：git 默认不跟踪空目录，若需入库需加 `.gitkeep`。计划内不加，执行时如实告知

* **push 鉴权**：当前 remote 是 HTTPS，之前 `git status` 显示同步说明已有可用凭据；若 403 失败需用户介入

* **README 内容来源**：严格基于 SKILL.md 已有的 9 心智模型、18 决策启发式、10 对话技巧、8 场景库、三层觉悟体系等内容，不添加 SKILL.md 中不存在的编造信息

## 验证点（执行完每步要检查）

| Step | 验证命令 / 方法                                                                                | 通过条件                |
| ---- | ---------------------------------------------------------------------------------------- | ------------------- |
| 1    | `ls /workspace/doudou-perspective/SKILL.md`                                              | 文件存在                |
| 2    | Python 脚本 exit code = 0 且打印 "YAML语法校验: 通过"                                               | 无 YAMLError，无敏感字段告警 |
| 3    | grep 命中行数 = 0                                                                            | 零匹配                 |
| 4    | `grep -c '见路不走\|三层觉悟\|心智模型\|叶子农对话场景\|Trae Work.*导入' /workspace/README.md` ≥ 5            | 关键章节齐全，内容非空         |
| 5    | `git status` 显示 up to date 或 rebase 成功完成                                                 | 无未解决冲突              |
| 6    | `git status` Changes to be committed 包含 `doudou-perspective/` 前缀 + `README.md`           | 无遗漏                 |
| 7    | `git log -1 --format=%s` == `feat:新增skill doudou-perspective`                            | message 精确匹配        |
| 8    | `git rev-parse HEAD` == `git rev-parse origin/main`                                      | 本地与远程对齐             |
| 9    | 链接格式正确（形如 `https://github.com/zhqzh123-cell/Facts-Seek-Truth/commit/<40位hash>`）；提醒文案原文输出 | 链接可访问、信息完整          |

## 风险与处理

| 风险                                 | 概率                        | 处理方案                                                                                                         |
| ---------------------------------- | ------------------------- | ------------------------------------------------------------------------------------------------------------ |
| pip install pyyaml 失败（网络/权限）       | 中                         | 降级：手写校验脚本，检查 `---` 闭合、`name`/`description` key 存在、`\|` 块缩进一致、无 Tab                                           |
| git pull rebase README.md 冲突       | 中-高（本次重写 README，远程可能也有变更） | **保留本地 ours 版本**，即新写的 README，用 `git checkout --ours README.md && git add README.md && git rebase --continue` |
| git pull doudou-perspective/ 内文件冲突 | 极低                        | 停止并输出冲突片段给用户人工裁决                                                                                             |
| git push 403 鉴权失败                  | 中                         | 输出"Push 失败：GitHub 凭据缺失/失效"，提示用户配置 token，不阻塞计划终止                                                              |
| git push 被拒（remote 比本地新）           | 中                         | 回到 Step 5 再 pull → rebase → push，最多自动重试 2 轮                                                                  |
| 复制后权限差异导致 mode diff                | 低                         | `git config core.filemode false` 忽略                                                                          |
| README 撰写遗漏关键章节                    | 低                         | 通过验证点 grep 关键词命中数 ≥ 5 把关；若不足则补写                                                                              |

