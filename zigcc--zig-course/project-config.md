---
trigger: always_on
description: > 本文件面向后续维护本目录的 **AI Agent / LLM** 与 **人类开发者**。
---

# AGENTS.md — zig-course PDF 导出器

> 本文件面向后续维护本目录的 **AI Agent / LLM** 与 **人类开发者**。
> 阅读后应能：理解整体设计、在不破坏既有特性的前提下扩展功能、把工具集成进
> zig-course（VitePress）项目，并避开已知的若干隐蔽陷阱。
>
> 适用范围：`scripts/pdf/` 目录下的全部代码。修改本目录代码时请优先遵循本文件的约定。

---

## 1. 这是什么

一个**不依赖无头浏览器**的离线 PDF 生成器，把整套 zig-course 课程渲染为带书签、
可点击站内/外链的单一 PDF（`books/zig_course.pdf`）。它替代了旧的
`vitepress-export-pdf`（基于 Puppeteer）方案。

核心实现思路：直接复用项目的 `course/.vitepress/sidebar.ts` 作为目录与顺序的**唯一数据源**，
逐篇读取 Markdown → 预处理与分词（`marked`）→ 用 [`jsPDF`](https://github.com/parallax/jsPDF)
矢量绘制 → 收集并绑定跳转链接 → 写入 PDF outline（书签）→ 输出文件。代码高亮使用
VitePress 同款的 [Shiki](https://shiki.style/) 引擎。

### 运行方式

```bash
bun pdf          # 全量构建 -> books/zig_course.pdf
bun pdf:sample   # 仅渲染几篇代表页 -> books/zig_course_sample.pdf（快速验证，秒级）
```

两个脚本都通过 [Bun](https://bun.sh) **直接执行 TypeScript**（`bun run scripts/pdf/main.ts`），
无需预编译或 tsx。类型检查（不产出文件）：

```bash
node_modules/.bin/tsc --noEmit -p scripts/pdf/tsconfig.json
```

---

## 2. 目录与模块职责

| 文件            | 职责                                                                                         |
| --------------- | -------------------------------------------------------------------------------------------- |
| `main.ts`       | 入口：`import sidebar`，扁平化为有序节点，逐页渲染，创建 PDF 书签（outline），写出文件       |
| `parse.ts`      | Markdown **预处理 + 分词**：展开代码引用、转换 GitHub alert、解析 VitePress 容器，产出 token |
| `renderer.ts`   | **核心布局引擎**：标题/段落/列表/表格/代码块/图片/提示框/引用块绘制，链接坐标收集与最终绑定  |
| `highlight.ts`  | 用 Shiki 把代码着色为 `{content, color}` 片段（只取 token 颜色，不生成 HTML）                |
| `utils.ts`      | sidebar 扁平化、站内链接归一化、`slugify`、图片路径解析、代码引用 `<<<@/...` 解析 + dedent   |
| `tsconfig.json` | 仅供本目录 `tsc --noEmit` 类型检查与编辑器使用，**不参与** VitePress 构建                    |
| `README.md`     | 面向使用者的简明说明                                                                         |
| `AGENTS.md`     | 本文件，面向维护者/Agent 的深入说明                                                          |

> 模块依赖方向（无环）：`main → {parse, renderer, highlight, utils}`；
> `parse → utils`；`renderer → {utils, highlight, parse(类型)}`。

---

## 3. 数据流（端到端）

```
sidebar.ts
   │  flattenSidebar()               (utils.ts)
   ▼
FlatNode[]  ──filter EXCLUDE/SAMPLE──►  有序页面/分组节点          (main.ts)
   │  对每个页面节点 route:
   │    readFile(route.md)
   ▼
parseMarkdown(content, courseDir)                                  (parse.ts)
   │  预处理 step A→D（见 §4）后 marked.lexer，再把 [[ADMONITION]] 收拢为自定义 token
   ▼
PdfToken[]
   │  renderer.renderPage(route, title, tokens)                    (renderer.ts)
   │    renderTokens() 按 token.type 分发到各 drawXxx；
   │    标题登记 anchors，页面登记 routeStart，链接坐标入 pendingLinks
   ▼
（全部页面渲染完）
   │  renderer.finalize()   用 anchors/routeStart 把 pendingLinks 绑定为 doc.link 跳转
   │  main.ts 同步用 outline.add() 写书签
   ▼
renderer.output()  ->  books/zig_course.pdf
```

---

## 4. Markdown 预处理管线（`parse.ts` 的 `preprocess`）

按**固定顺序**执行，顺序不可随意调换：

1. **step A — 去除 front matter**：删除文件开头 `--- ... ---` 区块。
2. **step B — 展开代码引用**：把 `<<<@/code/xxx.zig#anchor` 行替换为对应代码 fence。
   实际抽取在 `utils.resolveCodeImport`（见 §6）。
3. **step B2 — GitHub alert**：把 `> [!NOTE|TIP|IMPORTANT|WARNING|CAUTION|DETAILS]`
   多行 blockquote 转成统一的 `[[ADMONITION:type:title]] ... [[/ADMONITION]]` 标记块。
4. **step C — VitePress 容器**：用**栈**跟踪 `::: info/tip/warning/danger/details`、
   `code-group/raw/v-pre`（支持 `:::` 数量 ≥3 的嵌套）。提示类容器转成 `[[ADMONITION]]` 标记块；
   `code-group` 等仅去除围栏标记、保留内部内容。
5. **step D — 剔除属性指令**：去掉图片/链接尾部的 `{data-zoomable}`、`{.class}` 等 VitePress 属性。

之后 `parseMarkdown` 调用 `marked.lexer`，再用递归的 `collapse()` 把文本里的
`[[ADMONITION:...]] / [[/ADMONITION]]` 标记**收拢为嵌套的 `admonition` 自定义 token**。

> **设计动机**：把"容器/alert"在文本层先归一成统一标记，避免在 `marked` token 树里
> 处理 VitePress 私有语法；渲染器只需认识一种 `admonition` token。

---

## 5. 渲染引擎要点（`renderer.ts`）

### 5.1 坐标系与单位

- 单位为 **mm**（`new jsPDF({ unit: "mm", format: "a4" })`）。A4 = 210×297mm，页边距见 `MARGIN`。
- **字号是 pt**，坐标是 mm。两者换算 `1pt ≈ 0.3528mm`。涉及"按字号计算垂直偏移"时
  **必须显式换算**（如列表圆点的 `dotCy`），不要把 pt 当 mm 直接相加——这是历史 bug 来源。

### 5.2 三字体与正文/代码分流

构造时加载三套 glyf 型 TrueType 字体：

- `CJK`（思源宋体）→ 中文与 CJK 标点；
- `Sans`（Inter）→ **正文英文/数字**（无衬线比例字体）；
- `Mono`（JetBrains Mono）→ **代码块与行内代码**（等宽）。

分流规则（`isCjk(ch)` 逐字符判定 + 是否代码上下文）：CJK 字符走 `CJK`；代码（fence 代码块、
`codespan` 行内代码）走 `Mono`；其余正文拉丁走 `Sans`。字体子集**不含 emoji**，因此提示框
标题会先经 `cleanAdmonitionTitle` 去除 emoji，否则会出现缺字形并把文字推偏。

> 字体由 `build-fonts.ts`（纯 Bun/JS，`subset-font`/harfbuzz）从 Google Fonts 的 glyf 型
> **可变字体**「子集 + 钉轴（wght=400 等）」生成静态 TTF。**jsPDF 只能内嵌 glyf 型 TrueType**，
> 三个源（Noto Serif SC / Inter / JetBrains Mono）都是 glyf，无需任何 CFF→glyf 转换。
> 换字体务必选 glyf 型来源（CFF/OTF 会被 jsPDF 静默拒绝、渲染空白）。

### 5.3 两遍链接绑定（核心机制）

jsPDF 顺序绘制、无法回溯修改链接。因此采用"**先收集坐标，最后统一绑定**"：

- 渲染时：标题登记到 `anchors`（key = `` `${route}#${slug}` ``），页面登记到 `routeStart`，
  每个站内链接的矩形热区入 `pendingLinks`（含其所在页 `page`）。
- `finalize()`：对每个 `pendingLink` 查 `anchors[route#anchor]` → 退化到 `anchors[route#]`（页首）
  → 再退化到 `routeStart[route]`，命中后 `doc.setPage(link.page)` 并 `doc.link(...)` 绑定跳转。

### 5.4 页号的真实来源：`curPage()`（务必理解）

存在两个"页号"：

- `this.page`：**内部计数器**，`newPage()` 时自增，用于跨页测高的增量判断；
- `doc.getCurrentPageInfo().pageNumber`：jsPDF **文档真实页号**。

两者可能因两遍渲染产生 1 页漂移。**所有"用于记录跳转目标 / 链接热区"的页号必须用
`curPage()`**（real 阶段返回真实页号，dry 阶段返回内部计数）。历史上曾因为用 `this.page`
记录锚点导致**全书站内跳转整体错一页**，已通过 `curPage()` 修复。新增任何"登记 anchor /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zigcc/zig-course](https://github.com/zigcc/zig-course) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
