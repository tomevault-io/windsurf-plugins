---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains four standalone demos showcasing Pydantic AI integrated with Temporal workflows. The project demonstrates different patterns of agent orchestration, from simple single-agent workflows to complex multi-agent research systems with interactive user clarifications.

## Key Directories

- `pydantic_demos/workflows/` - Contains all workflow implementations
  - `hello_world_workflow.py` - Simple haiku-generating agent (Pydantic AI)
  - `tools_workflow.py` - Weather tool demo (Pydantic AI)
  - `research_bot_workflow.py` - Simple research workflow using PydanticSimpleResearchManager (Pydantic AI)
  - `interactive_research_workflow.py` - Interactive research workflow using PydanticInteractiveResearchManager
  - `simple_research_manager.py` - Simplified research orchestrator for basic workflow (Pydantic AI)
  - `interactive_research_manager.py` - Interactive research orchestrator for interactive workflow (Pydantic AI)
  - `pdf_generation_activity.py` - PDF generation activity using WeasyPrint
  - `research_agents/` - Research agent components for Pydantic AI
    - `research_models.py` - Data models for research interactions
    - `triage_agent.py` - Query analysis agent
    - `clarifying_agent.py` - Question generation agent
    - `planner_agent.py` - Research planning agent
    - `search_agent.py` - Web search agent
    - `writer_agent.py` - Report writing agent
    - `pdf_generator_agent.py` - PDF generation agent
- `pydantic_demos/run_*_workflow.py` - Client runners for each demo
- `pydantic_demos/run_worker.py` - Worker that registers all Pydantic AI workflows

## Architecture Overview

### Core Components

**Temporal Workflows**: All demos use Temporal's durable execution engine for reliable orchestration. Each workflow is registered in `run_worker.py` and has a corresponding client runner.

**Pydantic AI Integration**: Uses the `pydantic-ai` package for agent integration within Temporal workflows.

**Multi-Agent Research System**: The research demos implement a sophisticated multi-agent pipeline:
- **Triage Agent**: Analyzes queries and determines if clarifications are needed
- **Clarifying Agent**: Generates follow-up questions for better research parameters
- **Planner Agent**: Creates web search plans
- **Search Agent**: Performs web searches
- **Writer Agent**: Compiles final research reports
- **PDF Generator Agent**: Converts markdown reports to professionally formatted PDFs

Note: Query enrichment (combining user responses with original queries) is handled by the Interactive Research Manager via the `_enrich_query()` method, not by a separate agent.

### Workflow Patterns

**Simple Execution Pattern**: Direct workflow execution (Hello World, Tools, Simple Research)
```python
result = await client.execute_workflow(WorkflowClass.run, args, id="workflow-id", task_queue="pydantic-ai-task-queue")
```

**Interactive Long-Running Pattern**: Used in Interactive Research
- Uses `@workflow.update` for user interactions
- Uses `@workflow.query` for status checking
- Uses `@workflow.signal` for workflow termination
- Implements `workflow.wait_condition()` for long-running workflows

### Research Manager Architecture

**PydanticSimpleResearchManager** (`simple_research_manager.py`) - Used by basic research workflow:
- **Direct Flow**: Simple planner → search → writer pipeline
- **No Clarifications**: Streamlined for demonstration purposes  
- **Agent Coordination**: Basic orchestration without user interaction
- **No PDF Generation**: Markdown-only output for simplicity

**PydanticInteractiveResearchManager** (`interactive_research_manager.py`) - Used by interactive workflow:
- **Direct Flow**: `_run_direct()` for simple research
- **Clarification Flow**: `run_with_clarifications_start()` and `run_with_clarifications_complete()` for interactive research
- **Agent Coordination**: Manages the pipeline of triage → clarification → query enrichment → planner → search → writer → PDF generation
- **PDF Generation**: Optional PDF generation with graceful degradation

### Data Models

The `research_models.py` file defines key data structures:
- `ResearchInteraction`: Tracks state of interactive research sessions
- `ClarificationInput`/`SingleClarificationInput`: User input models
- `UserQueryInput`: Initial query input model

## Development Commands

### Environment Setup
The project uses `uv` for dependency management. Always ensure dependencies are synced:

```bash
# Install dependencies
uv sync

# Development dependencies are automatically included via [dependency-groups]
```

### Running Workflows with Workers

#### Standard Workflow Execution Pattern

For any workflow, use this pattern (adjust timeout based on workflow complexity):

```bash
# Start worker in background
uv run pydantic_demos/run_worker.py &
WORKER_PID=$!
echo "Worker started with PID: $WORKER_PID"

# Wait for worker initialization
sleep [TIME]

# Run the workflow (adjust timeout as needed)
echo "Running [workflow_name] workflow..."
timeout [SECONDS] uv run pydantic_demos/run_[workflow_name]_workflow.py

# Clean up worker

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [temporal-community/pydantic-ai-demos](https://github.com/temporal-community/pydantic-ai-demos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
