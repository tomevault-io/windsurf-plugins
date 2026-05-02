---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **Agent-based Business Idea Validator** that uses a multi-agent architecture with MCP (Model Context Protocol) servers to validate business ideas through social media research. The system scrapes Xiaohongshu (Little Red Book) for data, analyzes it with LLMs, and generates comprehensive market validation reports.

**Core Workflow:**
1. Generate search keywords from business idea (LLM)
2. Scrape relevant posts and comments from Xiaohongshu
3. Analyze content for pain points, solutions, and market signals
4. Generate comprehensive validation report with scores

## Quick Start

### Installation

```bash
cd agent_system
pip install -r requirements.txt
```

### Configuration

Create/edit `.env` file in `agent_system/`:

```env
# Required API Keys
OPENAI_API_KEY="your_openai_api_key"
OPENAI_BASE_URL="https://api.openai.com/v1"  # or use proxy like https://oa.api2d.net/v1
TIKHUB_TOKEN="your_tikhub_token"  # For Xiaohongshu data via TikHub

# Optional settings
SCRAPER_PAGES_PER_KEYWORD=2
SCRAPER_COMMENTS_PER_NOTE=20
ANALYZER_MAX_POSTS=20
REPORT_OUTPUT_DIR=reports
```

### Running the System

```bash
# Command line with argument
python run_agent.py 在深圳卖陈皮

# Interactive mode
python run_agent.py

# Fast mode (less data, faster execution)
# Select 'y' when prompted
```

**Two modes:**
- **Full mode**: 3 keywords × 2 pages × 20 comments
- **Fast mode**: Direct user input as keyword × 1 page × 5 comments

### Testing

```bash
# End-to-end test
python tests/test_e2e.py

# Integration test
python tests/test_integration.py
```

## Architecture

### High-Level Design

```
┌─────────────────────────────────────────────────────────┐
│                    OrchestratorAgent                    │
│         (Main workflow coordination & task dispatch)     │
└─────────────────────────────────────────────────────────┘
                          │
                          ├── KeywordAgent (LLM keyword generation)
                          ├── ScraperAgent (Xiaohongshu scraping)
                          ├── AnalyzerAgent (AI content analysis)
                          └── ReporterAgent (HTML report generation)
                          │
        ┌─────────────────┴─────────────────┐
        │                                     │
   ┌────▼────┐  ┌────────────┐  ┌──────────▼──┐
   │ XHS MCP │  │   LLM MCP   │  │  Storage    │
   │ Server  │  │   Server    │  │  MCP Server │
   └────┬────┘  └────────────┘  └─────────────┘
        │                                     │
    TikHub API                           OpenAI API
```

### Key Components

**1. MCP Servers** (`mcp_servers/`)
- `xhs_server.py`: TikHub API client for Xiaohongshu data (search notes, fetch comments)
- `llm_server.py`: OpenAI API wrapper for LLM calls (structured output with Pydantic)
- `storage_server.py`: File-based checkpoint/state persistence

**2. Agent System** (`agents/`)
- `base_agent.py`: Abstract base class for all agents with lifecycle management
- `orchestrator.py`: **Main entry point** - creates execution plan and coordinates subagents
- `context_store.py`: Shared state management between agents
- `config.py`: Configuration management from .env, YAML, or defaults

**3. Subagents** (`agents/subagents/`)
- `keyword_agent.py`: Uses LLM to generate search keywords from business idea
- `scraper_agent.py`: Scrapes Xiaohongshu posts with comments (via TikHub API)
- `analyzer_agent.py`: Analyzes posts/comments for market insights
- `reporter_agent.py`: Generates HTML validation reports

**4. Skills** (`agents/skills/`)
Each subagent has corresponding skills files containing actual business logic:
- `keyword_skills.py`: Prompt templates and LLM calls
- `scraper_skills.py`: Scraping orchestration with rate limiting
- `analyzer_skills.py`: Analysis prompts and batch processing
- `reporter_skills.py`: HTML template rendering

**5. Data Models** (`models/`)
- `agent_models.py`: TaskResult, ExecutionPlan, ProgressUpdate, OrchestratorState
- `context_models.py`: RunContext, ContextQuery, AgentState
- `business_models.py`: XhsNoteModel, XhsCommentModel, PostWithComments, XhsPostAnalysis, CombinedAnalysis

### Execution Flow

1. **run_agent.py** entry point → OrchestratorAgent
2. Orchestrator creates ExecutionPlan with 4-5 steps (depending on fast/full mode)
3. Each step delegated to appropriate subagent via `agent.execute(task, context)`
4. Subagents call MCP servers for external operations (API calls, storage)
5. Results passed through shared context between steps
6. Final result saved as HTML report in `reports/` directory

**Key pattern:** Agents are thin wrappers - business logic lives in Skills modules

### Fast Mode vs Full Mode

**Fast Mode** (`use_user_input_as_keyword=True`):
- Skips keyword generation step
- Uses user input directly as search keyword
- 1 page × 5 comments
- 4 total steps

**Full Mode** (`use_user_input_as_keyword=False`):
- Generates 3 keywords via LLM
- 2 pages per keyword × 20 comments
- 5 total steps (includes keyword generation)

## Checkpoint System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [liangdabiao/XHS_Business_Idea_Validator](https://github.com/liangdabiao/XHS_Business_Idea_Validator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
