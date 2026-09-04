# skills

[![skills.sh](https://skills.sh/b/jiahao-jayden/skills)](https://skills.sh/jiahao-jayden/skills)

个人 agent skills,用于 Claude Code 和 Codex。

## 流程:jn

一套需求流程，把想法整理成可执行的工作项，文件保存在项目的 `.jnative/` 目录。

只需要记住 `/jn`,它读 `.jnative/` 判断当前在哪一步,再交给对应阶段。

| skill | 调用 | 职责 |
|---|---|---|
| `jn` | 手动 | 流程入口：找到当前需求、保存文件、更新进度 |
| `jn-intent` | 自动 / 手动 | 通过成批提问，把模糊想法整理成需求说明 |
| `jn-plan` | 自动 / 手动 | 拆成计划、工作清单和一组能分别检查的工作项（spec） |
| `jn-grilling` | 自动 / 手动 | 成批提问：每题给选项、各自会带来的结果和推荐理由 |

各阶段只产内容,不知道文件存哪。单独调用时产出留在对话里;要落盘走 `/jn`。

## 调研:research

针对一个问题追到一手来源，产出不离开笔记就能复核的 Markdown：每处引证都是链接加原文摘录（源码片段或文档原句），固定一张「来源覆盖」表交代官方文档、作者本人说法、同类方案、issue 讨论和历史演变各查到了什么。机制类问题要求具体走一遍的 trace 和失败模式，对比类要求反方证据，可行性类要求真跑并贴命令输出。需要讲清复杂关系、流程、时间变化、多方案比较或 GitHub issue 讨论时，会额外生成同名的 HTML 报告。HTML 给所有人看，包括不写代码的人，分两层：上层是解释层，每一节按「一句话说清是什么 → 日常类比 → 展开 → 对我们意味着什么」写，术语第一次出现就标出并链到页尾术语表；下层是证据层，把 Markdown 的全部章节、表格、摘录、链接折叠在对应章节下面。页面用 Tailwind CDN 排版，带目录，图放在它解释的章节里，图上的标签用人话。`check_note.py` 检查笔记的结构和摘录，`check_report.py --note` 对照 Markdown 检查 HTML 没有丢内容，并检查每节有人话解释、每个术语有定义。GitHub issue 会区分维护者确认、具体复现案例和普通用户反馈，不把评论数量或表情反应伪装成普遍结论。独立于 `jn` 使用，`jn-grilling` 在事实落在仓库之外时也会调它；走流程时笔记落到 `.jnative/research/`，并从需求说明或计划里链回去。

四条核心规则:

- **按完成标准收敛，不按次数封顶。** 单点查证派 1 个 subagent;机制类、对比类、架构类按子系统或方案切分，并固定多派一个只查来源面(同类方案、作者说法、issue、历史)的 subagent。表里的工具调用数是下限，subagent 什么时候停看它负责的部分有没有达到完成标准
- **一手来源。** 第三方博客用来发现线索,不用来支撑结论。作者或维护者本人的博客、演讲、issue 回复算一手来源。搜索排名不等于可信度
- **链接加摘录。** 链接负责定位，摘录负责证明：源码引 5–15 行代码块，文档引原句。没有摘录的主张进不了结论，只能进「待验证」
- **把版本钉死。** 源码记 commit SHA,引用用带 SHA 和行号的 permalink;文档记版本号或访问日期。不钉版本,结论过几周就无法复核

产出固定三处:**结论在最前**(编号、自足),**来源覆盖在影响之前**(五行固定的表),**影响在最后**。中间按题目自己命名章节,数量随主题而定。多方案对比、逐维差距、能力矩阵用表格,维度做行、方案做列,单元格里直接嵌带 SHA 的证据链接。

### 工件

```
<repo>/.jnative/
├── CONTEXT.md                # 项目术语表,共识对齐用
├── research/                 # 调研笔记,跨特性共用
├── archive/                  # 已归档的特性,定位时跳过
└── <slug>/
    ├── intent.md             # 问题、期望结果、影响范围、边界
    ├── plan.md               # 整体方案、关键选择和整套计划是否已确认
    ├── todo.md               # 工作清单，也是唯一的进度记录
    └── specs/NN-<slug>.md    # 每项能单独完成、单独检查的工作
```

### 三个设计取舍

**开始前确认和完成前检查写在 spec 里，不写在 skill 里。** 做一项工作时一定会读它的 spec，但 skill 不一定每次都会加载。spec 模板会要求先说清会长期保存哪些数据、哪个模块维护它们；完成前也必须跑完检查，才能标记为完成。

**把这次必须遵守的项目规则原文摘进 plan 和 spec。** 规则虽然写在 `AGENTS.md` 里，实施时也容易漏掉，因为那时它不一定在当前上下文。只摘这次真正会碰到的条目；整份复制进来等于没有重点。

**todo.md 只当工作清单。** 详情在 spec 里。工作清单一旦塞进太多内容，读它就和读完所有 spec 一样慢。

**plan.md 和 spec 各管一层。** plan 写跨多项工作的整体做法、已经确认的选择、完整的规则/风险和要运行的检查；spec 写某一项工作的范围、用户和调用方能看到的行为、完成前检查和局部选择。spec 里摘出的规则是方便实施时查看，原始内容仍以 plan 为准。

**整套计划只需要确认一次。** 计划完成后，小需求请用户查看 `intent.md + plan.md + todo.md`，大需求再加上全部 specs。`plan.md` 只维护一个「⏳ 等待确认 / ✅ 已确认 · 可执行」状态，不逐个确认工作项。用户回复“确认”“开始”或“继续”后，agent 就按整套计划连续做下去；每完成一项不再停下来重复确认。

**归档要两个条件同时成立**：所有工作都完成，而且用户主动要求归档。不会在需求做完时顺手归档，也不会主动提议，因为归档是把内容移出当前视野，应该由用户决定什么时候做。

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

流程设计参考了 [mattpocock/skills](https://github.com/mattpocock/skills) 的调用轴模型(user-invoked 与 model-invoked 的分工)、grilling 的 design tree / frontier 结构,以及 Anthropic 的 [AI-native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook) 的工件链思路。

`research` 的规模分级、派活要素和先宽后窄的检索策略来自 Anthropic 的 [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system);笔记结构、摘录式引证、来源覆盖表与版本钉死的做法来自自己既有的调研实践;HTML 解释层的「是什么 → 类比 → 展开 → 对你意味着什么」结构和零未定义术语的要求参考了 [dreambigou/eli5](https://github.com/dreambigou/eli5)。
