---
trigger: always_on
description: <!-- This file is canonical. CLAUDE.md is a symlink to AGENTS.md. -->
---

# Agent Instructions

<!-- This file is canonical. CLAUDE.md is a symlink to AGENTS.md. -->

This is a map. Subsystem details live in nested `AGENTS.md` files next to code, auto-loaded rules live in `.claude/rules/`, and repository skills live in `.agents/skills/`. Claude reads the same skills through the `.claude/skills` symlink. Run `cargo xtask help` for build commands.

## Project positioning

nteract is a local-first, agent-ready notebook environment where humans, kernels, and AI agents work against the same live notebook document set. Describe that in concrete system terms: Automerge-backed notebook state, explicit runtime state, daemon-owned kernels, outputs, and execution, and programmatic control through the same runtime model. Avoid broad AI slogans; prefer the mechanics users and developers can verify.

## Documentation taxonomy

Use `docs/README.md` as the front door for repo documentation. Start working
notes in `.context/` while exploring. Do not create a persistent doc for routine
implementation notes, test plans, or context that only explains the current
patch; use the PR description, code comments, or final response instead.

Promote `.context/` notes into `docs/` only when they should persist for product,
design, engineering, research, or AI collaborators. Use `docs/memos/` for shared
thinking, research, options, and RFC-style proposals; do not file exploratory
work as a Draft ADR just because it mentions architecture. Graduate durable
technical decisions to `docs/adr/`, durable product requirements to `docs/prd/`,
scoped execution work to `docs/plans/`, evidence and follow-up lists to
`docs/audits/`, benchmark evidence to `docs/measurements/`, operational
procedures to `docs/runbooks/`, and time-bound transfer notes to
`docs/handoffs/`.

## Skills

Use `.agents/skills/` when the task matches:
- `architecture` — docs taxonomy, ADR/memo/PRD placement, cross-cutting architecture, and source-grounded proposal work
- `automerge-sync` — sync protocol internals, document model, reconnection, peer state, in-flight suppression, protocol design patterns, convergence debugging
- `daemon-dev` — daemon development, Python bindings, build system, kernel debugging, xtask workflows
- `execution-pipeline` — end-to-end cell execution: required_heads → ExecuteCell → CellQueued → RuntimeStateDoc polling → output-sync grace → output resolution
- `frontend-dev` — frontend development, TypeScript bindings (ts-rs), UI iteration workflows
- `mcp-session-lifecycle` — MCP proxy supervision, daemon watch loop, session state, rejoin/reconnect races, room eviction
- `pr-reviewer` — opencode-backed Bedrock PR reviews with isolated worktrees and structured findings
- `releasing` — version bumps, tag conventions, release procedures
- `testing` — choosing test strategies, running verification, E2E, diagnostics collection

## Subsystem guides

| Topic | Doc |
|------|-----|
| Architecture + daemon | `crates/runtimed/AGENTS.md` |
| Frontend architecture | `apps/notebook/src/AGENTS.md` |
| Hosted cloud rooms | `apps/notebook-cloud/AGENTS.md` |
| UI components (Shadcn + nteract) | `src/components/ui/AGENTS.md` |
| Wire protocol & sync | `crates/notebook-wire/AGENTS.md` |
| Widgets | `src/components/widgets/AGENTS.md` |
| Environments / trust | `crates/kernel-env/AGENTS.md` |
| Iframe sandbox & renderer plugins | `src/components/isolated/AGENTS.md` |
| CRDT mutation rules | `crates/notebook-doc/AGENTS.md` |
| Logging | `.claude/rules/logging.md` |

## MCP servers

Three may be visible. Pick by purpose. Full details in `.claude/rules/mcp-servers.md` (auto-loaded everywhere).

- **`nteract-dev`** — default for development. Per-worktree dev daemon, dev tools (`up`, `down`, `status`, `logs`, `vite_logs`) plus 26 proxied notebook tools. Prefer `up` over manual `cargo xtask dev-daemon`.
- **`nteract-nightly`** — system nightly daemon. Diagnostics only.
- **`nteract`** — system stable daemon. Diagnostics only.

If `nteract-dev` is unavailable, fall back to `cargo xtask` (derives the worktree env on its own). Use system MCP servers only for diagnostics.

## Required before commit

```bash
cargo xtask lint --fix
```

CI rejects unformatted PRs. `cargo xtask help` is the source of truth for build commands.

## Commit and PR title format

Conventional Commits: `<type>(<optional-scope>)!: <short imperative summary>`

Types: `feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `ci`, `build`, `perf`, `revert`.

## Load-bearing invariants

Most invariants auto-load from `.claude/rules/*.md` and nested `AGENTS.md` files when you edit matching paths. Two that don't fit any path scope:

### Tokio mutex guards stay within synchronous blocks

Hold a `tokio::sync::Mutex` or `RwLock` guard only within a synchronous block — release before any `.await`. Convoy deadlocks if the holder suspends. CI enforces via `cargo test -p runtimed --test tokio_mutex_lint`. Use block scoping (not `drop()`) so the lint can verify. Prefer owned state in `select!` loops over `Arc<Mutex<...>>`. Use `std::sync::Mutex` for sync-only access.

### Cell list uses stable DOM order


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nteract/nteract](https://github.com/nteract/nteract) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
