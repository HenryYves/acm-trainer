# Code Review & Hack Data Generation

## Output Principle

**Do not output the complete corrected file.** The user has a potentially very long template. Show only:

```
// 原代码（第 N 行）
<the buggy lines>

// 修复后
<the fixed lines>
// <brief explanation of why>
```

Only output the complete file when the user explicitly says "给我完整代码".

## Constraint-Based Viability Check

Before diving into detailed bug scanning, check whether the code is fundamentally viable given the problem constraints. This catches catastrophic issues even when the correct solution isn't obvious.

### Space Complexity Check
- Compare the code's memory usage against constraints. If `n,m ≤ 1e9`, any `vector<T>(n)` or `vector<T>(m)` is immediately invalid — flag it as **Fatal**.
- `vector<vector<T>>(n, vector<T>(m))` with n,m ≤ 1e9 would need exabytes. An O(n) or O(m) space allocation when n,m are at 1e9 scale is a showstopper.

### Time Complexity Check
- Estimate the code's time complexity from its loop structure. Compare against the constraint-implied viable complexity (see workflows.md Safe N table).
- Example: nested loops over n (n ≤ 1e9) → O(n²) → immediately TLE. Flag even if the logic seems correct.

### Per-Problem Constant Check
- If config has `per_problem_constants`, verify each against the problem's actual constraints. Flag any that are too small.

Report these as **Fatal** before the detailed bug scan — they mean the approach needs fundamental rethinking, not just bug fixes. Even when the correct solution is unclear, pointing out "your O(n) space allocation with n=1e9 is impossible" is valuable feedback.

## Template-Aware Review

If user config has `has_template: true`, apply these rules:

1. Skip reviewing lines 1 to `template_boundary` — these are template, not user code.
2. Focus review on code after `template_entry`.
3. Specifically check for template gotchas recorded in `.claude/acm-trainer/template-summary.md`. Common ones:
   - `#define max/min` macros — conflict with `std::max`/`std::min`, double-evaluation
   - `#define endl '\n'` — `cout << endl` does not flush
   - Custom I/O (`rd()`/`pr()`) — do not mix with `cin`/`cout`
   - Missing `init_win_env()` call on Windows
4. When macros appear in user code, mentally expand them to check for hidden bugs (e.g., `max(i++, j--)` expands to `((i++) > (j--) ? (i++) : (j--))` — double increment/decrement).
5. If `per_problem_constants` is non-empty, check whether each constant's current value is appropriate for the problem's constraints. For example, if `maxn` is `1e5+20` but the problem states N ≤ 2×10⁵, flag it. This check applies to array sizes, modulo values, and other problem-specific constants.

## Bug Scan

Systematically check:

### Off-by-One
- Loop bounds: `i < n` vs `i <= n`
- Array indices: 0-indexed vs 1-indexed
- Binary search boundaries

