# ACM Trainer

ACM/ICPC 算法竞赛训练助手。提供题解、代码审查、算法讲解、复杂度分析功能。支持用户配置持久化，一次初始化后续所有 session 受益。

## 安装

Plugin 位于项目 `.claude/plugins/acm-trainer/` 目录。启动 Claude Code 时使用：

```bash
cc --plugin-dir .claude/plugins/acm-trainer
```

## 使用

### 首次使用

```
/acm-trainer:acm-setup
```

运行初始化向导，配置代码位置、引导方式、模板分析、语言偏好。配置保存到 `.claude/acm-trainer.local.md`。

### 日常使用

```
/acm-trainer:acm          # 主入口（自动匹配：题目求解、代码审查、算法讲解、复杂度分析）
/acm-trainer:acm-config   # 查看/修改配置
```

主 skill 会自动激活——贴题目、贴代码、问算法时无需手动调用 `/acm-trainer:acm`。

### 配置管理

配置存储在项目根目录 `.claude/acm-trainer.local.md`（YAML frontmatter + markdown body）。建议加入 `.gitignore`：

```gitignore
.claude/*.local.md
```

## 结构

```
acm-trainer/
├── .claude-plugin/plugin.json
├── skills/
│   ├── acm/               # 主训练 skill
│   │   └── references/    # 详细工作流（按需加载）
│   ├── acm-setup/          # 初始化向导
│   └── acm-config/         # 配置管理
└── README.md
```

## 注意

与旧版 `.claude/skills/acm-trainer/` 不兼容。迁移到 plugin 后建议删除或归档旧版 skill。
