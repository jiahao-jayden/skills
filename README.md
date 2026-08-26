# skills

[![skills.sh](https://skills.sh/b/jiahao-jayden/skills)](https://skills.sh/jiahao-jayden/skills)

个人 agent skills,用于 Claude Code 和 Codex。

## 流程:jn

一套需求流程,把想法走成可执行的 spec,工件落在项目的 `.jnative/` 目录。

只需要记住 `/jn`,它读 `.jnative/` 判断当前在哪一步,再交给对应阶段。

| skill | 调用 | 职责 |
|---|---|---|
| `jn` | 手动 | 流程入口。**唯一知道环境**:目录约定、定位特性、落盘、回写 |
| `jn-intent` | 自动 / 手动 | 把模糊的想法拷问成一份意图 |
| `jn-plan` | 自动 / 手动 | 拆成计划、todo 索引和一组可独立验证的 spec |
| `jn-grilling` | 自动 / 手动 | 提问原语:成批提问,每题带选项、代价和推荐理由 |

各阶段只产内容,不知道文件存哪。单独调用时产出留在对话里;要落盘走 `/jn`。

## 调研:research

针对一个问题追到一手来源,产出带引证、可复核的 markdown 笔记。独立于 `jn` 使用,`jn-grilling` 在事实落在仓库之外时也会调它。

三条核心规则:

- **按规模定投入。** 单点查证不派 subagent;对比方案派 2-4 个;摸清整套协议派 4 个以上。给简单问题派一堆 subagent 是这类任务最常见的浪费
- **一手来源。** 第三方博客用来发现线索,不用来支撑结论。搜索排名不等于可信度
- **把版本钉死。** 源码记 commit SHA,引用用带 SHA 和行号的 permalink;文档记版本号或访问日期。不钉版本,结论过几周就无法复核

产出只固定两头:**结论在最前**(编号、自足),**影响在最后**。中间按题目自己命名章节,数量随主题而定。多方案对比、逐维差距、能力矩阵用表格,维度做行、方案做列,单元格里直接嵌带 SHA 的证据链接。

### 工件

```
<repo>/.jnative/
├── CONTEXT.md                # 在做的特性引入的新术语
└── <slug>/
    ├── intent.md             # 问题、期望结果、影响范围、边界
    ├── plan.md               # 背景、方案、硬约束、验收计划
    ├── todo.md               # 索引,唯一进度真相源
    └── specs/NN-<slug>.md    # 可独立验证的单元,自带门禁
```

### 三个设计取舍

**门禁写在 spec 里,不写在 skill 里。** spec 一定会被读,skill 只在被想起时才加载。所以「动手前列出触及的 durable fact 及其 owner」「跑完验证才能标完成」这两道门,实体在 spec 模板中。

**硬约束从项目约定文件原文摘录进 plan 和 spec。** 规则写在 `AGENTS.md` 里 agent 照样犯,不是因为规则不存在,而是因为动手那一刻它不在上下文里。只摘本次真正触及的条目,全文复制等于没摘。

**todo.md 只当索引。** 详情在 spec 里。索引一旦开始存内容,读它的成本就和读全部 spec 一样高。

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

`research` 的规模分级、派活四要素和先宽后窄的检索策略来自 Anthropic 的 [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system);笔记结构与版本钉死的做法来自自己既有的调研实践。
