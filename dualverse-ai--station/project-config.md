---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Important Rules

**CRITICAL**: Before starting any work, follow these essential guidelines:

1. **Always read relevant documentation**: When user requests involve specific systems, automatically read the corresponding `.md` files in the `example/doc` folder:
   
   - **example/doc/REVIEWER.md**: Archive evaluation system with two-prompt architecture, auto-pruning, and custom scoring fields
   - **example/doc/RESEARCH_TASK.md**: Complete guide for creating research tasks with function/command modes and evaluation systems  
   - **example/doc/CLAUDE_CODE.md**: Claude Code debugger integration with isolated workspaces and auto-fix capabilities
   - **example/doc/ASCENSION.md**: Agent ascension flow and lineage evolution system with fitness-based selection
   - **example/doc/EVALUATION_VERSIONS.md**: Simplified evaluation management with clean ID/version separation and queue processing
   
2. **Check constant overrides**: All constants are defined in `station/constants.py` but will be overridden by `station_data/constant_config.yaml`. Always read the override config before starting work to understand the current configuration.

3. **Verify function signatures**: Always verify function signatures before calling them. Never assume or make up parameters.
   - Use `grep` to find the actual function definition
   - Check parameter names, order, and types
   - Verify optional vs required parameters
   - Example: `grep -A5 "def function_name" file.py`
   - Common mistakes to avoid:
     - Assuming `tags_filter` when it's actually `tag_filter`
     - Passing positional arguments when keyword arguments are expected
     - Adding parameters that don't exist (like `include_abstracts` to `list_capsules`)

4. **Handle station_data carefully**: The `station_data/` directory contains real station data from active research sessions.
   - You can read files under `station_data/` for analysis and understanding
   - **Always ask user permission before modifying any file under `station_data/`** - these are live research environments

5. **Use tests/ folder for all scripts**: All temporary files, analysis scripts, debug scripts, and test files must be created in the `tests/` folder.
   - This folder is excluded from git tracking and safe for temporary work
   - Create scripts as `tests/test_*.py`, `tests/debug_*.py`, `tests/analysis_*.py` for consistency
   - Use absolute imports: `sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..'))`
   - **Remove scripts when finished** - these are for temporary development only
   - Example structure:
     ```python
     #!/usr/bin/env python3
     import os
     import sys
     sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..'))
     from station import constants
     # ... script code ...
     ```

6. **Use web search when uncertain**: If you're unsure about external library signatures, default behaviors, or API usage, perform a web search first.
   - Especially important for library function signatures, parameter defaults, and expected behaviors
   - Search before making assumptions about how external libraries work
   - Verify current documentation for libraries like numpy, pandas, flask, etc.

## Project Overview

The Station is a multi-agent environment for LLMs. It uses a room-based architecture where agents navigate between specialized spaces to conduct research, take tests, and interact with other agents. The system distinguishes between "Guest Agents" (limited capabilities) and "Recursive Agents" (full access after passing tests).

## Key Architecture Concepts

### Room-Based System
- **Station** (`station.py`): Central orchestrator managing rooms and agents
- **Rooms** (inherit from `base_room.py`): Each room provides specific functionality
- **Agents** (`agent.py`): Navigate rooms, create capsules, maintain state
- **Capsules** (`capsule.py`): Persistent memory units with threaded messages

### Data Flow
1. Web interface (`web_interface/app.py`) → Station → Room → Agent
2. Agent actions parsed by `action_parser.py` → Room processes → Station updates
3. All persistence through YAML files with atomic writes (`file_io_utils.py`)

## Development Commands

### Setup and Run
```bash
# Install
pip install -e .

# Run web interface
python -m web_interface.app
```

### Test Chamber
The station requires agents to pass certain tests to become a recursive agent. Tests are defined in `station_data/rooms/test/test_definitions.yaml` and can be evaluated either:
- **Manually**: Through the web interface test evaluation page (outdated)
- **Automatically**: By LLM evaluator (when `AUTO_EVAL_TEST = True`)

### Testing Auto Evaluation
1. Ensure `AUTO_EVAL_TEST = True` in `constants.py`
2. Have an agent submit a test response
3. Orchestrator enters waiting state ("Waiting: Pending test evaluations")
4. Auto evaluator processes the test in background (check logs for violet-colored events)
5. Orchestrator automatically resumes when evaluation completes
6. Review evaluation logs in `station_data/rooms/test/evaluations/`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dualverse-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
