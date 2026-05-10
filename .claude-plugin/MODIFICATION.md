# acm-trainer 修改指南
> 导航索引。skill-creator、plugin-dev:create-plugin 及其他修改本插件的 skill/模型读此文件后按路径找目标，不搜索。
> 改完改版本号：.claude-plugin/plugin.json → version

## 文件清单
路径相对插件根目录。

##=== 核心文件 ===##
skills/acm/SKILL.md             -- 主技能：启动配置解析 + 场景路由 + 行为规则
skills/acm/references/code-review.md  -- 代码审查工作流（重场景才加载）
skills/acm/references/workflows.md    -- 题解/算法讲解/复杂度分析（重场景才加载）
skills/acm-config/SKILL.md      -- 配置管理：查看/修改用户配置
skills/acm-setup/SKILL.md       -- 初始化向导：首次配置引导
README.md / README-EN.md        -- 用户文档

##=== 元数据 ===##
.claude-plugin/plugin.json          -- 插件清单（改版本号改这里）
.claude-plugin/marketplace.json     -- 市场信息
.claude-plugin/MODIFICATION.md      -- 你在这里
.claude-plugin/changelog/0.2.X.md   -- 每次修改的完整背景和教训

##=== 用户侧 ===##
.claude/acm-trainer.local.md    -- 用户配置文件（运行时读取，不在此插件内）

## 需求→文件 速查
| 我要做什么 | 读 |
|-----------|-----|
| 了解插件结构/修改规则 | MODIFICATION.md（本文件） |
| 改主技能行为 | skills/acm/SKILL.md |
| 改代码审查流程 | skills/acm/references/code-review.md |
| 改题解/算法讲解流程 | skills/acm/references/workflows.md |
| 改配置管理 | skills/acm-config/SKILL.md |
| 改初始化向导 | skills/acm-setup/SKILL.md |
| 看历史修改背景 | .claude-plugin/changelog/<版本>.md |
| 改 README | README.md / README-EN.md |

## 交叉引用清单
新增/修改配置项时全部检查：
1. skills/acm-setup/SKILL.md   -- 加收集步骤 + YAML模板字段
2. skills/acm-config/SKILL.md  -- 加展示行 + 修改选项 + Step3处理逻辑
3. skills/acm/SKILL.md         -- 加字段解析 + 默认回退值 + 行为规则
4. README.md                   -- 加YAML示例字段
5. README-EN.md                -- 同上
6. .claude/acm-trainer.local.md（用户侧）-- 不改，但考虑旧配置缺字段时的回退

规则：acm-setup步骤编号变→同步acm-config引用；字段名6处一致；默认值acm-setup与acm一致

## 依赖关系
acm/SKILL.md → 启动读 .claude/acm-trainer.local.md
acm/SKILL.md → 场景路由 code-review.md / workflows.md
acm/SKILL.md → 修改参考 MODIFICATION.md
acm-setup/SKILL.md → 读 template_path → 写 .claude/acm-trainer.local.md
acm-config/SKILL.md → 读/写 .claude/acm-trainer.local.md → 引用 acm-setup 步骤号
README.md / README-EN.md → 与 acm-setup YAML模板同步

## 配置项清单
字段 | 类型 | 默认 | acm-setup步骤 | acm章节
code_location_mode  | enum   | none        | Step 2  | Code Location
code_paths          | map    | {}          | Step 2  | Code Location
progressive_hints   | bool   | true        | Step 3  | Teaching Approach
auto_edit_code      | bool   | false       | Step 4  | Auto-Edit Behavior
has_template        | bool   | false       | Step 5  | Template-Aware Review
template_path       | string | ""          | Step 5  | Template-Aware Review
template_boundary   | int    | 0           | Step 6  | Template-Aware Review
template_entry      | string | ""          | Step 6  | Template-Aware Review
per_problem_constants|list  | []          | Step 6a | Template-Aware Review
terminology         | enum   | mixed       | Step 7  | Language Rules
solution_language   | enum   | cpp         | Step 8  | Solution Language
time_limit_baseline | int    | 100000000   | Step 9  | Complexity Analysis
config_version      | string | "0.2.4"     | Step 11 | 迁移检查（仅在配置格式变化时升）
remind_config_update| bool   | true        | Step 11 | 是否在配置版本落后时提醒
last_modified       | date   | ""          | Step 11 | 迁移检查

## 版本管理
- acm-setup: 首次创建写入配置版本+日期
- acm-config: 每次保存更新 last_modified；补全配置时更新 config_version
- **配置版本号 ≠ 插件版本号**：config_version 只在配置格式（字段增删）变化时升级，不与 plugin.json 版本同步
- acm: 启动时若 remind_config_update 为 true 且 config_version 落后，提醒用户运行 acm-config 补全
acm: 启动检查 config_version，过旧提示用户可重新初始化（不强制）

## 核心教训
- MODIFICATION.md写给AI不是人：用扁平列表不用ASCII树，用箭头不用框图，用紧凑格式不用装饰性分隔线和blockquote。省token=省时间。(v0.2.3)
- changelog文件也是AI向：README结构里只列第一个版本+省略号，避免每次新增版本都要改README。(v0.2.3)
- 不提工具名=不用该工具：指令里出现Glob/bash等工具名，模型就可能去用它。阻止行为的方式是完全不提。(v0.2.3)
- 路径必须写锚点：相对路径被模型猜错，写"project root / current working directory"。(v0.2.2)
- 参考文件按需加载：Scenario Routing只对"重"场景触发reference，简单问答用skill正文。(v0.2.2)
- 权限合并是追加式：acm-setup Step13不覆盖已有.claude/settings.local.json。(v0.2.2)
- config_version同步升级：改acm-setup Step11的YAML字段时，必须同步bump config_version，且acm/SKILL.md和acm-config/SKILL.md的版本检查阈值一起更新。4处硬编码保持一致。(v0.2.4)

## 更新历史
日期 | 版本 | 变更 | 详情
2026-05-10 | 0.2.5 | 权限配置+插件目录；配置版本≠插件版本；remind开关；Config独揽更新 | .claude-plugin/changelog/0.2.5.md
2026-05-10 | 0.2.4 | acm: Code Location段禁搜索; acm-config: 拆Step2+权限选项 | .claude-plugin/changelog/0.2.4.md
2026-05-09 | 0.2.3 | acm: 删Startup段Glob备选方案 | .claude-plugin/changelog/0.2.3.md
2026-05-09 | 0.2.2 | acm: 明确路径+启动顺序+跳过参考；acm-setup: Step13权限 | .claude-plugin/changelog/0.2.2.md
2026-05-07 | 0.2.1 | 新增auto_edit_code；配置版本字段；创建本指南 | —
2026-05-06 | 0.2.0 | 初始版本：三技能+基础配置 | —
