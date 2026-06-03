---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**rUvOS** (formerly Ruflo v4) is a Rust-native agent orchestration system being merged into the RuVector workspace. It's a ruthless rewrite of Ruflo from 631k TypeScript LOC + 323 MCP tools + 60+ agent types down to ~30k Rust LOC with 20 core tools and 12 agent archetypes.

**Core positioning:** RuVector is the self-learning vector + graph + local-AI substrate. rUvOS is the agent orchestration layer that runs on top of it. Single static binary (`ruvos`), zero Node.js runtime required.

## Architecture

### The Six New Crates (rUvOS's layer on top of RuVector)

| Crate | Budget | Purpose | Key files |
|-------|--------|---------|-----------|
| `ruvos-cli` | ≤8k LOC | clap-based CLI shell (`ruvos init`, `ruvos mcp`, `ruvos agent`) | — |
| `ruvos-mcp` | ≤6k LOC | JSON-RPC MCP server over stdio + the 20 tool handlers (memory, session, agent, hooks, intel, plugin, gov, workflow) | — |
| `ruvos-host` | ≤6k LOC | `CliHost` trait + Claude + Codex adapters, output normalizer for multi-CLI orchestration | — |
| `ruvos-plugin-host` | ≤4k LOC | Plugin discovery (markdown + YAML frontmatter), manifest parsing, shell command execution | — |
| `ruvos-hooks` | ≤3k LOC | 8 hooks (pre/post task, edit, command, session) + SONA learning integration | — |
| `ruvos-session` | ≤3k LOC | `.rvf` container write/read, fork (COW-branch), signature verification via `rvf-crypto` | — |

**Total: ≤30k LOC of new Rust.** Everything else is `use ruvector_*;` or `use sona::*;` or `use rvf::*;`.

### The 20 v1 MCP Tools (and 12 Agent Archetypes)

**Tools (by domain):**
- `memory.*` (4) — search, store, retrieve, list with MMR + recency
- `session.*` (3) — create, resume (restore from `.rvf`), fork
- `agent.*` (3) — spawn, status, message (for multi-agent swarms)
- `hooks.*` (3) — pre, post, route (unified hook dispatch + model recommendations)
- `intel.*` (2) — pattern_search (trajectory similarity), pattern_store (SONA learning)
- `plugin.*` (2) — list (discover), invoke (shell exec)
- `gov.*` (2) — witness_verify (`.rvf` signature chain), health (doctor / status)
- `workflow.*` (1) — run (orchestration templates: feature / bugfix / refactor / security)

**Agent archetypes:** coder, reviewer, tester, researcher, architect, planner, security, perf, devops, data, docs, coordinator (+ composable traits: `--trait=tdd`, `--trait=backend`, `--trait=frontend`, `--trait=mobile`, `--trait=ml`, `--trait=domain`, `--trait=cloud`, `--trait=db`, `--trait=audit`).

### Plugin Layout (Single Canonical Form)

```
./.ruvos/plugins/<name>/
├── plugin.toml              # Rust manifest
├── README.md
├── agents/*.md              # Claude Code agents (markdown + frontmatter)
├── skills/*/SKILL.md        # Claude Code skills
├── commands/*.md            # slash commands
└── hooks/*.toml             # hook bindings (optional)
```

Discovery order: project-local → user-global (`~/.ruvos/plugins/`) → env override → built-in registry.

## Development Workflow

### Phase Timeline

| Phase | What | Weeks |
|-------|------|-------|
| **0** | Scope ledger (this is it — you're in Phase 0 now) | 3–5 days |
| **1** | Merge into RuVector workspace, create 6 crate skeletons, CI green | 1 week |
| **2** | `ruvos mcp serve` ships hello-world tool to Claude Code, Codex CLI, Gemini CLI | 1 week |
| **3** | Plugin host (markdown discovery, shell exec, skill compatibility) | 1 week |
| **4** | 8 hooks + SQLite-backed queue (replaces in-process daemon, fixes Windows bug #1766) | 2 weeks |
| **5** | Memory + session (HNSW + RaBitQ + `.rvf` containers + witness chain) | 2 weeks |
| **6** | CliHost adapters (Claude + Codex normalized events; Gemini deferred) | 2 weeks |
| **7** | Cutover (npm shim ships, TS code → `legacy/`, v4 tag) | 1 week |

**Current state:** Phase 0 — scope ledger locked. No Rust yet.

### Build & Test (Phase 1+)

Once the Rust codebase exists:

```bash
# Build the binary
cargo build --release

# Run all tests
cargo test

# Run a single test
cargo test --lib <crate>::<module>::<test_name>

# Format + lint
cargo fmt && cargo clippy --all-targets --all-features

# Check code coverage (Phase 2+)
cargo tarpaulin --out Html

# Run doc tests
cargo test --doc
```

### Key Development Rules

**ZERO-DEFECT POLICY (highest priority, non-negotiable):**

The entire workspace must be **100% clean at all times** — zero errors, zero
warnings, zero failing tests, anywhere they occur, regardless of whether the
code is ours (`crates/`) or vendored substrate (`substrate/`). "It's not our
code / it's pre-existing / it's not in default-members" is **never** an
acceptable reason to leave a defect. If you touch the workspace and `cargo
build --workspace`, `cargo clippy --workspace --all-targets`, `cargo test
--workspace`, or `cargo fmt --check` surfaces anything, you fix it before
considering the task done. Validate with the full workspace, not just
default-members:

```bash
cargo build --workspace --jobs 4          # --jobs 4 avoids OOM/ICE on this 30+ crate tree
cargo clippy --workspace --all-targets --jobs 4 -- -D warnings
cargo test --workspace --jobs 4
cargo fmt --check
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dgdev25/ruvos](https://github.com/dgdev25/ruvos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
