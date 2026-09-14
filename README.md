# skills

[![skills.sh](https://skills.sh/b/jiahao-jayden/skills)](https://skills.sh/jiahao-jayden/skills)

个人 agent skills,用于 Claude Code 和 Codex。

## 流程:jn

用 GitHub Issues 管理需求和实施：**Grill → PRD → 子 Issues 与依赖 → Implement → Review**。一项任务也走同样的流程。

`/jn` 接受父或子 Issue 链接、编号，读取需求、确认和进度，再恢复对应阶段。父 Issue 保存 RFC 格式 PRD；子 Issue 保存任务、执行信息和检查证据。GitHub 是任务状态的唯一来源，不再生成本地 intent/plan/todo/spec。

| skill | 职责 | 单独调用 |
|---|---|---|
| `jn` | Issue 定位、发布、确认、实施调度和整体验收 | 手动入口 |
| `jn-grilling` | 分轮追问场景、边界和重要取舍 | 交付决定清单 |
| `jn-intent` | 生成带图的 RFC 格式 PRD，必须使用 `renhua` | 交付 PRD 内容 |
| `jn-plan` | 拆子 Issue 草案、依赖和验收条件 | 交付任务内容 |
| `research` | 针对一个问题独立调研 | 交付笔记及适用的报告 |
| `renhua` | 中文写作编辑，PRD 使用项目文档模式 | 可独立编辑文章或文档 |

Research 可单独使用，也可以在 JN 任一阶段按需调用。研究完成后返回原阶段，不必走完需求流程。内容阶段单独调用不会自动发布 Issues 或启动实施。

### PRD

采用 [InnerSource Commons 中文 RFC 模板](https://patterns.innersourcecommons.org/zh/fu-lu/e-wai/rfc)，保留原章节顺序，补充本次范围、验收标准和确认记录。

PRD 必须有解释需求的图，默认使用 Mermaid：主流程图说明使用过程；涉及模块、调用顺序、生命周期或迁移时，补关系图、时序图、状态图或前后对照。图放在相关正文旁，验证渲染和文字一致性，不另建 HTML PRD。

每次生成或实质修改 PRD 都必须应用 `renhua` 项目文档模式，再复核图示、术语、风险和验收条件。`renhua` 缺失或图示未验证时保留草稿，不能声称完成。使用 JN 时须同时安装 `jn-grilling`、`jn-intent`、`jn-plan` 和 `renhua`；需要调研时再加载 `research`。

### 标签

类型使用 `jn:prd`、`jn:task`，每个 Issue 选一个。打开时只保留一个阶段：`jn:clarifying`、`jn:awaiting-confirmation`、`jn:ready`、`jn:in-progress`、`jn:blocked` 或 `jn:review`。

阶段只在 label 中维护，正文保留执行者、工作位置和阻塞原因。确认看父 PRD 的确认记录，依赖看原生关系；ready 标签不能替代授权。关闭时移除阶段标签、保留类型，不增加 done/cancelled 标签。一个子任务阻塞而其他任务仍可推进时，父需求不标 blocked。

日常流程只管理这些 JN 标签；清理旧标签须由用户明确要求，不删除其他项目的分类。不引入优先级、模块或负责人标签，实际负责人使用 assignee。

### 实施与验收

PRD 和任务拆分整体确认一次，确认跨会话有效。发布文档与实施代码分别遵循用户授权；已经明确授权就继续，不逐项重复确认。范围或重大风险改变时更新 PRD、图示及受影响任务，并记录新的确认。

每个任务使用独立执行上下文，单次 `/jn` 默认顺序循环。不同会话可以并行处理无依赖任务，开始前核对执行者、前置产物和工作目录冲突。没有独立上下文能力时提供新会话交接，不假装已经隔离。不引入 PR、stack、worktree、看板或自动分支管理。

任务检查通过、证据回写后关闭子 Issue；失败先修复，无法解决则保持打开。用户取消的任务单独记录，不算完成。所有任务处理后，对照父 PRD 做整体验收，通过后才关闭父 Issue。实际复盘完成后再更新 RFC 的回顾记录。

### 本地资料与旧需求

本地保留共享调研 `.jnative/research/<area>/<topic>.md` 和已有术语表，不另存任务状态。Issue 引用可访问的证据，不能只给本机文件路径。

旧 `.jnative/<feature>/` 文件保留，只有用户要求才迁成父子 Issues。迁移后以 GitHub 为准，不继续同步旧文件。GitHub 不可访问时交付草稿并报告限制，不声称已发布。

## 调研:research

针对一个问题追到一手来源，产出不离开笔记就能复核的 Markdown：每处引证都是链接加原文摘录（源码片段或文档原句），固定一张「来源覆盖」表交代官方文档、作者本人说法、同类方案、issue 讨论和历史演变各查到了什么。机制类问题要求具体走一遍的 trace 和失败模式，对比类要求反方证据，可行性类要求真跑并贴命令输出。需要讲清复杂关系、流程、时间变化、多方案比较或 GitHub issue 讨论时，会额外生成同名的 HTML 报告。HTML 给所有人看，包括不写代码的人，分两层：上层是解释层，每一节按「一句话说清是什么 → 日常类比 → 展开 → 对我们意味着什么」写，术语第一次出现就标出并链到页尾术语表；下层是证据层，把 Markdown 的全部章节、表格、摘录、链接折叠在对应章节下面。页面用 Tailwind CDN 排版，带目录，图放在它解释的章节里，图上的标签用人话。`check_note.py` 检查笔记的结构和摘录，`check_report.py --note` 对照 Markdown 检查 HTML 没有丢内容，并检查每节有人话解释、每个术语有定义。GitHub issue 会区分维护者确认、具体复现案例和普通用户反馈，不把评论数量或表情反应伪装成普遍结论。可独立于 `jn` 使用，JN 任一阶段遇到外部事实疑问时也可调用；走流程时笔记落到 `.jnative/research/<area>/<topic>.md`，由父 PRD 或相关任务引用。独立调用遵循目标项目已有的笔记存放约定，不要求创建 JN 需求。

四条核心规则:

- **按完成标准收敛，不按次数封顶。** 单点查证派 1 个 subagent;机制类、对比类、架构类按子系统或方案切分，并固定多派一个只查来源面(同类方案、作者说法、issue、历史)的 subagent。表里的工具调用数是下限，subagent 什么时候停看它负责的部分有没有达到完成标准
- **一手来源。** 第三方博客用来发现线索,不用来支撑结论。作者或维护者本人的博客、演讲、issue 回复算一手来源。搜索排名不等于可信度
- **链接加摘录。** 链接负责定位，摘录负责证明：源码引 5–15 行代码块，文档引原句。没有摘录的主张进不了结论，只能进「待验证」
- **把版本钉死。** 源码记 commit SHA,引用用带 SHA 和行号的 permalink;文档记版本号或访问日期。不钉版本,结论过几周就无法复核

产出固定三处:**结论在最前**(编号、自足),**来源覆盖在影响之前**(五行固定的表),**影响在最后**。中间按题目自己命名章节,数量随主题而定。多方案对比、逐维差距、能力矩阵用表格,维度做行、方案做列,单元格里直接嵌带 SHA 的证据链接。

## 写作:renhua

中文技术写作去 AI 味,覆盖公开写作(推文、文章)和项目文档(spec、plan、README、ADR)两种场景,格式规则分开。

中文专属壳和工作流为自有;通用规则与「写出人味」一节融合自 [cursor/plugins 的 unslop](https://github.com/cursor/plugins/blob/main/pstack/skills/unslop/SKILL.md)。

## 安装

```bash
npx skills@latest add jiahao-jayden/skills
```

支持 Claude Code、Codex、Cursor、Copilot 等二十余种 agent,安装时可选装哪几个 skill、装到哪些 agent。

常用参数:

| 参数 | 作用 |
|---|---|
| `-l` | 只列出仓库里有哪些 skill,不安装 |
| `-g` | 装到用户级而非项目级 |
| `-s <名字>` | 只装指定的几个(`*` 表示全部) |
| `-a <agent>` | 指定装到哪些 agent(`*` 表示全部) |
| `--copy` | 复制文件,而不是符号链接到 agent 目录 |

其他命令(`skills.sh` 的文档页只写了 `add`,以下来自 `npx skills --help`):

| 命令 | 作用 |
|---|---|
| `list` / `ls` | 列出已安装的 skill |
| `update [skills...]` | 更新到最新版(`-g` 只更新全局,`-p` 只更新项目) |
| `remove [skills]` | 卸载 |
| `find [query]` | 交互式搜索 skill(`--owner` 限定某个 GitHub 用户) |
| `use <pkg>@<skill>` | 不安装,只生成使用该 skill 的 prompt |
| `experimental_install` | 从 `skills-lock.json` 还原全部依赖 |
| `init [name]` | 新建一个 skill 骨架 |

安装信息记在项目的 `skills-lock.json` 里,带来源和内容哈希,可以随代码一起提交。

<details>
<summary>或者直接 clone 并自己做符号链接</summary>

```bash
git clone git@github.com:jiahao-jayden/skills.git ~/skills
for s in ~/skills/*/; do
  ln -sfn "$s" ~/.claude/skills/$(basename "$s")   # Claude Code
  ln -sfn "$s" ~/.agents/skills/$(basename "$s")   # Codex 及其他
done
```

符号链接指回仓库,`git pull` 即更新。要改这些 skill 就用这种方式。

</details>

## 致谢

流程设计参考了 [mattpocock/skills](https://github.com/mattpocock/skills) 的调用轴模型(user-invoked 与 model-invoked 的分工)、grilling 的 design tree / frontier 结构,PRD 结构采用 [InnerSource Commons RFC 模板](https://patterns.innersourcecommons.org/zh/fu-lu/e-wai/rfc)，工件链思路参考 Anthropic 的 [AI-native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook)。

`research` 的规模分级、派活要素和先宽后窄的检索策略来自 Anthropic 的 [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system);笔记结构、摘录式引证、来源覆盖表与版本钉死的做法来自自己既有的调研实践;HTML 解释层的「是什么 → 类比 → 展开 → 对你意味着什么」结构和零未定义术语的要求参考了 [dreambigou/eli5](https://github.com/dreambigou/eli5)。
