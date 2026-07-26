---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ClimSight is a climate decision support system that integrates Large Language Models (LLMs) with climate data to provide localized climate insights. It uses a multi-agent architecture built on LangGraph to combine climate model data, geographic information, RAG (Retrieval Augmented Generation) from scientific reports, and LLM reasoning.

**Key Technologies:**
- **LangChain/LangGraph**: Multi-agent orchestration framework
- **Streamlit**: Web UI framework
- **xarray/NetCDF4**: Climate data processing
- **Chroma**: Vector database for RAG
- **GeoPandas/OSMnx**: Geospatial analysis
- **OpenAI API**: LLM backend (supports custom models via AITTA platform)
- **earthkit.data**: DestinE data retrieval via polytope
- **Arraylake**: ERA5 reanalysis data access

## Common Commands

### Environment Setup

```bash
# Using conda/mamba (recommended)
mamba env create -f environment.yml
conda activate climsight
python download_data.py  # Downloads ~8GB of climate data

# Using pip
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -e .
python download_data.py
```

### Running the Application

```bash
# Development mode (from repo root)
streamlit run src/climsight/climsight.py

# If installed via pip
climsight

# Testing mode (no OpenAI API calls)
streamlit run src/climsight/climsight.py skipLLMCall
```

### Testing

```bash
# Run all tests (DestinE tests excluded by default)
cd test
pytest

# Run specific test categories (see test/pytest.ini for all markers)
pytest -m geo           # Geographic functions
pytest -m climate       # Climate data functions
pytest -m env           # Environmental functions
pytest -m "not request" # Skip tests requiring HTTP requests

# DestinE tool tests (require ~/.polytopeapirc token + OPENAI_API_KEY)
pytest -m destine -v                    # All DestinE tests
pytest -m destine -v -k search          # RAG search only (fast)
pytest -m destine -v -k retrieve        # Data retrieval only

# Run single test file
pytest test_geofunctions.py

# Run with verbose output
pytest -v
```

### Linting

```bash
# Syntax errors and undefined names only (CI uses this)
flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
```

### Batch Processing (sequential/)

```bash
cd sequential

# Generate climate questions
python question_generator.py

# Visualize questions on map
streamlit run question_map.py

# Process questions through ClimSight
python question_runner.py --questions_file Q_1.json --llm_model gpt-4.1-nano
```

## Architecture

### Multi-Agent Workflow (LangGraph)

ClimSight uses a state machine with specialized agents that process user questions:

1. **intro_agent** ([climsight_engine.py:1079](src/climsight/climsight_engine.py#L1079))
   - Entry point that filters invalid requests
   - Uses exclusion-based logic to determine if query is climate-related
   - Routes to either FINISH (invalid) or CONTINUE (parallel agents)

2. **Parallel Information Gathering Agents:**
   - **ipcc_rag_agent** ([climsight_engine.py:1031](src/climsight/climsight_engine.py#L1031)): Searches IPCC reports via RAG
   - **general_rag_agent** ([climsight_engine.py:1052](src/climsight/climsight_engine.py#L1052)): Searches general climate literature via RAG
   - **data_agent** ([climsight_engine.py:854](src/climsight/climsight_engine.py#L854)): Extracts climate model data for location
   - **zero_rag_agent** ([climsight_engine.py:723](src/climsight/climsight_engine.py#L723)): Gathers geographic/environmental context
   - **smart_agent** ([smart_agent.py:71](src/climsight/smart_agent.py#L71)) — OPTIONAL:
     - Information gathering only (Wikipedia, RAG, ECOCROP)
     - Controlled by `use_smart_agent` in config
     - No Python REPL — that is now in data_analysis_agent

3. **prepare_predefined_data** ([climsight_engine.py:917](src/climsight/climsight_engine.py#L917))
   - Runs after all parallel agents complete
   - Extracts ERA5 climatology and generates predefined plots (climate comparison with ERA5 overlay, disaster summary, population projection)
   - Routes via `route_after_prepare`: if `use_powerful_data_analysis` → data_analysis_agent, otherwise → combine_agent

4. **data_analysis_agent** ([data_analysis_agent.py:476](src/climsight/data_analysis_agent.py#L476)) — OPTIONAL:
   - Receives all outputs from parallel agents + predefined plots
   - Performs data extraction, post-processing, and visualization using tool-calling
   - Tools: Python REPL (Jupyter kernel), ERA5 retrieval, DestinE search + retrieval, image viewer, reflection, visualization strategy
   - Controlled by `use_powerful_data_analysis` in config

5. **combine_agent** ([climsight_engine.py:1181](src/climsight/climsight_engine.py#L1181))
   - Synthesizes all agent outputs into final answer
   - Generates references and formatted response

**Routing Logic:**
- `route_fromintro` launches parallel agents (including smart_agent if enabled)
- All parallel agents converge to `prepare_predefined_data`
- `route_after_prepare` conditionally invokes `data_analysis_agent` or skips to `combine_agent`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CliDyn/climsight](https://github.com/CliDyn/climsight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
