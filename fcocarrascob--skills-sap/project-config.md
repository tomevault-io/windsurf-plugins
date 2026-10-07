---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

﻿# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Skills_SAP** is a SAP2000 automation framework that bridges Claude Code to SAP2000 structural analysis software via Model Context Protocol (MCP) and COM automation. It enables engineers to generate structural models, assign loads, run analyses, and extract results programmatically through conversational AI.

Architecture: User → Claude Code → MCP Server (Python) → SAP2000 COM objects → Results

## Core Architecture

### MCP Server Layer (`mcp_server/`)

The server exposes 12 MCP tools. Key modules:

- **`server.py`** — FastMCP entry point; registers all tools with `.mcp.tool()` decorator
- **`sap_bridge.py`** — Singleton COM connection manager to SAP2000; handles connection/disconnection and version detection
- **`sap_executor.py`** — Executes API functions and scripts in restricted sandbox (blocked imports: `os`, `subprocess`, `sys`, `pathlib`, `socket`, etc.; timeout: 120s). Navigates COM object hierarchy via dot-path resolution
- **`script_library.py`** — Persists executed scripts to `scripts/` with metadata headers; enables script search/reload
- **`doc_search.py`** — Full-text search over API documentation files in `API/` directory
- **`function_registry.py`** — Manages `scripts/registry.json` (catalog of 200+ verified functions); tracks verification status, wrapper scripts, and usage metadata

### Script Execution Model

Scripts passed to `run_sap_script` run in restricted Python environment with **pre-injected variables**:
- `SapModel` — Active model COM object
- `SapObject` — SAP2000 application COM object
- `result` — Dict for writing output (for verification)
- `sap_temp_dir` — Temp directory for `File.Save()` calls

All scripts follow the **Universal Pattern**:
1. Initialize model (`InitializeNewModel`, `NewBlank`)
2. Define materials (`PropMaterial.SetMaterial`, `SetMPIsotropic`)
3. Define sections (`PropFrame.Set*`, `PropArea.SetShell_1`)
4. Create geometry (`FrameObj.AddByCoord`, `AreaObj.AddByCoord`)
5. Add constraints (`PointObj.SetRestraint`, `ConstraintDef.*`)
6. Add loads (`LoadPatterns.Add`, `SetLoadDistributed`)
7. Run analysis (`File.Save`, `Analyze.RunAnalysis`)
8. Extract results (`Results.JointDispl`, `Results.AreaForceShell`)
9. Verify with assertions and populate `result`

### Function Registry Organization

**`scripts/registry.json`** tracks 200 verified functions across **22 categories**:
- File, PropMaterial, PropFrame, PropArea, Properties
- FrameObj, AreaObj, Object_Model
- Load_Patterns, Load_Cases, RespCombo
- Constraints, Groups, Design
- Analyze, Analysis_Results
- Database_Tables, Edit, Functions, Mass_Source, Select

Each entry includes: category, description, verification status, wrapper script path, usage notes.

### ByRef Convention (Critical)

SAP2000 API heavily uses ByRef parameters. In Python via COM, functions return **tuples** where:
- All ByRef outputs appear **first**
- Last element `raw[-1]` is **always the return code** (Long)
- Return code 0 = success, nonzero = error

**Pattern:**
```python
raw = SapModel.PointObj.AddCartesian(5, 3, 2, "", "PT1")
point_name = raw[0]   # ByRef Name output
ret_code = raw[-1]    # return code (ALWAYS last)
assert ret_code == 0, f"AddCartesian failed: {ret_code}"
```

Verified layouts in `scripts/registry.json` and documented in `.claude/skills/sap2000-api/references/enum-reference.md`.

## Development Commands

### Setup

```bash
# Create virtual environment
python -m venv .venv

# Activate (Windows PowerShell)
.venv\Scripts\Activate.ps1

# Install dependencies
pip install -r mcp_server/requirements.txt
```

### Running the MCP Server

**Manual test (for debugging):**
```bash
python mcp_server/server.py
```

**Automatic:** The server is registered at user scope (see [Connecting MCP to Claude Code](#connecting-mcp-to-claude-code)) and auto-starts over stdio when an MCP tool is invoked.

### Running Scripts

Scripts are executed via `run_sap_script` MCP tool. Example wrappers in `scripts/wrappers/func_*.py` demonstrate usage of individual API functions.

To test a wrapper manually (requires SAP2000 running):
```python
# Use the run_sap_script tool with the code from
# scripts/wrappers/func_FrameObj_AddByCoord.py
```

### Testing

Unit tests live in `mcp_server/tests/`. Run with pytest:
```bash
cd mcp_server
pytest tests/ -v
```

## Important Files & Their Roles

### Documentation
- **`README.md`** — Project overview, architecture diagram, component summary
- **`.claude/skills/sap2000-api/SKILL.md`** — Agent skill; API technical reference (ByRef layouts, templates, registry patterns) plus the scripting workflow (plan → research → execute → verify → register). Auto-loaded when working on SAP2000 scripts
- **`.claude/skills/sap2000-api/references/common-workflows.md`** — Step-by-step patterns (simple beam, portal frame, verification problem)
- **`scripts/README.md`** — Script library usage
- **`scripts/wrappers/README.md`** — Wrapper script naming and format
- **`scripts/database_tables/README.md`** — Database Tables module backend + GUI API

### Examples & Notebooks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fcocarrascob/Skills_SAP](https://github.com/fcocarrascob/Skills_SAP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
