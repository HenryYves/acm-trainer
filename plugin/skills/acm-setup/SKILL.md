---
name: acm-setup
description: This skill should be used when the user asks to "acm初始化", "配置acm", "acm setup", "setup acm", "初始化acm训练助手", "首次使用acm", or wants to configure ACM trainer settings for the first time. Provides a deterministic guided setup wizard that collects code location, progressive hints preference, template analysis, and language settings, then writes configuration to .claude/acm-trainer.local.md.
allowed-tools: Read, Write, AskUserQuestion, Glob, Grep
---

# ACM Trainer Setup

Deterministic setup wizard for the ACM trainer plugin. Collects user preferences through a fixed sequence of questions, analyzes template code if provided, and writes configuration.

## Step 1: Check Existing Config

Read `.claude/acm-trainer.local.md` if it exists.

- **Config does not exist** → proceed to Step 2 (full setup).
- **Config exists** → show a brief summary of current settings. Ask: "配置已存在，是否重新初始化？（这将覆盖当前配置）" If user confirms, proceed to Step 2. Otherwise exit.

## Step 2: Code Location

AskUserQuestion:
- header: "代码位置"
- question: "你的解题代码平时放在哪里？"
- multiSelect: false
- options:
  - "不固定，每次自己贴代码" — 无预设位置，每次手动粘贴
  - "单个文件夹" — 追问具体路径
  - "多个位置，用关键词区分" — 追问关键词到路径的映射（如 cf→D:\cf\solutions）

If user chooses "单个文件夹", ask for the absolute path. Validate it exists or warn.
If user chooses "多个位置", ask for comma-separated key→path pairs (e.g., "cf=D:\cf, luogu=D:\luogu").

## Step 3: Progressive Hints

AskUserQuestion:
- header: "引导方式"
- question: "是否默认启用渐进式引导？（先给提示，用户要求后再给完整解法）"
- multiSelect: false
- options:
  - "是，默认渐进式引导" — 先给方向/提示，等用户要求再展开
  - "否，直接给答案" — 直接给出醍醐灌顶句 + 完整解法

## Step 4: Template Code

AskUserQuestion:
- header: "模板代码"
- question: "是否有固定的 C++ 模板代码文件？"
- multiSelect: false
- options:
  - "没有模板" — 跳过模板分析
  - "有，请分析" — 追问模板文件路径

If user chooses "有，请分析", ask for the absolute path to the template `.cpp` file. Read it and proceed to Step 5.
Otherwise skip to Step 6.

## Step 5: Template Analysis

Read the template file. Identify:

1. **Boundary**: Look for separator comments (e.g., `//===><><=======<|-C-O-D-E-|>=====><><====//`) or function entry points (e.g., `solve()` function). Everything before the boundary is template; the boundary and beyond is user code.
2. **Key aliases**: `using ll = long long`, `using pii = pair<int,int>`, etc.
3. **Key macros**: `pre` (forward for), `erp` (reverse for), `max`/`min` (function-style macros), `endl` (redefined as `'\n'`), `IOS` (fast I/O toggle), etc.
4. **I/O functions**: `rd()` for input, `pr()`/`pt()`/`pp()` for output, `fast_write` namespace.
5. **Debug infrastructure**: `hs_f_debug` flag, `de(x)` macro, `strde`.
6. **Required init**: `init_win_env()` must be called in `main()` on Windows.
7. **Entry point**: Where does user code go? (e.g., `solve()` function, the line number where it starts).

**Template gotchas to flag** (these are the things that cause bugs):
- `#define max(a,b)` / `#define min(a,b)` — function-style macros, conflict with `std::max`/`std::min`, double-evaluation side effects.
- `#define endl '\n'` — `cout << endl` does NOT flush. Use `flush()` manually if needed.
- `#define IOS ios_base::sync_with_stdio(false)...` — typically commented out because template uses custom fast I/O.
- Custom `rd()` function — competes with standard `cin >>`, do not mix.
- `init_win_env()` — if forgotten on Windows, terminal color escapes won't work.

Save the analysis as markdown in the template summary body of the config file (Step 7).

## Step 6: Terminology Style

AskUserQuestion:
- header: "术语风格"
- question: "回答时采用哪种术语风格？"
- multiSelect: false
- options:
  - "中文为主，保留常见英文缩写" — DP、BFS、DFS 等保留，segment tree→线段树
  - "纯中文（英文术语也翻译）" — DP→动态规划，BFS→广度优先搜索

## Step 7: Solution Language

AskUserQuestion:
- header: "题解语言"
- question: "题解和代码注释用什么语言？"
- multiSelect: false
- options:
  - "中文"
  - "英文"
  - "中英双语"

## Step 8: Preview & Confirm

Show a summary of all choices:

```
=== ACM Trainer 配置预览 ===
代码位置: <summary>
渐进式引导: <是/否>
模板代码: <无 / 路径 + 摘要>
术语风格: <pure_chinese / mixed>
题解语言: <zh / en / bilingual>
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

## Step 9: Write Config

Generate `.claude/acm-trainer.local.md`:

**YAML frontmatter:**
```yaml
---
code_location_mode: <none|single|multi>
code_paths:
  default: <path or "">
  <keyword>: <path>
progressive_hints: <true|false>
terminology: <pure_chinese|mixed>
solution_lang: <zh|en|bilingual>
has_template: <true|false>
template_path: "<path or empty>"
template_boundary: <line number, 0 if no template>
template_entry: "<entry description or empty>"
---
```

**Markdown body** (only if has_template):
```markdown
# Template Summary

<template analysis from Step 5>

## Gotchas
<list of template-specific bugs to watch for>
```

Write the file with Write tool. Do NOT use Bash for file creation.

## Step 10: Verify

Confirm: "配置已保存到 `.claude/acm-trainer.local.md`。可以试试发一道算法题来测试。"

Also remind: "以后想改配置，用 `/acm-trainer:acm-config`。"
