---
trigger: always_on
description: 基于规范目录结构的学术论文排版助手。支持 PDF / .doc / .docx / .md 多种输入格式，自动选择 OCR 管道、重排版管道或 MD 直转管道。包含环境清理确认、断点恢复、智能配图裁剪、逐单元增量生成 DOCX、双单元质量核查、中间状态保存和 BibTeX 参考文献管理。所有中间文件放 resources/，最终产物放 outputs/。
---


# 学术论文专家（主路由）

> [!IMPORTANT]
> 本文件是**主路由入口 + Pipeline B/C 定义 + 排版格式规范库**。Pipeline A（OCR 管道）的详细规范请参见 `ocr_kb/SKILL.md`。
> 所有管道的 DOCX 生成均使用 `docx/SKILL.md` 中定义的方法（docx-js 创建新文档 / unpack-edit-pack 编辑现有文档）。

## 0. 目录规范 (Directory Convention)

> [!IMPORTANT]
> 所有中间文件和生成产物必须严格遵守以下目录结构，**禁止在项目根目录下放置任何生成文件**（包括 `.md`、`.py`、`.docx` 等）。

```
项目根目录/
├── resources/
│   ├── pages/          # Pipeline A: 切分出的单页 PNG 图片
│   ├── figures/        # 所有管道: 配图（裁剪的或提取的）
│   ├── md/             # 所有管道: 提取/拆分结果
│   │   ├── page_N.md       # Pipeline A 按页
│   │   └── section_N.md    # Pipeline B/C 按章节
│   ├── scripts/        # 所有 Python 辅助脚本（可跨任务复用，清理时不删除）
│   ├── compiled_paper.md  # 最终汇总的完整 Markdown
│   ├── config.json     # 任务配置（源文件、格式、管道类型），整个任务期间不变
│   └── checkpoint.json # 运行时进度（当前单元、计数器、失败项），随处理进度更新
├── outputs/            # 最终交付的 .docx / .bib 文件 + 核查点中间版本
├── ocr_kb/             # Pipeline A: OCR 工作流 Skill
│   └── SKILL.md
├── docx/               # Word 文档操作 Skill（DOCX 生成的技术实现基础）
│   └── SKILL.md
├── pdf/                # PDF 操作 Skill
│   └── SKILL.md
├── content_generation/ # Pipeline D: 论文内容智能生成 Skill
│   └── SKILL.md
├── SKILL.md            # 本文件（主入口路由 + Pipeline B/C + 排版规范库）
└── <source>.*          # 用户提供的原始文件 (.pdf / .doc / .docx / .md / 项目代码目录)
```

---

## 1. 新任务启动协议 (Step 0: Pre-flight)

> [!CAUTION]
> 以下四项检查必须在执行**任何**写作或排版动作之前**全部完成**，不可跳过。

### 1.0 技能版本检查与自动更新 (Version Check & Auto Update)

在每次启动处理时，必须首先对比用户本地的 skill 版本和现在的最新版本：
- 运行相应的命令（如 `git fetch origin && git status -uno`）检测本地仓库状态。
- **如果本地版本与最新版本不一致**：应当主动提示并帮助用户自动更新：
  > "检测到 `academic-paper-writer-pro` 技能存在最新版本，是否需要帮您自动更新？回复 **更新** 进行升级，或 **跳过** 继续当前任务。"
- 若用户选择更新，自动执行更新命令（如 `git pull` 或 `npx skills add https://github.com/tfboy1/academic-paper-writer --skill academic-paper-writer-pro`），待更新完成后再进入下一步。

### 1.1 环境清理确认

检测 `resources/` 下的 `pages/`、`md/`、`figures/` 以及 `resources/checkpoint.json` 是否存在旧文件。

- **若存在旧文件**：列出文件清单，明确询问用户：
  > "检测到上次任务的中间文件（N 个页面/章节、M 个 Markdown、K 张配图）。是否删除以避免干扰？回复 **删除** 清空，或 **保留** 继续上次任务。"
- **若不存在旧文件**：跳过此步。
- **用户选择"删除"时**：删除 `resources/pages/*`、`resources/md/*`、`resources/figures/*`、`resources/checkpoint.json`、`resources/config.json`。`resources/scripts/` **不删除**（脚本可复用）。
- **用户选择"保留"时**：读取 `resources/checkpoint.json`，进入断点恢复流程（见 §1.3）。

### 1.2 源文件确认与管道选择

扫描项目根目录下的所有支持格式文件（`.pdf`、`.doc`、`.docx`、`.md`）：

- **只有 1 个**：自动选定，根据扩展名决定管道。
- **有多个**：列出所有文件名、类型及大小，要求用户明确指定目标文件。
- **没有**：报错，要求用户放入源文件。

**管道自动选择规则**：

