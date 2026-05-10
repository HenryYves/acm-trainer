---
name: acm-setup
description: This skill should be used when the user asks to "acm初始化", "配置acm", "acm setup", "setup acm", "初始化acm训练助手", "首次使用acm", or wants to configure ACM trainer settings for the first time. Provides a deterministic guided setup wizard that collects code location, progressive hints preference, template analysis, solution language, and terminology settings, then writes configuration to .claude/acm-trainer.local.md.
allowed-tools: Read, Write, AskUserQuestion, Glob, Grep
---

# ACM Trainer Setup

Deterministic setup wizard for the ACM trainer plugin. Collects user preferences through a fixed sequence of questions, analyzes template code if provided, and writes configuration.

## Step 1: Check Existing Config

Read `.claude/acm-trainer.local.md` if it exists.

- **Config does not exist** → proceed to Step 2 (full setup).
- **Config exists** → show a brief summary of current settings. If `config_version` is old, mention: "检测到旧版配置，可运行 `/acm-trainer:acm-config` 补全配置。" Then ask: "是否完全重新初始化？（这将覆盖当前配置）" If user confirms, proceed to Step 2. Otherwise exit. (Config updates are handled by `/acm-trainer:acm-config`, not setup.)

## Step 2: Code Location

AskUserQuestion:
- header: "代码位置"
- question: "你的解题代码平时怎么存放？"
- multiSelect: false
- options:
  - "不固定，每次自己贴代码" — 无预设位置，每次手动粘贴代码到对话
  - "固定一个文件" — 所有题目写在一个文件里（如 my.cpp），追问具体路径
  - "一题一个文件" — 每道题一个独立文件（常见于浏览器插件获取题目自动创建文件的场景），追问文件存放目录
  - "多个文件，用关键词区分" — 几个固定文件用简短关键词标记（如 my→my.cpp, A→A.cpp），追问关键词→路径映射

**"不固定"** → set `code_location_mode: none`, skip to Step 3.

**"固定一个文件"** → ask for the absolute path. Validate it exists or warn. Set `code_location_mode: single`.

**"一题一个文件"** → ask for the directory where problem files are stored. If the user has a naming convention (e.g., `{problem_id}.cpp`), ask optionally. Set `code_location_mode: per_problem`. Record the directory in `code_paths.default`.

**"多个文件，用关键词区分"** → ask for comma-separated key→path mapping (e.g., "my=D:\my.cpp, A=D:\sub_project\A\A.cpp"). Set `code_location_mode: files`.

## Step 3: Progressive Hints

AskUserQuestion:
- header: "引导方式"
- question: "是否默认启用渐进式引导？（先给提示，用户要求后再给完整解法）"
- multiSelect: false
- options:
  - "是，默认渐进式引导" — 先给方向/提示，等用户要求再展开
  - "否，直接给答案" — 直接给出醍醐灌顶句 + 完整解法

## Step 4: Auto Edit Code

AskUserQuestion:
- header: "自动修改"
- question: "是否允许代码审查时自动发起代码修改请求？（选"不允许"则仅文字提示，不主动修改文件）"
- multiSelect: false
- options:
  - "不允许（默认）" — 只给出修改建议的文字说明，不主动编辑/创建代码文件
  - "允许" — 发现问题后可以直接用 Edit/Write 修改或创建代码文件

## Step 5: Template Code

AskUserQuestion:
- header: "模板代码"
- question: "是否有固定的 C++ 模板代码文件？"
- multiSelect: false
- options:
  - "没有模板" — 跳过模板分析
  - "有，请分析" — 追问模板文件路径

If user chooses "有，请分析", ask for the absolute path to the template `.cpp` file. Read it and proceed to Step 6.
Otherwise skip to Step 7.

## Step 6: Template Analysis

Read the template file. Identify:

