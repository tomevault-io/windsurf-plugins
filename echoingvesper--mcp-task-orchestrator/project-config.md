---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## MCP Tools Integration

@/home/aya/.claude/mcp-tools-core.md

Available MCP servers provide search, development, and automation capabilities. See user memory for detailed usage patterns.

## Project Overview

The MCP Task Orchestrator is a Model Context Protocol server that provides intelligent task orchestration, specialized AI roles, and persistent memory for AI-assisted development. It follows Clean Architecture and Domain-Driven Design principles with a complete layered structure.

## Critical Directives

### ***CRITICAL***: Task Orchestrator Failure Protocol

**If the MCP Task Orchestrator ever fails to function:**

1. **STOP** - Do not proceed with current task or "work around" the issue
2. **DIAGNOSE** - Immediately spawn a diagnostic agent to identify the issue:
   ```bash
   # Check MCP connection
   claude mcp list 2>/dev/null | grep task-orchestrator || echo "Not connected"
   
   # Run health check
   python tools/diagnostics/health_check.py
   
   # Check server logs
   tail -n 50 ~/.claude/logs/mcp-*.log 2>/dev/null || echo "No logs found"
   ```
3. **FIX** - Spawn a dedicated fix agent with these priorities:
   - Follow detailed procedures in `PRPs/protocols/orchestrator-fix-protocol.md`
   - Fix known issues in the orchestrator code
   - Restart the MCP server if needed: `claude mcp restart task-orchestrator`
   - If changes were made to server code: `pip install -e . && claude mcp restart task-orchestrator`
   - Update documentation if Claude Code restart is required
4. **VERIFY** - Test the fix with `orchestrator_health_check` tool
5. **RESUME** - Only continue original task after verification

**Never say "the orchestrator isn't working, let's continue without it"**

### ***CRITICAL***: Git Commit After Every Task

**After completing ANY development task:**

1. **ALWAYS** review changes: `git status && git diff`
2. **COMMIT** with descriptive message:
   ```bash
   git add -A
   git commit -m "type(scope): description"
   # Examples:
   # fix(orchestrator): resolve connection timeout issue
   # feat(prp): add orchestrator integration to PRP process
   # docs(claude): add critical failure protocol
   ```
3. Include orchestrator task ID in commit if applicable
4. Never leave uncommitted changes between tasks

### ***CRITICAL***: Auto-Recovery for Orchestrator Changes

**When modifying orchestrator code:**

1. After any change to `mcp_task_orchestrator/` files:
   ```bash
   # Reinstall and restart
   pip install -e . && claude mcp restart task-orchestrator
   
   # Verify connection
   claude mcp list | grep task-orchestrator
   ```
2. If restart fails, notify user that Claude Code restart may be needed
3. Test with health check before proceeding

## Commands

### Building and Testing

```bash
# Install in development mode
pip install -e ".[dev]"

# Run all tests
pytest

# Run specific test categories
pytest -m unit           # Unit tests only
pytest -m integration    # Integration tests
pytest -m "not slow"     # Skip slow tests

# Run single test file
pytest tests/test_server.py -v

# Alternative test runners (more reliable output)
python tests/test_resource_cleanup.py
python tests/test_hang_detection.py
python tests/enhanced_migration_test.py
```

### Linting and Formatting

```bash
# Format code with Black
black mcp_task_orchestrator/

# Sort imports
isort mcp_task_orchestrator/

# Type checking (if mypy is configured)
mypy mcp_task_orchestrator/

# Lint markdown files
markdownlint docs/ *.md
```

### Package Management

```bash
# Build distribution
python setup.py sdist bdist_wheel

# Install locally
pip install -e .

# PyPI release
python scripts/release/pypi_release_simple.py
```

### Server Modes

```bash
# Run server in dependency injection mode (default)
MCP_TASK_ORCHESTRATOR_USE_DI=true python -m mcp_task_orchestrator.server

# Run server in legacy mode
MCP_TASK_ORCHESTRATOR_USE_DI=false python -m mcp_task_orchestrator.server

# Use dedicated DI-only server
python -m mcp_task_orchestrator.server_with_di
```

### Debugging and Diagnostics

```bash
# Comprehensive health check and diagnostics
python tools/diagnostics/health_check.py

# Real-time performance monitoring
python tools/diagnostics/performance_monitor.py --monitor --duration 120

# Run diagnostic analysis
python tools/diagnostics/health_check.py --diagnostics

# Generate full system report
python tools/diagnostics/health_check.py --report system_report.json

# MCP protocol testing
python scripts/diagnostics/test_mcp_protocol.py
```

## Clean Architecture Overview

The MCP Task Orchestrator follows **Clean Architecture** and **Domain-Driven Design** principles:

### Architecture Layers

**1. Domain Layer** (`mcp_task_orchestrator/domain/`):
- **Entities**: Core business objects (Task, Specialist, OrchestrationSession, WorkItem)
- **Value Objects**: Immutable types (TaskStatus, SpecialistType, ExecutionResult, TimeWindow)
- **Exceptions**: Domain-specific error hierarchy with severity levels and recovery strategies
- **Services**: Domain business logic (TaskBreakdownService, SpecialistAssignmentService, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EchoingVesper/mcp-task-orchestrator](https://github.com/EchoingVesper/mcp-task-orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
