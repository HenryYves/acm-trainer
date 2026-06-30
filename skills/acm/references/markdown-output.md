# Markdown Output 格式规范

题解/算法讲解/代码审查输出到 `.md` 文件时的格式规则。目标兼容 Typora 等本地 Markdown 编辑器。

## 等宽字体 + `<pre>` —— 手画比例线的正确做法

### 问题

在非等宽字体（比例字体）下，每个字符宽度不同：`i` 窄、`W` 宽、`-` 和 `—` 宽度也各异。手工画 ASCII 比例线（如 `|----|------|`）时无法靠数字符来控制列宽比例，对不齐。

```markdown
<!-- ❌ 在比例字体下渲染出来对不齐 -->
|--------|------|
|  短列  |  长列  |
```

### 解决

用 HTML `<pre>` 标签 + 等宽字体。等宽字体每个字符宽度完全相同（英文 1em，中日文 2em），所以可以**直接数格子**来确定宽度比例——8 个 `-` 正好是 4 个 `-` 的两倍宽。

推荐开源等宽字体：**霞鹜文楷等宽** (LXGW WenKai Mono)。备选：更纱黑体 (Sarasa Mono SC)、Noto Sans Mono CJK SC。

```html
<pre style="font-family: 'LXGW WenKai Mono', 'Sarasa Mono SC', 'Noto Sans Mono CJK SC', 'Courier New', monospace;">
l=0: s[0..4] 原本 f=1 → 加上(3,4) → f=2  (+1)
l=1: s[1..4] 原本 f=0 → 加上(3,4) → f=1  (+1)
l=2: s[2..4] 原本 f=0 → 加上(3,4) → f=1  (+1)
l=3: s[3..4] 原本 f=0 → 加上(3,4) → f=1  (+1)
l=4: s[4..4] 不包含位置3，不受影响      → f=0  (+0)
</pre>
```

### 何时用 `<pre>` + 等宽

| 场景 | 方案 |
|------|------|
| 手画 ASCII 比例线（`|----|------|`） | `<pre>` + 等宽字体 |
| 多行对齐注释（每行 `←` 注释对齐到同一列） | `<pre>` + 等宽字体 |
| 数学公式 | LaTeX `$$...$$`（Typora 的 MathJax 自动渲染） |
| 简单键值对照、参数列表 | markdown table |
| 普通文本 | 正常段落 |

### 易错点

1. **不要在 markdown 段落外面手写 ASCII 线**（`|--------|----|`），非等宽下必错位。
2. **`<pre>` 内的内容不会被转义成 markdown 语法**——`|`、`*`、`_` 等字符安全，不会变成表格。
3. **HTML 标签之间不能有空行**，Typora 会拆成多个独立块。

## Typora 兼容规则

### 规则 1：HTML 标签之间不留空行

```html
<!-- ❌ -->
<details>
<summary>展开</summary>

<p>内容</p>
</details>

<!-- ✅ -->
<details>
<summary>展开</summary>
<p>内容</p>
</details>
```

### 规则 2：块级元素居中用 text-align

`<p>` 是块级元素，`margin: 0 auto` 必须配合显式 `width` 才生效。居中直接用 `text-align: center`。

### 规则 3：字数不等的标签用分散对齐

```html
<span style="display:inline-block;width:5em;text-align:justify;text-align-last:justify;">申请人</span>
```

### 规则 4：防止 inline-block 拆行用 flex

`white-space: nowrap` 管不住 inline-block 元素的换行。改用 `display: flex; flex-wrap: nowrap`。

```html
<p style="display:flex;flex-wrap:nowrap;align-items:baseline;">
  <span style="display:inline-block;width:5em;text-align:justify;text-align-last:justify;">标签</span>
  <span>：</span>
  <span style="display:inline-block;width:14em;">内容</span>
</p>
```

### 规则 5：HTML 标签内不解析 Markdown

`<p>`、`<div>` 等 HTML 标签内部不解析 Markdown 语法。加粗用 `<strong>`，代码用 `<code>`。

## 文件结构约定

每次追加写入一条记录，用 `---` 水平线分隔：

```markdown
## 2026-06-23 括号串权值 — 贡献法 + 贪心匹配

<内容>

---

## 2026-06-24 某题 — 线段树

<内容>

---
```

## 追加内容的双向链接

当向已有 `.md` 文件追加补充内容（如用户追问某个概念，需在原文相关章节后补充）时，必须建立双向锚点链接，方便在 Typora 中跳转。

### 锚点格式

GitHub / Typora 自动为每个 heading 生成 ID，规则：标题文字 lowercased + 空格变 hyphens + 去除标点符号。

```markdown
[文字](#heading-的-锚点形式)
```

更可靠的写法是显式 `<a>` 标签（不受标题文字变更影响）：

```markdown
<a id="my-anchor"></a>
### 某章节
```

### 链接模式

**原文中（插入前向链接）**——紧接被补充的段落之后：

```
> 📝 补充：[标题文字](#锚点名)
```

**新内容末尾（插入返回链接）**——新章节最后一行：

```
> ⬆ 返回：[被补充的章节标题](#锚点名)
```

### 完整示例

原始 `my.md`：

```markdown
## 核心推导

定义 $v_i$ = square-free core...

> 📝 补充：[三个 LCA 的树性质详解](#补充-lca)

### 贡献法计数
```

需要追加时，两步修改：

**1. 在原文段落之后插入前向链接（Edit 操作）**

```markdown
## 核心推导

...三个 LCA 中，2 个等于 lca(u,v,w)，第 3 个可能更深...

> 📝 补充：[三个 LCA 的树性质详解](#补充-lca)
```

**2. 在末尾 `---` 之前追加新章节 + 返回链接**

```markdown
### <a id="补充-lca"></a>补充：三个 LCA 的树性质详解

**第一步：三种点的两种 pattern**

...

> ⬆ 返回：[核心推导](#核心推导)

---
```

### 修改流程

1. 读 `.md` 文件，找到被补充的章节和标题
2. 在目标段落之后，用 Edit 插入 `> 📝 补充：[...]` 前向链接
3. 在文件末尾 `---` 之前，用 Edit 追加新章节（带 `<a id="...">` 锚点 + `> ⬆ 返回：[...]` 链接）
4. 若原文已有 `📝 补充` 链接，新链接追加在上一个后面