| 扩展名 | 管道 | 进度单位 | 核心 Skill |
|--------|------|----------|-----------|
| `.pdf` | **Pipeline A — OCR 管道** | 页 (page) | `ocr_kb/SKILL.md` |
| `.doc` / `.docx` | **Pipeline B — 重排版管道** | 章节 (section) | `docx/SKILL.md` + 本文件 §3.2 |
| `.md` | **Pipeline C — MD 直转管道** | 章节 (section) | `docx/SKILL.md` + 本文件 §3.3 |
| 无 / 目录 | **Pipeline D — 内容生成管道** | 章节 (section) | `content_generation/SKILL.md` |

### 1.3 断点恢复检测

检查 `resources/checkpoint.json` 是否存在且合法：

- **存在且 `status == "suspended"`**：
  向用户展示上次进度（已完成单元数 / 总单元数，管道类型），询问：
  > "检测到上次任务（Pipeline X，已完成 N/M 个单元）。是否从断点继续？回复 **继续** 从第 N+1 个单元开始，或 **重新开始** 清空所有中间文件。"
- **不存在或 `status == "completed"`**：正常启动新任务。

---

## 2. 文件完整性检查与资源准备

### 2.1 文件完整性检查

要求用户把需要排版的文稿和格式要求放入本目录下：
- **草稿 (Draft)**：用户的内容文件（`.pdf`、`.doc`、`.docx` 或 `.md`）。
- **格式要求 (Style Guide/Template)**：`.docx` 模板或 `.pdf` 指南（如 IEEE 模板）。
- **参考文献 (References)**：询问用户是否有 `.bib` 文件。如果有，**必须优先使用**。

### 2.2 格式规范解析（所有管道必执行）

> [!IMPORTANT]
> 在拆分内容或提取文字之前，**必须先完成格式规范解析**，确定所有排版参数。

1. **如果用户提供了 `.docx` 模板**（`config.template_file` 非空）：
   - 使用 `docx/SKILL.md` 的 unpack 方法打开模板
   - 提取样式定义（标题层级、正文字体/字号、行距、页边距、页眉页脚、分栏设置）
   - 将提取到的参数记录到 `config.json` 的 `format_params` 字段

2. **如果用户指定了格式名称**（如"IEEE"、"APA"）：
   - 从本文件 **§6 排版格式规范库** 中读取对应的预设参数
   - 将参数记录到 `config.json` 的 `format_params` 字段

3. **如果两者都有**：模板优先，预设参数作为兜底。

### 2.3 资源目录准备

确保 `resources/`（含子目录 `pages/`、`md/`、`figures/`、`scripts/`）和 `outputs/` 存在，不存在则自动创建。

### 2.4 任务配置持久化

创建或读取 `resources/config.json`（详细格式见 `ocr_kb/SKILL.md` §0.5），记录源文件路径、文件类型、管道类型、格式规范和排版参数，使后续续作无需重复指定。

---

## 3. 核心处理流程（按管道分发）

### 3.1 Pipeline A — OCR 管道（PDF 输入）

> [!IMPORTANT]
> Pipeline A 的**全部详细规范**定义在 `ocr_kb/SKILL.md` 中。本节仅做高层描述。

1. **切图**：运行 `resources/scripts/split_pdf.py` 将 PDF 分割为单页 PNG。
2. **逐页循环**：按 `ocr_kb/SKILL.md` 的规范逐页提取 + 逐页追加 DOCX。
3. **核查与悬挂**：每 2 页核查，每 4 页悬挂。
4. **汇总定稿**：合并所有 `page_N.md` → `compiled_paper.md` → 最终 DOCX。

**DOCX 生成方式**：使用 `docx/SKILL.md` 中的 docx-js 创建新文档，按 §6 格式规范库中的参数配置样式。

---

### 3.2 Pipeline B — 重排版管道（.doc / .docx 输入）

适用场景：用户已有 Word 文档，需要按新的格式规范（如 IEEE）无损排版。为了绝对保留原文原生复杂公式（OMML）、图表和批注，本管道**严禁**使用 Markdown 中转提取方案，必须采用底层 XML 外观平移（Unpack-Edit-Pack）。

#### 第一步：格式转换（仅 .doc）
如果输入是 `.doc`（旧格式），先转换为 `.docx`：
```bash
python docx/scripts/office/soffice.py --headless --convert-to docx <input.doc>
```

#### 第二步：解包 (Unpack) 与提取规范
1. 按 §2.2 解析目标格式参数。
2. 将输入 DOCX 解压到原生 XML 目录：
```bash
python docx/scripts/office/unpack.py <input.docx> resources/unpacked/
```

#### 第三步：XML 级别样式重构 (Edit XML)
此步骤不触碰主体内容逻辑。编写局部脚本直接从底层“换肤”：
1. **替换 Styles (`word/styles.xml`)**：针对格式规范（如 IEEE），将 `styles.xml` 内的全部核心样式强制替换为预设样式名称及字体。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TFboy1/academic-paper-writer](https://github.com/TFboy1/academic-paper-writer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
