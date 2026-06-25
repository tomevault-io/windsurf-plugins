---
trigger: always_on
description: > 本文件是项目所有 AI agent (Claude Code / Codex / Cursor / Cline / Aider) 的**唯一规范源**. 仓库根 `AGENTS.md` 是本文件的 symlink, 不是独立文档. 跨工具指引看本文件即可.
---

# 工作规范

> 本文件是项目所有 AI agent (Claude Code / Codex / Cursor / Cline / Aider) 的**唯一规范源**. 仓库根 `AGENTS.md` 是本文件的 symlink, 不是独立文档. 跨工具指引看本文件即可.

## 环境与工具链

- Python: 使用 `python3`
- LaTeX: 使用 `pdflatex`(BEDC 论文体内零中文; 顶层 README/CLAUDE/AGENTS 用中文但不进入 PDF)
- Lean 4: `lake build`; `lean4/` 为 **mathlib-free** 形式化, 从 first principles 起步
- 单个 `.tex` 文件不超过 800 行, 超过须**直接 split** 出 sibling 文件, 把相对独立的子主题搬过去并 `\input` 进来. **禁止用任何"压缩空行 / 删空白行 / 把多行合并成一行"等格式压缩动作来给文件腾空间**: 这类动作不传达任何理论内容, 是 code-debt churn, 也会让 git diff 噪音盖过真正的语义改动. 若 split 不出明显独立的子主题, 报告原因, 不要伪装成靠空行省下来的改动
- **Hub-only 索引文件**: hub-only 是结构分类, 不是 filename → role 映射. 只有自身正文只是结构性路由的 `.tex` 文件才按 hub-only 约束: 可放 1-2 句 orienting 段落、子文件 `\input{...}` 行、必要状态标注 (如 `\closureat`), 不放 `\begin{theorem}` / `\begin{definition}` / `\begin{lemma}` / `\begin{proof}` / `\begin{closurestatus}` 等正文环境. 持有 `\chapter` 与正文环境的文件是 content chapter, 受 800 行上限与正文检查约束.
- 二进制运算使用原生实现, 不用二进制字符串

## 旁路推理通道: nyxid oracle (ChatGPT Pro)

`nyxid oracle` 把推理任务路由到浏览器端 ChatGPT Pro, 可作为 codex / Claude 之外的旁路推理通道 (例: 让 ChatGPT Pro 跑一段独立推理 / 复核). 子命令、参数、输出字段清单以 `nyxid oracle --help` / `nyxid oracle ask --help` 为准, 本文档只记非显然用法:

- **两步异步 (省 token, 不轮询)**: `nyxid oracle ask company-chatgpt-pro "<问题>" --no-wait --output json` 拿 `task_id` (status=queued); 再 `nyxid oracle result <task_id> --output json` 取结果. status 走 `queued → dispatched(phase=sent) → completed`, `completed` 时 `response` 字段即答案 (附 `chatgpt_url`). 单次 `result` 即可, 未完成返回中间 status, 不要 busy-loop 轮询. 省 `--no-wait` 则 `ask` 同步阻塞最多 `--wait` 秒.
- **pool 是 org-visibility**: `company-chatgpt-pro` 限该 pool 所属 org 成员; 非成员 `ask` 返回 403 (error_code 1002 forbidden), `status` 返回 404. 先 `nyxid org join <邀请码>` 加入该 org, 再 `nyxid oracle pool list` 应能看到该 pool. 当前账号 (`aloning@gmail.com`) 已可用.
- 长 prompt 用 `--file -` 从 stdin 喂, 附件 `--pdf`, 多轮 `--new-conversation` / `--conversation <id>`; 配额与并发 (per-user inflight、worker tab 数) 以 `nyxid oracle pool list` / `nyxid oracle status <pool>` 实时读出, 不在此缓存数字.

## 语言与格式

- 工作语言默认中文, 英文版文档以 `_en.md` / `_en.tex` 结尾

## 写作纪律

