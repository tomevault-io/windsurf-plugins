---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with this MCP Memory Service repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this MCP Memory Service repository.

> **📝 Personal Customizations**: You can create `CLAUDE.local.md` (gitignored) for personal notes, custom workflows, or environment-specific instructions. This file contains shared project conventions.

> **Information Lookup**: Files first, memory second, user last. See [`.claude/directives/memory-first.md`](.claude/directives/memory-first.md) for strategy. Comprehensive project context stored in memory with tags `claude-code-reference`.

## 🔴 Critical Directives

**IMPORTANT**: Before working with this project, read:
- **`.claude/directives/memory-tagging.md`** - MANDATORY: Always tag memories with `mcp-memory-service` as first tag
- **`.claude/directives/README.md`** - Additional topic-specific directives

## ⚡ Quick Update & Restart (RECOMMENDED)

**ALWAYS use these scripts after git pull to update dependencies and restart server:**

```bash
# macOS/Linux - One command, <2 minutes
./scripts/update_and_restart.sh

# Windows PowerShell
.\scripts\service\windows\update_and_restart.ps1
```

**Why?** These scripts automate the complete update workflow:
- ✅ Git pull + auto-stash uncommitted changes
- ✅ Install dependencies (editable mode: `pip install -e .`)
- ✅ Restart HTTP server with version verification
- ✅ Health check (ensures new version is running)

**Without these scripts**, you risk running old code (common mistake: forget `pip install -e .` after pull).

See [Essential Commands](#essential-commands) for options (--no-restart, --force).

## Overview

MCP Memory Service is a Model Context Protocol server providing semantic memory and persistent storage for Claude Desktop with SQLite-vec, Cloudflare, and Hybrid storage backends.

> **🆕 v8.76.0**: **Official Lite Distribution Support** - New `mcp-memory-service-lite` package with 90% size reduction (7.7GB → 805MB), automated dual publishing workflow, conditional transformers loading, and multi-protocol port detection fixes (HTTP/HTTPS health checks, cross-platform fallback chain: lsof → ss → netstat → ps). See [CHANGELOG.md](CHANGELOG.md) for full version history.
>
> **Note**: When releasing new versions, update this line with current version + brief description. Use `.claude/agents/github-release-manager.md` agent for complete release workflow.

## Essential Commands

**Most Used:**
- `./scripts/update_and_restart.sh` - Update & restart (ALWAYS after git pull)
- `curl http://127.0.0.1:8000/api/health` - Health check
- `bash scripts/pr/pre_pr_check.sh` - Pre-PR validation (MANDATORY)
- `curl -X POST http://127.0.0.1:8000/api/consolidation/trigger -H "Content-Type: application/json" -d '{"time_horizon":"weekly"}'` - Trigger consolidation

**Full command reference:** [scripts/README.md](scripts/README.md)

## Architecture

**Core Components:**
- **Server Layer**: MCP protocol with async handlers, global caches (`src/mcp_memory_service/server.py:1`)
- **Storage Backends**: SQLite-Vec (5ms reads), Cloudflare (edge), Hybrid (local + cloud sync)
- **Web Interface**: FastAPI dashboard at `http://127.0.0.1:8000/` with REST API
- **Document Ingestion**: PDF, DOCX, PPTX loaders (see [docs/document-ingestion.md](docs/document-ingestion.md))
- **Memory Hooks**: Natural Memory Triggers v7.1.3+ with 85%+ accuracy (see below)

**Utility Modules** (v8.61.0 - Phase 3 Refactoring):
- `utils/health_check.py` - Strategy Pattern for backend health checks (5 strategies)
- `utils/startup_orchestrator.py` - Orchestrator Pattern for server startup (3 orchestrators)
- `utils/directory_ingestion.py` - Processor Pattern for file ingestion (3 processors)
- `utils/quality_analytics.py` - Analyzer Pattern for quality distribution (3 analyzers)

**Key Patterns:**
- Async/await for I/O, type safety (Python 3.10+), platform hardware optimization (CUDA/MPS/DirectML/ROCm)
- Design Patterns: Strategy, Orchestrator, Processor, Analyzer (all complexity A-B grade)

## Document Ingestion

Supports PDF, DOCX, PPTX, TXT/MD with optional [semtools](https://github.com/run-llama/semtools) for enhanced quality.

```bash
claude /memory-ingest document.pdf --tags documentation
claude /memory-ingest-dir ./docs --tags knowledge-base
```

See [docs/document-ingestion.md](docs/document-ingestion.md) for full configuration and usage.

## Interactive Dashboard

Web interface at `http://127.0.0.1:8000/` with CRUD operations, semantic/tag/time search, real-time updates (SSE), mobile responsive. Performance: 25ms page load, <100ms search.

**API Endpoints:** `/api/search`, `/api/search/by-tag`, `/api/search/by-time`, `/api/events`, `/api/quality/*` (v8.45.0+)

## Memory Quality System (v8.45.0+)

Local-first AI quality scoring (ONNX), zero-cost, privacy-preserving.

**Features:**
- Tier 1: Local ONNX (80-150ms CPU, $0 cost)
- Quality-boosted search: `0.7×semantic + 0.3×quality`
- Quality-based forgetting: High (365d), Medium (180d), Low (30-90d)

**Config:** `export MCP_QUALITY_SYSTEM_ENABLED=true`

→ Details: [`.claude/directives/quality-system-details.md`](.claude/directives/quality-system-details.md)
→ Guide: [docs/guides/memory-quality-guide.md](docs/guides/memory-quality-guide.md)

## Memory Consolidation System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lisophorm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
