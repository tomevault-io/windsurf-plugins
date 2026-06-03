---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个基于 AI 的智能书签摘要系统，自动化处理从 [bookmark-collection](https://github.com/zo0043/bookmark-collection) 收集的书签。系统使用 Jina Reader 提取网页内容，并通过 LLM 生成结构化摘要。

## 核心工作流程

### 主流程（bookmark_process_changes.py）

1. **监测书签变化**：从 bookmark-collection 仓库的 README.md 中读取最新书签
2. **内容提取**：使用 Jina Reader API (`https://r.jina.ai/{url}`) 获取 Markdown 格式的全文内容
3. **AI 摘要生成**：
   - 调用 OpenAI API 生成详细的列表式摘要
   - 生成一句话 TL;DR 总结
4. **文件组织**：
   - 按月份存储文件（`data/YYYYMM/` 目录）
   - 保存原始内容（`*_raw.md`）和摘要文件
   - 更新索引文件 `data/data.json`
5. **标签处理**：自动识别并归档带标签的书签

### 标签系统（process_tag_bookmark.py）

- 从书签 URL 行提取标签（格式：`#tag`）
- 按标签组织内容到 `data/tags/*.tag` 文件
- 生成标签摘要文件 `tag_summary.md`
- 特殊标签 `weekly` 用于标记需要进行周报分析的文章

### 关键词分析系统（keyword_analyzer.py）

**核心功能：** 自动识别书签中的高频关键词，生成深度分析报告和对比分析

**工作流程：**
1. **关键词提取**：使用 LLM 从书签标题和标签中提取 3-5 个核心关键词
2. **倒排索引构建**：构建关键词到文章的映射关系，保存到 `data/keyword_index.json`
3. **高频关键词筛选**：过滤频次 ≥ 3 的关键词（可配置）
4. **深度分析**：对每个高频关键词进行主题演变、核心观点、技术要点分析
5. **对比分析**：分析同一关键词下不同文章的共同点、差异和互补性
6. **报告生成**：生成独立的 Markdown 分析报告

**数据结构：**
- **关键词索引**: `data/keyword_index.json`
  ```json
  {
    "分布式系统": [
      {"url": "...", "title": "...", "timestamp": 123, "month": "202410", "summary_path": "..."},
      ...
    ]
  }
  ```
- **分析报告**: `data/keyword_analysis/{keyword-slug}.md`
  - 元数据（分析时间、文章数、时间跨度）
  - 文章列表（带链接）
  - 深度分析（主题演变、核心观点、技术要点）
  - 对比分析（共同观点、差异化视角、互补性、推荐阅读顺序）
- **分析索引**: `keyword_analysis_summary.md`

**增量更新机制：**
- 关键词索引支持增量构建（仅处理新书签）
- 每 10 个书签保存一次中间结果（防止中断丢失）
- 已处理的 URL 会被记录，避免重复提取

**容错策略：**
- 关键词提取失败时降级使用已有标签
- 摘要文件缺失时记录警告并跳过

## 环境配置

### 必需的环境变量

```bash
# OpenAI API 配置
OPENAI_API_KEY=sk-...           # OpenAI API 密钥
OPENAI_API_MODEL=gpt-4          # 可选，默认 gpt-4
OPENAI_API_ENDPOINT=https://... # 可选，自定义 API 端点
```

### Rye 包管理器使用

项目使用 [Rye](https://github.com/astral-sh/rye) 作为包管理器：

```bash
# 初始化环境
rye sync                    # 安装依赖，创建虚拟环境
rye add requests            # 添加新依赖
rye remove requests         # 移除依赖
rye run python script.py    # 在虚拟环境中运行命令

# 激活虚拟环境（手动操作时）
source .venv/bin/activate  # macOS/Linux
.venv\Scripts\activate     # Windows
```

## 开发环境设置

```bash
# 使用 Rye（推荐方式）
rye sync                    # 安装依赖
rye run python -m bookmark_ai_summary.bookmark_process_changes  # 运行模块

# 或使用传统 pip 方式
pip install -r requirements.lock
python -m bookmark_ai_summary.bookmark_process_changes

# 环境变量配置
# 项目根目录已存在 .env 文件，直接编辑添加必要的环境变量：
# OPENAI_API_KEY=sk-...
# OPENAI_API_MODEL=gpt-4（可选）
```

## 常用命令

### 手动处理书签

```bash
# 处理新书签并生成摘要
python -m bookmark_ai_summary.bookmark_process_changes
# 或使用 rye
rye run python -m bookmark_ai_summary.bookmark_process_changes

# 仅更新标签摘要
python -m bookmark_ai_summary.process_tag_bookmark

# 生成周报（需要 weekly 标签的文章）
python -m bookmark_ai_summary.build_weekly_release

# 关键词分析（增量更新）
python -m bookmark_ai_summary.run_keyword_analysis

# 关键词分析（强制重建索引）
python -m bookmark_ai_summary.run_keyword_analysis --force-rebuild

# 关键词分析（自定义最小频次）
python -m bookmark_ai_summary.run_keyword_analysis --min-frequency 5
```

### 项目维护脚本

```bash
# 关键词分析健康检查
./scripts/keyword-analysis-health.sh

# 清理关键词分析数据
./scripts/keyword-analysis-cleanup.sh

# 查看项目统计信息
./scripts/keyword-analysis-health.sh --stats
```

### GitHub Actions 自动化

- **每周日 20:00 (北京时间)**：自动生成周报 Release
- **每周日 00:00 (UTC)**：处理 weekly 标签的文章

可通过仓库的 Actions 页面手动触发工作流。

## 关键数据结构

### SummarizedBookmark

```python
@dataclass
class SummarizedBookmark:
    month: str       # 格式：YYYYMM
    title: str       # 书签标题
    url: str         # 原始 URL
    timestamp: int   # Unix 时间戳
    tags: List[str]  # 标签列表
```

### 文件路径模式

- 摘要文件：`data/YYYYMM/YYYY-MM-DD-{slugified-title}.md`
- 原始内容：`data/YYYYMM/YYYY-MM-DD-{slugified-title}_raw.md`
- 标签文件：`data/tags/{tag-name}.tag`
- 索引文件：`data/data.json`
- 关键词索引：`data/keyword_index.json`
- 关键词分析报告：`data/keyword_analysis/{keyword-slug}.md`
- 关键词分析索引：`keyword_analysis_summary.md`

## LLM Prompt 策略

### 详细摘要 Prompt

系统使用结构化的 Prompt 生成多层次的列表式摘要：
- 强调使用 Markdown 列表格式
- 支持多层缩进和子列表
- 每项开头包含简短描述词
- 统一使用中文输出
- 参考示例：Trello 产品分析文章

### 一句话总结 Prompt

限制在 100 字以内，基于详细摘要生成精炼版本。

### 关键词提取 Prompt

从书签标题和标签中提取 3-5 个核心关键词（技术术语或主题词），优先技术术语，避免泛化词汇，中文输出。

### 关键词深度分析 Prompt

分析关键词主题的演变趋势、核心观点和技术要点，基于多篇文章摘要，使用 Markdown 列表格式。

### 关键词对比分析 Prompt

对比同一关键词下不同文章的共同观点、差异化视角、互补性，并建议阅读顺序。

## 架构设计要点

### 容错机制

- **Wayback Machine 存档**：失败时记录警告但不中断流程
- **API 调用**：使用 `@log_execution_time` 装饰器记录执行时间
- **文件操作**：自动创建必要的目录结构

### 文件命名规则

- 使用 `slugify()` 函数清理文件名中的特殊字符
- 移除文件系统非法字符：`/\:*?"<>|`
- URL 编码用于 README.md 中的链接路径

### 数据同步

1. 读取 `bookmark-collection/README.md` 获取新书签
2. 对比 `data/data.json` 中已处理的 URL
3. 处理未摘要的书签
4. 更新 `bookmark-ai-summary/README.md` 和 `data.json`

## 与外部系统集成

### bookmark-collection 仓库

- 源仓库：存储原始书签列表
- 格式：Markdown 列表 `- [Title](URL) #tag1 #tag2`
- 本项目作为消费者，定期拉取最新内容

### Jina Reader API

- 端点：`https://r.jina.ai/{url}`
- 功能：将网页转换为 Markdown 格式
- 无需认证，直接 GET 请求

## 关键词分析系统配置

### 配置常量（keyword_analyzer.py）

```python
# 关键词索引文件路径
KEYWORD_INDEX_PATH = "bookmark-ai-summary/data/keyword_index.json"

# 关键词分析报告目录
KEYWORD_ANALYSIS_DIR = "bookmark-ai-summary/data/keyword_analysis"

# 分析索引文件路径
KEYWORD_ANALYSIS_SUMMARY = "bookmark-ai-summary/keyword_analysis_summary.md"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zo0043/bookmark-ai-summary](https://github.com/zo0043/bookmark-ai-summary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