- **禁止任何迭代叙事**: 不出现"新增""修订""修复了上一版本""v1.5.X 增量""patch""migration""frozen""supersede""increment""legacy"等字样
- 论文以**完整当前态**形式呈现; 理论扩展在原章节内追加, 不留版本号痕迹
- **禁止版本号 / round 编号在任何命名工件**: 文件名 / 命名空间 / 定理名 / 论文段落标题 / branch 名 / worktree 路径 / commit subject / PR title / codex worker 任务名 都不带 `V2` / `V6` / `V7` / `R11` / `Rxx` / `Round N` / `rN-X` / `v[0-9]+` 这类迭代标签. 命名按内容主题 (例: `feat-conf-closed-strip-completion` 而非 `feat-conf-general-v2`; `Typing.lean` 而非 `TypingV2.lean`; "Shift-aware binder context" 而非 "V7 substrate"). 派 codex worker 前自检 branch / worktree / file path 含 `v[0-9]+` / `V[0-9]+` / `r[0-9]+` / `Round` 字样 → 重命名
- 不生成修改记录、日志、总结、变更原因、报告
- 文件内备注从简: 允许必要的解释性注释(说清非显然的意图 / 约束 / 推导), 但不堆砌、不逐行注水; 仍禁止迭代叙事类注释(变更记录 / "新增""修订""上一版本" / Old-New 对照 / 迁移说明)
- 有问题直接改, 无需备份

## 数学符号写法

- 行内数学: `$...$`
- 展示数学: `$$...$$` (每个 `$$` 占独立一行)
- 多行展示: 在 `$$...$$` 内部使用 `\begin{aligned}...\end{aligned}` 或 `\begin{gathered}...\end{gathered}`
- **禁止顶层使用**: `\[...\]`, `\begin{equation}`, `\begin{equation*}`, `\begin{align}`, `\begin{align*}`, `\begin{eqnarray}`, `\begin{eqnarray*}`
- `papers/bedc/Makefile` 的 precheck 调 `scripts/check_math_env.sh`, 在 build 前**自动**把违规重写为 `$$\begin{aligned}...\end{aligned}$$` 形式; 写入禁用模式不会编译失败, 但会被静默替换
- 编写时直接用 `$$` 形式; 不要依赖 auto-rewrite 兜底

## LaTeX 常见 fail 模式

- **Math 宏在 text-mode 位置**: `\<Capital>Up` 类宏 (`\NatUp`, `\BoolUp`, ...) 是 `\mathsf{...}^{\uparrow}` 的封装, **必须在 math mode 内**. 典型错误包括 `\falsifiablePrediction{... \ClosedGenerationRefusalUp.}` 这类文本命令参数内裸用, 以及 `dyadicratcore/l10_sibling_dependency_route.tex:1: \DyadicRatCoreUp reads...` 这类文件首行直接 text mode. 修法: 包成 `$...$`. precheck check L 会拦.
- **死 `\input` 路径**: worker 把 region 内容迁到 subdir 后, sibling 文件若残留指向已搬走文件的 `\input{...}`, CI 会报 "File not found". 修法: 修路径或删行. precheck check M 会拦.
- **`\falsifiablePrediction{}` / `\independenceWitness{}` 参数语义**: 这两个宏是 text-mode 段落输出 (`\par\textbf{...}: #1\par`). 参数里任何 math 内容必须 `$...$` 包裹. 数学公式建议改用 `$$...$$` 展示式, 不要塞进参数.

## 项目结构

- `lean4/` — Lean 4 形式化, mathlib-free, 0 axiom 0 sorry
- `papers/bedc/` — BEDC LaTeX 论文 (现行态)
- concrete-instances namecert 章节有两种合法布局:
  - 单文件 content chapter: `papers/bedc/parts/concrete_instances/<NN>_<slug>_namecert_construction.tex`. 文件内可持有 `\chapter{...}` / `\label{ch:concrete-instances-<slug>-namecert}` / `\origin{...}` / 正文环境 / `closurestatus`, 受 800 行上限与 content checks 约束.
  - 多文件 hub + subdir: 当章节拆为多文件时, 顶层文件仍用 `papers/bedc/parts/concrete_instances/<NN>_<slug>_namecert_construction.tex`, 但只放 1-2 句 orienting 段、必要 `\closureat{<X>Up}{<level>Str}` 状态行、`\input{parts/concrete_instances/<slug>/<name>.tex}` 行; **禁** `\chapter` / `\begin{theorem}` / `\begin{definition}` / `\begin{lemma}` / `\begin{proof}` / `\begin{closurestatus}` / 任何正文环境.
  - Subdir spine: `papers/bedc/parts/concrete_instances/<slug>/namecert_construction.tex`. 持有拆分章节的 `\chapter{...}` / `\label{ch:concrete-instances-<slug>-namecert}` / `\origin{...}` / 章节 intro + 第一批定义.
  - 其它 sibling: `papers/bedc/parts/concrete_instances/<slug>/<descriptive_name>.tex`. 文件名 lowercase snake_case, 不带 `<NN>_<slug>_` 前缀.
  - Hub 内 `\input` 顺序: spine 第一, 其余按主题逻辑 (constructor → theorem → bridge → closurestatus).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [the-omega-institute/newmath](https://github.com/the-omega-institute/newmath) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
