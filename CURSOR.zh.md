# 在 Cursor 中使用本仓库

[English](./CURSOR.md) | 简体中文

本项目包含 **Cursor 项目规则**，用于在本仓库工作时自动应用受 Karpathy 启发的行为指南。

## 在本仓库中

1. 在 Cursor 中打开此文件夹。
2. 英文规则 [`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc) 已提交，并设置了 `alwaysApply: true`，无需额外安装。
3. 中文规则 [`.cursor/rules/karpathy-guidelines.zh.mdc`](.cursor/rules/karpathy-guidelines.zh.mdc) 可用于查看或复制；它默认不自动应用，避免同时启用两套同义规则。
4. 在 Cursor 中，你可以通过 **Settings → Rules**（或项目规则界面）确认 `karpathy-guidelines` 是否出现。

## 在其他项目中使用相同指南

**Cursor（推荐）：** 将 `.cursor/rules/karpathy-guidelines.mdc` 或 `.cursor/rules/karpathy-guidelines.zh.mdc` 复制到目标项目的 `.cursor/rules/` 目录中（如果目录不存在，请创建）。可按需调整或与已有规则合并。

**其他工具：** 如果工具只支持根目录指令文件，可复制 [`CLAUDE.md`](CLAUDE.md) 或 [`CLAUDE.zh.md`](CLAUDE.zh.md) 到目标项目，也可以合并到现有指令中。

## 可选：个人 Agent Skills

如果希望把相同内容作为 `~/.cursor/skills` 下的可复用 skill，可以使用 [`skills/karpathy-guidelines/SKILL.md`](skills/karpathy-guidelines/SKILL.md) 或 [`skills/karpathy-guidelines-zh/SKILL.md`](skills/karpathy-guidelines-zh/SKILL.md)。可复制或建立符号链接，目录布局按你现有的 skill 管理方式处理。

## Claude Code 与 Cursor

- **Claude Code：** 通过插件市场和 [`README.md`](README.md) 中的说明安装；插件会公开本仓库中的英文和中文 skill。按项目使用时也可以依赖 `CLAUDE.md` 或 `CLAUDE.zh.md`。
- **Cursor：** 使用上文提到的 `.cursor/rules/` 文件。Cursor 默认不会读取 `.claude-plugin/` 或 `CLAUDE.md`。

## 给贡献者

修改四个原则时，请保持 **[`CLAUDE.md`](CLAUDE.md)**、**[`CLAUDE.zh.md`](CLAUDE.zh.md)**、**[`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc)** 和 **[`.cursor/rules/karpathy-guidelines.zh.mdc`](.cursor/rules/karpathy-guidelines.zh.mdc)** 同步。如果发布的 skill 或插件文本也需要一致，请同时更新 **[`skills/karpathy-guidelines/SKILL.md`](skills/karpathy-guidelines/SKILL.md)** 和 **[`skills/karpathy-guidelines-zh/SKILL.md`](skills/karpathy-guidelines-zh/SKILL.md)**。
