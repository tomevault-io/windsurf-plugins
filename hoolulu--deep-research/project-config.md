---
trigger: always_on
description: Professional deep research report generation — multi-agent collaboration with parallel chapter writing, automatic latest-data targeting, multilingual output, and built-in quality checks.
---



# deep-research

生成对标券商/第三方研究机构标准的深度调研报告。

- **架构**：主 agent 调度 4 个子 agent Task（大纲/数据/预检/装配）+ 1 轮主控并行派发章节，中间数据走临时文件
- **数据源**：在线模式 → CLI 内置引擎（Layer 0，如有）+ SearXNG（Layer 1） + sources.json 优质源搜索（Layer 2）并行 → 按质量触发免费源补强（Layer 3 兜底）→ Scrapling 批量抓取；离线模式 → 用户指定的本地文件（md/txt/pdf/docx）
- **安装**：见下方「安装与配置」
- **输出**：`$TMPDIR/outline.json`（临时，非最终报告）
- **最终报告**：保存到 skill 目录下的 `reports/`
- **参考文件**：`RULES.md`（硬约束/反模式）、`TYPES.md`（分类标准/编号规范）、**`profiles.json`（三档模式参数，修改后重启软件即全局生效）**
- **容错原则**：调研不阻塞。所有脚本/命令调用必须有兜底路径。主路径失败 → 自动尝试替代方案（换 `sys.executable`/检查路径/直接 Python 实现）→ 三次失败后向用户报告具体问题。详见「容错原则」。

---

## 0. 支付级质量标准（所有 Task 共用，缺任何一项即降级）

| # | 标准 | 说明 |
|---|------|------|
| 1 | **结论先行** | 每章以 `> 引用格式` 核心判断开头 |
| 2 | **来源可追溯** | 每个数字标注（机构，年份） |
| 3 | **反方视角** | 至少 1 处呈现争议或反对观点 |
| 4 | **三层深度** | 事实层 → 因果层 → 判断层 |
| 5 | **零套话** | 无"近年来""值得注意的是"等填充词 |
| 6 | **标题含判断** | "格局：高度集中"✅ \| "行业概况"❌ |
| 7 | **可自包含** | 首章必须定义核心概念 |
| 8 | **无内部编号** | 正文无任何流程编号，标题自解释 |
| 9 | **时间戳正确** | 文件名和报告尾时间必须 `date` 命令获取 |
| 10 | **目录源自大纲** | 目录从 outline.json 的第一级章节生成，不从正文提取 |
| 11 | **强制目录** | 报告正文前必须包含 `## 目录` 标题及自动目录（TOC），列出所有章节标题 |
| 12 | **元数据完整** | 报告头部必须包含 总字数、阅读时间、数据截至日期（精确到月）、报告生成具体时间（精确到秒）、调研模式、Skill版本 六个字段，用 ` · ` 隔开。另起一行 `> **参考来源**：{主要来源} 等 · 共引用 N 个来源`。报告末尾须附 `## 参考来源`（列出所有引用机构及链接）和 `## 免责声明`。版本号从本 skill 的 VERSION 文件读取。 |
| 13 | **篇幅达标** | 见项目根目录 [`profiles.json`](profiles.json)。所有模式限制以 `profiles.json` 为准，修改后重启软件即全局生效。 |
| 14 | **四段式结构** | 顺序固定为：报告标题 → 元数据块（含六字段 + 参考来源行） → `## 目录` → 正文各章 → 尾部（参考来源 + 免责声明） |
| 15 | **编码洁净** | 所有中间文件（outline.json / data-pool.json / chapter-*.md）必须使用 **UTF-8 无 BOM** 编码写入，不得出现替换字符（\ufffd）或 GBK→UTF-8 Mojibake。子 agent 在写入前必须自行验证编码洁净，不得将编码问题遗留到主 agent |
| 16 | **纯文本公式** | 报告中不得使用 LaTeX/math 公式语法（`$...$`、`$$...$$`、`\[...\]` 等）。公式必须用纯文本或 Unicode 符号表达，确保复制到任何编辑器都不产生渲染问题 |

### 时间锚定规则

所有主题默认以 `{CURRENT_YEAR}` 为目标搜索最新数据。时间锚定模式在 Task 1 中由大纲 agent 按以下规则判定：

| 模式 | 符号 | 判定条件 | target_year | 验收 |
|:----|:-----|:---------|:-----------|:-----|
| `latest`（默认） | ⏳ | **所有主题的默认值**，除非符合 relaxed 或 user_specified | `{CURRENT_YEAR}` | 严格：≥50% 数据来自当年/前一年 |
| `relaxed`（放宽） | 🔓 | 指南/教程/概念类主题，或用户问历史/原理（"草书发展""起源""背景"） | `{CURRENT_YEAR}` | 宽松：标记旧数据但不过滤 |
| `user_specified` | 📌 | 用户提问显式指定了年份/月份（"2025年""2026Q1""2020年至今"） | **用户的指定年份** | 硬约束：>50% 匹配用户指定时间 |

