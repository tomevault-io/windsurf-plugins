---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DorkAgent is a LLM-powered automated Google Dorking tool for security research, bug bounty hunting, and penetration testing reconnaissance. It uses the CrewAI framework to coordinate AI agents that discover potential security vulnerabilities and exposed sensitive information in target domains through passive reconnaissance only.

## Commands

### Running the Tool
```bash
# Interactive mode (menu-driven interface)
python dorkagent.py

# Note: CLI mode (dorkagent-cli.py) is not yet implemented
```

### Package Installation
The application automatically installs missing dependencies on first run. Manual installation:
```bash
# Python 3.11.9 recommended
pip install python-dotenv crewai crewai-tools langchain-openai termcolor prompt-toolkit pyfiglet schedule
```

### Environment Setup
Create `.env` file with required API keys:
```bash
SERPER_API_KEY=        # Required - https://serper.dev/
OPENAI_API_KEY=        # Optional - set if using OpenAI
ANTHROPIC_API_KEY=     # Optional - set if using Anthropic
GEMINI_API_KEY=        # Optional - set if using Gemini (recommended for free usage)
```

## High-Level Architecture

### Modular Structure
The codebase follows a modular architecture after the v1.4 refactoring:

- **dorkagent.py**: Main entry point with orchestration logic, handles dependency installation, banner display, and CrewAI workflow execution
- **config.py**: LLM configuration and API key management (selection, validation, instantiation)
- **agents.py**: CrewAI agent definitions (searcher, bughunter, writer)
- **tasks.py**: CrewAI task definitions with Google Dork queries and analysis prompts
- **utils.py**: Utility functions for domain input, depth adjustment, filename sanitization, and notifications

### Core Workflow
The application follows a sequential workflow pattern coordinated by CrewAI:

1. **Initialization Phase** (dorkagent.py:98-103)
   - `select_llm_type()`: User selects LLM provider (returns "openai", "anthropic", or "gemini")
   - `ensure_api_keys()`: Validates and prompts for missing API keys, creates `.env` if needed
   - `create_llm()`: Instantiates LLM object after keys are verified
   - **CRITICAL**: Must follow exact order to avoid initialization errors

2. **Configuration Phase** (dorkagent.py:109-118)
   - `get_target_domains()`: Single domain or from file (utils.py:12-43)
   - `select_depth()`: Search depth selection 1-3 (utils.py:45-59)
   - `adjust_depth()`: Apply wildcard patterns (utils.py:74-91)
   - `integrate_notify()`: Optional notify tool integration (utils.py:61-72)

3. **Execution Phase** (dorkagent.py:125-160)
   - **Task 1** (tasks.py:6-120): `searcher` agent executes 30 Google Dork queries via SerperDevTool
   - **Task 2** (tasks.py:122-236): `bughunter` agent analyzes URLs for vulnerabilities via ScrapeWebsiteTool
   - **Task 3** (tasks.py:238-471): `writer` agent generates structured markdown security report
   - Reports saved to `./log/YYMMDD/YYMMDD_HHMMSS_domain.md`

### Critical Implementation Details

**Dependency Auto-Installation** (dorkagent.py:1-52):
- Runs BEFORE any third-party imports to avoid ImportError
- Uses try/except block to detect missing packages
- Exits after installation with message to restart
- Warns if Python version differs from 3.11.9

**LLM Configuration** (config.py):
- `select_llm_type()`: Interactive menu returns string identifier
- `ensure_api_keys()`: Reads/writes `.env`, prompts for missing keys using getpass
- `create_llm()`: Instantiates provider-specific objects:
  - OpenAI: ChatOpenAI with `gpt-4.1-mini-2025-04-14`
  - Anthropic: CrewAI LLM wrapper with `anthropic/claude-3-5-haiku-20241022`
  - Gemini: CrewAI LLM wrapper with `gemini/gemini-2.5-flash`

**Agent Configuration** (agents.py):
- Three specialized agents: searcher (Google Dorking), bughunter (vulnerability analysis), writer (report generation)
- All agents share same LLM instance passed from main
- Tools: SerperDevTool for searches, ScrapeWebsiteTool for content analysis
- No agent delegation (each operates independently)

**Google Dork Query Structure** (tasks.py:6-120):
- 30 hardcoded queries targeting different vulnerability categories:
  - Queries 1-15: Attack vectors (file exposure, admin panels, injection points, APIs)
  - Queries 16-30: Information disclosure (errors, logs, cloud storage, PII, infrastructure)
- Extensive exclusion criteria to filter false positives (tasks.py:59-90)
- JSON-structured output format for consistent parsing
- Must execute ALL 30 queries sequentially

**Depth Adjustment Logic** (utils.py:74-91):
- Level 1: `domain.com`
- Level 2: `*.domain.com`
- Level 3: `*.*.domain.com`
- Wildcard domains sanitized in filenames: `*` → `wildcard` (utils.py:93-101)

### Key Integration Points

**CrewAI Configuration** (dorkagent.py:140-146):
- `max_rpm=15`: Rate limiting for API providers
- `verbose=1`: Progress output
- `output_log_file=True`: Automatic logging
- Sequential task execution (no parallel processing)

**SerperDevTool Customization**:
- Default 10 results per query (modifiable in site-packages/crewai_tools/tools/serper_dev_tool/)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yee-yore/DorkAgent](https://github.com/yee-yore/DorkAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
