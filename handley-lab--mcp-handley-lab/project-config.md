---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ CRITICAL SECURITY RULES

**NEVER USE --break-system-packages FLAG**

- This is an externally managed environment (Arch Linux with pacman)
- NEVER run `pip install --break-system-packages` under any circumstances
- If package installation fails, use virtual environments: `python -m venv venv && source venv/bin/activate`
- System package management must remain intact for system stability
- Breaking system packages can corrupt the entire Python installation

**NEVER MODIFY FILES OUTSIDE THE PROJECT DIRECTORY**

- NEVER copy, move, overwrite, or delete files outside the project directory
- NEVER modify credential files in `~/` (home directory)
- NEVER touch system files, config files, or user data outside the project
- If testing requires different credentials, configure within the codebase using fixtures/environment variables
- When in doubt, ask the user before touching ANY file outside the project directory
- This prevents data loss and maintains system integrity

## Project Overview

This is an MCP (Model Context Protocol) framework project designed to bridge various external services and command-line utilities into a unified API. The framework provides a comprehensive toolkit for:

- **Code & Git Interaction**: Converting, flattening, and diffing codebases via `code2prompt`
- **AI Model Integration**: Unified LLM tools supporting Gemini, OpenAI, Claude, Grok, Mistral, and Groq
- **Productivity & Scheduling**: Google Calendar management
- **Academic Research**: ArXiv paper source code retrieval and analysis
- **Persistent Memory**: Agent management with conversational memory for LLMs

## Filing GitHub Issues for MCP Tool Bugs

When you encounter a bug or issue in an MCP tool, file a GitHub issue with `gh issue create`. Reference the correct source file path so developers can locate the code.

### MCP Tool Source Locations

| MCP Server | Tool File | Description |
|------------|-----------|-------------|
| `mcp-llm` | `src/mcp_handley_lab/llm/tool.py` | Chat, vision, image gen, transcribe, OCR, models |
| `mcp-llm-embeddings` | `src/mcp_handley_lab/llm/embeddings/tool.py` | Text embeddings & semantic search |
| `mcp-email` | `src/mcp_handley_lab/email/tool.py` (entry); `email/notmuch/tool.py` (read/update), `email/msmtp/tool.py` (send), `email/offlineimap/tool.py` (sync) | Email read/send/update/sync |
| `mcp-google-calendar` | `src/mcp_handley_lab/google_calendar/tool.py` | Calendar CRUD & search |
| `mcp-google-maps` | `src/mcp_handley_lab/google_maps/tool.py` | Directions & routes |
| `mcp-google-photos` | `src/mcp_handley_lab/google_photos/tool.py` | Photo search, browse, detail, download |
| `mcp-loop` | `src/mcp_handley_lab/loop/tool.py` | Persistent REPL sessions (Python, Bash, Julia, R, Claude) |
| `mcp-mathematica` | `src/mcp_handley_lab/mathematica/tool.py` | Wolfram Language evaluation |
| `mcp-word` | `src/mcp_handley_lab/microsoft/word/tool.py` | Word document read/edit |
| `mcp-excel` | `src/mcp_handley_lab/microsoft/excel/tool.py` | Excel workbook read/edit |
| `mcp-powerpoint` | `src/mcp_handley_lab/microsoft/powerpoint/tool.py` | PowerPoint presentation read/edit |
| `mcp-visio` | `src/mcp_handley_lab/microsoft/visio/tool.py` | Visio diagram read/edit |
| `mcp-search` | `src/mcp_handley_lab/search/tool.py` | Conversation transcript search |
| `mcp-screenshot` | `src/mcp_handley_lab/screenshot/tool.py` | Window/screen capture |
| `mcp-code2prompt` | `src/mcp_handley_lab/code2prompt/tool.py` | Codebase summarization |
| `mcp-arxiv` | `src/mcp_handley_lab/arxiv/tool.py` | ArXiv paper download |
| `mcp-otter` | `src/mcp_handley_lab/otter/tool.py` | Otter.ai live meeting transcripts |

### Issue Template

```bash
gh issue create \
  --title "fix(<server>): <short description>" \
  --body "## Description
<What happened vs what was expected>

## Source
Tool file: \`<path from table above>\`

## Steps to Reproduce
1. ...
2. ...

## Error Output
\`\`\`
<paste error>
\`\`\`"
```

## ⚠️ CRITICAL: VERSION MANAGEMENT REQUIRED FOR ALL CHANGES

**BEFORE ANY COMMIT OR PR: ALWAYS BUMP VERSION USING THE AUTOMATED SCRIPT**

```bash
# Use the automated version bump script with semantic versioning
python scripts/bump_version.py         # Auto-detect minimal bump (0.0.1b5 → 0.0.1b6, 0.0.1 → 0.0.2)
python scripts/bump_version.py patch   # For bug fixes (0.0.0b5 → 0.0.1)
python scripts/bump_version.py minor   # For new features (0.0.0b5 → 0.1.0)
python scripts/bump_version.py major   # For breaking changes (0.0.0b5 → 1.0.0)

# For development iterations:
python scripts/bump_version.py beta    # Beta versions (0.0.1 → 0.0.1b1, 0.0.1b5 → 0.0.1b6)

# For release process:
python scripts/bump_version.py rc      # Release candidate (0.0.0b5 → 0.0.0rc1)
python scripts/bump_version.py release # Final release (0.0.0rc1 → 0.0.0)

# Test first with dry-run:
python scripts/bump_version.py --dry-run
```

**The script automatically updates both pyproject.toml and PKGBUILD** - never edit version numbers manually.

**GitHub CI WILL FAIL** if versions don't match or aren't bumped from master. This is enforced automatically.

### Commit Quality Standards


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [handley-lab/mcp-handley-lab](https://github.com/handley-lab/mcp-handley-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
