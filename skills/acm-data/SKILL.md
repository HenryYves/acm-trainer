---
name: acm-data
description: This skill should be used when the user asks to export or import ACM training data — mistake collections and solutions. Triggers on "导出错误", "导出题解", "导入错误", "导入题解", "分享acm", "acm data", "备份acm", "恢复acm", or similar requests involving acm-trainer's collected data.
allowed-tools: Read, Write, Bash, Glob
---

# ACM Trainer Data — 导入导出

Handles export and import of `.claude/acm-trainer/mistakes.md` and `.claude/acm-trainer/solutions/` data.

## Export

Trigger: "导出错误", "导出题解", "分享acm数据", "备份acm", etc.

1. Read `.claude/acm-trainer/mistakes.md` and `.claude/acm-trainer/solutions/index.md` in the project root. If neither exists, say "暂无数据可导出。" and exit.
2. Create `acm-export/` in the current working directory (project root).
3. Copy `.claude/acm-trainer/mistakes.md` → `acm-export/mistakes.md` (if exists).
4. Copy `.claude/acm-trainer/solutions/` → `acm-export/solutions/` recursively (if exists). Use Bash: `cp -r .claude/acm-trainer/solutions acm-export/`.
5. Tell the user: "数据已导出到 `acm-export/` 文件夹。压缩后分享即可。"

## Import

Trigger: "导入错误", "导入题解", "恢复acm数据", etc. User specifies a source folder path.

1. Ask the user for the source folder path if not provided.
2. Verify the source folder exists and contains at least one of `mistakes.md` or `solutions/`.
3. **Merge mistakes.md**: If source has `mistakes.md`:
   - Read source mistakes.md and local `.claude/acm-trainer/mistakes.md` (if exists).
   - For each mistake entry in source, check if a similar entry (same category + similar description) exists locally.
   - New entries: append to local mistakes.md. Existing entries: skip or note "已存在".
   - If local mistakes.md doesn't exist, create it and copy source content with the standard header.
4. **Merge solutions/**: If source has `solutions/`:
   - Read source `solutions/index.md`. For each entry, check if it exists in local `.claude/acm-trainer/solutions/index.md` (by problem name or slug).
   - New entries: copy the detail file and append to local index.
   - Existing entries: skip (don't overwrite user's own solutions).
5. Report: "导入完成。新增 N 条错误记录，M 道题解。"
