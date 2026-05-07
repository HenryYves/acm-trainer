<div align="center">

<H1>🏆 ACM Trainer</H1>

<p><em>Claude Code 插件 · 算法竞赛智能训练助手</em></p>

<span>题解推导 · 代码审查 · 算法讲解 · 复杂度分析 · Hack 数据生成</span>

</div>

---

## ✨ 功能

| 功能 | 说明 |
|------|------|
| 📝 **题目求解** | 贴入题目描述，自动解析约束、识别题型、渐进式引导推导 |
| 🔍 **代码审查** | 扫描溢出、越界、未初始化、TLE 风险、模板陷阱，生成 Hack 数据 |
| 📖 **算法讲解** | 数据结构 & 算法精讲，附 C++ 模板代码和练习题推荐 |
| 📊 **复杂度分析** | 逐段分析时空复杂度，对比约束判断是否通过 |
| ⚙️ **配置持久化** | 一次初始化，代码位置 / 引导方式 / 模板分析 / 语言偏好全记住 |

## 📦 安装

### 方式一：市场安装（推荐）

在 Claude Code 中依次执行：

```
/plugin marketplace add HenryYves/acm-trainer
/plugin install acm-trainer
/reload-plugins
```

### 方式二：本地开发

```bash
cc --plugin-dir path/to/acm-trainer
```

## 🚀 快速开始

```
/acm-trainer:acm-setup     # 首次运行 — 5 步完成初始化
/acm-trainer:acm-config    # 随时查看 / 修改配置
/acm-trainer:acm           # 主入口 — 贴题目、贴代码、问算法
```

> 主 skill 会根据上下文自动激活——直接贴题目或代码即可，无需手动调用命令。

## 🧩 结构

```
acm-trainer/
├── .claude-plugin/
│   ├── marketplace.json          # 市场清单
│   └── plugin.json               # 插件清单
├── skills/
│   ├── acm/                      # 主训练 skill
│   │   └── references/           # 详细工作流（按需加载）
│   │       ├── workflows.md      #   题解 · 算法 · 复杂度
│   │       └── code-review.md    #   代码审查 · Hack 生成
│   ├── acm-setup/                # 初始化向导
│   └── acm-config/               # 配置管理
└── README.md
```

## ⚙️ 配置

初始化后配置保存在项目根目录 `.claude/acm-trainer.local.md`：

```yaml
---
code_location_mode: single        # none | single | multi
code_paths:
  default: D:\my\solutions
progressive_hints: false          # 是否渐进式引导
terminology: mixed                # pure_chinese | mixed
solution_lang: zh                 # zh | en | bilingual
has_template: true                # 是否有模板代码
template_path: D:\my\template.cpp
template_boundary: 80             # 模板行数
template_entry: "solve() // L87"  # 用户代码入口
---
# Template Summary
...
```

> 加入 `.gitignore`：`.claude/*.local.md`

## 🎯 使用示例

<div>
  <details>
    <summary><b>代码审查 — 找 Bug + 生成 Hack</b></summary>
    <pre><code>帮我看看这段代码为什么 WA

08 | const int N = 1e5 + 5;
09 | int a[N];
...
34 | for (int i = 0; i &lt;= n; i++) cin >> a[i];</code></pre>
    <p>回复：以醍醐灌顶句开头 → 指 off-by-one（<code>i &lt;= n</code>）→ 生成最小 Hack → 展示期望 vs 实际输出</p>
  </details>

  <details>
    <summary><b>题目求解 — 渐进式引导</b></summary>
    <pre><code>给定 n 个整数，求最长上升子序列长度。n ≤ 10⁶</code></pre>
    <p>回复：解析约束 O(N log N) → 提示"单调栈 + 二分"→ 逐步展开到完整代码 + 测试用例</p>
  </details>

  <details>
    <summary><b>算法讲解 — 精简 + 练习题</b></summary>
    <pre><code>讲讲线段树，顺便推荐几道题练手</code></pre>
    <p>回复：一句话醍醐灌顶 → 核心思路 → C++ 模板 → 复杂度 → CF/洛谷题号</p>
  </details>
</div>

---

<div style="text-align: center; margin: 1rem 0;">
  <div style="margin-bottom: 0.5rem;">
    <strong>Made with ❤️ for competitive programmers</strong>
  </div>
  <a href="https://opensource.org/licenses/MIT">
    <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License: MIT">
  </a>
</div>
