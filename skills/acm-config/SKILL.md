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
代码位置: <none | 单文件:path | 一题一文件:dir | 多文件:N个关键词>
渐进式引导: <是/否>
术语风格: <pure_chinese/mixed>
编程语言: <cpp/py/match_code>
模板代码: <路径 or 无>
变值常量: <无 or 常量名列表>
评测机速度: <1e8 / 5e7 / 3e7 / 2e8 / custom>
=========================
```

Then proceed to Step 2.

## Step 2: Choose What to Change

AskUserQuestion:
- header: "修改配置"
- question: "想修改哪些项？"
- multiSelect: true
- options:
  - "代码位置" — 重新选择代码存放方式
  - "引导方式" — 切换渐进式引导开关
  - "术语风格" — 切换纯中文 / 保留缩写
  - "编程语言" — 切换 C++ / Python / 跟随当前代码
  - "重新分析模板" — 重新指定模板文件并分析（含变值常量确认）
  - "变值常量" — 修改每题需要调整的常量列表（仅在已配置模板时显示）
  - "评测机速度" — 调整复杂度分析的 Safe N 基准值

If user selects nothing (or picks "Other" with empty/skip), say "没有改动。" and exit.

## Step 3: Apply Changes

For each selected item, ask the new value using AskUserQuestion. Use the same options as in the setup flow (acm-setup skill).

For "变值常量": present the full list of candidates from the template (re-scan the template file), let user toggle which ones vary. This replaces the existing `per_problem_constants` list.

For "重新分析模板": re-run the full template analysis (Step 4 + Step 5 + Step 5a of acm-setup), including per-problem constant confirmation.

For "评测机速度": use the same options as acm-setup Step 8. Update `time_limit_baseline`.

## Step 4: Preview & Confirm

Show before/after diff:

```
=== 变更预览 ===
渐进式引导: false → true
术语风格: mixed → pure_chinese
编程语言: cpp → match_code
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

Merge changes into the existing YAML frontmatter. Preserve the markdown body (template summary) unchanged unless "重新分析模板" or "变值常量" was selected — in that case update the relevant sections.

Write with Write tool. Confirm: "配置已更新。"
