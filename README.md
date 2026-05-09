<div align="center">

# 🏆 ACM Trainer

**Claude Code 插件 · 算法竞赛智能训练助手**

题解推导 · 代码审查 · 算法讲解 · 复杂度分析 · Hack 数据生成


[English](README-EN.md)

</div>

---

## ✨ 功能

| 功能 | 说明 |
|------|------|
| 📝 **题目求解** | 贴入题目描述，自动解析约束、识别题型、渐进式引导推导 |
| 🔍 **代码审查** | 扫描溢出、越界、未初始化、TLE 风险，生成 Hack 数据 |
| 📖 **算法讲解** | 数据结构 & 算法精讲，附 C++ 模板代码和练习题推荐 |
| 📊 **复杂度分析** | 逐段分析时空复杂度，对比约束判断是否通过 |
| ⚙️ **配置持久化** | 一次初始化，代码位置 / 引导方式 / 模板分析 / 语言偏好全记住 |

## 📦 安装

```bash
/plugin marketplace add HenryYves/acm-trainer # 添加市场
/plugin marketplace add https://github.com/HenryYves/acm-trainer # 用这个也可以


/plugin install acm-trainer@acm-trainer --scope project # 安装插件(后面的`--scope project`表示只为当前项目安装而不是全局安装,对于通常只在一个文件(项目)里面写题的人推荐添加这个参数或者手动修改skill的yaml使其不能被主Agent自动触发.)

/reload-plugins # 重载插件
```

## 🚀 快速开始

```
/acm-trainer:acm-setup     # 首次运行 — 5 步完成初始化;`acm-trainer:`为命名空间可省略
/acm-trainer:acm-config    # 随时查看 / 修改配置
/acm-trainer:acm           # 主入口 — 贴题目、贴代码、问算法
```

> 主 Agent 会根据上下文自动激活——直接贴题目或代码即可，无需手动调用。亦可使用命令`\acm`指定，可以节省少量token。

## 🧩 结构

```
acm-trainer/
├── .claude-plugin/
│   ├── MODIFICATION.md         # 修改指南（AI向，人类勿读）
│   ├── changelog/              # 修改详情（AI向，人类勿读）
│   │   └── 0.2.2.md ...        # 按需读，一个版本一个文件
│   ├── marketplace.json
│   └── plugin.json
├── skills/
│   ├── acm/               # 主训练 skill
│   │   └── references/    # 详细工作流（按需加载以节省token）
│   ├── acm-setup/          # 初始化向导
│   └── acm-config/         # 配置管理
├── README.md
└── README-EN.md
```

> `.claude-plugin/MODIFICATION.md` 和 `changelog/*.md` 是给 skill-creator (AI) 读的，格式为省 token 优化过（扁平列表、箭头、无装饰），不适合人类阅读。

## ⚙️ 配置

初始化后配置保存在项目根目录 `.claude/acm-trainer.local.md`：

```yaml
---
code_location_mode: files          # none | single | per_problem | files
code_paths:
  default: D:\my\my.cpp
  A: D:\my\sub_project\A\A.cpp
progressive_hints: true
auto_edit_code: false              # 是否允许自动发起代码修改
terminology: mixed                 # pure_chinese | mixed
solution_language: cpp             # cpp | py | match_code
time_limit_baseline: 100000000     # O(N) safe N per second
has_template: true
template_path: D:\my\template.cpp
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
- `maxn` (L45, 默认 1e5+20): 数组大小上限，每题需检查
```

---

<div align="center">
**Made with ❤️ for competitive programmers**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

</div>
