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
配置版本: <config_version>（最新: 0.2.4）
最后修改: <last_modified>
代码位置: <none | 单文件:path | 一题一文件:dir | 多文件:N个关键词>
渐进式引导: <是/否>
自动修改代码: <允许/不允许>
术语风格: <pure_chinese/mixed>
编程语言: <cpp/py/match_code>
模板代码: <路径 or 无>
变值常量: <无 or 常量名列表>
评测机速度: <1e8 / 5e7 / 3e7 / 2e8 / custom>
=========================
```

**Version check**: The latest config schema version is `0.2.4`. If the user's `config_version` is missing or older than `0.2.4`, warn: "检测到旧版配置文件（版本: <current>，最新: 0.2.4）。可运行 `/acm-trainer:acm-setup` 更新配置。" If `0.2.4` or newer, no warning.

Then proceed to Step 2.

## Step 2: Choose What to Change

Present options in 3 batches (AskUserQuestion max 4 options each). Collect all selections before proceeding to Step 3.

**Batch 1:**
AskUserQuestion:
- header: "修改配置 (1/3)"
- question: "想修改哪些项？"
- multiSelect: true
- options:
  - "代码位置" — 重新选择代码存放方式
  - "引导方式" — 切换渐进式引导开关
  - "自动修改代码" — 切换代码审查时直接修改文件
  - "术语风格" — 切换纯中文 / 保留英文缩写

**Batch 2:**
AskUserQuestion:
- header: "修改配置 (2/3)"
- question: "想修改哪些项？（继续）"
- multiSelect: true
- options:
  - "编程语言" — 切换 C++ / Python / 跟随当前代码
  - "评测机速度" — 调整复杂度分析的 Safe N 基准值
  - "权限配置" — 将项目目录加入 settings.local.json，避免 acm 读代码/配置时弹授权提示
  - "重新分析模板" — 重新指定模板文件并分析（含变值常量确认）

**Batch 3:**
AskUserQuestion:
- header: "修改配置 (3/3)"
- question: "模板相关？（继续）"
- multiSelect: true
- options:
  - "变值常量" — 修改每题需要调整的常量列表（仅在已配置模板时显示）
  - "以上都没有" — 不需要修改模板相关配置

If the user has no template configured (`has_template: false`), skip Batch 3 — 变值常量 requires a template.

After all batches, if the user selected nothing across all batches (or only "以上都没有"), say "没有改动。" and exit.

## Step 3: Apply Changes

For each selected item, ask the new value using AskUserQuestion. Use the same options as in the setup flow (acm-setup skill).

For "变值常量": present the full list of candidates from the template (re-scan the template file), let user toggle which ones vary. This replaces the existing `per_problem_constants` list.

For "自动修改代码": ask with the same question as acm-setup Step 4. Toggle `auto_edit_code`.

For "重新分析模板": re-run the full template analysis (Step 5 + Step 6 + Step 6a of acm-setup), including per-problem constant confirmation.

For "评测机速度": use the same options as acm-setup Step 9. Update `time_limit_baseline`.

For "权限配置": ask the user to confirm (same as acm-setup Step 13):

AskUserQuestion:
- header: "权限配置"
- question: "是否将项目目录加入 .claude/settings.local.json？之后 acm 读代码文件、读配置不会再弹出授权提示。（不影响其他项目）"
- multiSelect: false
- options:
  - "是，自动配置" — 自动把项目目录加入权限白名单
  - "不用了" — 跳过

If "是，自动配置":
1. Read `.claude/settings.local.json` in the project root (current working directory). If it doesn't exist, start with `{}`.
2. Merge the following into the existing JSON (preserve all existing settings, merge arrays without duplicates):

```json
{
  "permissions": {
    "allow": [
      "Bash(ls *)",
      "Bash(dir *)"
    ],
    "additionalDirectories": [
      "<current working directory with backslashes escaped>"
    ]
  }
}
```

3. Write the merged result back with the Write tool.
4. Confirm: "项目权限已配置。之后 acm 读代码、查配置不会弹授权了。"

If `.claude/settings.local.json` was newly created, also add it to `.gitignore` if one exists and the entry isn't there yet.

## Step 4: Preview & Confirm

Show before/after diff:

```
=== 变更预览 ===
渐进式引导: false → true
自动修改代码: false → true
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

Merge changes into the existing YAML frontmatter. Update `last_modified` to today's date (YYYY-MM-DD). Preserve the markdown body (template summary) unchanged unless "重新分析模板" or "变值常量" was selected — in that case update the relevant sections.

Write with Write tool. Confirm: "配置已更新。"