1. **Boundary**: Look for separator comments (e.g., `//===><><=======<|-C-O-D-E-|>=====><><====//`) or function entry points (e.g., `solve()` function). Everything before the boundary is template; the boundary and beyond is user code.
2. **Key aliases**: `using ll = long long`, `using pii = pair<int,int>`, etc.
3. **Key macros**: `pre` (forward for), `erp` (reverse for), `max`/`min` (function-style macros), `endl` (redefined as `'\n'`), `IOS` (fast I/O toggle), etc.
4. **I/O functions**: `rd()` for input, `pr()`/`pt()`/`pp()` for output, `fast_write` namespace.
5. **Debug infrastructure**: `hs_f_debug` flag, `de(x)` macro, `strde`.
6. **Required init**: `init_win_env()` must be called in `main()` on Windows.
7. **Entry point**: Where does user code go? (e.g., `solve()` function, the line number where it starts).
8. **Per-problem constants**: Scan `constexpr`/`const` definitions for values likely to change between problems:
   - Array size limits: `maxn`, `MAXN`, `N`, `MAX_N`
   - Modulo values: `MOD`, `mod`, `mod1`, `mod2`
   - Other "magic numbers" that look problem-specific (e.g., `inf`, `LIMIT`)
   
   Collect candidates into a list. Each candidate should note: name, line number, current value.

**Template gotchas to flag** (these are the things that cause bugs):
- `#define max(a,b)` / `#define min(a,b)` — function-style macros, conflict with `std::max`/`std::min`, double-evaluation side effects.
- `#define endl '\n'` — `cout << endl` does NOT flush. Use `flush()` manually if needed.
- `#define IOS ios_base::sync_with_stdio(false)...` — typically commented out because template uses custom fast I/O.
- Custom `rd()` function — competes with standard `cin >>`, do not mix.
- `init_win_env()` — if forgotten on Windows, terminal color escapes won't work.

### Step 6a: Confirm Per-Problem Constants

If candidate per-problem constants were found (step 8 above), present them to the user:

AskUserQuestion:
- header: "变值常量"
- question: "以下常量中，哪些每道题会需要改值？（多选）"
- multiSelect: true
- options: one per candidate, e.g., "maxn (当前值: 1e5+20, 第45行) — 数组大小上限"
  plus one option: "以上全都不变" — 所有值都是固定的，无需每题调整

Only save the constants the user selects into the config. Those not selected are treated as fixed template boilerplate and won't be flagged during code review.

If no candidates were found, skip to Step 7 without asking.

## Step 7: Terminology Style

AskUserQuestion:
- header: "术语风格"
- question: "回答时采用哪种术语风格？"
- multiSelect: false
- options:
  - "中文为主，保留常见英文缩写" — DP、BFS、DFS 等保留，segment tree→线段树
  - "纯中文（英文术语也翻译）" — DP→动态规划，BFS→广度优先搜索

## Step 8: Solution Language

AskUserQuestion:
- header: "编程语言"
- question: "题解代码用什么编程语言？"
- multiSelect: false
- options:
  - "C++" — 代码全部用 C++
  - "Python" — 代码全部用 Python
  - "与当前代码一致" — 用户贴什么语言就回什么语言；未贴代码时默认用 C++

## Step 9: Time Limit Baseline

AskUserQuestion:
- header: "评测机速度"
- question: "你常用 OJ 的评测机大概多快？（以 C++ 1秒能做多少次基础操作为基准）"
- multiSelect: false
- options:
  - "标准（约 10⁸）" — 大多数 OJ（Codeforces、AtCoder、洛谷等）的水平
  - "偏慢（约 5×10⁷）" — 较老或资源受限的 OJ
  - "较慢（约 3×10⁷）" — 明显偏慢的评测环境
  - "偏快（约 2×10⁸）" — 较新的高性能评测机
  - "自定义" — 用户自己输入一个值

If "自定义", ask for the specific number (N for O(N) in 1s). Record as `time_limit_baseline`.

This value determines the Safe N table used in complexity analysis (see workflows.md):

| Complexity | Safe N formula |
|-----------|----------------|
| O(N) | baseline |
| O(N log N) | baseline / 20 |
| O(N√N) | √(baseline) × 0.7 |
| O(N²) | √(baseline) × 0.5 |
| O(2^N) | log₂(baseline) × 0.8 |

Python solutions get approximately 1/30 of the C++ baseline.

