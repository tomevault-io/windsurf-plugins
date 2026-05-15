---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

Snowflake Fiction 是一个 Claude Code 插件，提供完整的小说创作工具链。包含核心技能：

| 技能 | 用途 | 触发词 |
|------|------|--------|
| snowflake-fiction | 雪花写作法创作小说（编排器） | 写小说、创作故事、雪花法 |
| outline-concept | 故事构思（步骤1→1.5c→2构思期，含写作风格配置） | 构思故事、验证创意、故事概念、帮我想一个故事 |
| character-design | 角色设计（步骤3→5→7人物深化链） | 角色设计、人物设计、新增角色、深化角色 |
| scene-plan | 场景规划（步骤8→9场景设计链） | 场景规划、规划场景、场景设计、场景清单、规划章节 |
| chapter-write | 创作期续写（步骤10，支持批量生成） | 续写、写章节、生成章节、写下一章、批量生成、继续写、写正文 |
| novel-review | 小说质量复核检查 | 小说复核、章节检查、一致性检查、质量检查 |
| humanize-text | AI文本人语化处理（24种检测模式+灵魂注入，支持指定路径和章节） | 人语化、去AI味、润色 |
| quality-check | 内容质量评估（冲突/情绪/期待感/节奏/钩子五维打分） | 质量检查、内容检查、综合评估、检查质量、小说质量 |
| character-check | 角色质量检查（扁平化/一致性/工具人/视角四维检测） | 角色检查、人设检查、人物检查、检查角色 |
| boring-detect | 流水账专项检测（无冲突/无情绪/无期待感/信息堆砌四维检测） | 流水账检测、检查流水账、平淡检测、流水账 |
| concept-check | 创意与选题检查（题材匹配/辨识度/书名质量/混搭/世界观五维检测） | 选题检查、创意检查、题材检查、检查选题、书名检查 |
| opening-check | 开篇质量检查（黄金三章法则，支持文件模式） | 开篇检查、黄金三章、检查开篇、检查前三章 |
| novel-export | 导出各平台格式 | 导出小说、番茄格式、起点格式 |

## 目录结构