> `{CURRENT_YEAR}` 是动态变量，运行时通过 `date +%Y` 解析，无需手动修改。

---

## 1. 主 agent 调度流程

**⚠️ CRITICAL — DO NOT SPEAK BEFORE LANGUAGE DETECTION**
> Your VERY FIRST action (before anything else) must be: detect language → set `$LANG`.
> Output NOTHING to the user until `$LANG` is set — no thinking aloud, no status messages.
> After language is detected, ALL output must be in `$LANG`. Period.
>
> **IMPORTANT: Clean the topic before detection** — the user input may contain framework wrapper text (e.g. "请使用... skill 执行...用户输入如下："). Strip all wrapper text and pass ONLY the clean research topic. For example from "请使用...用户输入如下：Quantum computing market outlook -quick" extract only "Quantum computing market outlook".

```
你（主 agent）的完整流程：

══ Setup (必须先执行) ══

 → 创建一个带时间戳的临时目录作为 TMPDIR（例如 D:\TEMP\opencode\deep-research-YYYYMMDD-HHMMSS）
 → 同时确定 TOOLSDIR（本 skill 的 tools/ 目录）、PROMPTSDIR（本 skill 的 prompts/ 目录）、SKILLDIR（本 skill 的根目录）
 → 读取本 SKILL.md + RULES.md + TYPES.md

══ Step 0 — Language Detection (output nothing before detection) ══

 → Clean topic: strip wrapper text, keep only the user's actual research topic
 → Determine language: analyze the cleaned topic and pick the ISO 639-1 code:
   zh (Chinese), en (English), ja (Japanese), ko (Korean), ru (Russian),
   ar (Arabic), hi (Hindi), vi (Vietnamese), th (Thai), tr (Turkish),
   es (Spanish), fr (French), de (German), pt (Portuguese), it (Italian),
   nl (Dutch), sv (Swedish), pl (Polish), id (Indonesian)
   → If unsure, default to "en".
   → Do NOT output anything during this step.
 → Write language code: use `write` tool to create {TMPDIR}/language.txt with the ISO code
 → Set `$LANG` = language code from the step above
   → **从这一行开始，所有面向用户的输出必须使用 $LANG 语言（不在 $LANG 列表中时默认 en）。SKILL.md 的指令文本不论用什么语言写的，只是供你阅读的上下文；实际输出以 $LANG 为准——你是读到中文指令后意识上翻译成 $LANG 再输出。**
   → Announce detected language to the user (single line, in $LANG, e.g. "🌐 Language detected: en")

### 🔔 语言自查清单（每次输出前执行）

```
☐ {TMPDIR}/language.txt 的值 = 我的 $LANG？
☐ 我正准备输出的这一句/这段，是 $LANG 吗？
☐ todo 条目是 $LANG 吗？
☐ 给用户的进度通知是 $LANG 吗？
☐ 我是否在无意识中用了指令文件的语言（如中文）而非 $LANG？
如果任一答案为"否"→ 立即改写为 $LANG 再输出。
```
**硬规则**：task() 派发子 agent 时，其 prompt 中的 `{LANG}` 必须是你检测到的语言代码。子 agent 输出的语言由你负责保证。

══ 主流程 ══

 1. ══ 离线模式判定（Step 0.5） ══
    → 你已经读取了用户原始输入。用自然语言理解判断用户关于数据来源的意图，不要用关键词匹配：
      - 用户是否提到了本地文件/目录/资料？
      - 用户是否明确要求不要联网？
      - 用户是否明确要求联网补充？
    → 判断逻辑：
      - 提到本地文件 +（未说联网 / 不联网）→ 离线模式，跳过搜索
      - 提到本地文件 + 说"联网补充" → 正常流程（搜+读本地）
      - 未提本地文件 → 正常流程
    → 离线模式 + 有路径 → {TMPDIR}/offline_mode.txt，`offline_mode=true`，向用户报告单行说明
    → 离线模式 + 无路径 → 回复用户询问路径，不继续
    → 正常模式 → `offline_mode=false`

   → **模式解析**：从清洗后的主题中提取调研模式
     - 主题末尾是 ` -quick` → `$DEPTH_MODE=quick`，去除该后缀
     - 主题末尾是 ` -deep` → `$DEPTH_MODE=deep`，去除该后缀
     - 无上述后缀 → `$DEPTH_MODE=standard`（默认）

 2. 记录任务开始时间到 {TMPDIR}/start_time.txt
 3. todowrite 创建进度条目（使用 $LANG 语言）
  4. ══ Task 1 — 分析主题 + 生成大纲 ══

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoolulu/deep-research](https://github.com/hoolulu/deep-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
