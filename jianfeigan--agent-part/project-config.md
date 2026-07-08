---
trigger: always_on
description: > **项目名称**: Product Visual Generator (商品视觉生成器)
---

# AGENTS.md - 项目架构文档

> **项目名称**: Product Visual Generator (商品视觉生成器)
> **版本**: 0.1.0
> **最后更新**: 2026-04-05

---

## 一、项目概述

### 1.1 项目定位

基于 LangChain/LangGraph 构建的**多 Agent 协作商品视觉内容自动生成系统**。系统通过多个专业 Agent 协同工作，实现商品营销图片和视频的自动化生成。

### 1.2 核心能力

| 能力 | 描述 |
|------|------|
| 🤖 多 Agent 协作 | 7 个专业 Agent 协同工作，分工明确 |
| 🖼️ 智能图片生成 | 主图、场景图、卖点图自动生成 |
| 🎬 视频分镜生成 | 智能分镜设计 + 视频合成 |
| 🎨 创意自动策划 | 风格推荐、配色方案设计 |
| ✅ 质量自动审核 | 内容质量检测、合规审核 |
| 📚 RAG 知识增强 | 企业知识库检索增强，提升生成质量 |

### 1.3 技术栈

| 层级 | 技术选型 |
|------|----------|
| **语言** | Python 3.11+ |
| **LLM 框架** | LangChain 0.3+, LangGraph 0.2+ |
| **主力 LLM** | 阿里云通义千问 (qwen3.5-flash) |
| **图像生成** | 阿里云通义万象 (wanx-v1) |
| **视频生成** | 可灵 AI (kling-v1) |
| **向量数据库** | PostgreSQL + PGVector |
| **Embedding** | BGE-large-zh (本地部署) |
| **API 框架** | FastAPI |
| **前端框架** | Vue 3 + TypeScript + Element Plus |
| **存储** | PostgreSQL (向量/关系数据) + Redis (任务状态) + 本地/OSS (资源文件) |
| **包管理** | uv |

---

## 二、系统架构

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Frontend (Vue 3 + TS)                        │
│                    商品录入 | 任务管理 | 结果展示                      │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      API Layer (FastAPI)                             │
│              /api/v1/products | /api/v1/tasks | /health              │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    LangGraph Workflow Engine                         │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    Agent 协作流程                              │   │
│  │                                                               │   │
│  │   Orchestrator ──▶ RequirementAnalyzer ──▶ CreativePlanner   │   │
│  │         │                                              │      │   │
│  │         │                                              ▼      │   │
│  │         │         ◀────────────────── VisualDesigner       │   │
│  │         │                    │                    │         │   │
│  │         │                    ▼                    ▼         │   │
│  │         │           ImageGenerator    ◀──▶  VideoGenerator  │   │
│  │         │                    │                    │         │   │
│  │         └────────────────────┴────────────────────┘         │   │
│  │                              │                               │   │
│  │                              ▼                               │   │
│  │                      QualityReviewer                         │   │
│  │                              │                               │   │
│  │                              ▼                               │   │
│  │                         [END]                                │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      External Services                               │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                │
│   │ DashScope   │  │  可灵 AI    │  │   Redis     │                │
│   │ (LLM/Image) │  │  (Video)    │  │  (State)    │                │
│   └─────────────┘  └─────────────┘  └─────────────┘                │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 数据流向

```
Product (商品信息)
    │
    ▼
GenerationRequest (生成请求)
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│                   AgentState (状态流转)                   │
│                                                          │
│  product_info → requirement_report → creative_plan      │
│       → generation_prompts → generated_images/video     │
│       → quality_reports → final_results                  │
└─────────────────────────────────────────────────────────┘
    │
    ▼
AssetCollection (最终产出)
```

---

## 三、模块结构

### 3.1 目录结构

```
agent_part/
├── main.py                 # FastAPI 应用入口
├── run_workflow.py         # CLI 工作流运行脚本
├── pyproject.toml          # 项目配置 (依赖、工具配置)
├── uv.lock                 # 依赖锁定文件
│
├── src/                    # 后端源码
│   ├── agents/             # Agent 实现
│   │   ├── base.py         # Agent 基类
│   │   ├── orchestrator.py # 编排调度 Agent
│   │   ├── requirement_analyzer.py  # 需求分析 Agent
│   │   ├── creative_planner.py      # 创意策划 Agent
│   │   ├── visual_designer.py       # 视觉设计 Agent
│   │   ├── image_generator.py       # 图片生成 Agent
│   │   ├── video_generator.py       # 视频生成 Agent
│   │   └── quality_reviewer.py      # 质量审核 Agent
│   │
│   ├── graph/              # LangGraph 状态图
│   │   ├── state.py        # AgentState 定义
│   │   └── workflow.py     # 工作流构建
│   │
│   ├── models/             # 数据模型
│   │   ├── product.py      # 商品信息模型
│   │   ├── creative.py     # 创意方案模型

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JianFeiGan/agent_part](https://github.com/JianFeiGan/agent_part) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
