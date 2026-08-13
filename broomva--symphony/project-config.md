---
trigger: always_on
description: Symphony is a long-running orchestration service that polls an issue tracker (Linear),
---


# AGENTS.md - Symphony

## Repository Purpose
Symphony is a long-running orchestration service that polls an issue tracker (Linear),
creates isolated per-issue workspaces, and runs coding agent sessions (Claude, Codex, etc.)
for each issue. It is a scheduler/runner, not a workflow engine.

## Architecture
Rust workspace with layered crates matching the spec's abstraction levels:

| Crate | Spec Layer | Responsibility |
|-------|-----------|----------------|
| `symphony-core` | Domain Model (S4) | Shared types: Issue, State, Session, Workspace |
| `symphony-config` | Config + Policy (S5-6) | WORKFLOW.md loader, typed config, file watcher |
| `symphony-tracker` | Integration (S11) | Linear GraphQL client, issue normalization |
| `symphony-workspace` | Execution (S9) | Per-issue directory lifecycle, hooks, safety invariants |
| `symphony-agent` | Execution (S10) | Agent subprocess, JSON-RPC + simple pipe modes |
| `symphony-orchestrator` | Coordination (S7-8) | Poll loop, dispatch, reconciliation, retry, drain |
| `symphony-observability` | Observability (S13) | Structured logging, HTTP server, dashboard, health, auth |
| `symphony-cli` (root) | CLI (S17.7) | Subcommands: start, stop, status, issues, validate, etc. |

## Key Design Decisions
- **In-memory state**: Orchestrator state is intentionally in-memory; recovery is tracker-driven
- **Single authority**: Only the orchestrator mutates scheduling state
- **Workspace isolation**: Coding agents run ONLY inside per-issue workspace directories
- **Dynamic reload**: WORKFLOW.md changes are detected and re-applied without restart
- **Liquid-compatible templates**: Strict variable/filter checking for prompt rendering
- **Graceful shutdown**: SIGTERM/SIGINT → drain mode → wait for workers → exit
- **Stall kill**: Worker abort handles tracked; stalled sessions killed + retried
- **Bearer auth**: Optional `SYMPHONY_API_TOKEN` protects `/api/v1/*`; health endpoints open

## Gathering Context from the Knowledge Graph

This repo is an Obsidian vault. All `.md` files form a wikilinked knowledge graph.

### How to orient before working:
1. **Start at `docs/Symphony Index.md`** — it links to everything
2. **Check `docs/roadmap/Project Status.md`** — current phase, test counts, known gaps
3. **Read the relevant `docs/crates/<name>.md`** — for the crate you're modifying
4. **Check `CONTROL.md`** — setpoints your changes must satisfy
5. **Check `PLANS.md`** — task breakdown for the current phase
6. **Traverse `[[wikilinks]]`** — follow links to find related context; the graph is designed for this

### Vault map:
```
Root governance:  CLAUDE.md  AGENTS.md  PLANS.md  CONTROL.md  EXTENDING.md  CONTRIBUTING.md
Docs index:       docs/Symphony Index.md
Architecture:     docs/architecture/Crate Map.md, Domain Model.md
Operations:       docs/operations/Control Harness.md, Configuration Reference.md
Roadmap:          docs/roadmap/Project Status.md, Production Roadmap.md
Per-crate:        docs/crates/symphony-core.md, symphony-config.md, ...
Planning state:   .planning/STATE.md, .planning/REQUIREMENTS.md
Examples:         examples/linear-claude.md, linear-codex.md, github-claude.md
```

## Development Commands
```bash
make smoke          # Compile + clippy + test (gate — runs in pre-commit hook)
make check          # Compile + clippy only
make test           # Run all workspace tests (includes CLI integration tests)
make build          # Release build
make control-audit  # Smoke + format check (before PR)
make fmt            # Auto-format code
make install        # Install binary locally

# CLI-specific testing
cargo test --test cli_integration          # Run CLI binary integration tests only
cargo test --test cli_integration -- init  # Run only init-related CLI tests
```

## Control Harness

The pre-commit hook at `.githooks/pre-commit` enforces the gate automatically. Activate:
```bash
git config core.hooksPath .githooks
```

### Before every commit (enforced by hook):
- `cargo check --workspace` passes
- `cargo clippy --workspace -- -D warnings` passes
- `cargo test --workspace` passes
- `cargo fmt --all -- --check` passes

### Before every push (agent obligation):
- Documentation updated per the rules in CLAUDE.md "Documentation Obligations"
- `CONTROL.md` deviation log updated if any setpoint was relaxed
- `docs/roadmap/Project Status.md` updated if changes are significant
- `docs/operations/Control Harness.md` test counts updated if tests were added

### The control loop:
```
Code change → make smoke (pre-commit) → tests pass → docs updated → push
     ↑                                                                 |
     └─── If smoke fails: fix before proceeding, never suppress ───────┘
```

### Harness validation:
- `make harness-audit` — validates governance files, hooks, CI, frontmatter, deviation log freshness
- `make entropy-check` — reports `#[allow]` count, TODO/FIXME/HACK markers, doc staleness, test count drift

## Consciousness Substrates

The development process is grounded in three substrates that provide persistent context:

| Substrate | Source | Content |
|-----------|--------|---------|
| Control Metalayer | `.control/policy.yaml`, `.control/state.json` | Behavioral governance: what MUST be true |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [broomva/symphony](https://github.com/broomva/symphony) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
