---
name: jn-setup
description: "一次性配置 JN 使用 GitHub 或本地任务 tracker，成功后删除当前安装项。Use when the user explicitly asks to set up, configure, or switch the JN tracker."
---

# JN Setup

为当前仓库写入 `.jnative/issue-tracker.md`。这是显式的一次性配置，不由 `jn` 自动调用；配置成功后删除当前 `jn-setup` 安装项。

## 检查

读取现有配置、`git remote` 和 `.jnative/task/`。配置为 GitHub 或准备切到 GitHub 时，确认 remote 唯一且 GitHub 可访问；只做读取检查，不创建 Issue、label 或其他远端对象。

已有 GitHub JN Issues 或本地父需求时说明它们仍留在原 tracker。setup 只决定新需求的默认 tracker，不迁移、复制或同步已有记录。

## 选择

- 用户明确指定 `github` 或 `local` 时直接采用。
- 未指定时，唯一 GitHub remote 可访问则推荐 `github`，否则推荐 `local`，等用户确认后再写。
- GitHub 的 fallback 固定为 `local`，不提供其他选项。

## 写入

只创建或更新 `.jnative/issue-tracker.md`：

```markdown
tracker: github
fallback: local
```

`tracker` 只允许 `github` 或 `local`，`fallback` 只允许 `local`。保留文件中不冲突的用户说明；无关项目文件不改。

## 成功后自删除

重新读取配置，确认值和用户选择一致后，删除本次加载的 `jn-setup` 安装项。调用本 skill 即授权在成功后执行这一步，不再单独询问。

- 优先使用可用的 skill 卸载机制，只卸载 `jn-setup`。
- 若当前入口是符号链接，只删除该链接；若是 `.agents/skills/`、`.claude/skills/`、`.codex/skills/` 等安装目录中的副本，只删除该 `jn-setup` 目录。
- 当前路径是技能源码仓库而不是安装项时，不删除源码；报告没有可安全删除的安装项。
- 配置写入或验证失败时保留 skill，便于重试。
- 不删除 `.jnative/issue-tracker.md`、其他 skills 或已有 tracker 记录。

完成后报告配置路径、主 tracker、fallback 和自删除结果。以后要改配置时重新安装 `jn-setup`，或由用户直接编辑配置文件。
