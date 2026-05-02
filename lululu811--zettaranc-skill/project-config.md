---
trigger: always_on
description: > 本文件面向 AI 编程 Agent。阅读前请确认你已通读本文件，再操作代码或文档。
---

# zettaranc-skill · Agent 指南

> 本文件面向 AI 编程 Agent。阅读前请确认你已通读本文件，再操作代码或文档。

---

## 项目概述

这是一个 **AI Skill（思维框架蒸馏包）**，而非传统软件工程仓库。

核心目标：将 B 站 UP 主 / 前私募基金经理 zettaranc（万千）的投资思维框架、决策启发式和表达 DNA，封装为可供 Claude Code / Cursor 等 AI 工具调用的 Skill 文件（`SKILL.md`）。

- **核心交付物**：`SKILL.md`（可直接被 AI 工具加载的角色扮演协议）
- **语料基础**：约 170 万字直播/付费课整理文章 + 12.7 万字视频 transcript + 3.3 万字交易心理系列
- **许可证**：MIT
- **版本**：当前 v1.5.0，采用语义化版本（见下方）

### 仓库结构

```
zettaranc-skill/
├── SKILL.md                    # 核心 Skill 文件（Agent 角色扮演协议）
├── README.md                   # 面向人类用户的项目介绍
├── CHANGELOG.md                # 版本变更日志（Keep a Changelog 格式）
├── CONTRIBUTING.md             # 贡献指南（含风格验证清单）
├── AGENTS.md                   # 本文件
├── LICENSE                     # MIT
├── .gitignore                  # 忽略原始语料（体积/版权）
├── scripts/                    # 语料采集与质量检查工具脚本
│   ├── batch_download_bilibili.py   # 批量下载 B 站 ztalk 音频（yt-dlp）
│   ├── batch_transcribe.py          # 批量音频转写（faster-whisper base）
│   ├── srt_to_transcript.py         # 字幕清洗为纯文本
│   ├── download_subtitles.sh        # YouTube 字幕下载（bash）
│   ├── merge_research.py            # 合并 6 个 Agent 调研结果
│   └── quality_check.py             # SKILL.md 质量自动检查
└── references/
    ├── research/               # 6 个调研提炼文件（蒸馏过程的中间产物）
    │   ├── 01-writings.md      # 著作与系统思考
    │   ├── 02-conversations.md # 长对话与即兴思考
    │   ├── 03-expression-dna.md# 碎片表达与风格 DNA
    │   ├── 04-external-views.md# 他者视角与批评
    │   ├── 05-decisions.md     # 决策记录与行动
    │   └── 06-timeline.md      # 人物时间线
    └── sources/                # 原始语料（被 .gitignore 忽略）
        ├── articles/           # ~407 篇粉丝整理文章（~170 万字，来源：知行课代表 / 知行小菜鸟 / 复盘专用 z / 大富翁小菜鸟 / TANGOO 公众号）
        ├── transcripts/        # 13 个 ztalk 视频转写（~12.7 万字）
        └── books/              # 书单等参考资料
```

**注意**：`references/sources/` 下的原始语料因版权和体积原因**不提交到 Git**。仓库中只保留调研提炼文件（`references/research/`）和 `SKILL.md`。

---

## 技术栈与运行时架构

### 技术栈

| 层级 | 技术 |
|------|------|
| 数据管道 | Python 3（标准库 + `pathlib`、`glob`、`re`、`json`） |
| 视频下载 | `yt-dlp`（Python 模块调用） |
| 语音转写 | `faster-whisper`（base 模型，CPU int8） |
| 字幕清洗 | Python 正则（SRT/VTT → 纯文本，去重、分段） |
| 文档格式 | Markdown（全部文档与语料） |
| 版本控制 | Git |

**没有** `pyproject.toml`、`package.json`、`requirements.txt`、`setup.py` 或 `Makefile`。所有脚本均为独立可执行文件，依赖通过系统级包管理器（如 `pip install yt-dlp faster-whisper`）安装。

### 运行时架构

本项目没有传统意义上的「编译/构建/运行」流程。其「生产流程」是一个**多阶段知识蒸馏流水线**：

```
Phase 1: 多 Agent 并行调研
    → 生成 references/research/01-06.md

Phase 1.5: 调研 Review
    → python scripts/merge_research.py <skill目录>
    → 输出 Markdown 摘要表格（来源数量、一手占比、关键发现、矛盾点）

Phase 2: 合成 SKILL.md
    → 人工/Agent 基于 6 份调研文件撰写最终 Skill 协议

Phase 4: 质量检查
    → python scripts/quality_check.py <SKILL.md路径>
    → 检查：心智模型数量(3-7)、模型局限性、表达DNA、诚实边界(≥3条)、
           内在张力(≥2处)、一手来源占比(>50%)

Phase 5: 交付
    → 将 SKILL.md 放入 Claude Code / Cursor 的 skill 目录
    → 或 npx skills add lululu811/zettaranc-skill
```

