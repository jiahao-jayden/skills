# skills

[![skills.sh](https://skills.sh/b/jiahao-jayden/skills)](https://skills.sh/jiahao-jayden/skills)

个人 agent skills,用于 Claude Code 和 Codex。

## 流程:jn

用可配置的 GitHub 或本地 tracker 管理需求和实施：**Grill → PRD → 子任务与依赖 → Implement → Review**。一项任务也走同样的流程。

`/jn` 接受 GitHub Issue 或本地任务路径，读取需求、确认和进度，再恢复对应阶段。通过 `/jn-setup` 可把默认 tracker 配置为 `github` 或 `local`；没有配置时使用内置默认值，GitHub 不可用则回退本地。同一需求只维护一套正式状态。

| skill | 职责 | 单独调用 |
|---|---|---|
| `jn` | Tracker 定位、发布、确认、实施调度和整体验收 | 可主动触发 |
| `jn-setup` | 一次性配置 tracker，成功后自删除 | 显式入口 |
| `jn-grilling` | 分轮追问场景、边界和重要取舍 | 交付决定清单 |
| `jn-intent` | 生成带图的 RFC 格式 PRD，必须使用 `renhua` | 交付 PRD 内容 |
| `jn-plan` | 拆子任务草案、依赖和验收条件 | 交付任务内容 |
| `research` | 针对一个问题独立调研 | 交付笔记及适用的报告 |
| `renhua` | 中文写作编辑，PRD 使用项目文档模式 | 可独立编辑文章或文档 |

Research 可单独使用，也可以在 JN 任一阶段按需调用。研究完成后返回原阶段，不必走完需求流程。内容阶段单独调用不会自动发布 tracker 记录或启动实施。

### PRD

采用 [InnerSource Commons 中文 RFC 模板](https://patterns.innersourcecommons.org/zh/fu-lu/e-wai/rfc)，保留原章节顺序，补充本次范围、验收标准和确认记录。

PRD 必须有解释需求的图，默认使用 Mermaid：主流程图说明使用过程；涉及模块、调用顺序、生命周期或迁移时，补关系图、时序图、状态图或前后对照。图放在相关正文旁，验证渲染和文字一致性，不另建 HTML PRD。

每次生成或实质修改 PRD 都必须应用 `renhua` 项目文档模式，再复核图示、术语、风险和验收条件。`renhua` 缺失或图示未验证时保留草稿，不能声称完成。使用 JN 时须同时安装 `jn-grilling`、`jn-intent`、`jn-plan` 和 `renhua`；需要调研时再加载 `research`。

### Tracker 与状态

GitHub 模式把父 PRD、子任务、执行证据和状态写入 Issues、评论、关系和 labels。本地模式把打开的父 PRD 和编号任务写入 `.jnative/task/<feature>/`，使用 frontmatter 保存类型、阶段、开关状态和关闭原因，文件链接保存父子关系与依赖。local 父需求关闭后按当天日期移入 `.jnative/archive/YYYY-MM-DD/<feature>/`。

两种模式都维护 `.jnative/task/index.md` 和 `history.md` 作为轻量导航，只记录父需求标题、阶段或结果、tracker 链接、更新时间和旧本地资料路径，不复制正文或执行证据。日常恢复只读当前索引，查询历史时才读历史视图。

`/jn-setup` 写入并验证 `.jnative/issue-tracker.md`，再用父需求元数据初始化轻量视图；不创建任务或迁移旧记录，成功后删除当前安装项，源码仓库不删。普通 JN 不加载 setup；配置缺失时，有可写 GitHub remote 就使用 GitHub，否则使用本地。GitHub 在首次写入前确认不可用时自动切到本地；远端结果不明或已部分创建时不会双写。

GitHub 类型使用 `jn:prd`、`jn:task`；本地使用等价的 `jn_type`。打开时只保留一个阶段：`clarifying`、`awaiting-confirmation`、`ready`、`in-progress`、`blocked` 或 `review`。

GitHub 阶段用 label，本地阶段用 frontmatter；正文保留执行者、工作位置和阻塞原因。确认看父 PRD，依赖看原生关系或文件链接；ready 不能替代授权。一个子任务阻塞而其他任务仍可推进时，父需求不标 blocked。

日常流程只管理这些 JN 标签；清理旧标签须由用户明确要求，不删除其他项目的分类。不引入优先级、模块或负责人标签，实际负责人使用 assignee。

### 实施与验收

PRD 和任务拆分整体确认一次，确认跨会话有效。发布文档与实施代码分别遵循用户授权；已经明确授权就继续，不逐项重复确认。范围或重大风险改变时更新 PRD、图示及受影响任务，并记录新的确认。

每个任务使用独立执行上下文，单次 `/jn` 默认顺序循环。不同会话可以并行处理无依赖任务，开始前核对执行者、前置产物和工作目录冲突。没有独立上下文能力时提供新会话交接，不假装已经隔离。不引入 PR、stack、worktree、看板或自动分支管理。

任务检查通过、证据回写后关闭子任务；失败先修复，无法解决则保持打开。用户取消的任务单独记录，不算完成。所有任务处理后，对照父 PRD 做整体验收，通过后才关闭父需求。实际复盘完成后再更新 RFC 的回顾记录。

### 本地资料与旧需求

本地可保留显式 tracker 配置、轻量任务视图、local 模式的正式任务记录、按日期归档的历史记录、共享调研 `.jnative/research/<area>/<topic>.md` 和已有术语表。GitHub 模式不另存任务正文或状态；Issue 引用可访问的证据，不能只给本机文件路径。

旧 `.jnative/<feature>/` 文件保留，只有用户要求才迁入配置的 tracker。迁移后只维护目标 tracker，不继续同步旧文件。

## 调研:research

针对一个问题追到一手来源，默认产出带来源链接和原文摘录的 Markdown 笔记。Research 可独立使用，JN 任一阶段也可以按需调用；独立调研不自动创建需求或实施任务。

只有用户明确要求 HTML 或网页报告时，才额外生成 HTML。“调研报告”“讲清楚”“加图”、复杂流程或 GitHub 讨论都不自动触发网页。HTML 按用户的问题重新组织：先给答案和条件，再用实际场景解释过程，最后说明对项目的建议与限制。可以合并章节、筛选结论，不能省略会改变判断的反证；完整源码、摘录和检索记录留在 Markdown，不全文搬进折叠区。

HTML 默认使用内嵌 CSS，正文和图表可离线阅读。`check_note.py` 检查笔记结构和引证；`check_report.py --note` 检查报告来源能否追溯到笔记，不要求两者标题、顺序、表格或结论数量一致。报告还需检查实际场景是否讲清，并做桌面与窄屏预览；脚本通过不代表报告好懂。

JN 调用时笔记放在 `.jnative/research/<area>/<topic>.md`，由 PRD 或任务引用；独立调用遵循项目已有存放约定。GitHub 讨论仍区分维护者确认、复现案例和普通反馈，不把评论数量当作普遍结论。

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

`research` 的规模分级、派活要素和先宽后窄的检索策略来自 Anthropic 的 [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system);笔记结构、摘录式引证、来源覆盖表与版本钉死的做法来自自己既有的调研实践;HTML 的通俗解释思路曾参考 [dreambigou/eli5](https://github.com/dreambigou/eli5)。
