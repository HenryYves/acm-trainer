# acm-trainer 修改指南

当通过 `/plugin-dev:create-plugin` 或直接编辑修改本插件时，**必须参考本文件**。每次修改后**更新本文件的更新历史**。

---

## 交叉引用清单

新增或修改一个配置项时，以下文件**全部需要检查**：

| # | 文件 | 需要做什么 |
|---|------|-----------|
| 1 | `skills/acm-setup/SKILL.md` | 在初始化向导中添加收集该配置的步骤；在预览 (Preview) 和写入配置 (Write Config) 的 YAML 模板中加入新字段 |
| 2 | `skills/acm-config/SKILL.md` | 在"当前配置展示"中加入该字段的展示行；在"修改选项"中加入可勾选的条目；在 Step 3 (Apply Changes) 中加入对应的处理逻辑 |
| 3 | `skills/acm/SKILL.md` | 在 Startup 解析列表中列出该字段及**默认回退值**；在行为规则中描述该配置如何影响技能行为 |
| 4 | `README.md` | 在 YAML 配置示例中加入该字段 |
| 5 | `README-EN.md` | 同上（英文版） |
| 6 | `.claude/acm-trainer.local.md` (用户侧) | 无需修改插件，但需考虑**旧配置文件缺少新字段时的回退行为** |

### 交叉引用更新规则

- **acm-setup 步骤编号变更**：如果新增/删除步骤导致编号变化，必须同步更新 acm-config 中对 acm-setup 步骤号的引用
- **字段名一致性**：同一个字段在以上 6 处必须使用完全相同的名称
- **默认值一致性**：acm-setup 的默认选项和 acm 的 fallback 值必须一致

---

## 添加新配置项的完整步骤

以本次 `auto_edit_code` 为例：

1. **acm-setup/SKILL.md**
   - [x] 在合适位置插入新 Step（Step 4: Auto Edit Code）
   - [x] 后续步骤全部重新编号（4→5, 5→6, ... 11→12）
   - [x] 预览 (Step 10) 加入 `自动修改代码: <允许/不允许>`
   - [x] YAML 模板 (Step 11) 加入 `auto_edit_code: <true|false>`
   - [x] 检查内部交叉引用：`skip to Step 6` → `skip to Step 7`

2. **acm-config/SKILL.md**
   - [x] 当前配置展示加入 `自动修改代码: <允许/不允许>`
   - [x] 修改选项加入 `"自动修改代码"` 条目
   - [x] Step 3 加入处理逻辑：`For "自动修改代码": ask with the same question as acm-setup Step 4`
   - [x] 更新对 acm-setup 步骤号的引用（Step 8→9, Step 4+5+5a→5+6+6a）
   - [x] 预览示例加入 `自动修改代码: false → true`

3. **acm/SKILL.md**
   - [x] Startup 解析列表加入 `` `auto_edit_code` — whether to auto-edit/create code files (default false) ``
   - [x] 新增 Auto-Edit Behavior 章节描述两种模式行为

4. **README.md + README-EN.md**
   - [x] YAML 配置示例加入 `auto_edit_code: false`

5. **回退兼容** (本次后新增规则)
   - [x] acm/SKILL.md 中为每个字段标注 `(default xxx)` fallback

---

## 文件依赖关系图

```
acm/SKILL.md (主技能)
    ├── 启动时读取 → .claude/acm-trainer.local.md (用户配置)
    ├── 场景路由 → references/code-review.md
    ├── 场景路由 → references/workflows.md
    └── 修改参考 → .claude-plugin/MODIFICATION.md (本文件)

acm-setup/SKILL.md (初始化向导)
    ├── 读取模板 → template_path (用户指定)
    ├── 写入配置 → .claude/acm-trainer.local.md
    └── 被引用 ← acm-config/SKILL.md

acm-config/SKILL.md (配置管理)
    ├── 读取配置 → .claude/acm-trainer.local.md
    ├── 写入配置 → .claude/acm-trainer.local.md
    └── 引用步骤 → acm-setup/SKILL.md

README.md / README-EN.md
    └── 展示配置示例 → 需与 acm-setup YAML 模板同步
```

---

## 配置项完整清单

| 字段 | 类型 | 默认值 | 设置位置 (acm-setup) | 使用位置 (acm) |
|------|------|--------|---------------------|---------------|
| `code_location_mode` | enum | `none` | Step 2 | Code Location |
| `code_paths` | map | `{}` | Step 2 | Code Location |
| `progressive_hints` | bool | `true` | Step 3 | Teaching Approach |
| `auto_edit_code` | bool | `false` | Step 4 | Auto-Edit Behavior |
| `has_template` | bool | `false` | Step 5 | Template-Aware Review |
| `template_path` | string | `""` | Step 5 | Template-Aware Review |
| `template_boundary` | int | `0` | Step 6 | Template-Aware Review |
| `template_entry` | string | `""` | Step 6 | Template-Aware Review |
| `per_problem_constants` | list | `[]` | Step 6a | Template-Aware Review |
| `terminology` | enum | `mixed` | Step 7 | Language Rules |
| `solution_language` | enum | `cpp` | Step 8 | Solution Language |
| `time_limit_baseline` | int | `100000000` | Step 9 | Complexity Analysis |
| `config_version` | string | `"0.2.0"` | Step 11 (auto) | — (迁移检查) |
| `last_modified` | date | `""` | Step 11 (auto) | — (迁移检查) |

---

## 配置文件版本管理

用户配置文件 `.claude/acm-trainer.local.md` 包含 `config_version` 和 `last_modified`：

- **acm-setup**: 首次创建时写入当前插件版本和日期
- **acm-config**: 每次保存修改时更新 `last_modified`
- **acm**: 启动时检查 `config_version`，如果缺失或过旧，提示用户考虑重新初始化（不强制）

---

## 更新历史

| 日期 | 版本 | 变更 |
|------|------|------|
| 2026-05-09 | 0.2.2 | acm: 明确配置路径在项目根目录、加启动加载顺序规则避免过早加载参考文件、简单问题跳过参考文件；acm-setup: 新增 Step 13 询问是否自动配置项目权限 |
| 2026-05-07 | 0.2.1 | 新增 `auto_edit_code` 配置项；配置新增 `config_version`/`last_modified` 字段；acm 添加所有字段的回退默认值；创建本修改指南 |
| 2026-05-06 | 0.2.0 | 初始版本：acm/acm-setup/acm-config 三个技能，code_location/progressive_hints/terminology/solution_language/time_limit_baseline/template 配置 |
