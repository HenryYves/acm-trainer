---
name: acm-config
description: This skill should be used when the user asks to "查看acm配置", "修改acm配置", "acm config", "acm设置", "改一下acm", or wants to view or modify existing ACM trainer settings. Shows current configuration and allows changing individual items without re-running full setup.
allowed-tools: Read, Write, AskUserQuestion
---

# ACM Trainer Config

View and modify ACM trainer configuration stored in `.claude/acm-trainer.local.md`.

## Step 1: Read Config

Read `.claude/acm-trainer.local.md`.

**Config does not exist** → say "尚未初始化。运行 `/acm-trainer:acm-setup` 完成首次配置。" Exit.

**Config exists** → parse YAML frontmatter. Show current values:

```
=== ACM Trainer 当前配置 ===
代码位置: <mode summary>
渐进式引导: <是/否>
术语风格: <pure_chinese/mixed>
题解语言: <zh/en/bilingual>
模板代码: <路径 or 无>
=========================
```

Then proceed to Step 2.

## Step 2: Choose What to Change

AskUserQuestion:
- header: "修改配置"
- question: "想修改哪些项？"
- multiSelect: true
- options:
  - "代码位置" — 重新选择代码存放位置
  - "引导方式" — 切换渐进式引导开关
  - "术语风格" — 切换纯中文 / 保留缩写
  - "题解语言" — 切换中文 / 英文 / 双语
  - "重新分析模板" — 重新指定模板文件并分析

If user selects nothing (or picks "Other" with empty/skip), say "没有改动。" and exit.

## Step 3: Apply Changes

For each selected item, ask the new value using AskUserQuestion (same options as in setup flow). Process one at a time.

## Step 4: Preview & Confirm

Show before/after diff:

```
=== 变更预览 ===
渐进式引导: false → true
术语风格: mixed → pure_chinese
===============
```

AskUserQuestion:
- header: "确认"
- question: "保存以上修改？"
- multiSelect: false
- options:
  - "确认保存"
  - "取消"

## Step 5: Write

Merge changes into the existing YAML frontmatter. Preserve the markdown body (template summary) unchanged unless "重新分析模板" was selected — in that case re-run the template analysis step from the setup skill.

Write with Write tool. Confirm: "配置已更新。"
