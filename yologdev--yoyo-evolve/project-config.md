---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A self-evolving coding agent CLI built on [yoagent](https://github.com/yologdev/yoagent). The agent spans multiple Rust source files under `src/`. A GitHub Actions cron job (`scripts/evolve.sh`) runs the agent hourly using a 3-phase pipeline (plan → implement → respond), which reads its own source, picks improvements, implements them, and commits — if tests pass. All runs use a flat 8h gap (~3/day). Sponsors get benefit tiers (issue priority, shoutout issues, listing eligibility) but no run-frequency speedup. One-time sponsors ($2+) get 1 accelerated run that bypasses the gap (only consumed when they have open issues; tracked in `sponsors/credits.json`).

**Sponsor benefit tiers:**

Monthly recurring (benefits only):
- $5/mo: Issue priority (💖)
- $10/mo: Priority + shoutout issue
- $25/mo: Above + SPONSORS.md eligible
- $50/mo: Above + README eligible

One-time (cumulative — each tier includes all benefits below it):
- $2: 1 accelerated run (bypasses 8h gap)
- $5: Accelerated run + issue priority (14 days)
- $10: Above + shoutout issue (30 days)
- $20: Above + SPONSORS.md eligible (30 days)
- $50: Above + priority for 60 days + SPONSORS.md + README eligible
- $1,000 💎 Genesis: All above + permanent priority + SPONSORS.md + README + journal acknowledgment (never expires)

## Build & Test Commands

```bash
cargo build              # Build
cargo test               # Run tests
cargo clippy --all-targets -- -D warnings   # Lint (CI treats warnings as errors)
cargo fmt -- --check     # Format check
cargo fmt                # Auto-format
```

CI runs all four checks (build, test, clippy with -D warnings, fmt check) on PR to main. A separate Pages workflow builds and deploys the website on push to main.

To run the agent interactively:
```bash
ANTHROPIC_API_KEY=sk-... cargo run
ANTHROPIC_API_KEY=sk-... cargo run -- --model claude-opus-4-6 --skills ./skills
```

To trigger a full evolution cycle:
```bash
ANTHROPIC_API_KEY=sk-... ./scripts/evolve.sh
```

## Architecture

**Multi-file agent** (`src/`):
- `main.rs` — agent core, REPL, streaming event handling, rendering with ANSI colors, sub-agent tool integration, AskUserTool (interactive question-asking)
- `hooks.rs` — Hook trait, HookRegistry, AuditHook, HookedTool wrapper, maybe_hook helper
- `tools.rs` — StreamingBashTool, RenameSymbolTool, AskUserTool, TodoTool, bash command safety analysis, tool builders
- `cli.rs` — CLI argument parsing, subcommands, configuration
- `config.rs` — permission config, directory restrictions, MCP server config, TOML parsing helpers
- `context.rs` — project context loading, file listing, git status, recently changed files
- `providers.rs` — provider constants (KNOWN_PROVIDERS), API key env vars, default/known models per provider
- `format/mod.rs` — Color, constants, utility functions, test output filtering, re-exports
- `format/highlight.rs` — syntax highlighting for code, JSON, YAML, TOML
- `format/cost.rs` — pricing, cost display, token formatting
- `format/markdown.rs` — MarkdownRenderer for streaming markdown output
- `format/tools.rs` — Spinner, ToolProgressTimer, ActiveToolState, ThinkBlockFilter
- `prompt.rs` — prompt construction for evolution sessions
- `prompt_budget.rs` — session wall-clock budget + audit log helpers (extracted from `prompt.rs`)

Uses `yoagent::Agent` with `AnthropicProvider`, `default_tools()`, and an optional `SkillSet`.

**Documentation** (`docs/`): mdbook source in `docs/src/`, config in `docs/book.toml`. Output goes to `site/book/` (gitignored). The journal homepage (`site/index.html`) is built by `scripts/build_site.py`. Both are built and deployed by the Pages workflow (`.github/workflows/pages.yml`), not during evolution.

**Evolution loop** (`scripts/evolve.sh`): pipeline:
1. Verifies build → fetches GitHub issues (community, self, help-wanted) via `gh` CLI + `scripts/format_issues.py` → scans for pending replies on previously touched issues
2. **Phase A** (Planning): Agent reads everything, writes task files to `session_plan/`
3. **Phase B** (Implementation): Agents execute each task (20 min each), with two fix loops: build/test failures get up to 10 fix attempts (10 min each), then the evaluator runs and rejections get up to 9 more fix attempts (10 min each). Reverts only after all fix attempts are exhausted. Max 3 tasks per session.
4. Verifies build, fixes or reverts → agent-driven issue responses (agent directly calls `gh issue comment`/`close`) → pushes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yologdev/yoyo-evolve](https://github.com/yologdev/yoyo-evolve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
