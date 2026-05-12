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
3. Specifically check for template gotchas recorded in the config's template summary body. Common ones:
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

### Hack Format

```
### Hack #N: <bug name — what bug this triggers>

// 输入
<copy-pasteable input in the problem's format>

// 期望输出
<correct answer>

// 你的代码实际输出
<what the buggy code produces, or error message>

// 触发位置
<file:line or logic section hit by this hack>
```

### Construction Strategies by Bug Type

- **Overflow**: Construct input that makes the expression exceed INT_MAX / LLONG_MAX
- **Off-by-one**: Construct input exactly at the boundary where loop runs one too many or too few
- **Logic error**: Construct input that violates the algorithm's core assumption
- **TLE risk**: Construct worst-case input for the algorithm's complexity
- **Float precision**: Construct values requiring high-precision comparison (e.g., 0.1 + 0.2 vs 0.3)

## Hack Verification

After generating all hack cases, check `exe_paths` from config. If the current keyword (the one the user used to refer to their code, e.g., "A") has an entry in `exe_paths`, auto-verify each hack:

1. Write the hack input to a temp file at `/tmp/acm_hack_in.txt`.
2. Run the exe via Bash, piping the input file:
   ```
   <exe_path> < /tmp/acm_hack_in.txt
   ```
   (On Windows with PowerShell fallback: `Get-Content /tmp/acm_hack_in.txt | <exe_path>`)
3. Capture stdout. Trim whitespace from both stdout and expected output before comparing.
4. Append to each hack entry:
   ```
   // 验证: ✅ 通过
   ```
   or
   ```
   // 验证: ❌ 失败 — 期望=X, 实测=Y
   ```

If `exe_paths` has no entry for the keyword, or its value is empty, skip verification silently. Do NOT fabricate paths or guess exe locations — only use the configured path.
