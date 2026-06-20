---
trigger: always_on
description: Generate visually striking PPT slides via OpenAI's gpt-image-2 -- use any style in styles/<id>.md or mimic a user-supplied .pptx template; outputs high-res slide PNGs and a 16:9 .pptx. Use when the user asks to make a presentation, slides, deck, pitch deck, investor PPT, magazine-style PPT, or 做一份 PPT / 生成幻灯片 / 用 gpt-image 生成 PPT / 按这个模板生成 PPT.
---


# gpt-image2-ppt -- 用 gpt-image-2 生成 PPT

把一份 markdown 大纲（或 `slides_plan.json`）+ 一种视觉风格，直接喂给 OpenAI 官方 Images API（`gpt-image-2`），逐页出图，最后打包成 16:9 .pptx。

## 可用风格

| 风格 ID | 一句话定位 | 适用场景 |
| --- | --- | --- |
| `gradient-glass` | Apple Vision OS / Spatial Glass | AI 产品发布、技术分享、创意提案 |
| `clean-tech-blue` | Stripe / Linear 级蓝白 | 融资路演、商业计划书、企业战略 |
| `vector-illustration` | 复古矢量插画 + 黑描边 | 教育培训、品牌故事、社区分享 |
| `editorial-mono` | Kinfolk / Monocle 编辑设计 | 品牌发布、文化访谈、读书分享 |
| `dark-aurora` | Linear / Vercel 深色霓虹 | AI 产品、开发者工具、技术分享 |
| `risograph` | Riso 双套色印刷 + 网点纹理 | 创意工作室、文创品牌、独立 zine |
| `japanese-wabi` | 无印 / 原研哉式侘寂 | 茶道、生活方式、奢侈品、文化讲座 |
| `swiss-grid` | Bauhaus / Vignelli 国际主义网格 | 学术报告、博物馆展陈、严肃汇报 |
| `hand-sketch` | Sketchnote / 白板手绘 | 工作坊、产品 brainstorming、培训 |
| `y2k-chrome` | Y2K 千禧液态金属 + 蝴蝶贴纸 | 潮牌、文娱、品牌联名、Z 世代营销 |
| `abstract-art-showcase` | 黑白极简、艺术展览感、超大字体和抽象画面并置 | 艺术策展、作品集、品牌调性展示 |
| `coal-industry-business-company-profile` | 工业棕黑、粗重标题、结构线和硬朗图标 | 能源、制造业、重资产公司介绍 |
| `college-candy-aesthetics-infographics` | 糖果色、校园感、圆润信息图和轻快装饰 | 教育、校园活动、轻量数据科普 |
| `creative-agency` | 创意机构气质、强视觉拼贴、鲜明版式节奏 | Agency 提案、品牌方案、创意汇报 |
| `culinary-innovation` | 餐饮创新感、食材摄影、暖色块和杂志式排版 | 餐饮品牌、食品创新、菜单/新品发布 |
| `data-science-consulting` | 数据咨询蓝灰、模块化布局、图表和技术感信息层级 | 数据分析、AI 咨询、企业数字化 |
| `mindfulness-in-the-classroom-breathing-techniques` | 柔和心理健康配色、留白、圆角块和安静插画感 | 心理健康、课堂活动、呼吸训练课程 |
| `mind-maps-workshop-professional` | 专业工作坊风、思维导图节点、清晰流程结构 | 培训工作坊、方法论、团队共创 |
| `meeting-agenda` | 会议议程感、干净网格、强信息分组和商务标题 | 例会、项目同步、管理层汇报 |
| `investment-company-business-plan` | 投资机构质感、深浅对比、稳重商务版式 | 投资计划、基金介绍、商业计划书 |
| `indigenous-cultures` | 文化纹样、自然色、手工质感和叙事型构图 | 文化课程、历史主题、公益教育 |
| `health-disparities-and-social-determinants-of-health-doctor-of-philosophy-phd-in-health-behavior-and-health-education` | 公共健康学术风、理性网格、柔和医疗色和论文感层级 | 医学论文答辩、公共健康报告、教育研究 |
| `geometric-duotone-thesis` | 双色几何、论文答辩感、斜切图形和强标题 | 学术答辩、研究报告、章节型内容 |
| `geometric-clinical-case` | 几何医疗风、冷静配色、病例卡片和清晰分栏 | 临床病例、医疗培训、诊疗汇报 |
| `geometric-business` | 商务几何块、稳健蓝绿调、简洁图表语言 | 商业计划、团队汇报、产品策略 |
| `formal-lavender-portfolio` | 淡紫正式感、作品集留白、优雅细线和柔和版式 | 个人作品集、设计简历、专业展示 |
| `flowery` | 花卉装饰、柔和色块、浪漫但有秩序的排版 | 生活方式、女性品牌、活动介绍 |
| `first-impressions` | 第一印象主题、强封面视觉、人物/标题的戏剧化关系 | 面试培训、个人品牌、沟通课程 |
| `final-year-project-thesis-defense` | 毕业设计答辩、学院派网格、清晰章节与数据页 | 毕业答辩、项目结题、研究展示 |
| `fashion-business-consulting-toolkit-aesthetic` | 时尚咨询感、高级拼贴、杂志排版和中性色 | 时尚商业、品牌咨询、趋势报告 |
| `economic-impact-of-coronavirus` | 经济影响报告风、严肃信息图、冷静色彩和数据叙事 | 宏观经济、政策分析、风险报告 |
| `eco-green-business-plan` | 鼠尾草绿、自然材质摄影、环保商务与极简分屏 | 可持续商业、环保品牌、健康生活方式 |

