---
trigger: always_on
description: 花叔的「md/html/docx 多向流水线」skill，四个能力 + 两种模式：(1) 用Microsoft markitdown把任意文件（PDF/DOCX/PPTX/XLSX/HTML/图片/音频/YouTube/EPub/ZIP）转成干净的md；(2) 用Pandoc + 4套精挑模板把md加工成出色的html——**兜底模式**（不耗token，pandoc 一键套版）+ **视觉艺术设计师模式**（AI 读懂内容、推荐 3 个差异化方向、为内容定制视觉表达），继承huashu-design的反AI slop审美；(3) 用html-to-markdown + trafilatura把html或URL无损转回md；(4) 用python-docx把md加工成出版社级docx（专业排版+自动嵌图+封面目录页眉页脚，专用于纸质书审校/投稿/出版交付）。落地花叔的「md生产，多端消费」方法论。触发词：md转html、html转md、pdf转md、docx转md、pptx转md、xlsx转md、文件转md、URL转md、文档转md、转markdown、做html、生成html、网页转
---


# huashu-md-html

> 你不再需要亲手编辑产物。md 是源代码，html / docx 是产物。这个 skill 把多端的最优解打通成一条流水线。

## 四个能力（决策树）

| 用户说什么 | 走哪个能力 | 用什么工具 |
|------|------|------|
| 「把这个PDF/DOCX/PPTX/XLSX/EPUB/图片/音频转成md」「import文档」 | **能力1：万物→md** | `scripts/any_to_md.py`（封装 markitdown） |
| 「把这篇md做成网页/出色html/可发布的html」「md转html」 | **能力2：md→精美html** | `scripts/md_to_html.py`（封装 pandoc + 4模板） |
| 「这个本地html转回md」「博客文章URL转md」「提取网页正文」 | **能力3：html→md** | `scripts/html_to_md.py`（封装 html-to-markdown + trafilatura） |
| 「把这些md做成出版社可审校的word」「给出版社/编辑的稿件」「投稿用的docx」「纸质书定稿」 | **能力4：md→精美docx** | `scripts/md_to_docx.py`（封装 python-docx + 专业排版） |
| 「这个产品页/技术文档URL转md」「带metadata一起拿」 | **能力1：万物→md**（也吃URL） | `scripts/any_to_md.py` |

**决策原则**：
- 能力1产出的md可以直接喂给能力2组成一条龙（如「PDF→精美阅读html」）
- 能力3用于反向归档（如「把已发布的html博客文章存回项目源」）
- **能力4是出版终点**——给人类编辑/出版社审校时用 docx，不要直接给 html 或 md，专业出版生态默认 docx

### URL 场景的进一步分流（2026-05 实测发现）

URL 输入时**两条路径都能跑**，但产出质量差异巨大。Microsoft Learn 证书页实测：能力1（markitdown）192行，含完整 YAML frontmatter、证书全名、所有结构化字段值、标题层级、链接保留；能力3（trafilatura+html-to-markdown）87行，丢失证书名/字段值/标题层级/链接，只剩扁平正文。

| 页面类型 | 走哪个 | 原因 |
|---------|--------|------|
| **结构化页面**：产品详情、技术文档、API doc、证书/课程页、电商商品页 | **能力1**（markitdown） | 保留 metadata、字段值、链接、标题层级——「信息完整版」 |
| **正文类页面**：博客、新闻、Essay、公众号文章、专栏长文 | **能力3**（trafilatura） | 自动去导航/侧栏/相关推荐/广告——「纯阅读版」 |
| **不确定** | **两个都跑一遍对比** | 看哪个产出对你的下游用途更合适 |

判断捷径：

> **URL 包含的内容是「读」的，还是「查」的？**
> 读 → 能力3（去噪）
> 查 → 能力1（保信息）

## 核心审美底线（继承自 huashu-design）

这个skill产出的每一份html都必须符合花叔的审美底线。**违反任一条都重做，不要交付**。

| 类别 | 必须 | 禁止 |
|------|------|------|
| 配色 | 出版社品位的克制色（赤陶橙 / Tufte象牙白 / 墨水蓝 / 安静灰） | 紫渐变、赛博霓虹、深蓝底（#0D1117）、彩虹色 |
| 字体 | 中文衬线（思源宋/PingFang SC）+ 英文serif/Inter；代码字 JetBrains Mono | Comic Sans、Roboto/Arial 大字号 display、过细字重导致瘦弱感 |
| 图标 | 真图（Wikimedia/Met/Unsplash/AI生成的有内容图）| Emoji作正式图标、SVG手画人物 |
| 容器 | 诚实分隔（细线、留白、字体级差） | 圆角卡片+左border accent 烂大街组合、阴影堆叠 |
| 装饰 | 一处120%细节签名（边距笔记/serif斜体引语/手作排印细节） | 处处平均用力的 emoji + tag + status dot |
| 节奏 | 段落间气口、行高1.75-1.85（中文）、最大宽度680-820px | 顶到边的密集排版、行高1.4以下、>900px宽体（眼动疲劳） |

