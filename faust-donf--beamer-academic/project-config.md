---
trigger: always_on
description: >
---


# Beamer Academic

Generate publication-quality Beamer slides from an academic paper in one automated pipeline.

## Pipeline Overview

```
论文 → 素材提取 → 大纲生成 → [用户确认] → 内容填充 → 编译 → [交互修改循环]
```

## Phase 0: Environment Check & Input Clarification

### 0.1 Locate Thesis File

Search current directory for thesis in priority order:
1. `.tex` (LaTeX source — best quality, figures/equations directly reusable)
2. `.docx` (Word — good for text extraction, figures embedded)
3. `.pdf` (PDF — acceptable but figure extraction may lose quality)

If multiple candidates found, ask user which one.
If none found, prompt: "请把论文文件放到当前目录（支持 .tex / .docx / .pdf）"

**Recommend .tex or .docx over PDF**: PDF figure extraction can produce low-quality raster images.
If user only has PDF, warn:
> 注意：PDF 提取的图片可能有质量损失。如果你有论文的 Word 或 LaTeX 源文件，建议优先使用。

### 0.2 Check LaTeX Environment

```bash
which xelatex
```

If missing, **recommend user to install** (this is a one-time setup, not optional):

> 你的系统还没有安装 LaTeX 环境。beamer PPT 需要 xelatex 编译，我来帮你装一下：

Then provide the command for user's OS and **offer to run it**:

| OS | Command |
|----|---------|
| macOS | `brew install --cask mactex-no-gui` (推荐，约3.5GB，装一次永久可用) |
| Ubuntu/Debian | `sudo apt install texlive-xetex texlive-lang-chinese texlive-fonts-recommended` |
| Fedora/RHEL | `sudo dnf install texlive-xetex texlive-xecjk` |
| Windows (WSL) | `sudo apt install texlive-xetex texlive-lang-chinese` |
| Arch | `sudo pacman -S texlive-xetex texlive-langchinese` |

Wait for installation to complete, verify with `which xelatex`, then proceed.

If user explicitly refuses to install (rare), fallback: deliver `.tex` + `.sty` + figures, suggest Overleaf.

### 0.3 Beamer Template (Ask User First)

Ask user about existing school beamer template:

> 你的学校/课题组有现成的 beamer 模板吗？（比如导师或师兄给过的 .sty 文件）
> - 有：请放到当前目录，我直接使用
> - 没有：我来帮你找或用内置通用模板

If user has a template: use it directly, skip the built-in theme.

If user has NO template:
1. Ask for school name
2. Search for existing public beamer templates for that school:
   - GitHub search: `{school name} beamer template`
   - Common sources: Overleaf Gallery, CTAN
3. If found: download and use (notify user which template)
4. If not found: use built-in `assets/beamerthemeAcademic.sty` with school logo
   - Ask: "你能提供学校 logo 图片吗？（放到当前目录即可）"
   - If no logo available: search online for the school's logo, or proceed without logo

### 0.4 Configuration

Check for `config.yaml` in current directory:
- Exists: read and use.
- Missing: ask user for basic info, then generate from template at `assets/config.yaml`.

Required user info (ask if not in config):
- Institution name and department
- Author name, supervisor, major
- Report type: `defense` | `proposal` | `conference`
- Color preference: `blue` | `red` | `green` | `purple` | `teal`
- Time limit (minutes): affects page count and content density
- Language: thesis language vs. PPT language (e.g., 英文论文 → 中文PPT)
- **Chapnote preference**: "每页要不要显示'对应论文 §X.X'的标注？"（some users find it helpful for committee, others find it cluttered）

### 0.5 Language Strategy

If thesis language ≠ PPT language, establish rules upfront:

> 你的论文是英文，PPT 要做中文版吗？
> 对于专业术语，我会：
> 1. 首次出现用"中文（English）"格式
> 2. 之后统一用中文
> 3. 公式/变量名保持英文不翻译
>
> 这样处理可以吗？

Build a **terminology mapping table** (stored in `materials/terms.md`):
```
| English | 中文 | 首次出现页 |
|---------|------|-----------|
| disparity | 形态多样性 | P4 |
| diversity | 分类丰富度 | P4 |
| Ornstein-Uhlenbeck | OU 过程 | P8 |
```

Ensure **术语一致性**: same concept uses same translation across all pages.

## Phase 1: Material Extraction

Create `materials/` directory. Extraction strategy depends on input format:

### From .tex source (best)
1. **Figures**: locate `\includegraphics` paths, copy originals → `materials/figures/`
2. **Tables**: extract `tabular`/`table` environments → `materials/tables/`
3. **Equations**: extract `equation`/`align`/`$$` blocks → `materials/equations.md`
4. **Structure**: parse `\chapter`/`\section` hierarchy → `materials/structure.md`

### From .docx
1. **Figures**: extract embedded images (python-docx or unzip) → `materials/figures/`
2. **Tables**: extract table contents → `materials/tables/`
3. **Equations**: extract OMML/LaTeX equations → `materials/equations.md`
4. **Structure**: parse heading hierarchy → `materials/structure.md`

### From .pdf (fallback)
1. **Figures**: use `pdfimages` or read PDF for embedded images → `materials/figures/`
   - ⚠️ Quality may degrade. Prefer vector (PDF/SVG) extraction when possible.
2. **Tables**: read and reconstruct → `materials/tables/`
3. **Equations**: read and convert to LaTeX → `materials/equations.md`
4. **Structure**: parse chapter/section from text → `materials/structure.md`

### 1.2 Material Confirmation (In-Conversation)

After extraction, present a **figure catalog** to user in conversation:

> ## 🖼️ 素材库（共提取 12 张图）
>
> | # | 文件名 | 来源 | 内容描述 |
> |---|--------|------|----------|
> | 1 | fig_001.png | 论文图1 | 地质年代柱状图 |
> | 2 | fig_002.png | 论文图4 | PCoA 碎石图 |
> | 3 | fig_003.png | 论文图8 | 形态空间散点图 |
> | ... | | | |
>
> **哪些图是你答辩必须展示的？** 可以说：
> - "1, 3, 5, 8 必须用"
> - "图7 不用了，跟图5 重复"
> - "全部都可能用到"

This ensures:
- Key figures won't be missed in layout assignment
- User has a mental model of available materials before the outline phase

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Faust-Donf/beamer-academic](https://github.com/Faust-Donf/beamer-academic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