所有可用风格都统一放在 `styles/` 下，使用方式完全相同。需要查看风格封面展示时，读取 `docs/distilled-styles.md`。

> 风格选择原则：先根据内容场景在 `styles/` 里选择最贴近的一套。技术类可优先看 `dark-aurora` / `gradient-glass` / `data-science-consulting`，商务类可优先看 `clean-tech-blue` / `editorial-mono` / `eco-green-business-plan` / `investment-company-business-plan`，文化生活类可优先看 `japanese-wabi` / `vector-illustration` / `culinary-innovation` / `flowery`，学术类可优先看 `swiss-grid` / `geometric-duotone-thesis` / `final-year-project-thesis-defense`，工作坊与培训类可优先看 `hand-sketch` / `mind-maps-workshop-professional` / `mindfulness-in-the-classroom-breathing-techniques`。

## 模板克隆模式

直接给 skill 一个 .pptx 模板，后续所有页都仿这个模板。

```bash
# 一行：自动渲染 + 模板分析 + 出图。需本机有可用 PPTX 渲染后端
python3 scripts/generate_ppt.py \
  --plan slides_plan.json \
  --template-pptx ./company-template.pptx \
  --template-strict
```

`--template-strict` 表示每页都把模板对应页作为 image reference 喂给 gpt-image-2，仿真度最高。

### 模板渲染：本机不需要操作 PowerPoint

skill 自带 `render_template.py`，把 .pptx 自动渲染成每页 PNG，存到 `<cwd>/template_renders/<stem>/page-NN.png`。

### Agent 前置检查（模板克隆时必须做）

**在跑任何 --template-pptx 命令之前，你必须先检查本机是否有可用 PPTX 渲染后端。**

检查方式：

- 首选：在 skill 目录运行 `python3 scripts/render_template.py --check`。它会验证后端是否真的可执行，而不是只看路径是否存在。
- macOS：优先检查 `/Applications/Keynote.app` 且 AppleScript 可执行；否则检查 `libreoffice --version || soffice --version`
- Windows：优先检查本机 PowerPoint COM 可启动；否则检查 `libreoffice --version` / `soffice --version`
- Linux / 兼容层：检查 `libreoffice --version || soffice --version`，不要只用 `which`

注意：鸿蒙 / Termux / 容器 / 特殊架构环境可能看起来像 Linux，但不能假设 Linux aarch64 的 LibreOffice 二进制可运行；必须以 `render_template.py --check` 或 `soffice --version` 的实际执行结果为准。不要把 `aspose-slides` 当默认兜底，它在很多移动/特殊 Python 环境没有可安装 wheel。

如果都没有可用后端，先告知用户模板渲染需要安装可执行的 LibreOffice，或让用户在桌面端手动把模板每页导出为 `page-01.png`、`page-02.png` 后通过 `--template-images` 传入。可选安装命令：

| 平台 | 安装命令 |
| --- | --- |
| Windows | `winget install LibreOffice.LibreOffice` |
| macOS | `brew install --cask libreoffice` |
| Linux (Debian/Ubuntu) | `sudo apt-get install -y libreoffice` |
| Linux (Fedora/RHEL) | `sudo dnf install -y libreoffice` |
| Linux (Arch) | `sudo pacman -S --noconfirm libreoffice-fresh` |

装完再次检查，确认存在可用渲染后端再继续后续流程。

> 注意：Windows 上 `winget` 是 Win10/11 自带，会弹 UAC 确认框，需要用户点确认；macOS 上 `brew` 需要先安装 Homebrew。

`render_template.py` 的渲染后端按优先级自动挑：
1. **Windows**：PowerPoint COM（本机有 Office 时优先，直出 PNG，跳过 PDF 步骤）> LibreOffice
2. **macOS**：Keynote AppleScript（本机有 Keynote 时优先，直出 PNG）> LibreOffice

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JuneYaooo/gpt-image2-ppt-skills](https://github.com/JuneYaooo/gpt-image2-ppt-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
