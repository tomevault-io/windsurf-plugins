---
trigger: always_on
description: LaTeX 编译防错与排版质量守护技能。专为东北大学学位论文模板(NEU-Thesis)设计，但适用于任何中文学位论文LaTeX模板。提供7类常见编译错误的快速诊断修复、表格排版规范、标准编译流程、12类PDF排版质量检测清单、编译后强制验证流程。关键词触发："编译论文"、"LaTeX报错"、"表格溢出"、"xelatex"、"tex文件"、"bibtex"、"Extra alignment"、"Overfull"、"Float too large"、任何涉及 .tex 文件编写、LaTeX 编译、表格制作、xelatex 命令执行、bibtex 引用处理、图表跨页分离、浮动体位置控制、minipage 绑定、longtable 长表格、FloatBarrier 浮动屏障 的场景均触发本技能。 新增批量浮动体修复模式：当需要系统性检查和修复整篇论文所有图表的跨页分离、位置漂移问题时触发此模式（"系统性检查浮动体"、"修复所有图表位置"、"批量优化排版"、"论文排版整体修复"）。
---


# LaTeX Guard — LaTeX 编译防错与排版质量守护技能

## 技能触发

任何涉及 .tex 文件编写、LaTeX 编译、表格制作、xelatex 命令执行、bibtex 引用处理、
图表跨页分离、浮动体位置控制、minipage 绑定、longtable 长表格、
FloatBarrier 浮动屏障 的场景均触发本技能。默认面向 NEU-Thesis 模板，但诊断方法同样适用于其他中文学位论文模板。

---

## 一、七大类常见编译错误

### 1.1 "Extra alignment tab has been changed to \cr."

**含义**：表格列数定义与实际数据列数不匹配。

**原因**：`\begin{tabular}{...}` 中定义的列数 < 数据行中 `&` 分隔符数 + 1。

**诊断**：数清列格式中 `l`/`c`/`r`/`p{}` 的个数，再数数据行中 `&` 的个数。例如 `{ccccccc}` 有7列，每行应恰好有6个 `&`。

**修复**：
- 修改 `\begin{tabular}{...}` 中的列数定义使其匹配数据行
- 或删除数据行中多余的 `&`

### 1.2 "Overfull \hbox" / "Float too large for page"

**含义**：表格宽度超出页面宽度。

**原因**：多列表格（≥6列）未做缩放处理。

**修复优先级（按表格宽度选择）**：

| 表格特征 | 方案 | 命令/环境 |
|---------|------|---------|
| 列数≤5 | 居中 + 五号字 | `\centering\zihao{5}` |
| 列数6-8 | 缩放适配 | `\tablefit{\begin{tabular}{...}...\end{tabular}}` |
| 列数≥9 | 横向页面 | `\begin{sidewaystable}...\end{sidewaystable}` |
| 超长表格 | 跨页表格 | `\begin{longtable}{...}...\end{longtable}` |
| 附录超大表 | 缩放 + 横向 | `sidewaystable` 内嵌 `\tablefit` |

### 1.3 图表、文字、表格跨页分离问题（系统性解决方案）

#### 1.3.1 问题本质

LaTeX 浮动体（figure/table）默认自由漂移，自动寻找"最佳"位置放置，但这常常导致图表和描述它的文字被分隔在不同页面。三种解决思路：**限制浮动范围** / **强制内容绑定** / **调整全局规则**。以下按可靠性从高到低展开。

#### 1.3.2 浮动体位置参数详解

位置参数控制浮动体的放置偏好。`\begin{figure}[参数]` 或 `\begin{table}[参数]` 中的可选字母：

| 参数 | 含义 | 优先级 | 说明 |
|------|------|--------|------|
| `h` | here | ★ | 尽量放在当前位置（此处） |
| `t` | top | ★★ | 尽量放在页面顶部 |
| `b` | bottom | ★★ | 尽量放在页面底部 |
| `p` | page | ★ | 放在单独的浮动页 |
| `!` | 强制执行 | — | 忽略 LaTeX 对页面上浮动体数量的限制 |
| `H` | 强制在此 | ★★★ | 必须放在此处（需 `\usepackage{float}`） |

**推荐组合**：
- **通用推荐**：`[hbt!]` —— 首选此处，不行放顶部，再不行放底部，忽略数量限制
- **精确控制**：`[H]` —— 强制锁定在代码位置（但需谨慎使用，可能造成大片空白）
- **顶部偏好**：`[t!]` —— 强制放在页面顶部

> ⚠️ **`[H]` 是"核武器"**：它完全绕过 LaTeX 浮动机制，可能导致页面底部大量空白。仅在定稿阶段、确认排版无问题后使用。日常写作优先用 `[hbt!]`。

#### 1.3.3 强制内容绑定（最可靠）

当必须确保"某段文字 + 某张图/表"绝对不跨页时，用盒子环境将它们打包在一起。

**① minipage 环境（首选，最可靠）**

LaTeX 保证绝不会在 `minipage` 盒子内部进行分页，是所有方案中最可靠的：

