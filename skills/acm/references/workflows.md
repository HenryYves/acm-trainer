# Workflows: Problem Solving, Algorithm Explanation, Complexity Analysis

## Problem Solving Walkthrough

When the user pastes a problem statement with constraints:

0. **Check solution collection** — if `.claude/acm-trainer/solutions/index.md` exists, scan for matching tags or similar problem patterns. If a match is found, read the detail file and reference its "如何想到" guide. Mention it: "之前在收录的题解里找到一道类似的题……"
1. **Parse constraints** — N, time limit, memory limit. These determine viable complexities:
   - N ≤ 10⁵ → O(N log N) max
   - N ≤ 5000 → O(N²) ok
   - N ≤ 20 → bitmask DP possible
2. **Infer expected complexity & algorithm family** — work backwards from constraints to the viable time complexity, then to what algorithm families could achieve it. State this explicitly even if the exact solution isn't clear yet. Examples:
   - n,m ≤ 1e9, k ≤ 1e5 → must be O(k log k) or O(k log m); O(nm) or O(n) are impossible
   - n ≤ 2×10⁵, single array → O(n log n) ceiling; segment tree / BIT / sorting / divide-and-conquer are candidates
   - n ≤ 20 → O(2ⁿ) or O(n·2ⁿ); bitmask DP or meet-in-the-middle
   This step rules out entire families of approaches and narrows the search space significantly.
3. **Tag the problem type** — DP, graph, greedy, math, data structure, string, geometry, etc.
4. **State the key constraint insight** — e.g., "每列至多一个障碍 → 可以按列处理", "N is only 2000, so O(N²) is fine"
5. **Give a high-level direction first** — 1-2 sentences, not code
6. If progressive hints mode: ask if the user wants to continue reasoning or get more hints
7. **Bail out if stuck** — after ~3 rounds of reasoning without a viable direction, or if you notice yourself cycling through the same ideas, admit difficulty (see main SKILL.md "Admitting Difficulty" section). Do not keep trying unproductive approaches indefinitely.
8. Discuss edge cases before showing final code
9. After providing solution, include 1-2 small test cases in OJ input format for the user to verify

## Algorithm Explanation

When the user asks about a specific algorithm or data structure:

1. **醍醐灌顶句** — one sentence: what problem this solves, when to use it, and how it differs from easily-confused alternatives
2. **Intuition first** — the core idea, not the mechanics
3. **Mechanics** — how it works step-by-step
4. **Code implementation** — clean, minimal code in the configured `solution_language` (C++ or Python). Only the core parts, not over-engineered. If `match_code`, use the language of the user's current code context.
5. **Complexity** — time and space
6. **When to use** — problem patterns that signal this algorithm
7. **Language-specific gotchas** — for C++: recursion depth, STL pitfalls, performance notes. For Python: recursion limit, input speed, list vs deque, default recursion depth.
8. **Practice problems** — 3-5 problems from CF/Luogu/AtCoder with difficulty order and short descriptions

Keep explanations concise. Practice problems are more valuable than long explanations.

## Complexity Analysis

When analyzing time/space complexity:

1. Break down each section of the code with its cost
2. State overall complexity in big-O
3. Compare against problem constraints — does it pass?
4. Identify the bottleneck if there is one
5. Suggest algorithmic improvements with reasoning (not just "use a segment tree" but WHY it helps)

### 1-Second Time Limit Guidelines

Compute Safe N from `time_limit_baseline` (config value, default 1e8 = O(N) safe per second):

```
B = time_limit_baseline

O(N log N) safe ≈ B / 20
O(N) safe     = B
O(N√N) safe   ≈ √B × 0.7
O(N²) safe    ≈ √B × 0.5
O(2^N) safe   ≈ log₂(B) × 0.8
```

| Complexity | Safe N (C++, B=1e8) | Safe N (Python, B/30) |
|-----------|---------------------|------------------------|
| O(N log N) | ~5×10⁶ | ~1.7×10⁵ |
| O(N) | ~1×10⁸ | ~3.3×10⁶ |
| O(N√N) | ~7×10³ | ~1.3×10³ |
| O(N²) | ~5×10³ | ~900 |
| O(2^N) | ~21 | ~21 |

Python is approximately 30× slower than C++. For Python solutions, use B/30 as the effective baseline.
