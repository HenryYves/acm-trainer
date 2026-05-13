# Windows NT Exit Codes Reference

When a C++ program compiled with MSVC crashes, the exit code is the NT status code. bash reports it as a large negative signed 32-bit integer (e.g., `-1073741819` = `0xC0000005`). Use this table to map the number back to a meaningful diagnosis.

## Full Table

| Hex (NTSTATUS) | Unsigned Dec | Signed Dec | Name | ACM常见原因 |
|----------------|-------------|------------|------|-------------|
| `0xC0000005` | 3221225477 | -1073741819 | ACCESS_VIOLATION | 数组越界读写、空指针、野指针、释放后使用 |
| `0xC00000FD` | 3221225725 | -1073741571 | STACK_OVERFLOW | 超大局部数组、无限递归、局部变量过多 |
| `0xC0000094` | 3221225620 | -1073741676 | INTEGER_DIVISION_BY_ZERO | `x / 0` 或 `x % 0` |
| `0xC000001D` | 3221225501 | -1073741795 | ILLEGAL_INSTRUCTION | 栈返回地址被覆写、执行了垃圾数据、UB导致代码跑飞 |
| `0xC0000409` | 3221226505 | -1073740791 | STACK_BUFFER_OVERRUN | 栈数组越界写触发 VS `/GS` 安全检查 |
| `0x80070005` | 2147942405 | -2147024891 | ACCESS_DENIED | `freopen`/`fopen` 写了无权目录、文件为只读 |
| `0xC0000008` | 3221225480 | -1073741816 | INVALID_HANDLE | `CloseHandle` 传了无效句柄（ACM 罕见） |
| `0xC0000022` | 3221225506 | -1073741790 | STATUS_ACCESS_DENIED/INVALID_PARAMETER | 函数参数非法、内存状态错误（ACM 罕见） |
| `0xC0000006` | 3221225478 | -1073741818 | IN_PAGE_ERROR | 内存映射文件被删、磁盘错误（ACM 罕见） |
| `0xC0000135` | 3221225781 | -1073741515 | DLL_NOT_FOUND | 缺少 VC++ 运行库（`vcruntime140.dll` 等），非代码问题 |
| `0xC0000142` | 3221225794 | -1073741502 | DLL_INIT_FAILED | DLL 加载成功但初始化失败，非代码问题 |

## How bash Reports These

When running via bash:

```bash
./my_program.exe < input.txt 2>&1; echo "EXIT:$?"
```

If the program crashes, bash prints `EXIT:3221225477` (unsigned) or `EXIT:-1073741819` (signed), depending on the shell. Both are equivalent to `0xC0000005`.

- **Signed value** (negative): bash on Windows (Git Bash, msys2) typically reports this
- **Unsigned value** (large positive): WSL or some CI environments
- **Hex value**: sometimes visible in stderr as `0xC0000005` before the program terminates

Any of these three forms maps to the same error. Match whichever form you see.

## Which Codes Matter for ACM

**高频**（代码 bug）: `0xC0000005`, `0xC00000FD`, `0xC0000094`, `0xC000001D`, `0xC0000409`

**低频**（环境/权限）: `0x80070005`（写保护目录用 `freopen`）

**几乎不出现**: `0xC0000008`, `0xC0000022`, `0xC0000006`, `0xC0000135`, `0xC0000142`（这些通常是系统/安装问题）
