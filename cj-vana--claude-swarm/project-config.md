---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

```bash
npm run build         # Compile TypeScript and copy dashboard assets
npm run dev           # Watch mode for TypeScript compilation
npm start             # Run the compiled MCP server
npm run inspector     # Debug with MCP Inspector
npx tsc --noEmit      # Type-check without emitting (CI validation)
npm test              # Run all tests with Vitest
npm run test:watch    # Run tests in watch mode
npm run test:coverage # Run tests with coverage report
npx vitest run src/utils/git-verification.test.ts  # Run a single test file
```

**Install the MCP server** into Claude Code:
```bash
claude mcp add claude-swarm --scope user -- node $(pwd)/dist/index.js
```

**Install the /swarm skill** for guided orchestration:
```bash
mkdir -p ~/.claude/skills/swarm && cp skill/SKILL.md ~/.claude/skills/swarm/
```

## Architecture Overview

This is an MCP (Model Context Protocol) server that orchestrates parallel Claude Code worker swarms via tmux sessions. The pattern separates concerns between an orchestrator (which plans and monitors) and workers (which implement individual features).

### Core Components

**src/index.ts** - MCP server entry point registering 55+ tools. All tool handlers are defined inline. Tool schemas use Zod for validation. The file is structured by tool category: core orchestration, worker management, competitive planning, confidence monitoring, feature management, session control, post-completion reviews, protocol management, protocol networking, context management, and repository setup.

**src/state/manager.ts** - Persistent state management using the "notebook pattern". Stores session state in `.claude/orchestrator/state.json` with atomic writes (temp file + rename). Also generates `claude-progress.txt` for human readability and `init.sh` for environment setup. Manages feature context (`FeatureContext`), routing config, and protocol bindings per feature.

**src/workers/manager.ts** - Manages Claude Code worker sessions via tmux. Key security pattern: prompts are passed via files (not shell strings, mode 0600) to prevent injection. Includes completion monitoring (10s polling) with circuit breaker pattern (`MAX_MONITOR_ERRORS = 5` auto-stops after repeated failures), heartbeat tracking, and conflict analysis for parallel execution. Supports competitive planning mode with dual planners.

**src/workers/confidence.ts** - Multi-signal confidence scoring combining tool activity patterns (35%), self-reported confidence (35%), and output analysis (30%). Detects struggling workers via stuck loops, error patterns, and frustration language.

**src/workers/enforcement-integration.ts** - Hooks protocol enforcement into the worker lifecycle. Validates constraints before worker spawns and monitors during execution.

**src/workers/review-manager.ts** - Orchestrates post-completion code and architecture reviews. Parses worker logs to identify modified files, builds review prompts with session context, and aggregates findings into structured JSON. Review workers have read-only tool access (no Bash). Notifies orchestrator when reviews complete via status file updates.

**src/workers/worktree-manager.ts** - Git worktree creation, merge-back, and cleanup. Each worker gets its own worktree directory for isolation from main working tree.

**src/workers/verification-runner.ts** - Verification command execution and validation. Runs pre-completion test/build commands and validates results.

**src/workers/lock-manager.ts** - Atomic file-based mutual exclusion. Prevents multiple workers from modifying the same files simultaneously.

**src/workers/plan-reviewer.ts** - Auto-approval engine for worker plans. Scores plans on test strategy, risk assessment, file specificity, and step count. Threshold-based approval (score >= 50). Also provides post-completion quality assessment.

**src/workers/ralph-loop.ts** - Ralph Loop script generation and configuration. Builds bash wrapper scripts that run fresh Claude sessions per iteration with filesystem-based state persistence (progress files, git diff).

### Protocol Governance System

The protocol system enables behavioral constraints on workers. Located in `src/protocols/`:

**schema.ts** - Zod schemas defining `Protocol`, `ProtocolConstraint`, constraint types (tool_restriction, file_access, output_format, behavioral, temporal, resource, side_effect), and `BaseConstraints`. All protocol data is validated against these schemas.

**registry.ts** - Protocol storage, activation status, and violation tracking. Persists to `.claude/orchestrator/protocols/`. Implements `ProtocolRegistryLike` interface for resolver compatibility. Maintains audit log of all protocol operations.

**enforcement.ts** - Pre/post execution validation engine. Validates worker actions against active protocol constraints. Supports learning mode for protocol development. Memory-bounded with `MAX_OBSERVED_PATTERNS = 100` and `MAX_ACTIVE_ALERTS = 50` using LRU eviction when limits are reached.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cj-vana/claude-swarm](https://github.com/cj-vana/claude-swarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
