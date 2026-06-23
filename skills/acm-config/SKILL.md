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
配置版本: <config_version>（最新: 0.2.14）
最后修改: <last_modified>
代码位置: <none | 单文件:path | 一题一文件:dir | 多文件:N个关键词>
渐进式引导: <是/否>
自动修改代码: <允许/不允许>
收录题解: <手动/自动> 权限: <已配置/未配置>
Markdown输出: <禁止/手动/自动> 权限: <已配置/未配置> 路径: <空=源码同目录/自定义路径>
术语风格: <pure_chinese/mixed>
编程语言: <cpp/py/match_code>
模板代码: <有/无>
变值常量: <无 or 常量名列表>
可执行文件: <无 / N个关键词(N个路径)>
错误收集: <手动/确认后收录/自动> 权限: <已配置/未配置>
评测机速度: <1e8 / 5e7 / 3e7 / 2e8 / custom>
版本更新提醒: <是/否>
=========================
```

**Version check**: The latest config schema version is `0.2.14`.

**Perm sync**: Read `.claude/settings.local.json`. Check the `permissions.allow` array for write permissions and sync the config's `perm` flags to match reality:
- If `"Write(.claude/acm-trainer/mistakes.md)"` is in `permissions.allow` → set `collect_mistakes.perm: true` (regardless of current value)
- If `"Write(.claude/acm-trainer/solutions/*)"` is in `permissions.allow` → set `auto_collect_solution.perm: true` (regardless of current value)
- If `"Write(*.md)"` is in `permissions.allow` → set `markdown_output.perm: true` (regardless of current value)
- If not found → keep current `perm` value (may be `false` or absent)

This runs on EVERY Config invocation, not just format upgrades — it prevents stale `perm: false` when the user manually added permissions. Do NOT mention this sync to the user unless a stale value was corrected (then note: "已同步权限状态：<feature> perm false→true").

**Format check**: Regardless of config_version, validate each field's format. If `collect_mistakes` is a plain string or bool (not a mapping with `mode` key), or `auto_collect_solution` is a plain bool (not a mapping with `mode` key), or `markdown_output` is a plain string or bool (not a mapping with `mode` key), the config has old-format fields that need migration. Even if `config_version` is current, old-format fields must be upgraded.

If config_version < `"0.2.14"` **or** any field is in old format:

1. Check for old-format fields specifically:
   - `collect_mistakes` is a string or bool → needs migration to `{mode, perm}`
   - `auto_collect_solution` is a bool → needs migration to `{mode, perm}`
   - `markdown_output` is a string or bool → needs migration to `{mode, perm}`
   
   If any old-format fields found: show a message listing them, e.g., "检测到 N 个配置项格式过旧（collect_mistakes: bool→mapping, markdown_output: bool→mapping），需更新。" Then AskUserQuestion with header "配置格式升级", single option "立即升级" (no skip — old format fields must be fixed). Convert old values: `collect_mistakes: true` → `{mode: "auto", perm: false}`, `false` → `{mode: "manual", perm: false}`, `"auto"/"manual"/"confirm"` string → `{mode: "<value>", perm: false}`. `auto_collect_solution: true` → `{mode: true, perm: false}`, `false` → `{mode: false, perm: false}`. `markdown_output: true` → `{mode: "auto", perm: false}`, `false` → `{mode: "disabled", perm: false}`, `"disabled"/"manual"/"auto"` string → `{mode: "<value>", perm: false}`.

2. Also check for missing fields against the complete field list (defaults from acm-setup):

   | Field | Setup Step | Default |
   |-------|-----------|---------|
   | `code_location_mode` | Step 2 | `"none"` |
   | `code_paths` | Step 2 | `{}` |
   | `progressive_hints` | Step 3 | `true` |
   | `auto_edit_code` | Step 4 | `false` |
   | `auto_collect_solution` | Step 4b | `{mode: false, perm: false}` |
   | `terminology` | Step 7 | `"mixed"` |
   | `solution_language` | Step 8 | `"cpp"` |
   | `time_limit_baseline` | Step 9 | `100000000` |
   | `exe_paths` | Step 2b | `{}` |
   | `collect_mistakes` | Step 4c | `{mode: "manual", perm: false}` |
   | `markdown_output` | Step 4d | `{mode: "disabled", perm: false}` |
   | `markdown_output_path` | Step 4d | `""` |
   | `has_template` | Step 5 | `false` |
   | `remind_config_update` | — | `true` |

3. If only missing fields (no old-format): AskUserQuestion with header "配置升级", listing missing fields. Options: "是，逐项配置" / "跳过". Same flow as before.

4. After all fixes, write config with `config_version: "0.2.14"`, `last_modified: <today>`.

### Auto Permission Check

After format upgrade (and ONLY during format upgrade — not on every Config run), check if any features have mode requiring write permission but `perm` is `false`:

- `collect_mistakes.mode` in `{"auto", "confirm"}` with `collect_mistakes.perm: false`
- `auto_collect_solution.mode: true` with `auto_collect_solution.perm: false`
- `markdown_output.mode` in `{"auto", "manual"}` with `markdown_output.perm: false`

If any, run the Permission Follow-up flow (see Step 3) for each. Then also offer general read permissions ("权限配置" from Step 2 Question 2).

**If user chooses "跳过"**: keep `perm: false`. This records the user's choice — do NOT re-offer the same feature later in this session, and do NOT auto-offer on future Config runs (format upgrade only happens once).

Then enter Step 2.

## Step 2: Choose What to Change

Present all options in a **single** `AskUserQuestion` call with 4 questions (each ≤4 options, per tool limit). The user navigates between them with `←` `→` arrow keys and submits once. Collect all selections across all 4 questions before proceeding to Step 3.

**Question 1:**
- header: "修改配置 (1/4)"
- question: "第 1 组：代码与交互类配置，想修改哪些？"
- multiSelect: true
- options:
  - "代码位置" — 重新选择代码存放方式
  - "引导方式" — 切换渐进式引导开关
  - "自动修改代码" — 切换代码审查时直接修改文件
  - "术语风格" — 切换纯中文 / 保留英文缩写

**Question 2:**
- header: "修改配置 (2/4)"
- question: "第 2 组：收录与工具类配置，想修改哪些？"
- multiSelect: true
- options:
  - "收录题解" — 切换手动/自动收录题解到本地知识库
  - "编程语言" — 切换 C++ / Python / 跟随当前代码
  - "评测机速度" — 调整复杂度分析的 Safe N 基准值
  - "权限配置" — 将项目目录加入 settings.local.json，避免 acm 读代码/配置时弹授权提示

**Question 3:**
- header: "修改配置 (3/4)"
- question: "第 3 组：模板与自动化配置，想修改哪些？"
- multiSelect: true
- options:
  - "重新分析模板" — 重新指定模板文件并分析（含变值常量确认）
  - "版本更新提醒" — 切换是否在配置版本落后时提醒（当前: <remind_config_update>）
  - "可执行文件路径" — 配置/修改 C++ exe 路径，hack 生成后自动跑验证
  - "错误收集" — 切换是否自动收集编码错误模式

**Question 4:**
- header: "修改配置 (4/4)"
- question: "第 4 组：其他配置，想修改哪些？"
- multiSelect: true
- options:
  - "变值常量" — 修改每题需要调整的常量列表
  - "Markdown输出" — 切换禁止/手动/自动输出题解到 .md 文件（Typora 兼容）
  - "以上都没有" — 不需要修改

If the user has no template configured (`has_template: false`), hide "变值常量" option in Question 4. If that leaves Question 4 with only 1 option, keep both (min 2 is satisfied).

**Important**: Send all 4 questions in a single `AskUserQuestion` call — not 4 separate calls. The user uses `←` `→` to flip between question pages, then submits everything at once.

After receiving answers, merge selections from all 4 questions. If the user selected "以上都没有" in Question 4, ignore all other selections and treat as "no changes". If nothing was selected across all 4 questions, say "没有改动。" and exit.

## Step 3: Apply Changes

For each selected item, ask the new value using AskUserQuestion. Use the same options as in the setup flow (acm-setup skill).

For "变值常量": present the full list of candidates from the template (re-scan the template file), let user toggle which ones vary. This replaces the existing `per_problem_constants` list.

For "自动修改代码": ask with the same question as acm-setup Step 4. Toggle `auto_edit_code`.

For "收录题解": ask with the same question as acm-setup Step 4b. Toggle `auto_collect_solution`.

For "重新分析模板": re-run the full template analysis (Step 5 + Step 6 + Step 6a of acm-setup), including per-problem constant confirmation. After analysis, read the existing `.claude/acm-trainer/template-summary.md` (if it exists). Compare the new analysis against the old summary — show a diff of what changed (新增/变化/移除) before asking the user to confirm. Do NOT say "无变化" without doing a section-by-section comparison (aliases, macros, IO, gotchas, per-problem constants).

For "可执行文件路径": present current state for each keyword (show all paths, not just one). Then AskUserQuestion:
- header: "可执行文件"
- question: "当前关键词及路径：<list current keywords with their paths>. 要如何修改？"
- options:
  - "添加路径" — 选择一个关键词，追加一个 exe 路径
  - "移除路径" — 选择一个关键词，从列表中删除某个路径
  - "重新自动寻找" — 对某个关键词重新搜索所有位置（替换该关键词的路径列表）
  - "手动重新指定" — 清空某个关键词的路径列表，手动输入新路径

For "添加路径": ask which keyword, then ask for the absolute path. Append to that keyword's list.
For "移除路径": ask which keyword, then list its paths for user to pick which to remove. If only 1 path left, warn "至少保留一个路径，或选"不配置"清空全部"。
For "重新自动寻找": use the same logic as acm-setup Step 2b auto-find. Search all standard locations, replace that keyword's list with all found paths.
For "手动重新指定": ask which keyword, then ask for one or more comma-separated paths. Replace that keyword's list.

After any change, the new list is ordered by modification time (newest first) for readability — runtime always picks the newest anyway.

If user chooses "不配置": clear all exe_paths (set to `{}`).


For "错误收集": AskUserQuestion — header: "错误收集", question: "代码审查发现 bug 后，如何记录错误模式？", options: "手动收集" (set mode to `"manual"`), "自动总结，确认后收录" (set mode to `"confirm"`), "自动静默收集" (set mode to `"auto"`). Set `collect_mistakes.mode`. Keep existing `collect_mistakes.perm` if present, otherwise default `false`. Then, if mode is `"auto"` or `"confirm"` and perm is `false`, run the per-feature permission flow (see "Permission Follow-up" below).

For "收录题解": AskUserQuestion with same question as acm-setup Step 4b. Set `auto_collect_solution.mode`. Keep existing `auto_collect_solution.perm` if present, otherwise default `false`. Then, if mode is `true` and perm is `false`, run the per-feature permission flow.

For "评测机速度": use the same options as acm-setup Step 9. Update `time_limit_baseline`.

For "版本更新提醒": AskUserQuestion — header: "版本提醒", question: "是否在配置版本落后时提醒更新？", options: "是，提醒我" (set `true`) / "不用提醒" (set `false`). Update `remind_config_update`.

For "Markdown输出": AskUserQuestion — header: "Markdown输出", question: "是否将题解/算法讲解/代码审查输出到 .md 文件？（与源码同名同目录，如 A.cpp → A.md）", options: "禁止（默认）" (set mode to `"disabled"`), "手动指定" (set mode to `"manual"`), "自动输出" (set mode to `"auto"`). Set `markdown_output.mode`. Keep existing `markdown_output.perm` if present, otherwise default `false`. Then ask for output path: AskUserQuestion — header: "输出路径", question: "Markdown 输出目录？默认留空（与源码同目录）。", options: "默认（与源码同目录）" (set `""`), "自定义目录" (ask and set path). Then, if mode is `"auto"` or `"manual"` and perm is `false`, run the per-feature permission flow (see "Permission Follow-up" below).

### Permission Follow-up

After setting `collect_mistakes.mode` (to `"auto"`/`"confirm"`), `auto_collect_solution.mode` (to `true`), or `markdown_output.mode` (to `"auto"`/`"manual"`), if the corresponding `perm` is `false`, offer to configure write permission immediately:

AskUserQuestion:
- header: "权限配置"
- question: "`<功能名>` 需要写入 `<文件路径>` 的权限，否则每次都会弹授权提示。是否现在配置？"
- multiSelect: false
- options:
  - "是，自动配置" — add the Write permission to settings.local.json, then set `perm: true`
  - "跳过" — keep `perm: false` (records user's choice; do NOT re-offer for this feature in this session)

If "是，自动配置":
1. Read `.claude/settings.local.json` in the project root. If it doesn't exist, start with `{}`.
2. Add the appropriate Write rule to `permissions.allow`:
   - For `collect_mistakes`: `"Write(.claude/acm-trainer/mistakes.md)"`
   - For `auto_collect_solution`: `"Write(.claude/acm-trainer/solutions/*)"`
   - For `markdown_output`: `"Write(*.md)"`
3. Merge without duplicates, write back.
4. Set the corresponding `perm` to `true`.
5. Confirm: "`<功能名>` 的写入权限已配置。"

For "权限配置": ask the user to confirm. Same logic as acm-setup Step 13 but also configurable here independently. Configures permissions based on what the user actually needs (not all unconditionally):

**Always added** (needed for any acm operation):
- `additionalDirectories`: project dir + plugin cache root (Read access)

**Conditional — only if `exe_paths` is non-empty** (hack verification needs these):
- `Bash(stat *)` — comparing exe vs source timestamps
- `Bash(*.exe *)` — running compiled exe for hack verification

Write permissions for mistake collection and solution collection are handled separately via "Permission Follow-up" above, not here.

AskUserQuestion:
- header: "权限配置"
- question: "是否自动配置权限？① 读代码/配置（必需）<if exe_paths non-empty: + ② 比较 exe 时间戳 ③ 运行 exe 验证 hack>。之后不再弹授权提示。（不影响其他项目）"
- multiSelect: false
- options:
  - "是，自动配置" — 自动添加所需权限
  - "不用了" — 跳过

If "是，自动配置":
1. Read `.claude/settings.local.json` in the project root. If it doesn't exist, start with `{}`.
2. Build the permission block. Start with Read permissions:
   ```json
   {"permissions": {"additionalDirectories": ["<project dir>", "<plugin cache root>"]}}
   ```
3. If `exe_paths` is non-empty, also add to `permissions.allow`: `"Bash(stat *)"`, `"Bash(*.exe *)"`.
4. Merge into existing JSON (preserve all existing settings, merge arrays without duplicates).
5. Write back. Confirm what was added: "已配置权限：Read 目录<if exe: + stat + exe 运行>"。

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

**Schema cleanup first**: To prevent deprecated config fields from accumulating (e.g., `duipai_exe_paths` after removal), strip unknown keys from the old YAML frontmatter before writing. The current config schema consists of these keys:

```
code_location_mode, code_paths, progressive_hints, auto_edit_code,
auto_collect_solution, markdown_output, markdown_output_path, terminology,
solution_language, time_limit_baseline, exe_paths, collect_mistakes,
has_template, template_boundary, template_entry, per_problem_constants,
config_version, remind_config_update, last_modified
```

1. From the parsed old YAML frontmatter, keep only keys that appear in the whitelist above. Drop any others.
2. Merge the new/changed values into the filtered config.
3. Set `config_version` to `"0.2.14"` and `last_modified` to today's date.
4. **Write fields in this exact order** (matches acm/SKILL.md parsing order, so the model reads sequentially without jumping):

```yaml
code_location_mode
code_paths
progressive_hints
auto_edit_code
terminology
solution_language
has_template
template_boundary        # only if has_template
template_entry           # only if has_template
per_problem_constants
exe_paths
collect_mistakes         # as mapping {mode, perm}
auto_collect_solution    # as mapping {mode, perm}
markdown_output          # as mapping {mode, perm}
markdown_output_path
time_limit_baseline
config_version
remind_config_update
last_modified
```

5. Config body: always write `<!-- 模板摘要见 .claude/acm-trainer/template-summary.md -->` (a one-line marker, no template analysis content).
6. If "重新分析模板" or "变值常量" was selected: write the updated template analysis to `.claude/acm-trainer/template-summary.md`.

Write both files with Write tool. Confirm: "配置已更新。"