## Step 10: Preview & Confirm

Show a summary of all choices:

```
=== ACM Trainer 配置预览 ===
代码位置: <none | 单文件:path | 一题一文件:dir | 多文件:N个关键词>
渐进式引导: <是/否>
自动修改代码: <允许/不允许>
模板代码: <无 / 路径 + 摘要>
变值常量: <无 / 选中的常量名列表>
术语风格: <pure_chinese / mixed>
编程语言: <cpp / py / match_code>
评测机速度: <1e8 / 5e7 / 3e7 / 2e8 / custom value>
=========================
```

AskUserQuestion:
- header: "确认"
- question: "以上配置是否正确？"
- multiSelect: false
- options:
  - "确认保存"
  - "取消"

If user cancels, exit without writing.

## Step 11: Write Config

Generate `.claude/acm-trainer.local.md`:

**YAML frontmatter:**
```yaml
---
code_location_mode: <none|single|per_problem|files>
code_paths:
  default: <path or dir or "">
  <keyword>: <path>  # only for files mode
progressive_hints: <true|false>
auto_edit_code: <true|false>
terminology: <pure_chinese|mixed>
solution_language: <cpp|py|match_code>
time_limit_baseline: <100000000 (1e8) or custom value>
config_version: "0.2.4"
remind_config_update: true
last_modified: "<today's date YYYY-MM-DD>"
has_template: <true|false>
template_path: "<path or empty>"
template_boundary: <line number, 0 if no template>
template_entry: "<entry description or empty>"
per_problem_constants:
  - name: <constant name>
    line: <line number>
    default_value: "<current value in template>"
---
```

**Markdown body** (only if has_template or per_problem_constants):
```markdown
# Template Summary

<template analysis from Step 5 — aliases, macros, I/O, debug, entry point>

## Gotchas
<list of template-specific bugs to watch for>

## Per-Problem Constants
<list of constants the user confirmed need per-problem adjustment, with default values>
```

Write the file with Write tool. Do NOT use Bash for file creation.

## Step 12: Verify

Confirm: "配置已保存到 `.claude/acm-trainer.local.md`。可以试试发一道算法题来测试。"

Also remind: "以后想改配置，用 `/acm-trainer:acm-config`。"

## Step 13: Auto-Configure Permissions

Reduce permission prompts by adding trusted directories to `.claude/settings.local.json`. Two directories need access:

1. **项目目录** — so config reads and code file lookups don't trigger permission prompts.
2. **插件缓存根目录** — so the acm skill can read its own reference files (`workflows.md`, `code-review.md`) without prompts. This skill file lives inside the plugin cache at `<root>/<version>/skills/acm-setup/SKILL.md`. Go up 3 levels from this skill file's directory to reach the plugin cache root (the `acm-trainer` directory). For a typical install: `~/.claude/plugins/cache/Yves-plugin/acm-trainer/` (Windows: `%USERPROFILE%\.claude\plugins\cache\Yves-plugin\acm-trainer\`).

AskUserQuestion:
- header: "权限配置"
- question: "是否自动配置项目权限？选"是"会将项目目录加入 settings.local.json，之后读取代码文件、搜索配置不会再弹出授权提示。（不会影响其他项目）"
- multiSelect: false
- options:
  - "是，自动配置" — 自动把项目目录加入权限白名单
  - "不用了" — 跳过，保留默认权限设置

If user chooses "不用了", skip to done.

**If user chooses "是，自动配置":**

1. Read `.claude/settings.local.json` in the project root (current working directory). If it doesn't exist, start with `{}`.
2. Parse existing JSON. Merge the following into it (preserve all existing settings):

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

3. If existing `permissions.allow` already has entries, merge — do NOT replace. If existing `permissions.additionalDirectories` already has entries, merge — do NOT replace.
4. Write the merged result back to `.claude/settings.local.json` with the Write tool.
5. Confirm: "项目及插件权限已配置。之后 acm 读代码、读 reference 文件、查配置不会弹授权了。"

If `.claude/settings.local.json` was newly created (didn't exist before), also add it to `.gitignore` if one exists and the entry isn't there yet.