```
snowflake-fiction/
├── .claude-plugin/
│   ├── plugin.json           # 插件元数据
│   └── marketplace.json      # Marketplace 发布配置
├── agents/
│   ├── outline-builder.md    # 大纲构建 agent（步骤4+6）
│   ├── snowflake-fiction.md  # 雪花写作法文件处理器（目录扫描+批量生成）
│   ├── chapter-write.md      # 创作期续写文件处理器（并行子代理架构）
│   ├── humanize-text.md      # 人语化文件处理器（并行子代理架构）
│   ├── novel-review.md       # 小说复核文件处理器（分批子代理架构）
│   ├── novel-export.md       # 格式导出文件处理器（并行子代理架构）
│   ├── quality-check.md      # 内容质量评估文件处理器（并行子代理架构）
│   ├── character-check.md    # 角色质量检查文件处理器（并行子代理架构）
│   ├── concept-check.md      # 创意与选题检查文件处理器（并行子代理架构）
│   ├── opening-check.md      # 开篇质量检查文件处理器（并行子代理架构）
│   └── boring-detect.md      # 流水账检测文件处理器（并行子代理架构）
├── skills/
│   ├── snowflake-fiction/    # 雪花写作法主技能
│   │   ├── SKILL.md          # 核心编排知识库（工作流程+委托规则）
│   │   └── references/       # 参考模板
│   │       ├── step-prompts.md              # 每步提示词
│   │       ├── character-template.md        # 人物卡片模板
│   │       ├── scene-template.md            # 场景规划模板
│   │       ├── export-format.md             # 导出格式说明
│   │       ├── long-novel-guide.md          # 长篇小说指南
│   │       └── million-word-webnovel-guide.md # 百万级网文指南
│   ├── novel-review/         # 小说复核技能
│   │   ├── SKILL.md          # 核心知识库（检查维度+报告格式）
│   │   └── references/
│   │       ├── consistency-check-prompt.md  # 一致性检查提示词
│   │       ├── character-state-template.md  # 角色状态追踪
│   │       ├── timeline-template.md         # 时间线追踪
│   │       ├── foreshadowing-tracker.md     # 伏笔追踪
│   │       └── review-report-template.md    # 复核报告模板
│   ├── character-design/     # 角色设计技能（步骤3+5+7）
│   │   └── SKILL.md
│   ├── character-check/      # 角色质量检查技能
│   │   ├── SKILL.md          # 核心知识库（检查维度+输出格式）
│   │   └── references/
│   │       ├── check-dimensions.md        # 四维度检测标准和修复示例
│   │       └── report-template.md         # 单章/批量报告模板
│   ├── scene-plan/           # 场景规划技能（步骤8+9）
│   │   └── SKILL.md
│   ├── chapter-write/        # 创作期续写技能（步骤10）
│   │   ├── SKILL.md          # 核心知识库（单章生成+流水账自检）
│   │   └── references/
│   │       └── writing-guide.md           # 写作提示词、检查清单、钩子设计
│   ├── opening-check/        # 开篇质量检查技能
│   │   ├── SKILL.md          # 核心知识库（黄金三章法则+输出格式）
│   │   └── references/
│   │       ├── golden-three-chapters.md   # 黄金三章法则详解和检查清单
│   │       ├── common-problems.md         # 常见开篇问题诊断和修复示例
│   │       └── report-template.md         # 单章/批量报告模板
│   ├── concept-check/        # 创意与选题检查技能
│   │   ├── SKILL.md          # 核心知识库（检查维度+输出格式）
│   │   └── references/
│   │       ├── check-dimensions.md        # 五维度检测标准和示例
│   │       ├── optimization-formulas.md   # 辨识度优化公式和改写示范
│   │       ├── book-title-guide.md        # 书名质量指南（7种起名思路+案例）
│   │       └── report-template.md         # 单章/批量报告模板
│   ├── quality-check/        # 内容质量评估技能
│   │   ├── SKILL.md          # 核心知识库（评估维度+输出格式）
│   │   └── references/
│   │       ├── evaluation-dimensions.md   # 五维度评分标准和检查清单
│   │       └── report-template.md         # 单章/批量报告模板
│   ├── humanize-text/        # 人语化处理技能
│   │   ├── SKILL.md          # 核心知识库（纯文本处理）
│   │   └── references/
│   │       ├── ai-patterns.md           # 24种AI写作模式详解
│   │       ├── soul-injection.md        # 灵魂注入技巧
│   │       ├── scene-modes.md           # 5种场景化处理模式
│   │       └── banned-words.md          # 禁止词汇清单
│   ├── boring-detect/        # 流水账检测技能
│   │   ├── SKILL.md          # 核心知识库（检测维度+输出格式）
│   │   └── references/
│   │       ├── detection-dimensions.md  # 四维度检测标准和示例
│   │       ├── fix-formulas.md          # 修复公式和改写示范
│   │       └── report-template.md       # 单章/批量报告模板
│   └── novel-export/         # 格式导出技能
│       ├── SKILL.md          # 核心知识库（格式转换规则）
│       └── references/
│           ├── platform-rules.md         # 各平台转换规则和示例
│           └── naming-convention.md      # 输出目录命名规范
└── README.md                 # 使用文档
```

## 技能架构

### snowflake-fiction（雪花写作法）

采用 Command / Skill / Agent 三层架构：

- **Command**（`commands/snowflake-fiction.md`）：用户入口，根据输入类型路由到 Skill 或 Agent
- **Skill**（`skills/snowflake-fiction/SKILL.md`）：核心编排知识库，定义工作流程和各步骤委托规则
- **Agent**（`agents/snowflake-fiction.md`）：文件处理器，扫描目录、判断进度、批量生成章节

支持三种模式：
- **短篇小说**（1-3万字）：12步流程

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hestudy/snowflake-fiction](https://github.com/hestudy/snowflake-fiction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
