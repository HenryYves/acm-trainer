<div align="center">

# 🏆 ACM Trainer

**Claude Code Plugin · Algorithmic Competitive Programming Tutor**

Problem Solving · Code Review · Algorithm Explanation · Complexity Analysis · Hack Generation


[中文](README.md)

</div>

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 📝 **Problem Solving** | Paste a problem statement, auto-parse constraints, identify problem type, progressive hint-based derivation |
| 🔍 **Code Review** | Scan for overflow, off-by-one, uninitialized variables, TLE risks, and generate targeted hack cases |
| 📖 **Algorithm Tutorials** | In-depth data structure & algorithm explanations with clean template code and practice problem recommendations |
| 📊 **Complexity Analysis** | Break down time/space complexity section by section, check against constraints |
| ⚙️ **Persistent Config** | One-time setup: code location, hint style, template analysis, language, OJ speed calibration |

## 📦 Installation

```bash
/plugin marketplace add https://github.com/HenryYves/acm-trainer
/plugin install acm-trainer@acm-trainer --scope project  # or --scope local
/reload-plugins
```

## 🚀 Quick Start

```
/acm-trainer:acm-setup     # First run — guided setup wizard
/acm-trainer:acm-config    # View / modify configuration anytime
/acm-trainer:acm           # Main entry — paste problems, code, or ask about algorithms
```

> The main skill activates automatically on Chinese-language CP queries. Just paste a problem or code directly.

## 🧩 Structure

```
acm-trainer/
├── .claude-plugin/
│   ├── MODIFICATION.md         # Modification guide (AI-oriented, not for humans)
│   ├── changelog/              # Version changelogs (AI-oriented, not for humans)
│   │   └── 0.2.2.md ...        # Loaded on demand, one file per version
│   ├── marketplace.json
│   └── plugin.json
├── skills/
│   ├── acm/               # Main training skill
│   │   └── references/    # Detailed workflows (loaded on demand to save tokens)
│   ├── acm-setup/          # Setup wizard
│   └── acm-config/         # Configuration management
├── README.md
└── README-EN.md
```

> `.claude-plugin/MODIFICATION.md` and `changelog/*.md` are for skill-creator (AI) to read. Their format is optimized for token efficiency (flat lists, arrows, no decoration) — not meant for human reading.

## ⚙️ Configuration

After setup, configuration is stored at `.claude/acm-trainer.local.md` in your project root:

```yaml
---
code_location_mode: files          # none | single | per_problem | files
code_paths:
  default: /path/to/my.cpp
  A: /path/to/sub_project/A/A.cpp
progressive_hints: true
auto_edit_code: false              # whether to auto-edit code files
terminology: mixed                 # pure_chinese | mixed
solution_language: cpp             # cpp | py | match_code
time_limit_baseline: 100000000     # O(N) safe N per second
has_template: true
template_path: /path/to/template.cpp
template_boundary: 80
template_entry: "solve() // L87"
per_problem_constants:
  - name: maxn
    line: 45
    default_value: "1e5 + 20"
config_version: "0.2.0"
last_modified: "2026-05-07"
---
# Template Summary
...
## Per-Problem Constants
- `maxn` (L45, default 1e5+20): array size limit, check per problem
```

### `code_location_mode` Options

| Mode | Use Case |
|------|----------|
| `none` | Paste code manually each time |
| `single` | One fixed file for all problems |
| `per_problem` | One file per problem (browser plugin workflow) |
| `files` | Multiple files with keyword mapping (e.g., `my`, `A`, `B`) |

### `time_limit_baseline`

Calibrates the Safe N complexity table to your OJ's speed. Safe N values for each complexity class are computed from this baseline.

### `per_problem_constants`

Template constants that need adjustment per problem (e.g., `maxn` for array size, `MOD` for modulo). The code reviewer will flag any that look wrong for the current problem's constraints.

---

<div align="center">
**Made with ❤️ for competitive programmers**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

</div>
