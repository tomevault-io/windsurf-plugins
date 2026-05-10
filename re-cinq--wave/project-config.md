---
trigger: always_on
description: **Wave** is a multi-agent pipeline orchestrator written in Go that wraps Claude Code and other LLM CLIs via subprocess execution. It composes personas, pipelines, contracts, and relay/compaction into a continuous development system.
---

# Wave Development Guidelines

**Wave** is a multi-agent pipeline orchestrator written in Go that wraps Claude Code and other LLM CLIs via subprocess execution. It composes personas, pipelines, contracts, and relay/compaction into a continuous development system.

## ACCOUNTABILITY — YOU FOUND IT, YOU FIX IT

> **URGENT — NON-NEGOTIABLE**: This is the single most important rule in this file. It survives context compaction. Re-read it if you are unsure.

If you discover a problem — any problem — you own it. Fix it immediately.

There is NO concept of "pre-existing issue" in this project.
If you touched it or saw it break — fix it.

If a validation step reveals issues in files you didn't modify — fix those too.

Never argue about whether something is your responsibility.

## Critical Constraints

1. **Single static binary** — no runtime dependencies except adapter binaries
2. **Test ownership** — every failing test is YOUR concern. Fix or delete (with justification), never ignore. Changes to personas, pipelines, contracts, or meta-pipelines require `go test ./...`
3. **Security first** — all inputs validated, paths sanitized, permissions enforced
4. **Constitutional compliance** — navigator-first architecture, fresh memory at step boundaries, contract validation at handovers, ephemeral workspace isolation, observable progress events
5. **Observable execution** — structured progress events for monitoring
6. **No backward compatibility constraint** during prototype phase — move fast, let tests catch regressions
7. **No `t.Skip()`** without a linked issue. Delete tests only with clear justification

## How Wave Works at Runtime

Each pipeline is a **topologically-sorted DAG** of steps. For every step:

1. **Workspace creation** — an ephemeral worktree is created under `.agents/workspaces/<pipeline>/<step>/`. Steps can share workspaces via `workspace.ref`. Mounts support readonly/readwrite modes
2. **Artifact injection** — outputs from prior steps are injected into `.agents/artifacts/` before execution begins. The system validates existence, enforces optional/required semantics, and checks schemas if `ref.SchemaPath` is specified
3. **Runtime CLAUDE.md assembly** — a per-step CLAUDE.md is generated from four layers:
   - Base protocol preamble (`.agents/personas/base-protocol.md`)
   - Persona system prompt (role, responsibilities, constraints)
   - Contract compliance section (auto-generated from step contract schema)
   - Restriction section (denied/allowed tools, network domains from manifest permissions)
4. **Adapter execution** — the persona runs in isolated context with fresh memory (no chat history inheritance)
5. **Contract validation** — step output is validated against its contract (json_schema, typescript_interface, test_suite, markdown_spec, format) **before** marking the step successful. Hard failures block; soft failures log warnings

Key source files: `internal/pipeline/executor.go`, `internal/adapter/claude.go`, `internal/contract/`, `internal/workspace/`

## Architecture

### Active Technologies
- Go 1.25+ with `gopkg.in/yaml.v3`, `github.com/spf13/cobra`
- SQLite for pipeline state, filesystem for workspaces and artifacts

### Core Components
- **Manifests** (`wave.yaml`) — single source of truth for configuration
- **Personas** — AI agents with specific roles, permissions, and system prompts
- **Pipelines** — multi-step workflows with dependency resolution
- **Contracts** — output validation (JSON schema, TypeScript, test suites)
- **Workspaces** — ephemeral isolated execution environments
- **State Management** — SQLite-backed persistence and resumption

### Security Model
- Fresh memory at every step boundary — no chat history inheritance
- Permission enforcement with deny/allow patterns — strictly enforced
- Ephemeral workspaces — isolated filesystem execution
- Contract validation — all outputs validated before step completion
- Audit logging — credential scrubbing and tool call tracking

## File Structure
```
internal/
├── adapter/      # Subprocess execution and adapter management
├── attention/    # Attention classifier driving dashboard notification badges
├── audit/        # Audit logging and credential scrubbing
├── bench/        # SWE-bench benchmarking and comparison
├── checks/       # Shared host-capability probes for preflight and doctor
├── classify/     # Issue domain/complexity classification and pipeline selection
├── continuous/   # Continuous pipeline execution
├── contract/     # Output validation (JSON, TypeScript, test suites)
├── cost/         # Cost ledger, iron rule enforcement, model pricing
├── defaults/     # Embedded default personas, pipelines, and contracts
├── display/      # Terminal progress display and formatting
├── doctor/       # Project health checking and optimization
├── event/        # Progress event emission and monitoring
├── fileutil/     # File and directory copy helpers
├── forge/        # Git forge/hosting platform detection (GitHub, GitLab, Gitea, Forgejo, Codeberg, Bitbucket, local)
├── github/       # GitHub API integration for issue enhancement

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [re-cinq/wave](https://github.com/re-cinq/wave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
