---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Environment Setup

- Install dependencies: `pip install -r requirements.txt`
- The project uses Python with Databricks-specific libraries (MLflow, LangChain, LangGraph)
- **Key Dependencies**:
  - `mlflow[databricks]==3.2.0`
  - `databricks-langchain==0.6.0`
  - `databricks-agents==1.4.0`
  - `langgraph==0.5.4`
  - `pydantic<2.12.0`
  - `nest-asyncio==1.6.0`
- No traditional build, test, or lint commands - this is a Databricks notebook-based project

### Databricks Notebook Structure

- **driver.py**: Main Databricks notebook with `# Databricks notebook source` magic commands
- **multiagent-genie.py**: Core Python module imported via `%run ./multiagent-genie`
- Notebook cells are separated by `# COMMAND ----------` markers
- Development workflow requires Databricks environment with cluster compute

### Environment Variables Required

- `DB_MODEL_SERVING_HOST_URL`: Databricks workspace URL for model serving
- `DATABRICKS_GENIE_PAT`: Personal Access Token for Genie space authentication
- Configure in Databricks secrets for production deployment

### Running the Agent

- Main entry point: `driver.py` (Databricks notebook format with `# Databricks notebook source` magic commands)
- Core agent implementation: `multiagent-genie.py` (Python module loaded via `%run ./multiagent-genie`)
- Configuration: `configs.yaml` (requires manual setup of Databricks resources)
- Test agent: Use sample questions in `driver.py` cells 141-143 for testing different complexity levels

### Development Workflow

1. Update `configs.yaml` with your Databricks resources (catalog, schema, workspace_url, etc.)
2. Create Genie space and obtain space ID from Databricks workspace
3. Set up Databricks PAT token in secrets (required for Genie space access)
4. Load data using `data/sec/ingest-sec-data.py` if needed
5. Run cells in `driver.py` sequentially to test, log, register, and deploy the agent
6. Use sample questions in cells 141-143 for testing different complexity levels

### Testing the Agent

- **Simple Questions**: Test with single metric queries (e.g., "What was AAPL's revenue in 2015?")
- **Complex Questions**: Test multi-company comparisons and trend analysis
- **Temporal Context Questions**: Test date-aware queries (e.g., "What is the current fiscal quarter performance?")
- **Sample Test Cases**: Use `sample_questions` array in `driver.py` cells 141-143
- **Response Testing**: Both `predict()` and `predict_stream()` methods available

## Architecture Overview

This is a **multi-agent system** built with LangGraph for financial data analysis using Databricks Genie:

### Core Components

1. **Supervisor Agent** (`supervisor_agent` function in multiagent-genie.py:100)
   - Routes queries between agents based on complexity
   - Uses structured output to determine routing strategy
   - Implements iteration limits (max 3 iterations)

2. **Genie Agent** (multiagent-genie.py:52)
   - Databricks GenieAgent for SQL-based financial data queries
   - Accesses SEC financial data (2003-2022) for AAPL, BAC, AXP
   - Primary data source for Income Statement and Balance Sheet metrics

3. **Parallel Executor Agent** (`research_planner_node` function in multiagent-genie.py:146)
   - Executes parallel queries for complex multi-step analysis using asyncio
   - Uses `asyncio.gather()` with `asyncio.to_thread()` for concurrent Genie queries
   - Preserves MLflow context during parallel execution (eliminates tracing warnings)
   - Synthesizes results from multiple data sources
   - Renamed from "Research Planner" to "Parallel Executor" for clarity

### Data Scope

- **Time Range**: SEC financial data from 2003-2022 (updated from original 2003-2017)
- **Companies**: Apple Inc. (AAPL), Bank of America Corp (BAC), American Express (AXP)
- **Data Types**: Income Statement and Balance Sheet metrics
- **Supported Metrics**: See `data/sec/genie_instruction.md` for full list of financial ratios and calculations

### Workflow Pattern

```text
User Query → Supervisor → [Parallel Executor OR Direct Genie] → Supervisor → Final Answer
```

### Key Technical Details

- **State Management**: Uses LangGraph's `AgentState` with typed state including research plans and results
- **MLflow Integration**: All agent calls are traced with `@mlflow.trace` decorators
- **Chat Interface**: Wrapped in `LangGraphChatAgent` class implementing MLflow's `ChatAgent` interface
- **Streaming Support**: Both `predict` and `predict_stream` methods available with status updates to prevent timeouts
- **Configuration**: Extensive YAML-based configuration for Databricks resources
- **Async Parallel Execution**: Uses asyncio with `asyncio.to_thread()` for concurrent Genie queries (max 3)
- **MLflow Context Preservation**: Eliminates "Failed to get Databricks request ID" warnings
- **Error Handling**: Individual query failures don't cancel other parallel queries (`return_exceptions=True`)
- **Authentication**: Uses Databricks PAT stored in secrets for Genie space access  
- **Temporal Context**: Automatic fiscal year/quarter awareness with real-time date injection into prompts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arifmarias/databricks-deep-research](https://github.com/arifmarias/databricks-deep-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