```latex
\noindent\begin{minipage}{\textwidth}
\vspace{4pt}                                     % 上部留一点空间
关于城市轨道交通运营效率的评价指标体系见表 \ref{tab:efficiency}。
\begin{table}[H]\centering\zihao{5}
  \caption{运营效率评价指标体系}\label{tab:efficiency}
  \tablefit{
    \begin{tabular}{clc}
      \toprule
      序号 & 指标名称 & 权重 \\
      \midrule
      1 & 准点率 & 0.35 \\
      2 & 客流强度 & 0.28 \\
      \bottomrule
    \end{tabular}
  }
\end{table}
\vspace{4pt}                                     % 下部留一点空间
\end{minipage}
```

**② \vbox（更轻量）**

纯 TeX 原语，比 minipage 更轻量，但功能较少（不支持 footnote 等）：

```latex
\vbox{
关于城市轨道交通运营效率的评价指标体系如下表所示：
\begin{table}[H]\centering\zihao{5}
  \caption{运营效率评价指标体系}\label{tab:efficiency}
  ...
\end{table}
}
```

**③ \parbox（适合短内容）**

用于包裹简短的内容组合，适合"一句话 + 小图表"的场景：

```latex
\parbox{\textwidth}{
该区域土地利用变化如图 \ref{fig:landuse} 所示：
\begin{figure}[H]\centering
  \includegraphics[width=0.8\textwidth]{Img/landuse.png}
  \caption{土地利用变化图}\label{fig:landuse}
\end{figure}
}
```

**④ samepage 环境（不推荐）**

LaTeX 标准 `samepage` 环境试图阻止分页，但效果有限：

```latex
\begin{samepage}
关于...见表 \ref{tab:xxx}。
\begin{table}[H]\centering ...
\end{table}
\end{samepage}
```

> ⚠️ **警告**：CMU 官方文档明确指出 `samepage` 在很多情况下是无效的。不推荐依赖它。

#### 1.3.4 浮动屏障（\FloatBarrier）

当不需要精确绑定文字和图表，但需要确保"在某个位置之前所有浮动体都必须落位"时使用。

**基础用法**：

```latex
\usepackage{placeins}

% ... 章节内容 ...

\FloatBarrier   % 此处之前所有浮动体必须落位，不能越过此屏障
```

**带 [section] 选项（推荐）**：

```latex
\usepackage[section]{placeins}
```

加上 `[section]` 参数后，LaTeX 自动在每个 `\section` 命令前插入 `\FloatBarrier`，确保每个章节内的图表不会跑到下一个章节中去。**这是学术论文最优雅的浮动控制方案。**

> 📌 **本项目已应用**：在 `Thesis.tex` 第 27 行已添加 `\usepackage[section]{placeins}`，自动控制每章浮动体范围。

#### 1.3.5 全局参数调整

通过调整 LaTeX 内部参数，让浮动体更容易在目标位置放置，减少漂移。将以下代码放入导言区（`\begin{document}` 之前）：

```latex
% ===== 浮动体全局参数调整 =====
\renewcommand{\topfraction}{0.9}        % 页面顶部最多可被浮动体占据 90%（默认0.7）
\renewcommand{\bottomfraction}{0.9}     % 页面底部最多可被浮动体占据 90%（默认0.3）
\renewcommand{\textfraction}{0.1}       % 页面至少保留 10% 给正文（默认0.2）
\renewcommand{\floatpagefraction}{0.7}  % 浮动页上浮动体至少占 70%（默认0.5）
\setcounter{topnumber}{3}               % 页面顶部最多 3 个浮动体（默认2）
\setcounter{bottomnumber}{2}            % 页面底部最多 2 个浮动体（默认1）
\setcounter{totalnumber}{5}             % 每页最多 5 个浮动体（默认3）
\raggedbottom                           % 允许底部留白，避免拉伸填充
```

**核心参数说明**：
- `\topfraction=0.9` + `\bottomfraction=0.9`：让 LaTeX 更愿意在目标位置放置浮动体（而不是拒绝后推到后面页面）
- `\raggedbottom`：页面底部不对齐，避免为填满页面而将浮动体推远

#### 1.3.6 特殊场景处理

**① 表格标题与表格分离**

当表格标题显示在一页底部、表格内容在下一页时：

```latex
\usepackage{caption}
\captionsetup[table]{position=above}       % 标题始终在表格上方
\captionsetup[table]{skip=2pt}             % 标题与表格间距
```

对于 figure 同理：
```latex
\captionsetup[figure]{position=below}      % 图标题在下方
```

**② 长表格跨页**

超过一页的表格必须用 `longtable`，不能用 `table` 环境：

```latex
\usepackage{longtable,booktabs}

{\zihao{5}
\begin{longtable}{ccccc}
  \caption{长表格示例}\label{tab:long-example}\\
  \toprule
  列1 & 列2 & 列3 & 列4 & 列5 \\
  \midrule
  \endfirsthead                                % 首页表头

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cghggchg765-create/latex-guard](https://github.com/cghggchg765-create/latex-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
