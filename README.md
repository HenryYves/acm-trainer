<div align="center">

# 🏆 ACM Trainer

**Claude Code 插件 · 算法竞赛智能训练助手**

题解推导 · 代码审查 · 算法讲解 · 复杂度分析 · Hack 数据生成

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

```
/plugin marketplace add HenryYves/acm-trainer
/plugin install acm-trainer@acm-trainer --scope project
/reload-plugins
```

## 🚀 快速开始

```
/acm-trainer:acm-setup     # 首次运行 — 5 步完成初始化
/acm-trainer:acm-config    # 随时查看 / 修改配置
/acm-trainer:acm           # 主入口 — 贴题目、贴代码、问算法
```

> 主 skill 会根据上下文自动激活——直接贴题目或代码即可，无需手动调用。

## 🧩 结构

```
acm-trainer/
├── .claude-plugin/
│   ├── marketplace.json
│   └── plugin.json
├── skills/
│   ├── acm/               # 主训练 skill
│   │   └── references/    # 详细工作流（按需加载）
│   ├── acm-setup/          # 初始化向导
│   └── acm-config/         # 配置管理
└── README.md
```

## ⚙️ 配置

初始化后配置保存在项目根目录 `.claude/acm-trainer.local.md`：

```yaml
---
code_location_mode: single        # none | single | multi
code_paths:
  default: D:\my\solutions
progressive_hints: false
terminology: mixed                # pure_chinese | mixed
solution_lang: zh                 # zh | en | bilingual
has_template: true
template_path: D:\my\template.cpp
template_boundary: 80
template_entry: "solve() // L87"
---
# Template Summary
...
```

> 加入 `.gitignore`：`.claude/*.local.md`

---

<div align="center">

**Made with ❤️ for competitive programmers**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

</div>