详细规则见 `references/anti-ai-slop.md`。

## Junior Designer 工作流

收到「转换/美化/导入」类任务时，**不要直接执行**。先问：

1. **能力是哪个**？三选一（用决策树自检）
2. **来源/去向**？文件路径 / URL / 字符串？输出到哪？
3. **能力2专属问**：模板选哪个？（article默认 / report / reading / interactive）
4. **特殊需求**？（图片处理：保留相对路径 还是 base64嵌入？语言：中文版/英文版？）

回答清楚再动手。不要默认猜，错了用户返工成本远大于多问一句。

## 能力1：万物 → md（`scripts/any_to_md.py`）

封装 [microsoft/markitdown](https://github.com/microsoft/markitdown) v0.1.5+，一份Python脚本兼容20+种格式。

### 调用

```bash
# 基本：自动按扩展名识别
python scripts/any_to_md.py input.pdf
python scripts/any_to_md.py input.docx -o output.md
python scripts/any_to_md.py "https://www.youtube.com/watch?v=xxx"

# 结构化网页/产品页/技术文档（保留 metadata + 标题层级 + 链接）
python scripts/any_to_md.py "https://learn.microsoft.com/en-us/credentials/certifications/modern-desktop/" -o cert.md

# 启用LLM图片描述（需要OPENAI_API_KEY环境变量）
python scripts/any_to_md.py photo.jpg --llm-describe
```

### 支持的格式

PDF、DOCX、PPTX、XLSX、XLS、HTML、CSV、JSON、XML、图片（EXIF/可选LLM描述）、音频（可选语音转写）、YouTube URL（自动抓字幕）、**普通网页URL**（带 YAML frontmatter）、EPub、ZIP（递归解包）、Outlook邮件（.msg）。

### 已知坑（写在脚本输出里提醒用户）

- 扫描PDF不做OCR，需要挂LLM client或Azure Doc Intelligence
- 复杂表格（合并单元格/嵌套）会丢失语义
- PPTX只保留文本+备注，动画排版完全丢
- 输出**为LLM消费设计**，给人读还要再过一道排版

依赖：`pip install 'markitdown[all]'`（自动检测，缺失时提示安装）。

完整cookbook见 `references/markitdown-cookbook.md`。

## 能力2：md → 精美html（`scripts/md_to_html.py`）

封装 [Pandoc](https://pandoc.org/) + 4套精挑模板，覆盖花叔写作场景全部需求。

### 调用

```bash
# 默认：article模板（Tufte风，适合essay/博客）
python scripts/md_to_html.py article.md

# 选模板
python scripts/md_to_html.py report.md --theme report      # 宽体多表格，适合技术报告/白皮书
python scripts/md_to_html.py article.md --theme reading    # Medium极简，适合公众号转接
python scripts/md_to_html.py book.md --theme interactive   # 折叠目录+SVG图，适合长文/橙皮书

# 输出位置
python scripts/md_to_html.py input.md -o out.html

# 图片处理
python scripts/md_to_html.py input.md --inline-images      # base64嵌入（自包含单文件）
python scripts/md_to_html.py input.md --copy-images        # 拷贝到output目录（默认保持相对路径）
```

### 4套模板速览

| 模板 | 哲学锚点 | 适合场景 |
|------|---------|---------|
| **article** | Tufte CSS启发，Pentagram式信息建筑 | essay、博客、深度阅读、独立文章 |
| **report** | 出版社白皮书风，多表格密度型 | 技术报告、调研、白皮书、产品文档 |
| **reading** | Medium风极简，单栏窄体大字 | 公众号转接、纯阅读、轻量分发 |
| **interactive** | 长文档导航型，折叠+目录+边栏 | 橙皮书章节、技术书籍、长教程 |

每个模板都是**自包含单CSS**，HTML打开即可用，不依赖外部CDN。

### 依赖

- `brew install pandoc`（必装，二进制）
- 脚本启动时自动检查`which pandoc`，缺失则提示安装命令

完整cookbook见 `references/md-to-html-themes.md`。

### 两种模式 · 兜底 vs 视觉设计师

能力 2 有两条路径——

| 模式 | 是否耗 token | 何时用 |
|------|------|------|
| **兜底**（4 主题套版） | ❌ 不耗 | 已知主题、要快、不挑细节——`md_to_html.py --theme xxx` 一条命令出活 |
| **设计师模式**（AI 介入定制） | ✅ 耗 | 让 AI 读懂内容、推荐 3 个设计方向、定制视觉表达 |

兜底模式跑 pandoc 二进制，5 秒出结果，全程不联网不耗 token——这是**默认行为**。
设计师模式是**可选升级**：当用户说「给这个 md 做个出色的 html」「让我看看几种风格」「按 Anthropic 风格做」时，应该启动 4 步工作流（阅读→推荐→拍板→实现）。

完整方法论 + 流派池 + 评审清单见 `references/visual-designer-mode.md`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alchaincyf/huashu-md-html](https://github.com/alchaincyf/huashu-md-html) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
