# acm-trainer 修改指南
> 导航索引。skill-creator、plugin-dev:create-plugin 及其他修改本插件的 skill/模型读此文件后按路径找目标，不搜索。
> 改完改版本号：.claude-plugin/plugin.json → version

## ⚠️⚠️⚠️ 改源目录，不改缓存（事不过三）
**你正在修改的是 Git 源目录**（`D:\a_my\project\Claude_code\plugins\acm-trainer\`），**不是** `~/.claude/plugins/cache/` 下的安装缓存。
缓存目录是用户安装插件时复制过去的副本，会被覆盖。你在源目录改完→用户重新安装→缓存更新。
**永远用 Write/Edit 改 `D:\a_my\project\Claude_code\plugins\acm-trainer\` 下的文件。** 如果不确定路径，先问用户。
本条警告优先级最高——之前两次都改错了缓存，第三次不能再犯。

## 文件清单
路径相对插件根目录。

##=== 核心文件 ===##
skills/acm/SKILL.md             -- 主技能：启动配置解析 + 场景路由 + 行为规则
skills/acm/references/code-review.md  -- 代码审查工作流（重场景才加载）
skills/acm/references/workflows.md    -- 题解/算法讲解/复杂度分析（重场景才加载）
skills/acm/references/windows-exit-codes.md -- Windows NT 退出码完整参考表（遇到未知退出码时加载）
skills/acm-config/SKILL.md      -- 配置管理：查看/修改用户配置
skills/acm-setup/SKILL.md       -- 初始化向导：首次配置引导
skills/acm-data/SKILL.md        -- 导入导出：错误记录和题解数据
README.md / README-EN.md        -- 用户文档

##=== 元数据 ===##
.claude-plugin/plugin.json          -- 插件清单（改版本号改这里）
.claude-plugin/marketplace.json     -- 市场信息
.claude-plugin/MODIFICATION.md      -- 你在这里
.claude-plugin/changelog/0.2.X.md   -- 每次修改的完整背景和教训

##=== 用户侧 ===##
.claude/acm-trainer.local.md           -- 用户配置文件（运行时读取，不在此插件内）
.claude/acm-trainer/template-summary.md -- 模板摘要（代码审查时按需加载，不在此插件内）

## 需求→文件 速查
| 我要做什么 | 读 |
|-----------|-----|
| 了解插件结构/修改规则 | MODIFICATION.md（本文件） |
| 改主技能行为 | skills/acm/SKILL.md |
| 改代码审查流程 | skills/acm/references/code-review.md |
| 改题解/算法讲解流程 | skills/acm/references/workflows.md |
| 改配置管理 | skills/acm-config/SKILL.md |
| 改初始化向导 | skills/acm-setup/SKILL.md |
| 改导入导出 | skills/acm-data/SKILL.md |
| 看历史修改背景 | .claude-plugin/changelog/<版本>.md |
| 改 README | README.md / README-EN.md |

## 交叉引用清单
新增/修改配置项时全部检查：
1. skills/acm-setup/SKILL.md   -- 加收集步骤 + YAML模板字段
2. skills/acm-config/SKILL.md  -- 加展示行 + 修改选项 + Step3处理逻辑 + Step5 whitelist
3. skills/acm/SKILL.md         -- 加字段解析 + 默认回退值 + 行为规则
4. README.md                   -- 加YAML示例字段
5. README-EN.md                -- 同上
6. .claude/acm-trainer.local.md（用户侧）-- 不改，但考虑旧配置缺字段时的回退

规则：acm-setup步骤编号变→同步acm-config引用；字段名6处一致；默认值acm-setup与acm一致；增删字段时同步更新acm-config Step5的schema whitelist

## 依赖关系
acm/SKILL.md → 启动读 .claude/acm-trainer.local.md
acm/SKILL.md → 场景路由 code-review.md / workflows.md
acm/SKILL.md → 修改参考 MODIFICATION.md
acm-setup/SKILL.md → 分析模板代码 → 写 .claude/acm-trainer.local.md（不记录模板路径）
acm-config/SKILL.md → 读/写 .claude/acm-trainer.local.md → 引用 acm-setup 步骤号
README.md / README-EN.md → 与 acm-setup YAML模板同步

## 配置项清单
字段 | 类型 | 默认 | acm-setup步骤 | acm章节
code_location_mode  | enum   | none        | Step 2  | Code Location
code_paths          | map    | {}          | Step 2  | Code Location
progressive_hints   | bool   | true        | Step 3  | Teaching Approach
auto_collect_solution|mapping|{mode:false, perm:false}| Step 4b | Solution Collection ({mode, perm})
auto_edit_code      | bool   | false       | Step 4  | Auto-Edit Behavior
has_template        | bool   | false       | Step 5  | Template-Aware Review
template_boundary   | int    | 0           | Step 6  | Template-Aware Review
template_entry      | string | ""          | Step 6  | Template-Aware Review
per_problem_constants|list  | []          | Step 6a | Template-Aware Review
terminology         | enum   | mixed       | Step 7  | Language Rules
solution_language   | enum   | cpp         | Step 8  | Solution Language
time_limit_baseline | int    | 100000000   | Step 9  | Complexity Analysis
exe_paths           | map    | {}          | Step 2b | Hack Verification
collect_mistakes    | mapping| {mode:"manual", perm:false} | Step 4c | Mistake Collection ({mode, perm})
config_version      | string | "0.2.13"    | Step 11 | 迁移检查（仅在配置格式变化时升）
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
- 永远改源目录不改缓存：用户指定开发目录（如 D:\...\plugins\acm-trainer\）是源，~/.claude/plugins/cache/ 是安装缓存会被覆盖。改插件前先确认源目录路径。(v0.2.6)
- 格式不兼容直接升级不搞兼容层：配置项格式变了就升级 config_version + 格式检测，让 acm-config 强制用户重配，不在 acm skill 里维护 backward compat 逻辑。(v0.2.16)
- AskUserQuestion 每问题最多 4 个选项：配置页面设计时每个 question 的 options 数组不能超过 4 项，否则报 `too_big` 错误。超限时拆成更多 question。(v0.2.14)

## 更新历史
日期 | 版本 | 变更 | 详情
2026-06-16 | 0.2.18 | exe_paths多路径+perm主动同步 | .claude-plugin/changelog/0.2.18.md
2026-05-26 | 0.2.17 | 权限自动配置+收紧+拒绝记忆 | .claude-plugin/changelog/0.2.17.md
2026-05-22 | 0.2.16 | 配置mapping格式+perm跟踪+acm-data导入导出；无向后兼容；config_version→0.2.13 | .claude-plugin/changelog/0.2.16.md
2026-05-19 | 0.2.15 | collect_mistakes三态枚举(manual/confirm/auto)；错误描述泛化；config_version→0.2.11 | .claude-plugin/changelog/0.2.15.md
2026-05-19 | 0.2.14 | acm-config选项数修正；错误收集增强（计数+时间+强制执行）；MODIFICATION.md醒目警告 | .claude-plugin/changelog/0.2.14.md
2026-05-19 | 0.2.13 | 模板摘要拆分为独立文件；重新分析模板需diff对比 | .claude-plugin/changelog/0.2.13.md
2026-05-13 | 0.2.12 | 移除duipai_exe_paths配置项；config_version→0.2.10；新增windows-exit-codes参考表 | .claude-plugin/changelog/0.2.12.md
2026-05-12 | 0.2.8 | acm-config: 3次AskUserQuestion合并为1次← →翻页 | .claude-plugin/changelog/0.2.8.md
2026-05-12 | 0.2.7 | 新增exe_paths配置+自动验证hack数据 | .claude-plugin/changelog/0.2.7.md
2026-05-12 | 0.2.6 | 删除template_path；Code Location防呆修复 | .claude-plugin/changelog/0.2.6.md
2026-05-10 | 0.2.5 | 权限配置+插件目录；配置版本≠插件版本；remind开关；Config独揽更新 | .claude-plugin/changelog/0.2.5.md
2026-05-10 | 0.2.4 | acm: Code Location段禁搜索; acm-config: 拆Step2+权限选项 | .claude-plugin/changelog/0.2.4.md
2026-05-09 | 0.2.3 | acm: 删Startup段Glob备选方案 | .claude-plugin/changelog/0.2.3.md
2026-05-09 | 0.2.2 | acm: 明确路径+启动顺序+跳过参考；acm-setup: Step13权限 | .claude-plugin/changelog/0.2.2.md
2026-05-07 | 0.2.1 | 新增auto_edit_code；配置版本字段；创建本指南 | —
2026-05-06 | 0.2.0 | 初始版本：三技能+基础配置 | —
