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
配置版本: <config_version>（最新配置格式: 0.2.6）
最后修改: <last_modified>
代码位置: <none | 单文件:path | 一题一文件:dir | 多文件:N个关键词>
渐进式引导: <是/否>
自动修改代码: <允许/不允许>
收录题解: <手动/自动>
术语风格: <pure_chinese/mixed>
编程语言: <cpp/py/match_code>
模板代码: <有/无>
变值常量: <无 or 常量名列表>
评测机速度: <1e8 / 5e7 / 3e7 / 2e8 / custom>
可执行文件: <无 / N个关键词→exe路径>
版本更新提醒: <是/否>
=========================
```

**Version check**: The latest config schema version is `0.2.6`.

If `config_version` is `"0.2.6"` or newer → no action, proceed to Step 2.

If `config_version` is missing or older than `"0.2.6"`:

1. Compare the parsed config against the complete field list (with defaults from acm-setup):

   | Field | Setup Step | Default |
   |-------|-----------|---------|
   | `code_location_mode` | Step 2 | `none` |
   | `code_paths` | Step 2 | `{}` |
   | `progressive_hints` | Step 3 | `true` |
   | `auto_edit_code` | Step 4 | `false` |
   | `auto_collect_solution` | Step 4b | `false` |
   | `terminology` | Step 7 | `mixed` |
   | `solution_language` | Step 8 | `cpp` |
   | `time_limit_baseline` | Step 9 | `100000000` |
   | `has_template` | Step 5 | `false` |
   | `remind_config_update` | — | `true` |

   (`template_boundary`, `template_entry`, `per_problem_constants` 只在 `has_template: true` 时有意义；如果 `has_template` 为 true 但这些字段缺失，引导用户用 Step 2 的"重新分析模板"补全，不在版本升级中处理。)

2. 找出用户配置中**缺失的字段**。忽略 `config_version`、`last_modified`（总是自动更新）。如果 `code_location_mode` 为 `none`，也忽略 `code_paths`。

3. **如果没有任何字段缺失**（只是版本号旧）：直接更新 `config_version` → `"0.2.6"`，`last_modified` → 今天日期。提示"配置内容已是最新，仅升级版本号。"然后进入 Step 2。

4. **如果有字段缺失**：列出缺失字段及用途。然后 AskUserQuestion：
   - header: "配置升级"
   - question: "检测到旧版配置（版本: <current>）。以下 N 个配置项尚未设置：[列出缺失字段名 + 一句话用途]。是否逐项配置？"
   - multiSelect: false
   - options:
     - "是，逐项配置" — 对每个缺失字段，用 acm-setup 对应步骤的问题引导选择（不重新走完整 setup，只问缺失的字段）
     - "跳过" — 保留当前配置不变，版本号不升级（下次仍会提示）

5. 如果选"是，逐项配置"：按 setup 步骤顺序，对每个缺失字段用对应的 AskUserQuestion 让用户选择。全部配置完后，写入配置文件：保留原有字段值 + 新字段值，`config_version` → `"0.2.6"`，`last_modified` → 今天日期。

6. 如果选"跳过"：不修改配置，继续 Step 2。

然后进入 Step 2。

## Step 2: Choose What to Change

Present all options in a **single** `AskUserQuestion` call with 3 questions. The user navigates between them with `←` `→` arrow keys and submits once. Collect all selections across all 3 questions before proceeding to Step 3.

**Question 1:**
- header: "修改配置 (1/3)"
- question: "想修改哪些项？"
- multiSelect: true
- options:
  - "代码位置" — 重新选择代码存放方式
  - "引导方式" — 切换渐进式引导开关
  - "自动修改代码" — 切换代码审查时直接修改文件
  - "术语风格" — 切换纯中文 / 保留英文缩写
  - "收录题解" — 切换手动/自动收录题解到本地知识库

**Question 2:**
- header: "修改配置 (2/3)"
- question: "想修改哪些项？"
- multiSelect: true
- options:
  - "编程语言" — 切换 C++ / Python / 跟随当前代码
  - "评测机速度" — 调整复杂度分析的 Safe N 基准值
  - "权限配置" — 将项目目录加入 settings.local.json，避免 acm 读代码/配置时弹授权提示
  - "重新分析模板" — 重新指定模板文件并分析（含变值常量确认）

**Question 3:**
- header: "修改配置 (3/3)"
- question: "想修改哪些项？"
- multiSelect: true
- options:
  - "版本更新提醒" — 切换是否在配置版本落后时提醒（当前: <remind_config_update>）
  - "可执行文件路径" — 配置/修改 C++ exe 路径，hack 生成后自动跑验证
  - "变值常量" — 修改每题需要调整的常量列表
  - "以上都没有" — 不需要修改

If the user has no template configured (`has_template: false`), hide "变值常量" option in Question 3. If that leaves Question 3 with only 3 options, keep it as-is (min 2 is satisfied).

**Important**: Send all 3 questions in a single `AskUserQuestion` call — not 3 separate calls. The user uses `←` `→` to flip between question pages, then submits everything at once.

After receiving answers, merge selections from all 3 questions. If the user selected "以上都没有" in Question 3, ignore all other selections and treat as "no changes". If nothing was selected across all 3 questions, say "没有改动。" and exit.

## Step 3: Apply Changes

For each selected item, ask the new value using AskUserQuestion. Use the same options as in the setup flow (acm-setup skill).

For "变值常量": present the full list of candidates from the template (re-scan the template file), let user toggle which ones vary. This replaces the existing `per_problem_constants` list.

For "自动修改代码": ask with the same question as acm-setup Step 4. Toggle `auto_edit_code`.

For "收录题解": ask with the same question as acm-setup Step 4b. Toggle `auto_collect_solution`.

For "重新分析模板": re-run the full template analysis (Step 5 + Step 6 + Step 6a of acm-setup), including per-problem constant confirmation.

For "可执行文件路径": use the same question flow as acm-setup Step 2b. Ask: 不配置 / 手动指定 / 尝试自动寻找. If manually specifying or re-running auto-find, use the current keywords from `code_paths`. Update `exe_paths`.

For "评测机速度": use the same options as acm-setup Step 9. Update `time_limit_baseline`.

For "版本更新提醒": AskUserQuestion — header: "版本提醒", question: "是否在配置版本落后时提醒更新？", options: "是，提醒我" (set `true`) / "不用提醒" (set `false`). Update `remind_config_update`.

For "权限配置": ask the user to confirm. Same logic as acm-setup Step 13 but also configurable here independently. Adds both the project directory and the plugin cache root directory (so acm skill can read its `references/*.md` files without prompts).

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
      "Bash(dir *)",
      "Glob(**/*)"
    ],
    "additionalDirectories": [
      "<current working directory with backslashes escaped>",
      "<plugin cache root with backslashes escaped>"
    ]
  }
}
```

The plugin cache root is the `acm-trainer` directory — go up 3 levels from this skill file's directory (`<version>/skills/acm-config/`) to reach it. For a typical install: `~/.claude/plugins/cache/Yves-plugin/acm-trainer/`.

3. Write the merged result back with the Write tool.
4. Confirm: "项目及插件权限已配置。之后 acm 读代码、读 reference 文件、查配置不会弹授权了。"

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
