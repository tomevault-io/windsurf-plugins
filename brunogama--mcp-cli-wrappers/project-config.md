---
trigger: always_on
description: - **Type**: CLI Tool Collection (Monolithic)
---

# CLI Wrappers - MCP Tool Collection

## Overview

- **Type**: CLI Tool Collection (Monolithic)
- **Stack**: Python 3.10+ with `uv` (PEP 723 script execution)
- **Purpose**: Progressive-disclosure CLI wrappers for Model Context Protocol (MCP) tools
- **Architecture**: 13 self-contained, single-file Python scripts
- **Location**: `~/cli-wrappers/`

This CLAUDE.md is the authoritative source for development guidelines.

---

## MANDATORY: Before Using Any Tool

**ALWAYS run these commands first:**
```bash
uv run TOOL.py --help    # Quick overview
uv run TOOL.py list      # See all functions
```

---

## CLI Quick Reference

| Tool | Purpose | When to Use |
|------|---------|-------------|
| **crawl4ai** | Web scraping | Scrape pages, extract content |
| **firecrawl** | Advanced scraping | JS-heavy sites, structured extraction |
| **ref** | Doc search | Find API/framework documentation |
| **deepwiki** | Repo docs | Understand GitHub repos, ask questions |
| **gitmcp** | GitHub docs | Fetch llms.txt, READMEs from repos |
| **semly** | Code analysis | Outline code, find patterns |
| **repomix** | Pack codebases | Prepare code for AI analysis |
| **github** | GitHub API | Issues, PRs, file operations |
| **sequential-thinking** | Reasoning | Complex problems, design decisions |
| **claude-flow** | Orchestration | Multi-agent workflows |
| **flow-nexus** | Cloud execution | Sandboxed code execution |

**Full reference**: See `.claude/CLI_REFERENCE.md`

---

## Universal Development Rules

### Code Quality (MUST)

- **MUST** use PEP 723 headers for uv script execution
- **MUST** return valid JSON from all functions
- **MUST** validate arguments before execution
- **MUST** check API keys and credentials exist before MCP calls
- **MUST** follow 4-level progressive disclosure pattern
- **MUST NOT** add external CLI dependencies beyond `httpx`, `pydantic`, `click`, `rich`
- **MUST NOT** execute shell commands with `subprocess` without validation

### Best Practices (SHOULD)

- **SHOULD** use `rich` library for formatted output
- **SHOULD** provide clear error messages with recovery steps
- **SHOULD** include timeout limits on network requests (30s default)
- **SHOULD** support `--format json|text|table` output options
- **SHOULD** cache API responses when reasonable (5-minute TTL)
- **SHOULD** document all environment variables required
- **SHOULD** include helpful examples in `FUNCTION_EXAMPLES`

### Anti-Patterns (MUST NOT)

- **MUST NOT** use `subprocess.run()` with `shell=True`
- **MUST NOT** store credentials in code (use environment variables)
- **MUST NOT** execute user input directly
- **MUST NOT** suppress errors silently (always log/display)
- **MUST NOT** block on long-running operations (use async when possible)
- **MUST NOT** skip argument validation before MCP calls

---

## 4-Level Progressive Disclosure Pattern

All wrappers implement this pattern for token efficiency (60-70% savings):

### Level 1: `--help`
```bash
uv run WRAPPER.py --help
```
- Quick overview (10-15 lines)
- Available functions listed
- Usage examples
- Pointer to next level
- **Tokens**: ~30

### Level 2: `list` and `info FUNCTION`
```bash
uv run WRAPPER.py list                    # All functions
uv run WRAPPER.py info FUNCTION_NAME      # Detailed docs
```
- Full function signature
- All parameters with types
- Return type and format
- Related functions
- **Tokens**: ~150

### Level 3: `example FUNCTION`
```bash
uv run WRAPPER.py example FUNCTION_NAME
```
- 2-3 real working examples
- Success and error cases
- Realistic sample data
- Copy-paste ready
- **Tokens**: ~200

### Level 4: `FUNCTION --help`
```bash
uv run WRAPPER.py FUNCTION_NAME --help
```
- Complete argparse reference
- All OPTIONS with detailed explanations
- USAGE section
- ERROR CASES with recovery hints
- **Tokens**: ~500

---

## Project Structure

### Core Wrappers (MCP Tools)
- **`crawl4ai.py`** → Web scraping and crawling
- **`firecrawl.py`** → Advanced web data extraction
- **`ref.py`** → Documentation search
- **`repomix.py`** → Repository analysis and packing
- **`deepwiki.py`** → GitHub wiki documentation
- **`github.py`** → GitHub repository operations
- **`semly.py`** → Code search and analysis
- **`exa.py`** → Web search with AI
- **`gitmcp.py`** → GitHub docs via SSE (gitmcp.io)
- **`sequential-thinking.py`** → Multi-step reasoning
- **`claude-flow.py`** → Agentic workflow orchestration
- **`flow-nexus-cli-wrapper.py`** → Cloud orchestration

### Tooling & Documentation
- **`generate-all-wrappers.py`** → Master introspection script
- **`README.md`** → Quick start and overview
- **`INDEX.md`** → Complete tool index
- **`ref.md`** → Ref.py documentation
- **`exa-cli.md`** → Exa.py documentation
- **`.claude/`** → Claude Code configuration
- **`CLAUDE.md`** → This file

---

## Core Commands & Workflows

### Quick Start (After `cd ~/cli-wrappers/`)

#### Explore Any Wrapper (4 Levels)
```bash
# Level 1: Quick overview
uv run WRAPPER.py --help

# Level 2: See all functions
uv run WRAPPER.py list

# Level 2: Detailed function docs
uv run WRAPPER.py info FUNCTION_NAME

# Level 3: Working examples
uv run WRAPPER.py example FUNCTION_NAME

# Level 4: Complete reference
uv run WRAPPER.py FUNCTION_NAME --help
```

### Common Tasks

#### Scrape a Website
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brunogama) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
