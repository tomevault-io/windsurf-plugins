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

## 🔴 Operational Rules

**These rules apply to every session. Violations cause real incidents — follow them exactly.**

### Memory Storage
- **Always use MCP Memory Server** (`mcp__memory__memory_store`) for storing context, learnings, and decisions
- **Never write to `MEMORY.md` or local memory files** unless the user explicitly asks for file-based storage
- Tag all memories with `mcp-memory-service` as the first tag (per `memory-tagging.md`)

### MCP Configuration
- **MCP server configs go in `.mcp.json`**, not in `settings.json`
- `settings.json` is for Claude Code settings (hooks, plugins, permissions) only

### SSH / Network Safety
- **Before any SSH or network task**: confirm machine identity with `hostname` and verify connection direction (source → target)
- **Never assume** which machine you're on or which direction a connection flows — always verify first
- This prevents accidental operations on production machines or reverse-direction tunnels

### Auto-Save Learnings
- **After completing tasks**: automatically save key learnings, decisions, and patterns to MCP Memory without being asked
- Include relevant tags: `mcp-memory-service`, task-specific tags, and `learnings`

### Release Workflow Checklist
Before merging or releasing:
1. Verify CI is green on the target branch (`gh run list --branch <branch>`)
2. Check landing page version (`docs/index.html`) against latest tag — update if stale (MINOR/MAJOR only)
3. Clean up merged branches after release (`git branch -d`, `git push origin --delete`)
4. Use `github-release-manager` agent — never manually bump versions

## Overview

MCP Memory Service is a semantic memory layer for AI applications, accessible via REST API and MCP transport. It provides persistent storage for 14+ AI clients including Claude Desktop, OpenCode, LangGraph, CrewAI, and any HTTP client. It uses vector embeddings for semantic search, supports multiple storage backends (SQLite-vec, Cloudflare, Hybrid), and includes advanced features like memory consolidation, quality scoring, and OAuth 2.1 team collaboration.

**Current Version:** v10.60.0 - feat(consolidation): temporal contradiction detection + fix(milvus): instance-level graph cache + fix(hooks): tunnel/reverse-proxy port fix + feat(benchmarks): mem0 adapter — ~2,005 tests — see [CHANGELOG.md](CHANGELOG.md) for details

> **🎯 v10.0.0 Milestone**: This major release represents a complete API consolidation - 34 tools unified into 12 with enhanced capabilities. All deprecated tools continue working with warnings until v11.0. See `docs/MIGRATION.md` for migration guide.

> **📊 Q1 2026 Status** (Feb 1, 2026): Quarterly roadmap review completed - 6/9 high-priority items delivered ahead of schedule including Python 3.14 support, backup scheduler fix, and full CI/CD stability. See [Development Roadmap](https://github.com/doobidoo/mcp-memory-service/wiki/13-Development-Roadmap) and [Issue #399](https://github.com/doobidoo/mcp-memory-service/issues/399) for details.

## Essential Commands

### Development Server

**Recommended (lifecycle CLI):**
```bash
# Start HTTP server in background with PID tracking, logs, health check
memory launch                              # Background (default)
memory launch --foreground                 # Foreground (same as server --http)
memory launch --storage-backend hybrid     # With specific backend
memory launch --debug                      # With debug logging

# Check if server is running
memory info

# Stop server
memory stop

# Restart (preserves --storage-backend and --debug from running server)
memory restart

# View logs
memory logs
memory logs -n 50
```

**Legacy shell scripts (still available, but CLI is preferred):**
```bash
# MCP server (for Claude Desktop integration)
python -m mcp_memory_service.server

# HTTP API server (dashboard + REST API)
python scripts/server/run_http_server.py

# Both servers simultaneously
./start_all_servers.sh

# Quick update after git pull
./scripts/update_and_restart.sh
```

> **Note:** The `memory launch/stop/restart/info/logs` CLI commands are the
> preferred way to manage the server going forward. The shell scripts
> (`start_all_servers.sh`, `update_and_restart.sh`) still work but may be
> deprecated in a future release. For new deployments, use the CLI.

### Testing
```bash
# Run all tests (~1,780 tests total)
pytest

# Run specific test file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doobidoo/mcp-memory-service](https://github.com/doobidoo/mcp-memory-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