### 脚本用法

| 脚本 | 用法 | 说明 |
|------|------|------|
| `batch_download_bilibili.py` | `cd scripts && python batch_download_bilibili.py` | 下载 B 站 ztalk 合集音频到 `../references/sources/transcripts/` |
| `batch_transcribe.py` | `cd scripts && python batch_transcribe.py` | 将 `*_audio.m4a` 转写为 `*_transcript.txt`（跳过已存在） |
| `srt_to_transcript.py` | `python srt_to_transcript.py input.srt [output.txt]` | 清洗字幕为干净文本，默认输出 `input_transcript.txt` |
| `download_subtitles.sh` | `./download_subtitles.sh <YouTube_URL> [输出目录]` | 下载人工/自动字幕（中文优先） |
| `merge_research.py` | `python merge_research.py <skill目录路径>` | 扫描 `references/research/01-06.md`，输出调研摘要 |
| `quality_check.py` | `python quality_check.py <SKILL.md路径>` | 对 SKILL.md 执行 6 项质量检查，退出码 0=全部通过 |

**路径约定**：`batch_download_bilibili.py` 和 `batch_transcribe.py` 使用硬编码的相对路径 `../references/sources/transcripts/`，**必须在 `scripts/` 目录内执行**。其他脚本通过命令行参数接收路径。

---

## 代码组织与模块划分

### 目录职责

- **`SKILL.md`**：唯一核心产出。包含角色扮演规则、Agentic Protocol（问题分类 → 研究 → 回答）、5 个核心心智模型、23 条决策启发式、表达 DNA、人物时间线、诚实边界、智识谱系。**修改此文件是项目最主要的工作。**
- **`references/research/`**：6 个维度的调研中间产物。当需要更新/扩展 SKILL.md 中的内容时，应先查阅或更新对应的调研文件，确保改动有语料支撑。
- **`scripts/`**：辅助工具，不参与 Skill 运行时逻辑，仅服务于语料采集和质量检查。

### 文件修改优先级

1. **`SKILL.md`** —— 直接影响 Skill 表现，任何改动都需语料支撑。
2. **`references/research/*.md`** —— 调研档案，补充新语料或修正旧发现时更新。
3. **`README.md` / `CHANGELOG.md`** —— 项目对外文档，版本发布时同步更新。
4. **`scripts/`** —— 工具脚本，仅在数据管道或检查逻辑需要改进时修改。

---

## 开发规范与约定

### 版本规则

采用语义化版本，但含义针对本项目定制：

| 位 | 含义 | 示例 |
|----|------|------|
| MAJOR | 心智模型级别的重构 | v1.3.0：将 6 个心智模型重组为 5 个 |
| MINOR | 新增战术/启发式/语料 | v1.2.0：新增 B1/B2/B3、双枪战法等 5 个子模块 |
| PATCH | 排版修正或数字更新 | v1.2.1：优化排版、拆分行内引文 |

### 内容修改原则

1. **最小改动原则**：只改确实不准确的部分。
2. **有依据**：任何改动都需要语料支撑，不能凭印象。优先来源：
   - zettaranc 本人直接产出（视频、直播、付费课、雪球专栏）
   - 权威媒体报道（澎湃新闻等）
   - 证券业协会公示资料
   - **不应作为主要依据**：知乎回答、非本人微信公众号、股吧/雪球帖子（除本人账号外）。
3. **保持角色一致性**：修改后的回答仍需符合 zettaranc 的表达 DNA（见 `CONTRIBUTING.md` 风格验证清单）。

### 风格验证清单（来自 CONTRIBUTING.md）

修改 SKILL.md 后，用以下问题自检：

- [ ] 是否用「我」而非「Z 哥认为...」？
- [ ] 是否包含职业背书开场？
- [ ] 是否分 1/2/3/4 点拆解？
- [ ] 是否用了具体数字或案例？
- [ ] 是否以金句或反问收尾？
- [ ] 是否避免跳出角色的表述？
- [ ] 交易建议是否包含具体的进场/止损/止盈规则？

### 代码规范（scripts/）

- 所有脚本文件头包含 `#!/usr/bin/env python3` 或 `#!/bin/bash`。
- 使用中文编写文档字符串和注释。
- 使用标准库为主，避免引入不必要的第三方依赖。
- 正则表达式用于文本解析和统计，保持简单可维护。

---

## 测试与质量保障

### 没有传统单元测试

本项目为知识/文档工程，不采用 pytest 等单元测试框架。

### 质量检查工具：`quality_check.py`

```bash
python scripts/quality_check.py SKILL.md
```

检查 6 个维度，输出 PASS/FAIL：

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lululu811/zettaranc-skill](https://github.com/lululu811/zettaranc-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
