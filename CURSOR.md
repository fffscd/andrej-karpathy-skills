# Using this repo with Cursor

[English](./CURSOR.md) | [简体中文](./CURSOR.zh.md)

This project includes a **Cursor project rule** so the Karpathy-inspired behavioral guidelines apply automatically when you work here.

## In this repository

1. Open the folder in Cursor.
2. The rule [`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc) is committed with `alwaysApply: true`, so you do not need extra installation steps.
3. The Chinese rule [`.cursor/rules/karpathy-guidelines.zh.mdc`](.cursor/rules/karpathy-guidelines.zh.mdc) is available for reading or copying. It is not auto-applied by default, so this repo does not load duplicate equivalent rules.
4. In Cursor, you can confirm it under **Settings → Rules** (or the project rules UI), where `karpathy-guidelines` should appear.

## Use the same guidelines in another project

**Cursor (recommended):** Copy `.cursor/rules/karpathy-guidelines.mdc` or `.cursor/rules/karpathy-guidelines.zh.mdc` into that project’s `.cursor/rules/` directory (create the folders if needed). Adjust or merge with existing rules as you like.

**Other tools:** If a stack only supports a root instruction file, copy [`CLAUDE.md`](CLAUDE.md) or [`CLAUDE.zh.md`](CLAUDE.zh.md) into that project instead (or merge its contents into your existing instructions).

## Optional: personal Agent Skills

If you want the same content as a reusable skill under `~/.cursor/skills`, use [`skills/karpathy-guidelines/SKILL.md`](skills/karpathy-guidelines/SKILL.md) or [`skills/karpathy-guidelines-zh/SKILL.md`](skills/karpathy-guidelines-zh/SKILL.md). You can copy or symlink it into your personal skills directory; use whatever layout you use for other skills.

## Claude Code vs Cursor

- **Claude Code:** Install via the plugin marketplace and [`README.md`](README.md) instructions; the plugin exposes the English and Chinese skills from this repo. Per-project use can also rely on `CLAUDE.md` or `CLAUDE.zh.md`.
- **Cursor:** Use the committed `.cursor/rules/` file as described above. Cursor does not read `.claude-plugin/` or `CLAUDE.md` by default.

## For contributors

When you change the four principles, keep **[`CLAUDE.md`](CLAUDE.md)**, **[`CLAUDE.zh.md`](CLAUDE.zh.md)**, **[`.cursor/rules/karpathy-guidelines.mdc`](.cursor/rules/karpathy-guidelines.mdc)**, and **[`.cursor/rules/karpathy-guidelines.zh.mdc`](.cursor/rules/karpathy-guidelines.zh.mdc)** in sync. If the published skill/plugin text should match, update **[`skills/karpathy-guidelines/SKILL.md`](skills/karpathy-guidelines/SKILL.md)** and **[`skills/karpathy-guidelines-zh/SKILL.md`](skills/karpathy-guidelines-zh/SKILL.md)** as well.
