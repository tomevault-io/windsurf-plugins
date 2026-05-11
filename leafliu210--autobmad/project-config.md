---
trigger: always_on
description: **项目名称**: DocuSwarm Multi-Agent Orchestration System
---

# Claude Code 指导文档

**项目名称**: DocuSwarm Multi-Agent Orchestration System
**版本**: 3.2
**最后更新**: 2026-04-28

---

## 📋 目录

1. [项目概述](#1-项目概述)
2. [快速导航](#2-快速导航)
3. [核心开发原则](#3-核心开发原则)
4. [AI助手工作流程](#4-ai助手工作流程)
5. [开发工作流](#5-开发工作流)
6. [常用命令](#6-常用命令)
7. [质量保证](#7-质量保证)
8. [更新记录](#8-更新记录)

---

## 1. 项目概述

### 1.1 项目性质

DocuSwarm是一个**多代理文档编排系统**,基于BMAD方法论,集成了:

- **LangGraph** - 多代理工作流的状态机框架
- **Claude Agent SDK** - Anthropic 官方 SDK
- **Claude 3.5/4 Sonnet** - 大上下文窗口LLM(200K tokens)
- **BMAD (Breakthrough Method of Agile AI-driven Development)** - AI驱动的敏捷开发方法论
- **Dual-Agent Pattern** - 双代理模式(Independent + Evaluator)
- **Context Isolation** - 三层上下文隔离机制
- **Sequential Pipeline** - 5个BMAD阶段的顺序执行

### 1.2 核心理念

本项目采用 **"Occam's Razor"** 原则:

- **简单优先**: 使用LangGraph而非自定义NodeExecutor(节省8-12周)
- **双代理模式**: Independent Agent + Evaluator(比三代理模式简化33%)
- **顺序执行**: MVP使用顺序流程,DAG并行延迟到Phase 2
- **单一LLM**: 仅使用Claude Sonnet,无需多provider抽象
- **无RAG系统**: 256K上下文窗口足够,移除向量数据库复杂度

### 1.3 架构演进

当前系统已完成核心重构，详见 [TDD重构方案](docs/solution/README.md):

| 阶段 | 内容 | 状态 |
|------|------|------|
| **Phase 1 (P0)** | CheckpointManager + ContextValidator 提取 | ✅ 已完成 |
| **Phase 2 (P1)** | SDK统一异常处理 + 消息格式切换 | ✅ 已完成 |
| **Phase 3 (P2)** | 质量保障增强 + 测试覆盖提升 | 🔄 进行中 |

**关键TDD方案**:
- [TDD-01](docs/solution/TDD-01-CheckpointManager-Refactor.md) - CheckpointManager提取 (DRY修复)
- [TDD-02](docs/solution/TDD-02-ContextValidator-Refactor.md) - ContextValidator提取 (职责拆分)
- [TDD-03](docs/solution/TDD-03-ToolResultExtractor-Refactor.md) - 纯工具输出模式 (12-Factor对齐)
- [TDD-04](docs/solution/TDD-04-ContextResolver-Refactor.md) - @路径注入系统
- [TDD-05](docs/solution/TDD-05-SDKWrapper-Refactor.md) - SDK替换 (kimi→claude)

### 1.4 项目依赖

核心技术栈:
- **[LangGraph](https://langchain-ai.github.io/langgraph/)** - 状态机和工作流编排
- **[claude-agent-sdk](https://github.com/anthropics/claude-agent-sdk)** - Anthropic Claude Agent SDK
- **[Anthropic Claude](https://docs.anthropic.com/)** - 主要LLM提供商（200K上下文窗口）
- **[BMAD Method](https://github.com/bmad-code-org/BMAD-METHOD)** - 方法论来源
- **SQLite with WAL** - 状态持久化
- **Python 3.12+** - 实现语言

---

## 2. 快速导航

### 2.1 详细文档位置

📖 **完整文档位于 `claude_docs/` 目录**：

| 文档 | 描述 | 何时使用 |
|------|------|----------|
| **[core_principles.md](claude_docs/core_principles.md)** | 四大开发原则详解（DRY、KISS、YAGNI、奥卡姆剃刀） | 需要理解核心原则时 |
| **[bmad_methodology.md](claude_docs/bmad_methodology.md)** | BMAD开发方法论完整说明 | 团队协作、敏捷开发时 |
| **[ai_workflow.md](claude_docs/ai_workflow.md)** | AI助手三阶段工作流程 | 任何开发任务的开始 |
| **[development_rules.md](claude_docs/development_rules.md)** | 编码规范、代码风格 | 编写代码时 |
| **[testing_guide.md](claude_docs/testing_guide.md)** | 测试规范和实践 | 编写和运行测试时 |
| **[quality_assurance.md](claude_docs/quality_assurance.md)** | 质量保证流程和工具 | QA审查、质量门控 |
| **[technical_specs.md](claude_docs/technical_specs.md)** | 技术规范和配置 | 技术决策、配置管理 |
| **[workflow_tools.md](claude_docs/workflow_tools.md)** | autoBMAD工作流详解 | 自动化任务时 |
| **[quick_reference.md](claude_docs/quick_reference.md)** | 常用命令速查 | 快速查找命令时 |
| **[project_tree.md](claude_docs/project_tree.md)** | 项目结构说明 | 了解项目布局时 |
| **[venv.md](claude_docs/venv.md)** | 虚拟环境管理 | **运行任何py程序时** |

### 2.2 重构与架构文档

📋 **重构方案位于 `docs/solution/` 目录**：

| 文档 | 描述 | 何时使用 |
|------|------|----------|
| **[solution/README.md](docs/solution/README.md)** | TDD重构方案总览和实施路线图 | 规划重构工作时 |
| **[TDD-SDK-Migration](docs/solution/TDD-SDK-Migration-2026-03-25.md)** | SDK迁移方案 | kimi→claude迁移 |

📊 **研究文档位于 `docs/research/` 目录**：

| 文档 | 描述 | 何时使用 |
|------|------|----------|
| **[Context Refactor Overview](docs/research/2026-03-13-docuswarm-context-refactor-overview.md)** | 上下文重构概览 | 理解重构背景时 |
| **[Dependency Drift](docs/research/dependency-drift-2026-03-25/README.md)** | 依赖漂移分析 | 了解SDK迁移时 |

🏗️ **架构文档位于 `docs/architecture/` 目录**：

| 文档 | 描述 | 何时使用 |
|------|------|----------|
| **[Project Structure](docs/architecture/project-structure.md)** | 项目结构规范 | 理解项目布局时 |
| **[Tech Stack](docs/architecture/tech-stack.md)** | 技术栈规范 | 理解技术选型时 |
| **[Pipeline Architecture](docs/architecture/03_PIPELINE_ARCHITECTURE.md)** | 管道执行架构 | 理解节点执行时 |

### 2.3 核心目录结构

```
project/
├── autoBMAD/                 # 主源代码
│   ├── docuswarm/            # DocuSwarm系统 ⭐
│   │   ├── agents/           # Agent实现
│   │   ├── cli/              # CLI命令
│   │   ├── context/          # 上下文管理
│   │   ├── llm/              # LLM集成
│   │   ├── node_execution/   # 节点执行
│   │   ├── nodes/            # 节点定义
│   │   ├── pipeline/         # 管道编排
│   │   ├── prompts/          # 提示模板
│   │   ├── storage/          # 存储层
│   │   ├── tools/            # 工具系统
│   │   ├── utils/            # 工具函数
│   │   ├── README.md         # DocuSwarm文档
│   │   └── CONFIGURATION.md  # 配置说明
│   └── epic_automation/      # Epic自动化系统
├── nodes/                    # 节点配置（BMAD personas）
├── tests/                    # 测试代码
├── docs/                     # 项目文档 ⭐
│   ├── architecture/         # 架构文档
│   ├── research/             # 研究报告
│   ├── solution/             # TDD方案
│   ├── epics/                # Epic定义
│   └── ...                   # 其他文档（PRD、设计、评估等）
├── docs-test/                # 测试示例文档
│   ├── bubble-sort/          # Bubble Sort示例
│   └── calc-one-plus-one/    # 计算器示例
├── claude_docs/              # 开发规范文档 ⭐
├── pyproject.toml            # 项目配置
└── README.md                 # 项目概览
```

---

## 3. 核心开发原则

### 3.1 四大黄金法则


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeafLIU210/autoBMAD](https://github.com/LeafLIU210/autoBMAD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