### Integer Overflow
- Multiplication and accumulation — flag if values can exceed INT_MAX (~2.1×10⁹)
- Suggest `long long` (`ll` in the user's template) for counters, accumulators, and products
- `mid = (l + r) / 2` overflows when `l + r > INT_MAX`

### Uninitialized Variables
- Local variables read before assignment
- Arrays/vectors partially initialized

### Undefined Behavior
- Signed integer overflow
- Null pointer dereference
- Out-of-bounds access
- `stoi`/`stoll` on empty or non-numeric strings

### Modulo Mistakes
- Forgetting `+ MOD` before `% MOD` on negative numbers
- Using `%` where modulo arithmetic needs careful handling

## TLE Risk Scan

- Nested loops that could be optimized — e.g., O(N²) where O(N log N) exists
- `unordered_map` without custom hash — can degrade to O(N) per operation on adversarial inputs
- Unnecessary copying of large structures — pass by const reference
- Note: the user's template redefines `endl` as `'\n'`, so `cout << endl` does NOT flush. This is actually good for performance. Only flag if the user manually writes `std::endl`.

## WA (Wrong Answer) Risk Scan

- Missing edge cases: n=1, all equal values, negative numbers, empty input
- Floating-point comparison without epsilon
- Off-by-one in binary search boundaries
- Incorrect greedy assumption — does the greedy choice actually hold?

## RE (Runtime Error) Risk Scan

- Array/vector out-of-bounds access
- Stack overflow from deep recursion (suggest iterative or pragma)
- Division by zero
- Empty container access (`.front()`, `.back()`, `.top()` on empty)

## STL Review

| Pattern | Note |
|---------|------|
| `vector<bool>` | Not a proper container — it's a bitset. Use `vector<char>` or `bitset` instead. |
| `unordered_map` vs `map` | `unordered_map` is O(1) average but can degrade. Use `map` for deterministic O(log N). |
| `priority_queue` | Default is max-heap. Comparator direction is counterintuitive. |
| `set::lower_bound` vs `std::lower_bound` | The former is O(log N), the latter O(N) on set. |

## Hack Data Generation

After the bug scan, produce hack test cases. Each hack must target a specific bug found — do not produce generic boundary tests.

### Principles

1. **Target-specific**: Each hack precisely triggers one identified bug. If you found integer overflow, construct a case that overflows. If you found off-by-one, construct a case at the boundary.
2. **Follow the problem's input format**: The user should be able to copy-paste directly into stdin / the OJ test panel.
3. **Keep cases small** (≤ 20 elements for arrays) for manual verification. If the bug only manifests at large scale (e.g., overflow at N > 2×10⁵), show a conceptual case demonstrating the math plus a smaller runnable case.
4. **NO extra labels in data blocks**: The input and output blocks must contain ONLY the raw data in OJ format. Do NOT add Chinese labels like "输入:"、"期望:"、"输出:" before the data — these make the input non-copy-pasteable. The only allowed labels are the `//` comment lines in the hack template below, which are outside the data blocks.

### Hack Format

```
### Hack #N: <bug name>

// 输入（可直接复制到 OJ / stdin）
<raw input — EXACT problem format, no Chinese labels>

// 期望输出
<correct answer — raw value only>

// 实际输出
<what the buggy code produces>

// 触发位置
<file:line or logic section>
```

The `//` lines are markdown comments that stay in the output for readability. The data blocks between them must be pure raw data — the user can select and copy without editing.

**Correct example:**
```
### Hack #1: sort 破坏原序

// 输入
4 2
1 100 2 99

// 期望输出
101

// 实际输出
3
```

**Wrong (DO NOT do this):**
```
输入:
4 2
1 100 2 99
期望: 101
```
The labels "输入:" and "期望:" would be included when the user copy-pastes.

### Construction Strategies by Bug Type

- **Overflow**: Construct input that makes the expression exceed INT_MAX / LLONG_MAX
- **Off-by-one**: Construct input exactly at the boundary where loop runs one too many or too few
- **Logic error**: Construct input that violates the algorithm's core assumption
- **TLE risk**: Construct worst-case input for the algorithm's complexity
- **Float precision**: Construct values requiring high-precision comparison (e.g., 0.1 + 0.2 vs 0.3)

## Hack Verification

After generating all hack cases, check `exe_paths` from config. If the current keyword (the one the user used to refer to their code, e.g., "A") has an entry in `exe_paths`, auto-verify each hack:

1. **Resolve paths**: Get the value for the keyword. If it's a string, treat as a single-element list. If it's already a list, use as-is.
2. **Pick the newest**: For each path in the list, check if the file exists (e.g., `test -f "<path>"`). Among the existing ones, compare modification times with `stat -c %Y "<path>"`. Pick the one with the highest timestamp. If none exist, skip verification with a brief note: "exe 未找到（已检查 N 个路径）".
3. **Freshness check**: Compare the newest exe's modification time with the source `.cpp` file's modification time (the file at `code_paths[keyword]`).
   - If the newest exe is older than the source → warn: "⚠️ 所有 exe 都比源代码旧，可能未重新编译。建议重新编译后再验证。" Then skip verification (stale output is misleading).
   - If the newest exe is newer or same → proceed.
   - If multiple paths had newer exes, the freshness check still only compares the single newest one against the source.

4. Write the hack input to a temp file at `/tmp/acm_hack_in.txt`.
5. Run the selected exe via Bash, capturing both stdout and exit code:
   ```
   <selected_exe_path> < /tmp/acm_hack_in.txt 2>&1; echo "EXIT:$?"
   ```
   (On Windows with PowerShell fallback: `Get-Content /tmp/acm_hack_in.txt | <selected_exe_path> 2>&1; echo "EXIT:$?"`)
6. Extract the exit code from the `EXIT:N` marker. Interpret it:

### Exit Code Interpretation

When running a C++ exe compiled with MSVC on Windows via bash:

| Exit code / Pattern | 诊断 | 说明 |
|----------------------|------|------|
| `0` | 正常退出 | 继续比较输出 |
| `3` or `139` or `EXIT:-1073741819` | 段错误 (ACCESS_VIOLATION) | 数组越界、空指针解引用、野指针、访问已释放内存 |
| `136` or `EXIT:-1073741676` | 除零异常 (INTEGER_DIVISION_BY_ZERO) | 整数除零或模零操作 |
| `134` or `EXIT:-1073740940` | 断言/退出 (ABORT) | `assert()` 触发或 `abort()` 被调用 |
| `EXIT:-1073741571` | 栈溢出 (STACK_OVERFLOW) | 递归过深或局部数组太大 |
| 其他非零 `EXIT:N` | 程序异常退出 | 报告 exit code，检查是否有 stderr 输出 |

**When the program crashes**: Report as `💥 崩溃 (<诊断类型>)` rather than treating it as wrong output. Include the exit code and suggested cause. Example: "💥 崩溃: 段错误 (exit -1073741819) — 很可能是数组越界或空指针。"

7. If the program ran successfully (exit 0), capture stdout. Trim whitespace from both stdout and expected output before comparing.
8. Append to each hack entry:
   ```
   // 验证: ✅ 通过
   ```
   or
   ```
   // 验证: ❌ 失败 — 期望=X, 实测=Y
   ```
   or
   ```
   // 验证: 💥 崩溃 — 段错误 (exit -1073741819), 很可能是数组越界
   ```

If `exe_paths` has no entry for the keyword, or the entry is an empty list, skip verification silently. Do NOT fabricate paths or guess exe locations — only use the configured paths.


## Mistake Collection

`collect_mistakes.mode` has three modes: `"manual"` (never auto, only on explicit command), `"confirm"` (auto-summarize → ask user), `"auto"` (auto-save silently). See `acm/SKILL.md` Mistake Collection section for full mode descriptions. Before writing in auto/confirm modes, check `collect_mistakes.perm` — if false, warn about pending permission prompt.

### Saving Format (all modes)

1. Summarize each bug into a **generalizable** pattern — not tied to the specific problem. Categories:

   | 分类 | 示例 |
   |------|------|
   | 段错误/越界 | `k==0 后无 return，空 vector 被访问` |
   | 逻辑反转 | `相邻列 nx>px 与 nx<px 的转移公式互换` |
   | 未初始化 | `nowdown 在 know<klast 分支未赋值` |
   | 溢出 | `乘法前未取模，int 乘积溢出` |
   | 取模 | `减法取模忘 +MOD` |
   | 边界条件 | `指数 m-2 应为 m-1` |
   | 变量混淆 | `now==m 应改为 last==m（now 在 k=1 时未赋值）` |
   | 哨兵值设置错误 | `min/max初始化用了具体数字而非INF/-INF，初始值意外成为答案` |

2. Read `.claude/acm-trainer/mistakes.md` in the project root. If it doesn't exist, create it with a header:
   ```markdown
   # 常见编码错误记录
   
   自动收集，代码审查时参考。
   
   ```

3. For each pattern, check if it already exists (same category + same description text).
   - **New pattern**: append a new line.
   - **Existing pattern**: update the count and last-seen date (do NOT create a duplicate line).

4. Format each entry:
   ```
   - [<category>] <generalizable pattern description> — 次数: <N>, 首次: <YYYY-MM-DD>, 最近: <YYYY-MM-DD>
   ```
   - When creating: 次数 = 1, 首次 = 最近 = today.
   - When updating existing: increment 次数, update 最近 to today, leave 首次 unchanged.

5. Mention briefly: "已记录 N 个错误模式到知识库（其中 M 个为新增）。"

**Good vs bad descriptions:**
- ❌ `ans初始化为n导致n=3时错误` — 绑定具体题面
- ✅ `哨兵值设置错误 — min/max维护的初始值用了具体数字而非INF/-INF`
- ❌ `sqrt(x)应为sqrt(n)，小x时漏掉最优函数` — 脱离题面无意义
- ✅ `变量混淆 — 搜索半径/循环上限等算法参数使用了错误的变量`

### Referencing Mistakes

**Before or during** any code review (when `collect_mistakes.mode` is not `"manual"`), read `.claude/acm-trainer/mistakes.md`. Scan for patterns that match the current code. If a match is found, flag it:

```
⚠️ 历史错误再现：[<category>] <pattern> — 你之前在 <date> 犯过类似错误（共 <count> 次）。
```

Only flag if the pattern genuinely matches — don't force false positives.

### Manual Triggers

- "记录这个错误" / "收藏这个bug" → save the most recently discussed bug pattern (works in all modes)
- "查看错误记录" / "mistakes" → read and display `.claude/acm-trainer/mistakes.md`
- "清空错误记录" → delete the file (confirm first)
